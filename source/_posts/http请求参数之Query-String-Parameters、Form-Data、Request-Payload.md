---
title: http请求参数之Query String Parameters、Form Data、Request Payload
date: 2019-10-23 23:13:43
tags: http Query String Parameters Form Data Request Payload
---
### 背景
跟接口对接的时候经常会发现这边传的格式和位置，跟接口套不上，糊里糊涂的。

### Query String Parameters、Form Data、Request Payload
HTTP请求中不同的请求方式和设置不同的Content-Type时，参数传递的方式会不一样，
一起了解这三种形式：Query String Parameters、Form Data、Request Payload；

这三种形式在chrome devTools 里也会有体现。

### Get请求
GET请求时，参数会以 `url string` 的形式进行传递，即?后的字符串则为其请求参数，并以 `&` 作为分隔符。

```
Request URL: http://test.com?from_type=省&from_name='四川省'
Request Method: GET
```

![query-string-parameters](/images/http/querystringparameters.png)

### Post请求
有两种形式的请求体：`FormData`，`Request Payload`

#### FormData
当发起一次Post请求，若未指定 `Content-type`，则默认 `content-type` 为 `application/x-www-form-urlencoded`，
即参数会以 `FormData` 的形式进行传递，不会显示出现在请求URL中。传递给接口的是 `key1=value1&key2=value2` 的形式。

![form-data](/images/http/formdata.png)

##### formData()方法
当我们遇到一些文件上传功能时，我们需要使用原生的 `formData()` 来进行数据组装，
且 `content-type` 需要设置为 `multipart/formdata` 

http请求头
```
Request URL: http://test.com/upload
Request Method: POST
```
![form-data1](/images/http/formdata1.png)

其中，`WebKitFormBoundarysBkB6WoEBvbCRkmh` 为浏览器随机生成的 `boundary` ，作为分隔参数，作用等同于 `&`。

#### Request Payload
当发起一次post请求，若 `Content-Type` 为 `application/json` ，则参数会以 `Request Payload` 的形式进行传递（数据格式为json），
不会显示出现在请求url中。传递给接口的是 `JSON字符串` 的形式。

![request-payload](/images/http/requestpayload.png)

### 特殊
在实际对接中，即使知道里上面这些分类，也还是会出错，因为还有一种情况。

我们讲的 `Get`，`Post` 都是基于应用层的，基于 `http` 协议的，然后应用层下面是传输层，`tcp` 协议，也就是在 `post` 和 `get`，在传输上没啥区别，
也就是说 `post请求` 也可以以 `url string` 的形式进行传递，即?后的字符串则为其请求参数，并以 `&` 作为分隔符。重点看服务端接不接。

```
  // axios文档的摘要

  // `transformRequest` 允许在向服务器发送前，修改请求数据
  // 只能用在 'PUT', 'POST' 和 'PATCH' 这几个请求方法
  // 后面数组中的函数必须返回一个字符串，或 ArrayBuffer，或 Stream
  transformRequest: [function (data, headers) {
    // 对 data 进行任意转换处理
    return data;
  }],

  // `transformResponse` 在传递给 then/catch 前，允许修改响应数据
  transformResponse: [function (data) {
    // 对 data 进行任意转换处理
    return data;
  }],

  // `params` 是即将与请求一起发送的 URL 参数
  // 必须是一个无格式对象(plain object)或 URLSearchParams 对象
  params: {
    ID: 12345
  },

   // `paramsSerializer` 是一个负责 `params` 序列化的函数
  // (e.g. https://www.npmjs.com/package/qs, http://api.jquery.com/jquery.param/)
  paramsSerializer: function(params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  // `data` 是作为请求主体被发送的数据
  // 只适用于这些请求方法 'PUT', 'POST', 和 'PATCH'
  // 在没有设置 `transformRequest` 时，必须是以下类型之一：
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属：FormData, File, Blob
  // - Node 专属： Stream
  data: {
    firstName: 'Fred'
  },
```














