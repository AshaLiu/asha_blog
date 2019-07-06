---
title: h5的history对象-无刷新页面跳转
date: 2017-03-11 11:23:26
tags: html5 history 无刷新跳转
---
### 前言
怎么说呢，公司有个官网，当然希望按照seo规则，能够愉快得被搜索引擎抓住。  
其中有一个我要买车的筛选页，主要功能是根据用户的需求选择车的品牌，车系，价位等筛选出车辆，大致这样的一个展示页。  
搜索引擎蜘蛛并不是浏览器，获取的只是从后端服务器传回来的数据。  
首先要想被搜索引擎抓取，那就不能按照之前常用的ajax模式(你得前后端分离，页面交给前端人员去渲染)。所以我们采取的是第一次由服务端去渲染，然后之后的在页面上的操作筛选交给我们前端的童鞋。 

### url的模式
为了响应SEO，于是着我们和服务端约定我们的url将会是这样的`/car/大众/polo/红色`，(当然实际上不会这样，这只是一个大致的比喻，我们实际试是这样的`http://www.cheok.com/car/Dazhong-DZJKJKC/ci_0_cp_1`)  
这个时候，可能会有童鞋发出疑问，为什么我不能`car?brand="大众"&color="红色"`？ 搜索引擎把？的这种页面叫动态页面。它们比较倾向静态页。所以很多网站都会做伪静态(就是动态伪装成静态)。  
问题来了，`/car/大众/polo/红色`意味着要像服务器请求三次，对于用户来说，会出现三次的页面刷新重新渲染，体验非常不好，在网络差的情况，很可能会失去用户。  
怎么办呢，前端童鞋上场了~ 我h5有一个history对象，可以既让你url变了，又可以不去服务端重新渲染，我ajax自己局部刷新。
所以，扯了这么多，终于引入正题。

### history对象
#### 定义
`history`是挂在`window`下面的，打开控制台输入`history`，可以看到：
```javascript
   History {length: 2, state: null, scrollRestoration: "auto"}
       length:2 //当前窗口的浏览记录的条数
       scrollRestoration:"auto" //用来设置是否回到之前滚动位置的，A->B，B一开始的滚动位置就看这个属性了，此属性可以是自动的（auto）或者手动的（manual）。
       state:null //当前页面存储的一个对象，比如你的筛选条件
       __proto__:History
           back:back()
           constructor:History()
           forward:forward()
           go:go()
           length:(...)
           get length:()
           pushState:pushState()
           replaceState:replaceState()
           scrollRestoration:(...)
           get scrollRestoration:()
           set scrollRestoration:()
           state:(...)
           get state:()
           Symbol(Symbol.toStringTag):"History"
           __proto__:Object
```
#### history.pushState(state, title, url)
功能：往历史记录里面添加记录
```javascript
   if (!!(window.history && history.pushState)){
     // 支持History API
     history.pushState('', '', '');
   } else {
     // 不支持
     location.href = newUrl;
   } 
```
> state：一个与指定网址相关的状态对象，popstate事件触发时，该对象会传入回调函数。如果不需要这个对象，此处可以填null。  
  title：新页面的标题，但是所有浏览器目前都忽略这个值，因此这里可以填null。  
  url：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个网址。

假如当前的url是`www.cheok.com/car/cp_1`，然后你这样：
```javascript
   var oParams = {
       modeID: 1
   };
   history.pushState(oParams, '我要买车', '/cp_2');
```
这个时候，观察下地址栏，会发现你的url悄无声息得变成了`www.cheok.com/car/cp_2`，其实这个页面是根本不存在的，神奇吧。然后尝试着点击后退的按钮看看效果，你会发现什么也没变。  

#### history.replaceState(state, title, url)
功能：和pushState的用法一样，只不过这是修改当前的记录
```javascript
    //假如你当前的url为 http://www.cheok.com/car.html
    history.pushState({page: 1}, 'title 1', '?page=1');
    history.pushState({page: 2}, 'title 2', '?page=2');
    history.replaceState({page: 3}, 'title 3', '?page=3');
    
    history.back()
    // url显示为http://www.cheok.com/car.html?page=1
    
    history.back()
    // url显示为http://www.cheok.com/car.html
    
    history.go(2)
    // url显示为http://www.cheok.com/car.html?page=3
```
#### history.state
功能：返回当前页面的state对象
```javascript
    history.pushState({page: 1}, 'title 1', '?page=1');
    
    history.state;
    // { page: 1 }
```
### popstate事件
每当同一个文档的浏览历史（即history对象）出现变化时，就会触发popstate事件。

需要注意的是，仅仅调用pushState方法或replaceState方法 ，并不会触发该事件，只有用户点击浏览器倒退按钮和前进按钮，或者使用JavaScript调用back、forward、go方法时才会触发。另外，该事件只针对同一个文档，如果浏览历史的切换，导致加载不同的文档，该事件也不会触发。

使用的时候，可以为popstate事件指定回调函数。这个回调函数的参数是一个event事件对象，它的state属性指向pushState和replaceState方法为当前URL所提供的状态对象（即这两个方法的第一个参数）。
```javascript
    window.onpopstate = function (event) {
      console.log('location: ' + document.location);
      console.log('state: ' + JSON.stringify(event.state));
    };
    
    // 或者
    
    window.addEventListener('popstate', function(event) {
      console.log('location: ' + document.location);
      console.log('state: ' + JSON.stringify(event.state));
    });
    //event.state就是history.state对象
```
注意，页面第一次加载的时候，在load事件发生后，Chrome和Safari浏览器（Webkit核心）会触发popstate事件，而Firefox和IE浏览器不会。

### 总结
来一段高清无码的代码
```javascript
    var DEFALUT_PARAMS = {
            province: "",	//省份缩写
            city:"", 		//城市缩写
            brand: "",		//品牌缩写
            series: "",		//车型缩写
            minPrice: "",	//最小价格
            maxPrice: "",	//最大价格
            minCarAge: "",	//最小车龄
            maxCarAge: "",	//最大车龄
            minKmNum: "",	//最小公里数
            maxKmNum: "",	//最大公里数
            emissionStandard: "",	//排放标准 国Ⅱ：0；国Ⅲ：1；国Ⅳ：2；国Ⅴ：3
            color: "",	//车身颜色
            baseType: "",	//特色类型（0：all；1:一手好车；2：准新车；3：最新上架；4：里程少）
            sortType: "",	//(默认排序0，价格1，车龄2，里程3)；
            sortRule: "",	//(低到高：0;高到低：1)
            currentPage: 1,	//当前页数
            pageSize: "",	//每页显示数量
            queryKey: "",	//关键字
            carCategory: ""	//车型("轿车", 0),("SUV", 1),("MPV", 2), ("商用车", 3);
        },
        //随便举个例子，存的是当前的筛选条件
        oParam = {
            minPrice: "10",	//最小价格
            maxPrice: "15",	//最大价格
            minCarAge: "1",	//最小车龄
            maxCarAge: "7"	//最大车龄
        },
        paramModel = $.extend(false, DEFALUT_PARAMS, oParam)
    ;

    /**
     * 占位符替换工厂。
     * 
     * @method
     * @param {String} sContent 含占位符的字符串。
     * 	当要被替换的内容中含未知替换数据，则会保留当前点位符。
     * @param {Object} oData 要替换的点位符数据，依据对象的键名与点位符一一对应，功能类似 KISSY.substitute。
     * @return {String} 返回替换后的字符串。
     */
    function substitute (sContent, oData) {
        if (!oData) {
            return sContent;
        }

        for (var p in oData) {
            sContent = sContent.replace(new RegExp("\\{" + p + "\\}", "g"), oData[p]);
        }

        return sContent;
    }
    
    /**
     * 获取车辆列表url地址
     * @param  {Object}  oParamModel  参数对象
     * @return {String}  Url    对应的地址
     */
    function getCartListUrl (oParamModel){
        var sUrl = '/{province}-{city}/car/{brand}-{series}/',
            sParamStr
        ;
        
        sUrl = substitute(sUrl, $.extend(oParamModel, {
            province: oParamModel.province == 0 ? "" : oParamModel.province,
            city: oParamModel.city == 0 ? "" : oParamModel.city
        }))
            .replace(/\{[a-zA-Z\d]+\}/ig, "")   //过滤空参数
            .replace(/-\//ig, "/")  //过滤城市和车系为空的情况
            .replace(/c-\//i, "/") //品牌和车系都为空，过滤这一层
            .replace(/\/\//ig, "/")     //过滤空的省市或者品牌车系
            // .replace(/\{[a-zA-Z\d]+\}/ig, "")   //过滤空参数
            // .replace(/_0\//ig, "_/")            //过滤0，针对省市
        ;
        
        sParamStr = getCartListParamsUrl(oParamModel);            
        sUrl += sParamStr;

        return sUrl;
    }
    
    /**
     * 获取车辆列表参数url区段
     * @param  {Object}  oParamModel  参数对象
     * @return {String}
     */
    function getCartListParamsUrl (oParamModel){
        var sParamStr = "_b_{baseType}" 
            + "_st_{sortType}" 
            + "_sr_{sortRule}" 
            + "_ip_{minPrice}" 
            + "_xp_{maxPrice}" 
            + "_ia_{minCarAge}" 
            + "_xa_{maxCarAge}" 
            + "_in_{minKmNum}" 
            + "_xn_{maxKmNum}" 
            + "_es_{emissionStandard}" 
            + "_ci_{color}" 
            + "_cp_{currentPage}" 
            + "_pg_{pageSize}" 
            + "_k_{queryKey}" 
            + "_cm_{carCategory}"
        ;
        
        sParamStr = substitute(sParamStr, $.extend(oParamModel, {
            currentPage: oParamModel.currentPage ? oParamModel.currentPage : 1
        }))
            .replace(/_[a-z]{1,2}_(?=_|$)/ig, "")   //过滤空的
            .replace(/_[a-z]{1,2}_\{[a-zA-Z\d]+\}/ig, "")   //过滤没有被替换的
            // .replace(/_cp_0/ig, "")
            .replace(/_st_0/ig, "")
            // .replace(/_sr_0/ig, "")
        ;

        if(sParamStr){
            sParamStr = sParamStr.substr(1, sParamStr.length - 1);
        }
       
        return sParamStr;
    }
    
    /**
     * 生成跳转地址
     * @param oParamModel 搜索参数
     */
    function fnCreateUrl (oParamModel){
        return getCartListUrl(oParamModel);
    }
    
    /**
     * 读取车辆列表数据
     * @return {void}
     */
    function getData(oParamModel, isInitPager){
        var sApiUrl = "/{province}/{city}/{brand}/{series}/";
        sApiUrl = substitute(sApiUrl, $.extend({}, oParamModel, {
            province: oParamModel.province || 0,
            city: oParamModel.city || 0,
            brand: oParamModel.brand || 0,
            series: oParamModel.series || 0
        }));
        sApiUrl += getCartListParamsUrl(oParamModel);
        
        //之后ajax请求渲染页面就可以了
        $ajax().then(function(){});
    }
    
    /**
     * 改变搜索条件
     * @param  {Boolean} isInitPager 是否重新从第一页开始，默认是
     * @return {void}
     */
    function doSearch(isInitPager){
        if(isInitPager){
            paramModel.currentPage = 1;
        }

        var sUrl = fnCreateUrl(paramModel);

        if (!!(window.history && history.pushState)){
            history.pushState(paramModel, document.title, sUrl);

            getData(paramModel, isInitPager);
            return false;
        } else {
            location.href = sUrl;
        }
    }
    
    //然后每次更改筛选条件的时候，更新paramModel的值
    paramModel.color = 'red';
    doSearch(true);
    
    //一些规则具体要和服务端约定好
```
大致的流程就是，先跟服务端约定好，各种筛选条件的规则，然后`getCartListParamsUrl()`生成当前的筛选条件对应的url去`pushState()`，至于刷新的时候，如何获取先前的筛选条件，服务端会在页面里面搞一个隐藏域给我们放数据。
感觉讲得有点混乱，但是代码绝对是清晰的，哈哈。