# QingStor Demo Signature Server API Specs
English | [中文](./README.zh_CN.md)

Specifications of QingStor Demo Signature Server API.

## Specification
We use [OpenAPI Specification (Swagger) v2.0](https://swagger.io/) to describe QingStor Demo Signature Server API.

## About QingStor Demo Signature Server
When a user need to visit a Bucket and call services of Object Storage after a Bucket was success created, Object Storage will use symmetrical encryption to verify a visitor's identification, so a pair of [Access Key](https://docs.qingcloud.com/qingstor/api/common/signature.html) to sign the request is needed. For an Application developer, it is unsafe to store Access Key inside Application clients, cause someone else can illegally use the Bucket by getting the Access Key intentionly. One solution is that the developer implements a signature server by using signature methods provided by QingStor, which can keep the Access Key safe by putting it inside signature server while clients don't need Access Key anymore.

QingStor Demo Signature Server provides two kinds of API(Query parameters and Authorization header) for clients to sign the request before sending it to the QingStor Object Storage.

## Data Flow
Assume that we have a client that knows nothing about Access Key (ACCESS_KEY_ID and SECRET_ACCESS_KEY) before requiring service from QingStor Object Storage. If the client wants to access to non-public bucket directly, it needs the signature generated from QingStor Demo Signature Server for every request.

Here's a diagram describes the data flow:
```
+---------------------------------------------------------------------------+
|                                                                           |
|                           +-----------------------------+                 |
|                           |                             |                 |
|               +---------> |   QingStor Object Storage   |                 |
|               |           |                             |                 |
|               |           +--------------+--------------+                 |
|       3. Send |                          |                                |
|        Signed |    +---------------------+                                |
|       Request |    |   4. Response                                        |
|               |    v                                                      |
|               |                                                           |
|      +--------+--------+                                                  |
|      |                 |        2. Get Signature                          |
|      |     Client      | <-----------------------------+                  |
|      |                 |                               |                  |
|      +--------+--------+                    +----------+-----------+      |
|               |                             |                      |      |
|               +---------------------------> |   Signature Server   |      |
|                      1. Sign Request        |                      |      |
|                                             +----------------------+      |
|                                                                           |
+---------------------------------------------------------------------------+
```

## Examples of the API of QingStor Demo Signature Server
1\. **Sign operation by Query parameters**

_Request Example:_
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
_Response Example:_
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

2\. **Sign operation by Authorization header**

_Request Example:_
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
    "body": "put-test-file-content\n",
    "headers": {
       "Date": "Wed, 16 Aug 2017 07:56:30 GMT",
       "Content-Length": "22",
       "User-Agent": "qingstor-sdk-go/2.2.6 (Go v1.8.3; linux_amd64_gc)"
    }
}
```

_Response Example:_
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

3\. **Sign operation by Query parameters**

_Request Example:_
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

_Response Example:_
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

4\. **Sign operation by Authorization header**

_Request Example:_
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

_Response Example:_
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

## Validation
There are lots of tools to validate JSON file with its JSON schema, we choose [z-schema](https://github.com/zaggino/z-schema) to do this.

**Notice:** NodeJS is required.
```bash
$ git clone https://github.com/yunify/qingstor-demo-signature-server-api-specs
$ cd qingstor-demo-signature-server-api-specs
$ npm install
$ npm t # or 'npm test'
```

### Reference Documentation
* [Integration Solution of Mobile App](https://docs.qingcloud.com/qingstor/solutions/app_integration.html)
* [JSON Schema](http://json-schema.org/)

### LICENSE
[The Apache License (Version 2.0, January 2004).](http://www.apache.org/licenses/LICENSE-2.0.html)
