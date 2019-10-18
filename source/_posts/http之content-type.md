---
title: http之content-type
date: 2019-10-18 14:21:31
tags: content-type http
---
## 背景
没啥背景，就是想整理下，发现年纪大了，记不住了。

## 定义
content-type: `实体头部` 用于指示资源的 `媒体类型（MIME类型）`

## MIME类型
### 语法
> type/subtype

由类型与子类型两个字符串中间用'/'分隔而组成。不允许空格存在。type 表示可以被分多个子类的独立类别。subtype 表示细分后的每个类型。
MIME类型对大小写不敏感，但是传统写法都是小写。

### types
#### 独立类型

类型 | 描述 | 示例
---- | --- | ----
text | 表明文件是普通文本，理论上是人类可读 | text/plain（默认）, text/html, text/css, text/javascript
image | 表明是某种图像。不包括视频，但是动态图（比如动态gif）也使用image类型 | image/gif, image/png, image/jpeg, image/bmp, image/webp, image/x-icon, image/vnd.microsoft.icon
audio | 表明是某种音频文件 | audio/midi, audio/mpeg, audio/webm, audio/ogg, audio/wav
video |	表明是某种视频文件 | video/webm, video/ogg
application | 表明是某种二进制数据 | application/octet-stream（默认）, application/pkcs12, application/vnd.mspowerpoint, application/xhtml+xml, application/xml,  application/pdf

#### multipart类型
Multipart 类型表示细分领域的文件类型的种类，经常对应不同的 MIME 类型。这是复合文件的一种表现方式。

`Content-Type: multipart/mixed; boundary=gc0p4Jq0M2Yt08jU534c0p`
其中boundary字段是必须的，用于区分消息体中的数据边界，一般是两个'-'的格式作为该端的开头，例如： 
`--gc0p4Jq0M2Yt08jU534c0p`

1. multipart/form-data 
    用于HTML表单，从浏览器发送信息给服务器。
    ```html
   <!-- html表单 -->
   <form>
     <div class="form-item">
       <label for="name">姓名：</label>
       <input type="text" name="name" id="name" />
     </div>
     <div class="form-item">
       <label for="pic">照片：</label>
       <input type="file" name="image" id="image" />
     </div>
     <div class="form-item"><span id="commit">提交</span></div>
   </form>
    ```
   
   ```javascript
   function upload() {
     var formData = new FormData();
     var name = document.querySelector("#name").value;
     var image = document.querySelector("#image").files[0];
     formData.append("name", name);
     formData.append("image", image);

     var xhr = new XMLHttpRequest();
     xhr.open("POST", "/user");
     xhr.onreadystatechange = function() {
       if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
         console.log(xhr.responseText);
         alert(xhr.responseText);
       }
     };
     xhr.send(formData);
   }

   document.querySelector("#commit").addEventListener(
     "click",
     function() {
       upload();
     },
     false
   );
   ```
   ![Form-data](/images/article/contentType/multipart:form-data.png)

   从图中可以看出，这个 `Form-data`，是个复合数据。表单有两个值，一个是 `name`，一个是 `iamge`。
   
   ##### Content-Disposition小科普
   当页面上有表单，并且我们选择的表单提交方式为 multipart/form-data 时，Content-Disposition 会出现在请求体中。其作用是说明对应的表单项的字段名是什么，表单中上传的文件名是什么。在该场景下第一个参数总是固定不变的 form-data，另外有两个可选参数。name 表示对应的表单项的字段名，filename 表示对应的上传的文件名。参数之间使用 ; 进行分割。
   Content-Disposition 在 multipart/form-data 的请求体头中可能会这样出现：
   ```
    Content-Disposition: form-data
    Content-Disposition: form-data; name="fieldName"
    Content-Disposition: form-data; name="fieldName"; filename="filename.jpg"
    ```

2. multipart/byteranges


### 常见的类型
#### 静态资源、图片、媒体类
`text/xxx 、image/xxx 、video/xxxx`

#### application
##### application/x-www-form-urlencoded
是表单提交中默认的类型，要求传送的数据是 `键值对`。
```
key1=value1&key2=value2
```

数据被编码成以 '&' 分隔的键-值对, 同时以 '=' 分隔键和值. 非字母或数字的字符会被 `百分号编码（percent-encoding，又称URL编码）`: 这也就是为什么这种类型不支持二进制数据的原因 (应使用 multipart/form-data 代替).

但是回想我们平时请求接口的时候，不会自己做去这个事情，因为我们依赖的库 `jquery` / `axios` 帮我们做了这个事情。
在 `chrome devTools` 里面，会有一个 `Form Data` 的栏目，工具会帮我们解析好，展示出来。可以点一下右边的 `view source` 看本来的格式。

当然对于服务端来说收到的就是一个 `key1=value1&key2=value2` 字符串，需要自己去取下

##### application/json
JSON格式支持比键值对复杂得多的结构化数据。发给服务端的数据就是一个 `JSON 字符串`。
在 `chrome devTools` 里面，会有一个 `Request Payload` 的栏目，工具会帮我们解析好，展示出来。同样可以 `view source` 看本来的格式。

对于服务端来说收到的就是一个 `JSON 字符串`，需要自己parse一下。

返回来，服务端的接口返回的 `response header` 里面的 `content type` 一般来说是 `application/json`，所以我们接到数据后，应该 `parse` 一下，当然，依赖的库会帮我们做掉

#### multipart/form-data
见之前的

## content-type的指令
1. media-type
资源或数据的 MIME type 。
2. charset
字符编码标准。
3. boundary
对于多部分实体，boundary 是必需的，其包括来自一组字符的1到70个字符，已知通过电子邮件网关是非常健壮的，而不是以空白结尾。它用于封装消息的多个部分的边界。


