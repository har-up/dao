## 全局配置
Android工程中有一个项目根目录的build.gradle,每个Module也都有一个私有的build.gradle。全局配置对应项目根目录下的build.gradle，私有配置
则是Module下的build.gradle

- 设定UTF-8
```gradle
allProjects{
    repositories {
       jcenter()
       mavenCentral()
    }

    tasks.withType(JavaCompile){
      options.encoding = "UTF-8"
    }
}
```