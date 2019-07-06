---
title: ES6的Promise
date: 2017-03-17 13:50:18
tags: es6 promise
---
### 前言
刚跨入前端的行业的时候，那时候啥也不懂，项目也很小，完全是纯jquery走天下，多个ajax也是一个一个的内嵌的套起来，那时候就觉得这样好难看有点别扭，又不知道咋办。  
后来换了公司之后，没事的时候翻翻老大搭的框架，看到了`promise`。
### 定义
中文翻译过来就是承诺，它是一个对象，里面存储了未来（异步）的才能知道的情况的信息：假如明天下雨了，我就宅家里，假如明天是天晴，我就出去骑自行车。
ES6里面，定义它是一个构造函数：
```javascript
var promise = new Promise(function(resolve, reject) {
    // ... some code

    if (/* 异步操作成功 */){
        resolve(value);
    } else {
        reject(error);
    }
});

promise.then(function(value) {
    // success
}, function(error) {
    // failure
});
```
其常见的用法(配合ajax)
```javascript
var carService = {
    getList: function(o) {
        var URL = "/car/getList",
            promise = new Promise(function(resolve, reject) {
                $.ajax({
                    url: URL,
                    method: 'get',
                    data: o,
                    success: function(data) {
                        resolve(data);
                    },
                    error: function(xhr, statusText) {
                        reject(statusText);
                    }
                });
            })
        ;
        return promise;
    }
};

carService.getList({id: 1001}).then(function(data) {
    //成功ajax之后拿到数据之后的操作
}, function(){
    //ajax不成功该怎么办
});
```

### Promise.prototype.then()
then方法的第一个参数是Resolved状态的回调函数，第二个参数（可选）是Rejected状态的回调函数。

then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法，而且前一个then方法会将返回结果作为参数传递给后面的then方法。
```javascript
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function funcA(comments) {
  console.log("Resolved: ", comments);
}, function funcB(err){
  console.log("Rejected: ", err);
});
```

### Promise.prototype.catch()
`Promise.prototype.catch`方法是`.then(null, rejection)`的别名，用于指定发生错误时的回调函数。
另外，then方法指定的回调函数，如果运行中抛出错误，也会被catch方法捕获。
```javascript
getJSON('/posts.json').then(function(posts) {
  // 处理成功事件
  throw new Error('test');//这里抛出错误，能被后面的catch()接收到
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
```

### Promise.all()
Promise.all方法用于将多个Promise实例，包装成一个新的Promise实例。
```javascript
var p = Promise.all([p1, p2, p3]);
```
上面代码中，Promise.all方法接受一个数组作为参数，p1、p2、p3都是Promise对象的实例，如果不是，就会先调用下面讲到的Promise.resolve方法，将参数转为Promise实例，再进一步处理。（Promise.all方法的参数可以不是数组，但必须具有Iterator接口，且返回的每个成员都是Promise实例。）

p的状态由p1、p2、p3决定，分成两种情况。

* 只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

* 只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

下面是一个具体的例子：
```javascript
// 生成一个Promise对象的数组
var promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON("/post/" + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```
另一个例子：
```javascript
const databasePromise = connectDatabase();

const booksPromise = databasePromise
  .then(findAllBooks);

const userPromise = databasePromise
  .then(getCurrentUser);

Promise.all([
  booksPromise,
  userPromise
])
.then(([books, user]) => pickTopRecommentations(books, user));
```
上面代码中，`booksPromise`和`userPromise`是两个异步操作，只有等到它们的结果都返回了，才会触发`pickTopRecommentations`这个回调函数。

### Promise.race()
和`Promise.all()`类似，只不过`all()`是`&`的意思，`race()`是`||`的意思。
```javascript
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);
p.then(response => console.log(response));
p.catch(error => console.log(error));
```
如果5秒之内`fetch`方法无法返回结果，变量`p`的状态就会变为`rejected`，从而触发`catch`方法指定的回调函数。

### Promise.resolve()
作用是将现有对象转为`Promise`对象，`Promise.resolve`方法就起到这个作用。
```javascript
var jsPromise = Promise.resolve($.ajax('/whatever.json'));
```
上面代码将jQuery生成的deferred对象，转为一个新的Promise对象。

`Promise.resolve`等价于下面的写法。
```javascript
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```
`Promise.resolve方`法的参数分成四种情况。
* 如果参数是Promise实例，那么`Promise.resolve`将不做任何修改、原封不动地返回这个实例。
* `thenable`对象指的是具有then方法的对象，比如下面这个对象。
    ```javascript
     let thenable = {
       then: function(resolve, reject) {
         resolve(42);
       }
     };
    ```
    `Promise.resolve`方法会将这个对象转为Promise对象，然后就立即执行`thenable`对象的then方法。
    ```javascript
    let thenable = {
      then: function(resolve, reject) {
        resolve(42);
      }
    };
    
    let p1 = Promise.resolve(thenable);
    p1.then(function(value) {
      console.log(value);  // 42
    });
    ```
    上面代码中，`thenable`对象的then方法执行后，对象p1的状态就变为`resolved`，从而立即执行最后那个`then`方法指定的回调函数，输出`42`。
* 如果参数是一个原始值，或者是一个不具有then方法的对象，则`Promise.resolve`方法返回一个新的Promise对象，状态为`Resolved`。
    ```javascript
    var p = Promise.resolve('Hello');
    
    p.then(function (s){
      console.log(s)
    });
    // Hello
    ```
    上面代码生成一个新的Promise对象的实例p。由于字符串`Hello`不属于异步操作（判断方法是它不是具有then方法的对象），返回Promise实例的状态从一生成就是`Resolved`，所以回调函数会立即执行。`Promise.resolve`方法的参数，会同时传给回调函数。
* `Promise.resolve`方法允许调用时不带参数，直接返回一个Resolved状态的Promise对象。
  
  所以，如果希望得到一个Promise对象，比较方便的方法就是直接调用Promise.resolve方法。
  上面代码的变量p就是一个Promise对象。
  
  需要注意的是，立即resolve的Promise对象，是在本轮“事件循环”（event loop）的结束时，而不是在下一轮“事件循环”的开始时。
  ```javascript
    setTimeout(function () {
      console.log('three');
    }, 0);
    
    Promise.resolve().then(function () {
      console.log('two');
    });
    
    console.log('one');
    
    // one
    // two
    // three
  ```
  上面代码中，`setTimeout(fn, 0)`在下一轮“事件循环”开始时执行，`Promise.resolve()`在本轮“事件循环”结束时执行，`console.log(’one‘)`则是立即执行，因此最先输出。
  
### Promise.reject()
`Promise.reject(reason)`方法也会返回一个新的`Promise`实例，该实例的状态为`rejected`。
```javascript
var p = Promise.reject('出错了');
// 等同于
var p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
```
注意，`Promise.reject()`方法的参数，会原封不动地作为`reject`的理由，变成后续方法的参数。这一点与`Promise.resolve`方法不一致。
```javascript
const thenable = {
  then(resolve, reject) {
    reject('出错了');
  }
};

Promise.reject(thenable)
.catch(e => {
  console.log(e === thenable)
})
// true
```

### Promise.done()（封装的）
`Promise`对象的回调链，不管以`then`方法或`catch`方法结尾，要是最后一个方法抛出错误，都有可能无法捕捉到（因为`Promise`内部的错误不会冒泡到全局）。因此，我们可以提供一个`done`方法，总是处于回调链的尾端，保证抛出任何可能出现的错误。
```javascript
asyncFunc()
  .then(f1)
  .catch(r1)
  .then(f2)
  .done();

//实现方式
Promise.prototype.done = function (onFulfilled, onRejected) {
  this.then(onFulfilled, onRejected)
    .catch(function (reason) {
      // 抛出一个全局错误
      setTimeout(() => { throw reason }, 0);
    });
};
```

### Promise.finally()（封装的）
`finally`方法用于指定不管`Promise`对象最后状态如何，都会执行的操作。它与`done`方法的最大区别，它接受一个普通的回调函数作为参数，该函数不管怎样都必须执行。
```javascript
server.listen(0)
  .then(function () {
    // run test
  })
  .finally(server.stop);
//实现方式
Promise.prototype.finally = function (callback) {
  let P = this.constructor;
  return this.then(
    value  => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => { throw reason })
  );
};
```

### 总结
大致就是这样的，其实基本都看的是[阮一峰的ES6](http://es6.ruanyifeng.com/#docs/promise#Promise-reject)