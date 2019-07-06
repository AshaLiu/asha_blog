---
title: 如何使用微信的jsSDK
date: 2017-06-07 14:16:44
tags: 微信 jsSDK
---
### 前言
现在的微信用户太强大了，公司的一些运营的活动啊，总是希望能够定制微信的自定义分享，但是需要`jsSDK`，我们又没做公众号开发，只能闲暇的时候看看怎么弄了。  
以前还能投机取巧些说分享图片取第一张图片，现在微信升级之后，好像是在6.8.5版本之后，如果不及接入jsSDK的话，就默认使用微信官方的一个锁的那个默认图，怎么说呢，体验会不是很好。

### 申请微信公众号
公众号分什么订阅号，服务号，个人号，企业号，不同需求不同功能嘛，都有开发者模式。但是我作为个人开发者是申请不到服务号的，怎么办呢，腾讯微我们准备了测试账号，通过这个账号，我们可以获得微信服务号的所有功能和接口调用权限。  
[申请地址](http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)  
申请之后你会拿到一个`appid`和`appsecret`，这个其实差不多就是你的公众号的秘钥一样，在拿`access_token`需要用到的。

### 准备工作
其实微信[官方文档](https://mp.weixin.qq.com/advanced/wiki?t=t=resource/res_main&id=mp1421140183)说的很清楚。  
微信是一个app，我们h5是一个网页，那我们怎么在app里面分享、拍照上传等动作呢，这时候就需要一个h5和app的桥，叫jsSDK。那使用这个jsSDK的前提是什么？

#### 步骤一：绑定域名
登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS接口安全域名”。  
备注：登录后可在“开发者中心”查看对应的接口权限。  
测试账号的话直接拉倒下面就可以看到填写得地方了。注意，我这里卡了好久。首先，先解释下什么叫二级域名  
这里我举个例子就能明白了：
   - .com 顶级域名
   - baidu.com 一级域名
   - www.baidu.com 二级域名
   - bbs.baidu .com 二级域名
   - tieba.baidu .com 二级域名
     
设置js安全域名后，公众号开发者可以在该域名下调用微信的jsSDK，js接口的安全域名要求是一级或一级以上，可以填写三个。  
如果调用js的域名是二级域名，而在JS接口安全域名里面没有配置该二级域名，那么可以直接配置成主域名。比如二级域名是weixin.test.com,那么JS接口安全域名可以配置成test.com   
我个人建议是填写一级域名，因为这么多的运营活动，你才三个坑。  
另外截止到现在为止，公众平台接口调用仅支持80端口。

#### 步骤二：引入JS文件
在需要调用JS接口的页面引入如下JS文件，（支持https）：http://res.wx.qq.com/open/js/jweixin-1.2.0.js
备注：支持使用 AMD/CMD 标准模块加载方法加载

#### 步骤三：通过config接口注入权限验证配置
所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用（同一个url仅需调用一次，对于变化url的SPA的web app可在每次url变化时进行调用,目前Android微信客户端不支持pushState的H5新特性，所以使用pushState来实现web app的页面会导致签名失败，此问题会在Android6.2中修复）。
```javascript
wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: '', // 必填，公众号的唯一标识
    timestamp: '', // 必填，生成签名的时间戳
    nonceStr: '', // 必填，生成签名的随机串
    signature: '',// 必填，签名
    jsApiList: [] // 必填，需要使用的JS接口列表，所有JS接口列表
});
```
等等，上面的这些啥玩意儿？
`appId`就是上面提到的，timestamp时间戳自己生成的，nonceStr一个随机字符串自己生成的，signature签名，这怎么来？
##### 签名算法
签名生成规则如下：  
参与签名的字段包括`noncestr`（随机字符串）, 有效的`jsapi_ticket`, `timestamp`（时间戳）, `url`（当前网页的URL，不包含#及其后面部分） 。  
对所有待签名参数按照字段名的ASCII 码从小到大排序（字典序）后，使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1。  
这里需要注意的是所有参数名均为小写字符。对string1作sha1加密，字段名和字段值都采用原始值，不进行URL 转义。  
即`signature=sha1(string1)`。 示例：
```javascript
 noncestr = 'Wm3WZYTPz0wzccnW';
 jsapi_ticket = 'sM4AOVdWfPE4DxkXGEs8VMCPGGVi4C3VM0P37wVUCFvkVAy_90u5h9nbSlYy3-Sl-HhTdfl2fzFy1AOcHKP7qg';
 timestamp = '1414587457';
 url = 'http://mp.weixin.qq.com?params=value';
``` 
步骤1.  
    对所有待签名参数按照字段名的ASCII 码从小到大排序（字典序）后，使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1：
    jsapi_ticket=sM4AOVdWfPE4DxkXGEs8VMCPGGVi4C3VM0P37wVUCFvkVAy_90u5h9nbSlYy3-Sl-HhTdfl2fzFy1AOcHKP7qg&noncestr=Wm3WZYTPz0wzccnW&timestamp=1414587457&url=http://mp.weixin.qq.com?params=value  
步骤2.  
    对string1进行sha1签名，得到signature：0f9de62fce790f9a083d5c99e95740ceb90c27ed  
注意事项  
   - 签名用的noncestr和timestamp必须与wx.config中的nonceStr和timestamp相同。  
   - 签名用的url必须是调用JS接口页面的完整URL。
   - 出于安全考虑，开发者必须在服务器端实现签名的逻辑。  
`jsapi_ticket`又是啥？
##### jsapi_ticket
`jsapi_ticket`是公众号用于调用微信JS接口的临时票据。  
正常情况下，`jsapi_ticket`的有效期为7200秒，通过`access_token`来获取。  
由于获取`jsapi_ticket`的`api`调用次数非常有限，频繁刷新`jsapi_ticket`会导致`api`调用受限，影响自身业务，开发者必须在自己的服务全局缓存`jsapi_ticket` 。  
说白了就是调用下微信的接口，`https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=ACCESS_TOKEN&type=jsapi`  ，`GET请求`，它会给你你需要的`jsapi_ticket`。
```javascript
//返回的json数据示例
{
    "errcode": 0,
    "errmsg": "ok",
    "ticket": "bxLdikRXVbTPdHSM05e5u5sUoXNKd8-41ZO3MhKoyN5OfkWITDGgnr2fwJ0m9E8NYzWKVZvdVtaUgWvsdshFKA",
    "expires_in": 7200
}
```
`access_token`又是啥？

##### access_token
access_token是公众号的全局唯一接口调用凭据，公众号调用各接口时都需使用access_token。  
开发者需要进行妥善保存。access_token的存储至少要保留512个字符空间。  
access_token的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的access_token失效。  

*公众平台的API调用所需的access_token的使用及生成方式说明： * 
- 建议公众号开发者使用中控服务器统一获取和刷新Access_token，其他业务逻辑服务器所使用的access_token均来自于该中控服务器，不应该各自去刷新，否则容易造成冲突，导致access_token覆盖而影响业务；
- 目前Access_token的有效期通过返回的expire_in来传达，目前是7200秒之内的值。中控服务器需要根据这个有效时间提前去刷新新access_token。在刷新过程中，中控服务器对外输出的依然是老access_token，此时公众平台后台会保证在刷新短时间内，新老access_token都可用，这保证了第三方业务的平滑过渡；
- Access_token的有效时间可能会在未来有调整，所以中控服务器不仅需要内部定时主动刷新，还需要提供被动刷新access_token的接口，这样便于业务服务器在API调用获知access_token已超时的情况下，可以触发access_token的刷新流程。

其实也是请求微信的一个接口：  
```javascript
/**
* https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET
* 请求方式 GET
* 参数 grant_type    获取access_token填写client_credential
* 参数 appid		    第三方用户唯一凭证
* 参数 secret		第三方用户唯一凭证密钥，即appsecret
*/
正常的返回json
{
    "access_token":"ACCESS_TOKEN",
    "expires_in":7200
}
失败的返回json
{
    "errcode":40013,
    "errmsg":"invalid appid"
}
/**
* -1	系统繁忙，此时请开发者稍候再试]
* 0	请求成功
* 40001	AppSecret错误或者AppSecret不属于这个公众号，请开发者确认AppSecret的正确性
* 40002	请确保grant_type字段值为client_credential
* 40164 调用接口的IP地址不在白名单中，请在接口IP白名单中进行设置
*/
```
好了，这样一步一步过来就能拿到调用jsSDK需要的必备参数了。完美，皆大欢喜！
#### 步骤四：通过ready接口处理成功验证
```javascript
wx.ready(function(){
    // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
});
```

#### 步骤五：通过error接口处理失败验证
```javascript
wx.error(function(res){
    // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名。
});
```

#### 接口调用说明
```javascript
所有接口通过wx对象(也可使用jWeixin对象)来调用，参数是一个对象，除了每个接口本身需要传的参数之外，还有以下通用参数：
 1. success：接口调用成功时执行的回调函数。
 2. fail：接口调用失败时执行的回调函数。
 3. complete：接口调用完成时执行的回调函数，无论成功或失败都会执行。
 4. cancel：用户点击取消时的回调函数，仅部分有用户取消操作的api才会用到。
 5. trigger: 监听Menu中的按钮点击时触发的方法，该方法仅支持Menu中的相关接口。
 备注：不要尝试在trigger中使用ajax异步请求修改本次分享的内容，因为客户端分享操作是一个同步操作，这时候使用ajax的回包会还没有返回。
 以上几个函数都带有一个参数，类型为对象，其中除了每个接口本身返回的数据之外，还有一个通用属性errMsg，其值格式如下：
 调用成功时："xxx:ok" ，其中xxx为调用的接口名
 用户取消时："xxx:cancel"，其中xxx为调用的接口名
 调用失败时：其值为具体错误信息
```
### 实施
这个题目，不知道该叫啥，哈哈。
之前不是说js安全域名不能填本地的ip嘛，那怎么办，我本地开发，我又是一个前端。我需要服务端的人支持，运维的人支持，服务端的人每次初始化的时候给我签名，时间戳啥啥啥的。  
但是我又想自己搞清楚，要不然一塌糊涂，做缓存，调接口，我node也可以呀。重要的是我怎么把我的内网地址能让外网访问。  
之前QQ浏览器自带了一个微信调试这样的一个功能，它其中有个功能就是服务端调试，把你的内网穿透，能映射让外网访问。这样就可以在测试公众号上填写js安全域名了。  
很可惜，我说的是之前。所以，百度啊百度，百度了一个叫花生壳的工具，有这样的一个功能。  

注册一个账号，并且去申请一个免费的壳域名，设置一下，映射到本地的你起的这个服务的网址上，这样就做到了内网穿透。具体的大家可以百度花生壳内网穿透的教程。  
做这个的时候还是挺心酸的，我用的是mac，花生壳还没有mac版的客户端，我又跑去找到封尘以久的联想，然后...  

我申请到一个`ashasmile.imwork.net`，然后在js安全域名填了`imwork.net`。

### 代码
上面的都是一些概念啊，流程啊，下面开始晒代码了啊。

#### 基本配置 /config/wechat.config.js
```javascript
module.exports = {
    grant_type: 'client_credential',
    appid: 'wx5f87160ac8f64d3d',
    secret: 'ad89c2d17bfeddce0905e6761b42a3d2',
    noncestr:'Wm3WZYTPz0wzccnW',
    accessTokenUrl:'https://api.weixin.qq.com/cgi-bin/token',
    ticketUrl:'https://api.weixin.qq.com/cgi-bin/ticket/getticket',
    cache_duration:1000*60*60*24
}
```

#### 生成签名算法 /sign/signature.js
```javascript
var request = require('request'),
    cache = require('memory-cache'),
    sha1 = require('sha1'),
    config = require('../config/wechat.config');

exports.sign = function (url,callback) {
    var noncestr = config.noncestr,
        timestamp = Math.floor(Date.now()/1000), //精确到秒
        jsapi_ticket;
    if(cache.get('ticket')){
        jsapi_ticket = cache.get('ticket');
        console.log('1' + 'jsapi_ticket=' + jsapi_ticket + '&noncestr=' + noncestr + '&timestamp=' + timestamp + '&url=' + url);
        callback({
            noncestr:noncestr,
            timestamp:timestamp,
            url:url,
            jsapi_ticket:jsapi_ticket,
            signature:sha1('jsapi_ticket=' + jsapi_ticket + '&noncestr=' + noncestr + '&timestamp=' + timestamp + '&url=' + url)
        });
    }else{
        //获取accessToken
        request(config.accessTokenUrl + '?grant_type=' + config.grant_type + '&appid=' + config.appid + '&secret=' + config.secret ,function(error, response, body){
            if (!error && response.statusCode == 200) {
                var tokenMap = JSON.parse(body);
                request(config.ticketUrl + '?access_token=' + tokenMap.access_token + '&type=jsapi', function(error, resp, json){
                    if (!error && response.statusCode == 200) {
                        var ticketMap = JSON.parse(json);
                        cache.put('ticket',ticketMap.ticket,config.cache_duration);  //加入缓存
                        console.log('jsapi_ticket=' + ticketMap.ticket + '&noncestr=' + noncestr + '&timestamp=' + timestamp + '&url=' + url);
                        callback({
                            noncestr:noncestr,
                            timestamp:timestamp,
                            url:url,
                            jsapi_ticket:ticketMap.ticket,
                            signature:sha1('jsapi_ticket=' + ticketMap.ticket + '&noncestr=' + noncestr + '&timestamp=' + timestamp + '&url=' + url)
                        });
                    }
                })
            }
        })
    }
}
```

#### 客户端js  /static/index.js
```javascript
document.getElementById('refresh').onclick = function(){location.reload();}

/**
 *  以下内容多摘自官方demo
 *
**/

//alert(appId + ' ; ' + timestamp + ' ; ' + nonceStr + ' ; ' + signature);
wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: appId, // 必填，公众号的唯一标识
    timestamp: timestamp, // 必填，生成签名的时间戳
    nonceStr: nonceStr, // 必填，生成签名的随机串
    signature: signature,// 必填，签名，见附录1
    jsApiList: ['checkJsApi',
        'onMenuShareTimeline',
        'onMenuShareAppMessage',
        'onMenuShareQQ',
        'onMenuShareWeibo',
        'hideMenuItems',
        'showMenuItems',
        'hideAllNonBaseMenuItem',
        'showAllNonBaseMenuItem',
        'translateVoice',
        'startRecord',
        'stopRecord',
        'onRecordEnd',
        'playVoice',
        'pauseVoice',
        'stopVoice',
        'uploadVoice',
        'downloadVoice',
        'chooseImage',
        'previewImage',
        'uploadImage',
        'downloadImage',
        'getNetworkType',
        'openLocation',
        'getLocation',
        'hideOptionMenu',
        'showOptionMenu',
        'closeWindow',
        'scanQRCode',
        'chooseWXPay',
        'openProductSpecificView',
        'addCard',
        'chooseCard',
        'openCard'] // 必填，需要使用的JS接口列表，
});

wx.ready(function(){
 // 1 判断当前版本是否支持指定 JS 接口，支持批量判断
  document.querySelector('#checkJsApi').onclick = function () {
    wx.checkJsApi({
      jsApiList: [
        'getNetworkType',
        'previewImage'
      ],
      success: function (res) {
        alert(JSON.stringify(res));
      }
    });
  };

   // 2. 分享接口
  // 2.1 监听“分享给朋友”，按钮点击、自定义分享内容及分享结果接口
  document.querySelector('#onMenuShareAppMessage').onclick = function () {
    wx.onMenuShareAppMessage({
      title: '互联网之子',
      desc: '在长大的过程中，我才慢慢发现，我身边的所有事，别人跟我说的所有事，那些所谓本来如此，注定如此的事，它们其实没有非得如此，事情是可以改变的。更重要的是，有些事既然错了，那就该做出改变。',
      link: location.href,
      imgUrl: 'http://demo.open.weixin.qq.com/jssdk/images/p2166127561.jpg',
      trigger: function (res) {
        // 不要尝试在trigger中使用ajax异步请求修改本次分享的内容，因为客户端分享操作是一个同步操作，这时候使用ajax的回包会还没有返回
        alert('用户点击发送给朋友');
      },
      success: function (res) {
        alert('已分享');
      },
      cancel: function (res) {
        alert('已取消');
      },
      fail: function (res) {
        alert(JSON.stringify(res));
      }

    });
    alert('已注册获取“发送给朋友”状态事件');
  };

    // 5 图片接口
  // 5.1 拍照、本地选图
  var images = {
    localId: [],
    serverId: []
  };
  document.querySelector('#chooseImage').onclick = function () {
    wx.chooseImage({
      success: function (res) {
        images.localId = res.localIds;
        alert('已选择 ' + res.localIds.length + ' 张图片');
      }
    });
  };
    // 5.2 图片预览
  document.querySelector('#previewImage').onclick = function () {
    wx.previewImage({
      current: 'http://img5.douban.com/view/photo/photo/public/p1353993776.jpg',
      urls: [
        'http://img3.douban.com/view/photo/photo/public/p2152117150.jpg',
        'http://img5.douban.com/view/photo/photo/public/p1353993776.jpg',
        'http://img3.douban.com/view/photo/photo/public/p2152134700.jpg'
      ]
    });
  };

    // 7.2 获取当前地理位置
  document.querySelector('#getLocation').onclick = function () {
    wx.getLocation({
      success: function (res) {
        alert(JSON.stringify(res));
      },
      cancel: function (res) {
        alert('用户拒绝授权获取地理位置');
      }
    });
  };

    // 9 微信原生接口
  // 9.1.1 扫描二维码并返回结果
  document.querySelector('#scanQRCode0').onclick = function () {
    wx.scanQRCode();
  };

});

wx.error(function(res){
    JSON.stringify(res)
});
```

#### node起服务  /server.js
```javascript
var express = require('express'),
jade = require('jade');

var app = express();

app.set('view engine', 'jade'); // 设置模板引擎
app.set('views', './views');  // 设置模板相对路径(相对当前目录)

app.locals.basedir = './'

var port = 4567 ;  //BAE 百度应用引擎默认端口号
 //中间件定义
app.use(express.logger());
app.use(express.compress());
app.use(express.bodyParser());
app.use(express.methodOverride());
app.use(express.cookieParser());

//静态资源

app.use(express.static('./static'));

//启动服务
app.listen(port, function() {
    console.log('服务启动成功！请访问 http://localhost:' + port);
});


//启动路由分发
require('./router/index').init(app);
```

#### package.json
```javascript
{
  "name": "wechat-sdk-demo",
  "version": "0.0.0",
  "description": "a demo with wechat js-sdk based on nodejs",
  "main": "server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/liaobin312716/wechat-sdk-demo.git"
  },
  "keywords": [
    "wechat",
    "js-sdk",
    "node"
  ],
  "author": "liaobin",
  "license": "MIT",
  "dependencies": {
    "express": "~3.17.5",
    "jade": "^1.11.0",
    "memory-cache": "^0.1.4",
    "request": "^2.58.0",
    "sha1": "^1.1.1"
  },
  "bugs": {
    "url": "https://github.com/liaobin312716/wechat-sdk-demo/issues"
  },
  "homepage": "https://github.com/liaobin312716/wechat-sdk-demo"
}
```

#### router /route/index.js
```javascript
exports.init = function (app) {
    var wechat_cfg = require('../config/wechat.cfg');
    var http = require('http');
    var cache = require('memory-cache');
    var sha1 = require('sha1'); //签名算法
    // var url = require('url');
    var signature = require('../sign/signature');
    
    app.get('/',function(req,res){
        //var url = req.protocol + '://' + req.host + req.path;
        var url = req.protocol + '://' + req.host + req.originalUrl; //获取当前url
        console.log(url);
        signature.sign(url,function(signatureMap){
            signatureMap.appId = wechat_cfg.appid;
            res.render('index',signatureMap);
        });
    });
};
```

#### 前端静态资源css /static/index.css
```javascript
body{-webkit-tap-highlight-color: rgba(0,0,0,0);}
h3{text-align: center;color: red;}
.panel{text-align: center;}
.panel h5{
    text-align: left;
    color: #888;
    margin: 0;
}
.panel button{
    width: 90%;
    height: 40px;
    line-height: 40px;
    margin: 5px auto;
    border: none;
    border-radius: 5px;
    background: #e2e2e2;
}
```

#### 前端静态资源 html /views/index.jade
```javascript
doctype html
html(lang='en')
    head
        meta(http-equiv='Content-Type',content='text/html; charset=utf-8')
        meta(name='viewport',content='width=device-width,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no,minimal-ui')
        meta(name='apple-mobile-web-app-status-bar-style',content='black')
        title 微信SDK demo
        link(rel='stylesheet',type="text/css",href='index.css')
    body
        h3 微信SDK demo
        div.panel
            h5 1.判断是否支持特定JS-API
            button#checkJsApi checkJsApi
            br
            br
            h5 2.分享接口
            button#onMenuShareAppMessage 发送给朋友
            br
            br
            h5 3.图像接口
            button#chooseImage 选择图片
            br
            button#previewImage 预览图片
            br
            br
            h5 4.位置接口
            button#getLocation 获取位置
            br
            br
            h5 5.微信扫一扫
            br
            button#scanQRCode0 扫一扫
            br
            button#refresh 刷新当前页面
        script(src='http://res.wx.qq.com/open/js/jweixin-1.0.0.js')
        script(type='text/javascript').
            var signature = '#{signature}';
            var nonceStr = '#{noncestr}';]
            var timestamp = '#{timestamp}';
            var appId = '#{appId}';
            var jsapi_ticket = '#{jsapi_ticket}';
        script(src='index.js')
```

### 总结
比较重要的是生成签名的算法，微信官方也给出了各个语言的版本[示例](http://demo.open.weixin.qq.com/jssdk/sample.zip)，`java`、`php`、`node `等。 

参考资料：  
> [微信公众平台技术文档](https://mp.weixin.qq.com/wiki) 
> [基于nodejs 的微信 JS-SDK 简单应用](http://www.cnblogs.com/albin-gecker/p/4628355.html)
> [微信公众平台接口调试工具](https://mp.weixin.qq.com/debug/)
> [jsSDKdemo](http://203.195.235.76/jssdk/)