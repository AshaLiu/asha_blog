---
title: 虚拟DOM
date: 2017-11-26 18:53:56
tags: js 虚拟DOM
---
### 闲话
自从分配到事业线的前线后，博客这边就荒废了。人总是这样，隔一段时间打一次鸡血。

### 关于DOM操作历程。
凡是个前端开发者应该心里都清楚，操作DOM的代码是昂贵的。  
1. 最初的时候是直接用js直接操作DOM，这样的缺点是复杂且要考虑兼容性
2. 后来出现了`jquery`，`jquery`都封装好了，我们只需要调用一下就OK了，完全不用考虑兼容性。但是呢，缺点是会频繁操作DOM，因为你得自己写增删改查(视图更新逻辑)啊，每个动作都要DOM更新。
3. 然后就出现了`MVVM`的概念，它实现了数据双向绑定，封装了一系列增删改查的方法，又一次解放了程序员的双手，不用考虑兼容性，不用自己增删改查(视图更新逻辑)。缺点也是有的，性能堪忧啊。
4. 最后就出现了`虚拟DOM`。

### 概念
DOM树大家都知道，虚拟DOM字面意思就是模拟的DOM，用什么模拟的呢，当然是javascript啦，为什么呢，因为一棵js模拟的树的改变比直接操作DOM高效啊。  

### webkit引擎的处理流程（引用）
浏览器加载一个html，呈现在我们眼前，都会发生什么呢？ 
 ![流程图](/images/article/vritualDom/bowser-render-chart.jpg)
创建DOM tree –> 创建Style Rules -> 构建Render tree -> 布局Layout –> 绘制Painting
第一步，一旦浏览器接收到一个HTML文件，渲染引擎（render engine）就开始解析它，并根据HTML元素（elements）一一对应地生成DOM 节点（nodes），组成一棵DOM树。

第二步：同时，浏览器也会解析来自外部CSS文件和元素上的inline样式。通常浏览器会为这些样式信息，连同包含样式信息的DOM树上的节点，再创建另外一个树，一般被称作渲染树（render tree） 

第三步：将上面的DOM树和样式表，关联起来，构建一颗Render树。这一过程又称为Attachment。每个DOM节点都有attach方法，接受样式信息，返回一个render对象（又名renderer）。这些render对象最终会被构建成一颗Render树。  

第四步：
   - WebKit内核的浏览器上，处理一个节点的样式的过程称为attachment。DOM树上的每个节点都有一个attach方法，它接收计算好的样式信息，返回一个render对象（又名renderer）
    
   - Attachment的过程是同步的，新节点插入DOM树时，会调用新节点的attach方法。
    
   - 构建渲染树时，由于包含了这些render对象，每个render对象都需要计算视觉属性（visual properties）；这个过程通过计算每个元素的样式属性来完成。
    
第五步：有了Render树后，浏览器开始布局，会为每个Render树上的节点确定一个在显示屏上出现的精确坐标值。 

第六步：浏览器将会通过遍历渲染树，调用每个节点的paint方法来绘制这些render对象。paint方法根据浏览器平台，使用不同的UI后端API（agnostic UI backend API）。 通过绘制，最终将在屏幕上展示内容。  

当你用传统的原生api或jQuery去操作DOM时，浏览器会从构建DOM树开始从头到尾执行一遍流程。比如当你在一次操作时，需要更新10个DOM节点，理想状态是一次性构建完DOM树，再执行后续操作。但浏览器没这么智能，收到第一个更新DOM请求后，并不知道后续还有9次更新操作，因此会马上执行流程，最终执行10次流程。显然例如计算DOM节点的坐标值等都是白白浪费性能，可能这次计算完，紧接着的下一个DOM更新请求，这个节点的坐标值就变了，前面的一次计算是无用功。  

虚拟DOM就是为了解决这个浏览器性能问题而被设计出来的。例如前面的例子，假如一次操作中有10次更新DOM的动作，虚拟DOM不会立即操作DOM，而是将这10次更新的`diff`内容保存到本地的一个js对象中，最终将这个js对象一次性attach到DOM树上，通知浏览器去执行绘制工作，这样可以避免大量的无谓的计算量。  

### 实现虚拟DOM
```html
<div id="real-container">
    <p>Real DOM</p>
    <div>cannot update</div>
    <ul>
        <li className="item">Item 1</li>
        <li className="item">Item 2</li>
        <li className="item">Item 3</li>
    </ul>
</div>
```
上面的DOM转换成js对象来就是这样的
```javascript
const tree = Element('div', { id: 'virtual-container' }, [
    Element('p', {}, ['Virtual DOM']),
    Element('div', {}, ['before update']),
    Element('ul', {}, [
        Element('li', { class: 'item' }, ['Item 1']),
        Element('li', { class: 'item' }, ['Item 2']),
        Element('li', { class: 'item' }, ['Item 3']),
    ])
]);

const root = tree.render();
document.getElementById('virtualDom').appendChild(root);
```
这么做之后，页面的更新可以先全部反映在js对象上，等更新完后，再将最终的js对象映射成真实的DOM，交由浏览器去绘制。

```javascript
/**
 * 通过 JS 对象来表示 DOM
 * @param {*} tagName 标签名
 * @param {*} props 属性
 * @param {*} children 子节点 
 */
* 
/**
* 第一个参数是节点名（如div） 
* 第二个参数是节点的属性（如class） 
* 第三个参数是子节点（如ul的li）
* 除了这三个参数会被保存在对象上外，还保存了key和count。
*/
function Element(tagName, props, children) {
    if (!(this instanceof Element)) {
        return new Element(tagName, props, children);
    }

    this.tagName = tagName;
    this.props = props || {};
    this.children = children || [];
    this.key = props ? props.key : undefined;

    let count = 0;
    this.children.forEach((child) => {
        if (child instanceof Element) {
            count += child.count;
        }
        count ++;
    });
    this.count = count;
}
```
```javascript
/**
* 将js对象生成真正的DOM
*/
Element.prototype.render = function() {
    const el = document.createElement(this.tagName);
    const props = this.props;

    for (const propName in props) {
        setAttr(el, propName, props[propName]);
    }

    this.children.forEach((child) => {
        const childEl = (child instanceof Element) ? child.render() : document.createTextNode(child);
        el.appendChild(childEl);
    });

    return el;
};
```
来感受下现在的DOM图
![流程图](/images/article/vritualDom/dom1.jpeg)

### Virtual DOM算法
- 用 JavaScript 对象结构表示 DOM 树的结构；然后用这个树构建一个真正的 DOM 树，插到文档当中
- 当状态变更的时候，重新构造一棵新的对象树。然后用新的树和旧的树进行比较，记录两棵树差异
- 把2所记录的差异应用到步骤1所构建的真正的DOM树上，视图就更新了

### Diff算法
我们已经完成了创建虚拟DOM并将其映射成真实DOM的工作，这样所有的更新都可以先反映到虚拟DOM上，如何反映呢？需要明确一下Diff算法。

两棵树如果完全比较时间复杂度是O(n^3)，但参照《深入浅出React和Redux》一书中的介绍，React的Diff算法的时间复杂度是O(n)。要实现这么低的时间复杂度，意味着只能平层地比较两棵树的节点，放弃了深度遍历。这样做，似乎牺牲了一定的精确性来换取速度，但考虑到现实中前端页面通常也不会跨层级移动DOM元素，所以这样做是最优的。

我们新创建一棵树，用于和之前的树进行比较
```javascript
const newTree = Element('div', { id: 'virtual-container' }, [
    Element('h3', {}, ['Virtual DOM']),                     // REPLACE
    Element('div', {}, ['after update']),                   // TEXT
    Element('ul', { class: 'marginLeft10' }, [              // PROPS
        Element('li', { class: 'item' }, ['Item 1']),
        // Element('li', { class: 'item' }, ['Item 2']),    // REORDER remove
        Element('li', { class: 'item' }, ['Item 3']),
    ])
]);
```

只考虑平层地Diff的话，就简单多了，只需要考虑以下4种情况
- 第一种**节点类型变了**。例如下图中的P变成了h3。我们将这个过程称之为`REPLACE`。直接将旧节点卸载（componentWillUnmount）并装载新节点（componentWillMount）就行了。
![节点变了](/images/article/vritualDom/diff1.jpeg)  
旧节点包括下面的子节点都将被卸载，如果新节点和旧节点仅仅是类型不同，但下面的所有子节点都一样时，这样做显得效率不高。但为了避免O(n^3)的时间复杂度，这样做是值得的。这也提醒了React开发者，应该避免无谓的节点类型的变化，例如运行时将div变成p就没什么太大意义。

- 第二种是**节点类型一样，仅仅属性或属性值变了**。我们将这个过程称之为`PROPS`。此时不会触发节点的卸载（componentWillUnmount）和装载（componentWillMount）动作。而是执行节点更新（shouldComponentUpdate到componentDidUpdate的一系列方法）。  
```javascript
renderA: <ul>
renderB: <ul class: 'marginLeft10'>
=> [addAttribute class "marginLeft10"]
```
    我们来看下diff算法
```javascript
function diffProps(oldNode, newNode) {
    const oldProps = oldNode.props;
    const newProps = newNode.props;

    let key;
    const propsPatches = {};
    let isSame = true;

    // find out different props
    for (key in oldProps) {
        if (newProps[key] !== oldProps[key]) {
            isSame = false;
            propsPatches[key] = newProps[key];
        }
    }

    // find out new props
    for (key in newProps) {
        if (!oldProps.hasOwnProperty(key)) {
            isSame = false;
            propsPatches[key] = newProps[key];
        }
    }

    return isSame ? null : propsPatches;
}
```
- 第三种**文本变了**，文本对也是一个Text Node，也比较简单，直接修改文字内容就行了，我们将这个过程称之为`TEXT`。

- 第四种是**移动，增加，删除子节点**，我们将这个过程称之为`REORDER`。  
![diff2](/images/article/vritualDom/diff2.jpeg)  
在中间插入一个节点，程序员写代码很简单：$(B).after(F)。但如何高效地插入呢？简单粗暴的做法是：卸载C，装载F，卸载D，装载C，卸载E，装载D，装载E。如下图：
 
![diff3](/images/article/vritualDom/diff3.jpeg)  
我们写JSX代码时，如果没有给数组或枚举类型定义一个key，就会看到下面这样的warning。React提醒我们，没有key的话，涉及到移动，增加，删除子节点的操作时，就会用上面那种简单粗暴的做法来更新。虽然程序运行不会有错，但效率太低，因此React会给我们一个warning。
  
![diff5](/images/article/vritualDom/diff5.jpeg)  

如果我们在JSX里为数组或枚举型元素增加上key后，React就能根据key，直接找到具体的位置进行操作，效率比较高。如下图：
  
![diff4](/images/article/vritualDom/diff4.jpeg)  
  
常见的最小编辑距离问题，可以用Levenshtein Distance算法来实现，时间复杂度是O(M*N)，但通常我们只要一些简单的移动就能满足需要，降低点精确性，将时间复杂度降低到O(max(M, N)即可。具体可参照采用深度剖析：如何实现一个 Virtual DOM 算法里的一个算法一文。或自行阅读例子中的源代码
最终Diff出来的结果如下：
```javascript
{
    1: [ {type: REPLACE, node: Element} ],
    4: [ {type: TEXT, content: "after update"} ],
    5: [ {type: PROPS, props: {class: "marginLeft10"}}, {type: REORDER, moves: [{index: 2, type: 0}]} ],
    6: [ {type: REORDER, moves: [{index: 2, type: 0}]} ],
    8: [ {type: REORDER, moves: [{index: 2, type: 0}]} ],
    9: [ {type: TEXT, content: "Item 3"} ],
}
```
深度遍历DOM将Diff的内容更新进去：
```javascript
function dfsWalk(node, walker, patches) {
    const currentPatches = patches[walker.index];

    const len = node.childNodes ? node.childNodes.length : 0;
    for (let i = 0; i < len; i++) {
        walker.index++;
        dfsWalk(node.childNodes[i], walker, patches);
    }

    if (currentPatches) {
        applyPatches(node, currentPatches);
    }
}
```
具体更新的代码如下，其实就是根据Diff信息调用源生API操作DOM：
```javascript
function applyPatches(node, currentPatches) {
    currentPatches.forEach((currentPatch) => {
        switch (currentPatch.type) {
            case REPLACE: {
                const newNode = (typeof currentPatch.node === 'string')
                    ? document.createTextNode(currentPatch.node)
                    : currentPatch.node.render();
                node.parentNode.replaceChild(newNode, node);
                break;
            }
            case REORDER:
                reorderChildren(node, currentPatch.moves);
                break;
            case PROPS:
                setProps(node, currentPatch.props);
                break;
            case TEXT:
                if (node.textContent) {
                    node.textContent = currentPatch.content;
                } else {
                    // ie
                    node.nodeValue = currentPatch.content;
                }
                break;
            default:
                throw new Error(`Unknown patch type ${currentPatch.type}`);
        }
    });
}
```
### 总结
没啥好总结的，因为都是道听途说看人家总结的精华的。所以写的过程也很痛苦，因为不知不觉的就在抄。 
文中说的这些个例子是react的虚拟DOM的概念， 下面拉了两篇文章是vue2引入的虚拟DOM。

### 参考
[虚拟DOM介绍](http://www.jianshu.com/p/616999666920)
[解析vue2.0的diff算法](https://github.com/aooy/blog/issues/2)
[深入 Vue2.x 的虚拟 DOM diff 原理](https://www.qcloud.com/community/article/648055)
    
