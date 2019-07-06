---
title: angularJS中ng-repeat的track by
date: 2017-03-07 16:15:18
tags: angularJS ng-repeat trackby
---
### 引言
新年博客更新的速度突然间就这么慢了下来，本来也慢。其实我是积攒了一堆要总结的。
说到`ng-repeat`,网上一堆说要使用`track by`来优化性能，一直在使用，自己也没有实际求做过验证。

### ng-repeat
ng-repeat指令遍历一个JavaScript数组，当数组中有重复元素的时候，AngularJS会报错
```javascript
<html>
  <head>  
    <script src="angular-1.2.2/angular.js"></script>  
    <script>    
         function rootController($scope,$rootScope,$injector) {  
            $scope.dataList = [1,2,1];  
         }  
    </script>  
  </head>  
      <body ng-app ng-controller="rootController">  
            <div ng-repeat="data in dataList">  
                {{data}}      
            </div>  
      </body>  
</html>  
```
但是当你的`$scope.dataList = [{id:1},{id:1}];`是不会报错的
很简单，angular认为对于数字或者字符串等基本数据类型来说，它的id就是它自身的值。因此数组中是不允许存在两个相同的数字的，而如果是javascript对象类型数据，那么就算内容一摸一样，ng-repeat也不会认为这是相同的对象。
为了规避这个错误，需要定义自己的`track by`表达式。

* 业务上自己生成唯一的id
`item in items track by item.id`

* 或者直接拿循环的索引变量$index来用
`item in items track by $index`

### ng-repeat的DOM刷新
当scope中的dataList数据发生了变化，界面会自动刷新。
* 删除之前所有存在的dom，然后重新生成dom
* 重用之前的dom元素，仅仅更新dom元素的属性

在没有使用`track by`的情况下，angular默认采用的是第一种方式，它会往数组里每个元素加一个`$$hashKey`的属性。
这个`key`是由 Angular 内部的`nextUid()`方法生成，类似数据库自增，但是是使用字符串。所以当数组变化的时候，每次生成的$$hashKey都不一样，导致dom会删了再重建。
但是当你采用`track by`的时候，angular会根据的by的东西，判断新的数据和旧的数据是不是一致，一致的话就直接改变dom属性，不一致的话就会删了重建。

### 总结
我觉得`track by item.id`和`$index`来说，还是`$index`好一点。
依靠服务端的话，万一数据是重复的，比如简单的基础类型数据，前端会报错，如果`[1,2,3]`变成`[2,3,1]`,那么dom还是会删除再重建。

### 参考资料
[Improving ng-repeat Performance with “track by”](http://www.codelord.net/2014/04/15/improving-ng-repeat-performance-with-track-by/)
