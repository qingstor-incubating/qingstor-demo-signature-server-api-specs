# 签名服务器接口规范文档
[![Build Status](https://travis-ci.org/Colin0114/qingstor-demo-signature-server-api-specs.svg?branch=add_travis_ci)](https://travis-ci.org/Colin0114/qingstor-demo-signature-server-api-specs)

[English](../../README.md) | 中文

本文主要描述了签名服务器的接口。

## 规范
本文使用 [OpenAPI Specification (Swagger) v2.0](https://swagger.io/) 来描述签名服务器的接口。

## 签名服务器
用户成功创建了一个对象存储 Bucket 后，当需要访问 Bucket 并请求对象存储服务时，对象存储会使用对称加密的方法来验证请求者的身份，因此用户首先需要获取一对用于签名的[密钥（Access Key）](https://docs.qingcloud.com/qingstor/api/common/signature.html)。对于应用开发者来说，如果将密钥内嵌在应用客户端，必然存在安全隐患，很可能被他人恶意获取密钥，从而非法使用 Buckek 。为了保证密钥的安全，开发者可以根据 QingStor 签名方法，自己搭建并实现一个签名服务器。用于签名的密钥只需要在服务端保存，客户端不需要拿到，从而避免了认证信息泄漏的隐患。

签名服务器为客户端提供两类接口（请求参数和授权头）来对请求进行签名。

## 数据流
假设客户端请求 QingStor 对象存储服务前没有访问密钥对（ACCESS_KEY_ID 和 SECRET_ACCESS_KEY），
如果客户端想访问私有 Bucket，则必须先从签名服务器对每个请求进行签名。
下图描述了请求数据的流动过程：
```
+---------------------------------------------------------------------------+
|                                                                           |
|                           +-------------------------------+               |
|                           |                               |               |
|               +---------> |        QintStor 对象存储       |               |
|               |           |                               |               |
|               |           +----------------+--------------+               |
|       3、发送  |                            |                              |
|        已签名  |    +-----------------------+                              |
|        的请求  |    |        4、响应                                        |
|               |    v                                                      |
|               |                                                           |
|      +--------+--------+                                                  |
|      |                 |            2、获得签名                             |
|      |      客户端      | <-----------------------------+                  |
|      |                 |                               |                  |
|      +--------+--------+                    +----------+-----------+      |
|               |                             |                      |      |
|               +---------------------------> |      签名服务器        |      |
|                      1、签名请求              |                      |      |
|                                             +----------------------+      |
|                                                                           |
+---------------------------------------------------------------------------+
```

## 签名服务器接口实例
1\. **通过查询参数进行签名**

_请求实例_
```HTTP
POST /operation/query HTTP/1.1
Content-Type: application/json;
Host: 127.0.0.1:9000
Connection: close
User-Agent: Go-http-client/1.1
Content-Length: 272

{
   "method": "GET",
   "host": "pek3a.qingstor.com",
   "port": "443",
   "path": "/signature-test-bucket",
   "query": {
        "prefix": "test"
   },
   "protocol": "https",
   "headers": {
       "Date": "Wed, 16 Aug 2017 07:56:30 GMT",
       "Content-Length": "0",
       "User-Agent": "qingstor-sdk-go/2.2.6 (Go v1.8.3; linux_amd64_gc)"
    },
    "expires": "1502870310"
}
```
_响应实例:_
```HTTP
HTTP/1.1 200 OK
Content-Type: application/json;
Content-Length: 147
Date: Wed, 16 Aug 2017 07:56:30 GMT
Connection: close

{
    "access_key_id": "BCJGERIHUBJTBOEBRFKT",
    "signature": "yc5BeAHAXJ/3XjJb1YPucSX+NWAErY2kFJFj3n8t0us=",
    "expires": 1502870310
}
```

2\. **通过认证头进行签名**

_请求实例:_
```HTTP
POST /operation/header HTTP/1.1
Content-Type: application/json;
Host: 127.0.0.1:9000
Connection: close
User-Agent: Go-http-client/1.1
Content-Length: 297

{
    "method": "PUT",
    "host": "pek3a.qingstor.com",
    "port": "443",
    "path": "/signature-test-bucket/put-test-file",
    "protocol": "https",
    "headers": {
       "Date": "Wed, 16 Aug 2017 07:56:30 GMT",
       "Content-Length": "22",
       "User-Agent": "qingstor-sdk-go/2.2.6 (Go v1.8.3; linux_amd64_gc)"
    }
}
```

_响应实例:_
```HTTP
HTTP/1.1 200 OK
Content-Type: application/json;
Content-Length: 92
Date: Wed, 16 Aug 2017 07:56:30 GMT
Connection: close

{
    "authorization": "QS BCJGERIHUBJTBOEBRFKT:Y3nmxKVj4GMCBgTAujwCa9iDUwToI5hmPjW3rDsu/yg="
}
```

3\. **通过查询参数进行签名**

_请求实例:_
```HTTP
POST /string-to-sign/query HTTP/1.1
Content-Type: application/json;
Host: 127.0.0.1:9000
Connection: close
User-Agent: Go-http-client/1.1
Content-Length: 99

{
    "string_to_sign": "GET\n\n\n1502870311\n/signature-test-bucket/put-test-file",
    "expires": 1502870311
}
```

_响应实例:_
```HTTP
HTTP/1.1 200 OK
Content-Type: application/json;
Content-Length: 147
Wed, 16 Aug 2017 07:56:32 GMT
Connection: close

{
    "access_key_id": "BCJGERIHUBJTBOEBRFKT",
    "signature": "ESMKCKSGMyhdZ8Mo+7DAGoL4PlEUnNqCYDrrW6fneAg=",
    "expires": 1502870311
}
```

4\. **通过认证头进行签名**

_请求实例:_
```HTTP
POST /string-to-sign/header HTTP/1.1
Content-Type: application/json;
Host: 127.0.0.1:9000
Connection: close
User-Agent: Go-http-client/1.1
Content-Length: 106

{
    "string_to_sign": "DELETE\n\n\nWed, 16 Aug 2017 07:56:32 GMT\n/signature-test-bucket/signature-test-file",
}
```

_响应实例:_
```HTTP
HTTP/1.1 200 OK
Content-Type: application/json;
Content-Length: 92
Date: Wed, 16 Aug 2017 07:56:30 GMT
Connection: close

{
    "authorization": "QS BCJGERIHUBJTBOEBRFKT:IgA9wFwuSy+OYvZskldV+bl4VgvH9UNXsAXqcqVFM/A="
}
```

## 有效性校验
用来对 JSON 模式进行校验的工具有许多，本文选择 [z-schema](https://github.com/zaggino/z-schema) 来实现。

**注意：** 确保有 NodeJS 环境.
```bash
$ git clone https://github.com/yunify/qingstor-demo-signature-server-api-specs
$ cd qingstor-demo-signature-server-api-specs
$ npm install
$ npm t # or 'npm test'
```
## 参考文档
* [Integration Solution of Mobile App](https://docs.qingcloud.com/qingstor/solutions/app_integration.html)
* [JSON Schema](http://json-schema.org/)

## 许可
[The Apache License (Version 2.0, January 2004).](http://www.apache.org/licenses/LICENSE-2.0.html)
