[![Build Status](https://travis-ci.org/joostvunderink/angular-jsonrpc-client.svg?branch=master)](https://travis-ci.org/joostvunderink/angular-jsonrpc-client)

# 背景知识

angular-jsonrpc-client provides a configurable client to perform [JSON-RPC 2.0][JSONRPC2] calls via HTTP to one or more servers.

angular-jsonrpc-client 提供可配置的客户端去执行  [JSON-RPC 2.0][JSONRPC2] 请求，并支持配置多个服务器端点

[JSONRPC2]: http://www.jsonrpc.org/specification

JSON-RPC is a protocol where you send a JSON object with a method name and method parameters to a server, and you get a response with the result of the operation. Let's say you send the following JSON:

JSON-RPC 是一种让你通过发送 JSON 对象，通过方法、方法参数传送给服务端，然后获取服务端的响应结果。比如下面就是一个常见的发送请求：

```json
    {
        "jsonrpc": "2.0",
        "id": "1",
        "method": "buy_fruit",
        "params": {
            "type": "banana",
            "amount": 42
        }
    }
```

The server might reply with an object like this:

服务器返回这样：

```json
    {
        "jsonrpc": "2.0",
        "id": "1",
        "result": {
            "cost": 24.65
        }
    }
```

Or, if an error happens, the server would reply with an object like this:

如果服务器发生错误，可能会返回给你这样的结果：

```json
    {
        "jsonrpc": "2.0",
        "id": "1",
        "error": {
            "code": 666,
            "message": "There were not enough bananas.",
            "data": {
                "available_bananas": 17
            }
        }
    }
```

In addition to such a JSON-RPC server error, it could also happen that something goes wrong with sending the request to the server. For example, the server could be down, or the client could be configured with the wrong location of the server. In that case, you would also experience an error situation, although different from JSON-RPC errors.

JSON-RPC 服务器出错是常事，在客户端发送的过程中也可能会出错。比如，服务器不在线，或者客户端配错了服务器地址。这种情况下也会给你一个错误。

# 介绍

This client takes care of the communication and the error handling when communicating with a JSON-RPC server. By default it returns a `$q` promise, which either resolves to a result value via `.then()` or results in an error via `.catch()`.

这个客户端负责与 JSON-RPC 服务器的通讯与错误处理。默认情况下处理完后返回 `$q` promise，promise 要么成功，调用 `.then()` 或者出错，调用 `.catch()`

Currently, it can only handle JSON-RPC servers that are reachable via HTTP.

现在这个客户端还只支持通过 HTTP 的 JSON-RPC 

The client is configured in the Angular configuration phase via the `jsonrpcConfig` provider, and the client itself is injected as `jsonrpc`.

本客户端在 Angular 配置阶段提供 `jsonrpcConfig` provider 对象，客户端本身注入为 `jsonrpc`

# 开始动手吧 

```javascript
// 单一服务器配置:
// (大部分情况下这样的配置没问题)
angular
    .module('MyApp', ['angular-jsonrpc-client'])
    .config(function(jsonrpcConfigProvider) {
        jsonrpcConfigProvider.set({
            url: 'http://example.com:8080/rpc'
        });
    })
    .controller('MyController', ['$scope', 'jsonrpc', function($scope, jsonrpc) {
        jsonrpc.request('version', {})
            .then(function(result) {
                $scope.result = result;
            })
            .catch(function(error) {
                $scope.error = error;
            });
    }]);

// 或者看看这里，多服务器配置
angular
    .module('MyApp', ['angular-jsonrpc-client'])
    .config(function(jsonrpcConfigProvider) {
        jsonrpcConfigProvider.set({
            servers: [
                {
                    name: 'first',
                    url: 'http://example.com:8080/rpc'
                },
                {
                    name: 'second',
                    url: 'http://example.net:4444/api',
                    headers: {
                        'Authorization': 'Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ=='
                    }
                }
            ]
        });
    })
    .controller('MyController', ['$scope', 'jsonrpc', function($scope, jsonrpc) {
        jsonrpc.request('first', 'version', {})
            .then(function(result) {
                $scope.result = result;
            })
            .catch(function(error) {
                $scope.error = error;
            });
    }]);
```

# 配置

3个可配置项

| 参数                | 是否可选 | 类型      | 默认值   | 描述                                      |
| ----------------- | ---- | ------- | ----- | --------------------------------------- |
| url               | 可选   | string  | null  | JSON-RPC HTTP 服务器的地址.                   |
| servers           | 可选   | array   | null  | 多个服务器的配置.                               |
| returnHttpPromise | 可选   | boolean | false | 是否返回一个 `$http` promise 或者 `$q` promise. |

You must provide either `url` or `servers` to configure which backend(s) will be used. If you provide the `url` argument, a single backend called `main` will be created internally.

 `url` 和 `servers`  必须配置一个。如果只配置 `url`  参数，会默认创建一个叫 `main` 的服务器配置

The argument `url` is a string with the URL of the JSON-RPC server.

The argument `servers` is an array of objects. Each object contains three keys: `name` for the symbolic name of this backend, `url` for the URL of that JSON-RPC server, and an optional key `headers` to make the client pass custom header to each request.

The argument `returnHttpPromise` can be used to return a `$http` promise instead of a `$q` promise, if you want to handle the $http errors yourself. See the [Http Promise](#http-promise) section for more information.

# 调用 `jsonrpc.request()`

The method `jsonrpc.request()` can be called with 1, 2 or 3 arguments:

`jsonrpc.request()` 可以最多有 3 个参数

`jsonrpc.request(requestObject)`

`jsonrpc.request(methodName, args)`

`jsonrpc.request(serverName, methodName, args)`

If it's called with 2 arguments, the serverName is set to `main` internally. This is the same internal name as when you call jsonrpcConfig with the `url` parameter.

如果提供 2 个参数，那么 serverName 会被设置为 `main`，就像前面说的配置 `url`一样

If called with 1 argument, you can use the following keys:

如果提供 1 个参数的对象，那么可以用下面这些 key

| Key        | Description       |
| ---------- | ----------------- |
| serverName | 服务器名字             |
| methodName | 方法名               |
| methodArgs | 方法的参数（数组）         |
| config     | 提供一个传递给 $http 的对象 |

# 运行中设置请求头

Sometimes, you don't have the authentication headers during the Angular configuration phase yet, for example because you will only receive them as result of a "log in" JSON-RPC call. It is possible to add headers later on, via `jsonrpc.setHeaders(serverName, headers)`. For example:

有时候你想设置一个认证头，可能在 Angular 配置阶段不合适，因为你还没有登录。但是不要担心，在登录成功后你可以这样设置，通过 `jsonrpc.setHeaders(serverName, headers)`. 比如下面例子：

```
jsonrpc.setHeaders('main', {
    AuthToken: 'my auth token'
});
```

# JSONRPC 批量处理

To send several Request objects at the same time, the Client may send an Array filled with Request objects.

如果你想把几个 Request 对象一起发送，可以通过下面的方法

## 调用 `jsonrpc.batch()`

The method `jsonrpc.batch()` can be called with either 0 or with 1 argument:

`jsonrpc.batch(serverName)`

`jsonrpc.batch()`

If it's called with 0 arguments, the serverName is set to `main` internally. This is the same internal name as when you call jsonrpcConfig with the `url` parameter.

The method returns a new `batch` object.

## 使用 batch 请求

To use the batch request, you have to create a new batch object with `jsonrpc.batch()`. You can add new requests with `batch.add(methodName, args)` and send the batch request with `batch.send()`.

首先要调用 `jsonrpc.batch()`  得到一个 batch 对象，然后通过 `batch.add(methodName, args)` 添加请求，然后通过 `batch.send()` 来发送请求

The method `batch.add()` returns by default a `$q` promise like `jsonrpc.request()`. If you configure usage of `$http` promise the return value is the id of the request to identify the response.

默认情况下 `batch.add()`  返回一个 `$q` promise，但如果你配置为返回 `$http`，这种情况下将返回一个请求的 id

The method `batch.send()` returns just as `jsonrpc.request()` by default a `$q` promise. The result of the resolved `$q` promise isn't set. After calling `batch.send()` the batch data is cleared.

方法  `batch.send()` 默认也是返回一个 `$q` promise，但是请注意这个 `$q`  resolve 时没有数据返回的。

For example:
```
    // 创建批量对象
    var batch = jsonrpc.batch();
    
    // 添加请求（以及请求的处理 then）
    batch.add("foo.bar", [])
        .then(handleFooBar);
    batch.add("bar.foo", [])
        .then(handleBarFoo);
        
    // 发送批量请求
    batch.send()
        .then(handleSend);
```

# Return value

By default, the return value of `jsonrpc.request` is a `$q` promise, which resolves into the `result` value of the JSON-RPC response. If anything goes wrong with handling the request, the code ends up in `$q.catch()`.

# Handling errors

When dealing with JSON-RPC, there are 2 kinds of errors. The first kind is when your request did not arrive at the server, for example because it was down or because the wrong URL was used. The second kind of error is when the server has received the request and returns an error structure as specified by [JSON-RPC 2.0][JSONRPC2].

It would be possible to handle errors like this:

```
    // Note: this is NOT how this module works! Don't use this code!
    jsonrpc.request(methodName, params)
        .then(function(response) {
            if (response.result) {
                // The JSON-RPC call was successful.
                $scope.result = response.result;
            }
            else {
                // The request was received but an error occurred during processing.
                $scope.jsonError = response.error;
            }
        })
        .catch(function(error) {
            // This is an HTTP transport error, for example the wrong URL.
            $scope.httpError = error;
        });
```

The disadvantage of this method, is that you need to check for `response.result` or `response.error` in the `.then()` block for every single request you do.

This module moves the JSON-RPC error to the `.catch()` block. That means that if any error has occurred, the code flow ends up in the `.catch()` path. So, in the `.then()` block, you can be sure that the JSON-RPC call has been performed successfully. This leads to the following code flow on the client side:

```
    jsonrpc.request(methodName, params)
        .then(function(result) {
            // The JSON-RPC call was successful.
            $scope.result = result;
        })
        .catch(function(error) {
            // This could be either a JSON-RPC error or an HTTP transport error.
            $scope.error = error;
        });
```

This leads to simpler and less client code for the vast majority of the situations, where you are interested in what happens if the JSON-RPC call succeeds.

To differentiate between the two types of errors, we use the type of the `error` object that is rejected by `jsonrpc.request()` and thus is the only argument for `.catch(function(error) { ... })`. This `error` argument is an object and is either of type `JsonRpcTransportError` (for the first type of errors) or `JsonRpcServerError`.

In both cases, `error.message` is set. In case of a `JsonRpcServerError`, `error.error` contains the error object as it is specified by [JSON-RPC 2.0][JSONRPC2].

# Http Promise

If, for some reason, you want to handle the `$http` response yourself, you can do that via:

```
jsonrpcConfigProvider.set({
    returnHttpPromise: true
});
```

This changes the return value of `jsonrpc.request()` and `jsonrpc.batch()` from a `$q` promise into the return value of `$http.request()`. See `examples/example2.html` for a working example of this. Doing this makes your client code more complex and is not recommended.

# Cancelling a JSON-RPC call

By using the 1 argument way of calling `jsonrpc.request`, it's possible to pass any options to the internal `$http` request that's made. You can use this to cancel a JSON-RPC call.

```
    var canceller = $q.defer();

    $scope.cancelRequest = function() {
        canceller.resolve('cancelled');
    };

    jsonrpc.request({
        serverName: serverName,
        methodName: methodName,
        methodArgs: methodArgs,
        config: {
            timeout: canceller.promise
        }
    });
```

# Monitoring progress

In the same way you can cancel a request, you can also monitor the progress of a request. You will need AngularJS 1.5.4 or later for this to work. Here is an example:

```
    function logProgress(progressEvent) {
        console.log('Received progress event:');
        console.log(progressEvent);
    }

    jsonrpc.request({
        serverName: serverName,
        methodName: methodName,
        methodArgs: methodArgs,
        config: {
            eventHandlers: {
                progress: logProgress
            }
        }
    });
```

Note that you might need to use `uploadEventHandlers` if `eventHandlers` does not do the trick. See https://docs.angularjs.org/api/ng/service/$http for more information.

# Examples

In the `examples` dir, there are a few examples. You can look at the code there, and you can also see the code in action. To do that, you need to make sure that a the local example JSON-RPC server is running:

```
npm install
cd example
node jsonrpc.server.js
```

Now you can open the example html files in your browser and the calls should work.

# TODO

- Maybe add support for JSON-RPC v1?
- Add support for JSON-RPC over TCP

# Contributors

The following people have contributed code to this module:

- [Gerrit Kaul](https://github.com/GerritK)
- [Jens](https://github.com/derfux)
