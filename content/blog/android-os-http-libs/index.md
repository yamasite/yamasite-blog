---
title: HTTP 库 - OkHttp 和 AFNetworking
date: "2021-10-17T22:12:03.284Z"
description: "重新温习 Android 和 iOS 系统中的 HTTP 库"
---

最近由于工作原因，重新温习了 Android 和 iOS 系统中的 HTTP 库。除去系统原生 API 之外，还有一些社区封装的库。比较流行的莫属 OkHttp 和 AFNetworking。下文会对这两个库做一些梳理。

## OkHttp

OkHttp 应该是 Android 最流行的开源 HTTP 库。根据官方文档的描述，OkHttp 支持以下特性：

- HTTP/2。允许所有对同一个 host 的请求使用同一个 socket。
- 连接池。可以在不能使用 HTTP/2 的情况下降低请求延时。
- 透明压缩。OkHttp 会自动加入 gzip 请求头 `Accept-Encoding:gzip`。所以，当返回的数据带有 gzip 响应头 `Content-Encoding=gzip` 时，OkHttp 会自动解压数据。
- 响应缓存。支持在不消耗网络资源的情况下处理重复的请求。

### 基本概念

OkHttp 定义了一些基本概念，理解这些概念对高效使用 OkHttp 库非常重要。

#### Call

Call 在 OkHttp 中的实体是一个接口，原型如下：

```java
public interface Call
extends Cloneable
```

可以看到 Call 扩展了 Java 原生接口 `Cloneable`。最核心的成员方法是：

```java
// 执行 Request 并返回 Response
Response execute()
          throws IOException
```

Call 还嵌套了 `Call.factory` 接口。`Call.factory` 接口的成员方法是：

```java
Call newCall(Request request)
```

这个方法创建了一个 Call。Call 中的 Request 会在你调用 `execute()` 的时候执行。

#### OkHttpClient

```java
public class OkHttpClient
extends Object
implements Cloneable, Call.Factory, WebSocket.Factory
```

从原型上可以看出，`OkHttpClient` 实现了 `Cloneable`、`Call.Factory` 和 `WebSocket.Factory` 接口。而 `Call.Factory` 接口只有一个 `newcall` 方法。这也是 `OkHttpClient` 最常用的方法，同时也是 OkHttp 的核心逻辑。

#### Request

HTTP 请求对象。

#### Response

HTTP 响应对象。

### 获取 OkHttp

目前在 [Maven Central](https://search.maven.org/artifact/com.squareup.okhttp3/okhttp) 上托管。通过 gradle 即可获取：

```gradle
implementation("com.squareup.okhttp3:okhttp:4.9.2")
```

> OkHttp 4.x 使用了 Kotlin 语言对库进行了重构。OkHttp 3.x 及之前版本是基于 Java 实现的。

值得一提的是，OkHttp 提供了 [Bill of Materials (BOM) 方法](https://docs.gradle.org/6.2/userguide/platforms.html#sub:bom_import)，让你轻松解决依赖兼容问题。

```gradle
    dependencies {
       // define a BOM and its version
       implementation(platform("com.squareup.okhttp3:okhttp-bom:4.9.2"))

       // define any required OkHttp artifacts without version
       implementation("com.squareup.okhttp3:okhttp")
       implementation("com.squareup.okhttp3:logging-interceptor")
    }
```


### 基本使用方法

OkHttp 使用起来非常简单。官方提供了一个极简示例项目，清晰地展示了 OkHttp 的基本用法。下面对这个极简示例项目进行解析：

```java
package okhttp3.guide;

import java.io.IOException;
//  OkHttpClient 类。
import okhttp3.OkHttpClient;
// Request 类。
import okhttp3.Request;
// Response 类。
import okhttp3.Response;

public class GetExample {
  // 创建一个 OkHttpClient 对象。
  // 使用 final 是因为 OkHttp 推荐所有 HTTP call 共用一个 OkHttpClient 实例。
  // 这样所有 call 可以共享一个连接池和线程池，从而显著降低延迟、节省内存。
  final OkHttpClient client = new OkHttpClient();

  String run(String url) throws IOException {
    // 创建一个 Request 对象并设置参数
    Request request = new Request.Builder()
        .url(url)
        .build();

    // 1. 创建 Response 对象。
    // 2. 使用 OkHttpClient 对象创建一个 call 并执行 call。
    // 3. 返回 response 的 body。
    try (Response response = client.newCall(request).execute()) {
      return response.body().string();
    }
  }

  public static void main(String[] args) throws IOException {
    GetExample example = new GetExample();
    // 设置请求 URL
    String response = example.run("https://raw.github.com/square/okhttp/master/README.md");
    // 打印 response 的 body
    System.out.println(response);
  }
}
```


## AFNetworking

AFNetworking 应该是 iOS、macOS、watchOS 和 tvOS 最流行的开源 HTTP 库。AFNetworking 基于 [Foundation URL Loading System](https://developer.apple.com/documentation/foundation/url_loading_system) 构建。


### 基本概念

下文介绍 AFNetworking 中的核心基本概念。

#### AFURLSessionManager 和 AFHTTPSessionManager

`AFURLSessionManager` 对象基于 `NSURLSessionConfiguration` 对象创建并维护一个 `NSURLSession` 对象。

```objective-c
@interface AFURLSessionManager : NSObject <NSURLSessionDelegate, NSURLSessionTaskDelegate, NSURLSessionDataDelegate, NSURLSessionDownloadDelegate, NSSecureCoding, NSCopying>
...
```

`AFURLSessionManager` 包含上传、下载、数据发送等任务类型。

```objective-c
// 上传任务
- (NSURLSessionUploadTask *)uploadTaskWithRequest:(NSURLRequest *)request
                                         fromFile:(NSURL *)fileURL
                                         progress:(nullable void (^)(NSProgress *uploadProgress))uploadProgressBlock
                                completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject, NSError  * _Nullable error))completionHandler;
```

```objective-c
// 下载任务
- (NSURLSessionDownloadTask *)downloadTaskWithRequest:(NSURLRequest *)request
                                             progress:(nullable void (^)(NSProgress *downloadProgress))downloadProgressBlock
                                          destination:(nullable NSURL * (^)(NSURL *targetPath, NSURLResponse *response))destination
                                    completionHandler:(nullable void (^)(NSURLResponse *response, NSURL * _Nullable filePath, NSError * _Nullable error))completionHandler;
```

```objective-c
// 数据发送任务
- (NSURLSessionDataTask *)dataTaskWithRequest:(NSURLRequest *)request
                               uploadProgress:(nullable void (^)(NSProgress *uploadProgress))uploadProgressBlock
                             downloadProgress:(nullable void (^)(NSProgress *downloadProgress))downloadProgressBlock
                            completionHandler:(nullable void (^)(NSURLResponse *response, id _Nullable responseObject,  NSError * _Nullable error))completionHandler;

```

`AFHTTPSessionManager` 是 `AFURLSessionManager` 的子类，包含了 HTTP 请求操作的相关逻辑，例如 GET、POST 等。

```objective-c
@interface AFHTTPSessionManager : AFURLSessionManager <NSSecureCoding, NSCopying>
...
```

```objective-c
// 发送 GET 请求
- (nullable NSURLSessionDataTask *)GET:(NSString *)URLString
                            parameters:(nullable id)parameters
                               headers:(nullable NSDictionary <NSString *, NSString *> *)headers
                              progress:(nullable void (^)(NSProgress *downloadProgress))downloadProgress
                               success:(nullable void (^)(NSURLSessionDataTask *task, id _Nullable responseObject))success
                               failure:(nullable void (^)(NSURLSessionDataTask * _Nullable task, NSError *error))failure;
```

### 获取 AFNetworking

你可以通过 CocoaPods 获取 AFNetworking。

```pod
target "MyTargetName" do
    source 'https://github.com/CocoaPods/Specs.git'

    ...

    pod 'AFNetworking', '~> 4.0'

    ...

end
```

也可以用 Swift Package Manager 安装：

```swift
dependencies: [
    .package(url: "https://github.com/AFNetworking/AFNetworking.git", .upToNextMajor(from: "4.0.0"))
]
```

### 基本使用方法

1. 引入 AFNetworking。

```objective-c
#import <AFNetworking/AFNetworking.h>
```

2. 创建 `NSURLSessionConfiguration` 对象和 `AFURLSessionManager` 对象。

```objective-c
// 创建 NSURLSessionConfiguration 对象
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];

// 创建 AFURLSessionManager 对象
AFURLSessionManager *manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:configuration];
```

3. 使用 `AFURLSessionManager` 对象执行上传、下载、或数据发送任务。

```objective-c
// 执行上传任务
NSURL *URL = [NSURL URLWithString:@"http://example.com/upload"];
NSURLRequest *request = [NSURLRequest requestWithURL:URL];

NSURL *filePath = [NSURL fileURLWithPath:@"file://path/to/image.png"];
NSURLSessionUploadTask *uploadTask = [manager uploadTaskWithRequest:request fromFile:filePath progress:nil completionHandler:^(NSURLResponse *response, id responseObject, NSError *error) {
    if (error) {
        NSLog(@"Error: %@", error);
    } else {
        NSLog(@"Success: %@ %@", response, responseObject);
    }
}];
[uploadTask resume];
```

```objective-c
// 执行下载任务
NSURL *URL = [NSURL URLWithString:@"http://example.com/download.zip"];
NSURLRequest *request = [NSURLRequest requestWithURL:URL];

NSURLSessionDownloadTask *downloadTask = [manager downloadTaskWithRequest:request progress:nil destination:^NSURL *(NSURL *targetPath, NSURLResponse *response) {
    NSURL *documentsDirectoryURL = [[NSFileManager defaultManager] URLForDirectory:NSDocumentDirectory inDomain:NSUserDomainMask appropriateForURL:nil create:NO error:nil];
    return [documentsDirectoryURL URLByAppendingPathComponent:[response suggestedFilename]];
} completionHandler:^(NSURLResponse *response, NSURL *filePath, NSError *error) {
    NSLog(@"File downloaded to: %@", filePath);
}];
[downloadTask resume];
```

```objective-c
// 执行数据发送任务
NSURL *URL = [NSURL URLWithString:@"http://httpbin.org/get"];
NSURLRequest *request = [NSURLRequest requestWithURL:URL];

NSURLSessionDataTask *dataTask = [manager dataTaskWithRequest:request completionHandler:^(NSURLResponse *response, id responseObject, NSError *error) {
    if (error) {
        NSLog(@"Error: %@", error);
    } else {
        NSLog(@"%@ %@", response, responseObject);
    }
}];
[dataTask resume];
```

AFNetworking 的仓库无论是代码和文档都有相当一段时间没有更新了，活跃程度远不及基于 Swift 的 Alamofire。而且 Alamofire 的功能比 AFNetworking 更全面，文档也更加丰富（参考 [Alamofire 官方文档](https://alamofire.github.io/Alamofire/index.html)）。

同样是发送一个 request，Alamofire 的实现是：

1. 引入 Alamofire。

```swift
import Alamofire
```

2. 实现请求。

```swift
AF.request("https://httpbin.org/get").response { response in
    debugPrint(response)
}
```

