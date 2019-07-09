---
layout: post
title: "总结Java中异步的几种方法"
subtitle: "Talk about the way of Java async"
author: "qingshan"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.4
tags:
  - 工作
  - Java
---


> 在Java中，如使用Tomcat，一个请求会分配一个线程进行请求处理，该线程负责获取数据、拼装数据或模板然后返回给前端；在同步调用获取数据接口的情况下（等待依赖系统返回数据），整个线程是一直被占用并阻塞的。如果有大量的这种请求，每个请求占用一个线程，但线程一直处于阻塞，降低了系统的吞吐量，这将导致应用的吞吐量下降；我们希望在调用依赖的服务响应比较慢，此时应该让出线程和CPU来处理下一个请求，当依赖的服务返回了再分配相应的线程来继续处理。而这应该有更好的解决方案：异步/协程。而Java是不支持协程的（虽然有些Java框架说支持，但还是高层API的封装），因此在Java中我们还可以使用异步来提升吞吐量。目前java一些开源框架（HttpClient\HttpAsyncClient、dubbo、thrift等等）大部分都支持。

 

# 几种调用方式

### 1 同步阻塞调用

即串行调用，响应时间为所有服务的响应时间总和；

 

### 2 半异步(异步Future)

线程池，异步Future，使用场景：并发请求多服务，总耗时为最长响应时间；提升总响应时间，但是阻塞主请求线程，高并发时依然会造成线程数过多，CPU上下文切换；

 

### 3 全异步(Callback)

Callback方式调用，使用场景：不考虑回调时间且只能对结果做简单处理，如果依赖服务是两个或两个以上服务，则不能合并两个服务的处理结果；不阻塞主请求线程，但使用场景有限。

 

### 4 异步回调链式编排

异步回调链式编排（JDK8 CompletableFuture），使用场景：其实不是异步调用方式，只是对依赖多服务的Callback调用结果处理做结果编排，来弥补Callback的不足，从而实现全异步链式调用。


# 代码示例 

### 1 同步阻塞调用

```java
public class Test {
   public static void main(String[] args) throws Exception {
       RpcService rpcService = new RpcService();
       HttpService httpService = new HttpService();
       //耗时10ms
       Map<String, String> result1 = rpcService.getRpcResult();
       //耗时20ms
       Integer result2 = httpService.getHttpResult();
       //总耗时30ms

    }

   static class RpcService {
       Map<String, String> getRpcResult() throws Exception {
           //调用远程方法（远程方法耗时约10ms，可以使用Thread.sleep模拟）
       }
    }

   static class HttpService {
       Integer getHttpResult() throws Exception {
           //调用远程方法（远程方法耗时约20ms，可以使用Thread.sleep模拟）
           Thread.sleep(20);
           return 0;
       }
    }
}
```
 

### 2 半异步(异步Future)
```java
public class Test {
   final static ExecutorService executor = Executors.newFixedThreadPool(2);
   public static void main(String[] args) {
       RpcService rpcService = new RpcService();
       HttpService httpService = new HttpService();
       Future<Map<String, String>> future1 = null;
       Future<Integer> future2 = null;

       try {
           future1 = executor.submit(() -> rpcService.getRpcResult());
           future2 = executor.submit(() -> httpService.getHttpResult());

           //耗时10ms
           Map<String, String> result1 = future1.get(300, TimeUnit.MILLISECONDS);
           //耗时20ms
           Integer result2 = future2.get(300, TimeUnit.MILLISECONDS);
           //总耗时20ms
       } catch (Exception e) {
           if (future1 != null) {
                future1.cancel(true);
           }

           if (future2 != null) {
                future2.cancel(true);
           }
           throw new RuntimeException(e);
       }
    }

   static class RpcService {
       Map<String, String> getRpcResult() throws Exception {
           //调用远程方法（远程方法耗时约10ms，可以使用Thread.sleep模拟）
       }
    }

   static class HttpService {
       Integer getHttpResult() throws Exception {
           //调用远程方法（远程方法耗时约20ms，可以使用Thread.sleep模拟）
       }
    }
}
```
 

### 3 全异步(Callback) 
```java
public class AsyncTest {
public staticHttpAsyncClient httpAsyncClient;
   public static CompletableFuture<String> getHttpData(String url) {
       CompletableFuture asyncFuture = new CompletableFuture();
       HttpPost post = new HttpPost(url);
       HttpAsyncRequestProducer producer = HttpAsyncMethods.create(post);
       AsyncCharConsumer<HttpResponse> consumer = newAsyncCharConsumer<HttpResponse>() {
            HttpResponse response;
           protected HttpResponse buildResult(final HttpContext context) {
                return response;
           }

…...

       };

       FutureCallback callback = new FutureCallback<HttpResponse>() {
           public void completed(HttpResponse response) {
               asyncFuture.complete(EntityUtils.toString(response.getEntity()));
           }

…...

       };

       httpAsyncClient.execute(producer, consumer, callback);
       return asyncFuture;
    }

 

   public static void main(String[] args) throws Exception {
       AsyncTest.getHttpData("网页链接);
       Thread.sleep(1000000);

    }

}
```

#### 参考资料
[网页链接](https://wely.iteye.com/blog/2346288)