---
title: 将扁平数组json转成有父子关系的一棵树
date: 2018-12-29 10:34:32
tags: 编程题
---
```
题目

入参：
let params = [
  {id: 1, text: 't11', parentId: 0},
  {id: 2, text: 't11', parentId: 0},
  {id: 3, text: 't11', parentId: 1},
  {id: 4, text: 't11', parentId: 1},
  {id: 5, text: 't11', parentId: 3},
  {id: 6, text: 't11', parentId: 2}
];

出参：
let x = [{
  'id': 1,
  'text': 't11',
  'parentId': 0,
  'child': [{
    'id': 3,
    'text': 't11',
    'parentId': 1,
    'child': [{
      'id': 5,
      'text': 't11',
      'parentId': 3
    }]
  },
  {
    'id': 4,
    'text': 't11',
    'parentId': 1
  }]
},
{
  'id': 2,
  'text': 't11',
  'parentId': 0,
  'child': [{
    'id': 6,
    'text': 't11',
    'parentId': 2
  }]
}];
```

```javaScript
let params = [
  {id: 1, text: 't11', parentId: 0},
  {id: 2, text: 't11', parentId: 0},
  {id: 3, text: 't11', parentId: 1},
  {id: 4, text: 't11', parentId: 1},
  {id: 5, text: 't11', parentId: 3},
  {id: 6, text: 't11', parentId: 2}
];
json2Tree(params);

function json2Tree (params) {
  let result = params.filter(parent => {
    params.filter(child => {
      if (parent.id === child.parentId) {
        parent.child = parent.child ? parent.child : [];
        parent.child.push(child);
        child.isChild = true;
      }
    });
    if (parent.isChild) {
      delete parent.isChild;
      return false;
    } else {
      return true;
    }
  });
  console.log(JSON.stringify(result));
  return result;
}

```

#### 总结
复杂度是O(n^2)

两个知识点：

1. filter不改变原有数组
2. 利用js对象的引用关系

我发现我老是长时间不用这个api就记不住细节用法，哪些是要修改原来对象的，哪些是不修改返回新对象的
