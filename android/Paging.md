# Paging

Jet Paging给您一个优雅的数据分页解决方案。使app ui按需加载数据。

## Paging library components

官方建议app有如下几个架构组件架构

- DataBase:一个本地数据库，作为app ui 显示的单一真是数据源

- Web API:网络请求

- Repository:仓库，提供统一的数据接口。包装本地数据库操作和网络请求操作

- ViewModel:为UI提供特定数据

- UI：显示数据

  

Paging库中的几个组要组成部分有

- PagedList : 页码的数据集合，异步加载数据

- DataSource/DataSource-Factory:DataSource 是数据快照加载到PageList的基类/DataSource-Factory负责创建DataSource

- LivePageListBuilder:创建LiveData<PageList>

- BoundaryCallback:PageList：PageList集合到达最后项的信号回调

- PageListAdapter: RecyclerView Adapter的子类

  

## 使用PageList存放列表数据

一般而言，ui显示列表数据需要LiveData<List<Data>>这样的数据类型，但这并不是分页数据。所以需要使用

LiveData<PageList<Data>>来替代。

PageList是一种以块的形式加载数据的List



## Pageing整合

- **db/RepoDao**中 LiveDada<PageList<repo>>  ——> DataSource.Factory<Int,Repo>

  ```kotlin
  fun reposByName(queryString: String): DataSource.Factory<Int, Repo>
  ```

- **db/GithubLocalCache** 依照上一步改返回值

  ```kotlin
      fun reposByName(name: String): DataSource.Factory<Int, Repo> {
          // appending '%' so we can allow other characters to be before and after the query string
          val query = "%${name.replace(' ', '%')}%"
          return repoDao.reposByName(query)
      }
  ```

- **data/GithubRepository**         获取DataSource.Factory对象和单页数据数量构造PageList<>对象

  ```kotlin
  fun search(query: String): RepoSearchResult {
      // Get data source factory from the local cache
      val dataSourceFactory = cache.reposByName(query)
  
      // Get the paged list
      val data = LivePagedListBuilder(dataSourceFactory, DATABASE_PAGE_SIZE).build()
  
       // Get the network errors exposed by the boundary callback
       return RepoSearchResult(data, networkErrors)
  }
  ```

- 添加边界回调
  
  - 新建个类 **RepoBoundaryCallback** 继承 PagedList.BoundaryCallback,重写onZeroItemsLoaded
  
  （）、onItemAtEndLoaded（）方法
  
  - 在方法中请求获取数据
  
    ```kotlin
        private fun requestAndSaveData(query: String) {
            if (isRequestInProgress) return
    
            isRequestInProgress = true
            searchRepos(service, query, lastRequestedPage, NETWORK_PAGE_SIZE, { repos ->
                cache.insert(repos) {
                    lastRequestedPage++
                    isRequestInProgress = false
                }
            }, { error ->
                _networkErrors.postValue(error)
                isRequestInProgress = false
            })
        }
    ```
  
  - 在获取DataSource.Factory对象是设置监听
  
    ```kotlin
       val data = LivePagedListBuilder(dataSourceFactory, DATABASE_PAGE_SIZE)
                    .setBoundaryCallback(boundaryCallback)
                    .build()
    ```
  
- RecyclerView的adapter继承PageListAdapter

