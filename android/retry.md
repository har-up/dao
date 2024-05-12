## **OkHttp 拦截器实现请求重试**

### **RetryAndFollowUpInterceptor**

RetryAndFollowUpInterceptor是OkHttp自带重试重定向拦截器，我们可以在构建okhttpClient是设置

retryOnConnectionFailure（false）来关闭重试。

RetryAndFollowUpInterceptor在两种异常的时候才会重试。

- RouteException 网络连接的异常
- IOException IO异常

RetryAndFollowUpInterceptor除了以上两种异常会重试外，会判断是否需要重定向，如果需要重定向会

发出重定向请求，但也有次数限制，不超过20次。



### **自定义重试拦截器**

项目中请求可能会遇到后台服务异常时，并不是以上RetryAndFollowUpInterceptor拦截的异常，所以还是需要我们自己定义一个重试拦截器。

依据需求只需定义拦截次数，代码比较简单，如下。当然后面也可以做出扩展定义某段时间内拦截，以及某段时间内

拦截多少次等。

```
class HttpRetryInterceptor : Interceptor{

  companion object{
    const val RETRY_TIMES = "retry_times"
  }

  override fun intercept(chain: Interceptor.Chain): Response {
    var times: Int? = try { chain.request().headers[RETRY_TIMES]?.toInt() }catch (e:NumberFormatException){0}
    try {
      while (times != null && times >= 0) {
        val response = chain.proceed(chain.request())
        if (response.isSuccessful || times == 0) {
          return response
        } else {
          response.close()
          times -= 1
        }
      }
    }catch (e:Exception){
      return chain.proceed(chain.request())
    }
    return chain.proceed(chain.request())
  }
}
builder.addInterceptor(HttpRetryInterceptor())
```



```
/**
     * 用户刷新应用上报应用信息
     */
@POST("app/v1/user/refreshDeviceInfoss")
@Headers(
  "Content-Type:application/json", "${AUTHORIZATION}:true",
  "${HttpRetryInterceptor.RETRY_TIMES}:1"
)
suspend fun refreshUserInfo(@Body req: String): BaseResponse<Boolean>
```



## **Kotlin 实现请求重试**

关键函数repeat

repeat是kotlin得标准高阶函数,逻辑很简单，对入参函数做一个for循环运行。

```
public inline fun repeat(times: Int, action: (Int) -> Unit) {
  contract { callsInPlace(action) }

  for (index in 0 until times) {
    action(index)
  }
}
```

由于是repeat是内联函数，我们可以通过对action得结果做判断，直接返回结果中断循环，否则重试。

```
suspend fun <T> retry(fallbackValue: T?, times: Int, block: suspend () -> T): T? {
  try {
    repeat(times) { current ->
      try {
        return block()
      } catch (e: Exception) {
        BaseLog.d("retry $current,${e.message}")
      }
    }
  } catch (e: Exception) {
    return fallbackValue
  }
  return fallbackValue
}
```

直接使用上面得retry，我们需要在外部做逻辑判断是否需要重试（如果结果不满足，抛出异常）。实例如下：

```
retry(null, 3) {
  val result =
  RetrofitUtil.requestForBaseResponse {
    appService.refreshUserInfo(jsonObject.toString())
  }
  if (result.code == HttpResultCode.SUCCESS) {
    result
  } else {
    throw Exception(result.msg)
  }
}
```

由于网络请求我们可以通用一套判断逻辑是否需要重试，所以可以再封装一下方便使用，代码如下：

```
suspend fun <T : Any> requestRetry(
  times: Int,
  request: suspend () -> BaseResponse<T>
    ): Result.Success<T>? {
  val result = retry(null, times) {
    val invokeResult = request.invoke()
    if (invokeResult.code == 0) {
      requestForBaseResponse {
        invokeResult
      }
    } else {
      throw Exception(invokeResult.msg)
    }
  }
  return result
}
requestRetry(3){
  appService.refreshUserInfo(jsonObject.toString())
}
```



## **两种方式总结对比**

- 拦截器的方式使用简单，但灵活度欠缺。不如kotlin协程处理方式灵活，当有特殊逻辑判断重试时可以使用kotlin的重试方式。通用的逻辑使用拦截器的方式就可以了
- 重试判断不一样，拦截器目前拦截重试的逻辑是：响应码以2开头则不重试，其他的都会重试；kotlin的方式以okhttp+retrofit解析到的业务逻辑结果来处理，如果接收到业务逻辑结果code==0则不重试，否则重试。