---
title: vue-cli2创建自己的模板
date: 2019-06-13 15:47:18
tags:
---

<a name="eb18b1e8"></a>
## 指令

<a name="dbd0d432"></a>
### 官方模板指令

[vue-cli2.x 文档](https://github.com/vuejs/vue-cli/tree/v2#vue-cli)<br />`vue init <templateName> <myProject>`<br />官方提供几个模板：

> 1. **browserify** - A full-featured Browserify + vueify setup with hot-reload, linting & unit testing.
> 1. **browserify-simple** - A simple Browserify + vueify setup for quick prototyping.
> 1. **pwa** - PWA template for vue-cli based on the webpack template
> 1. simple - The simplest possible Vue setup in a single HTML file
> 1. **webpack** - A full-featured Webpack + vue-loader setup with hot reload, linting, testing & css extraction.
> 1. **webpack-simple** - A simple Webpack + vue-loader setup for quick prototyping.


<a name="ca85194f"></a>
### 自定义模板指令

<a name="1296ebdc"></a>
#### mpvue模板指令

`vue init mpvue/mpvue-quickstart my-project`

```
GitHub - github:owner/name or simply owner/name
vue init mpvue/mpvue-quickstart my-project

GitLab - gitlab:owner/name
vue init git@git.souche-inc.com:digital-marketing/xxxxname#my-branch my-project

Bitbucket - bitbucket:owner/name
```

[git@git.souche-inc.com]():digital-marketing/businesses/tangeche-miniprogram-alipay.git
<a name="d41d8cd9"></a>
####
<a name="35cdd9f3"></a>
### 本地模板指令

`vue init ~/fs/path/to-custom-template my-project`

<a name="ee2fd5ec"></a>
## 编写自定义的模板

<a name="6363a843"></a>
### 一定要有 `template` 文件夹，放模板文件

<a name="dceca03b"></a>
### 可以有模板的元数据文件，可以是 `meta.js` 或者 `meta.json`

里面可以包含以下字段：

- `prompts <Object>`： 做命令行交互配置
- `filters <Object>`： 根据命令行交互结果过滤将要渲染的项目文件
- `metalsmith <Object>`：配置 `Metalsmith` 插件，文件会像 `gulp.js` 中的 `pipe` 一样依次经过各个插件的处理
- `completeMessage <String>`：将模板渲染为项目后输出一些提示信息，取值为字符
- `complete <Function>`：与 `completeMessage` 功能相同，二选其一，取值为函数，函数最后要返回输出的字符串，这里也可以配置让其自动安装项目依赖
- `helpers <Object>`：自定义的 `Handlebars` 辅助函数

<a name="prompts"></a>
### prompts

```
{
    prompts: {
        name: {
            type: "input",
            message: "项目名"
        },
        author: {
            type: "input",
            message: "作者"
        },
        email: {
            type: "input",
            message: "邮箱",
            validate: function(answer){
                if(/@/g.test(answer)){
                    return true;
                }
                return "邮箱应该含有@符号";
            }
        },
        vuex: {
            type: "confirm",
            message: "你的项目中需要安装vuex吗",
            default: true
        }
    }
}
```

- type(类型)：input(输入，默认类型), confirm（y/n）, list(列表), rawlist（带下标的列表）, expand（下标是字母的列表）, checkbox（复选框）, password（密码）, editor（编辑大篇文字）。
- massage(提示信息——问题)：字符串或者函数，如果是函数，返回值需要时字符串；默认是name值，如上面的name,author,email,vuex。
- default（默认值）：这个需要跟据类型和选项来给出对应的默认值，这个就不多说了，大家都明白。
- choices（选项）：list,rawlist,expand,checkbox类型需要给出相应的choice供用户选择，数组类型，数组的每个元素可以是字符串也可以是对象。对象可以有name（选之前的提示信息）、value（选的结果）、short（选之后显示的结果）。
- validate（有效性验证）：函数类型，参数是用户输入的结果，验证通过返回true，否则返回一个验证失败的提示。
- filter（过滤）：函数类型，参数是用户输入的结果，filter的处理结果会改变用户输入的结果，这个与后天提到的transformer不同。
- transformer（转换）：函数类型，参数是用户的输入结果，transformer的处理结果是用来展示出来的，不会改变用户输入的最终结果，仅仅是显示的不同。
- when（控制条件）：函数类型，参数是用户的输入结果，当前面的某个结果符合条件时才会询问此问题。
- pageSize（每页显示的数量）：当有choice选项的时候可以给用这个来控制每页显示的数量。

> 当所有问题问完之后，template 目录下的所有文件将会用 Handlebars 进行渲染. 用户输入的数据会作为模板渲染时的使用数据。


<a name="filters"></a>
### filters

filters 字段是一个包含文件过滤规则的对象, 键用于定义符合 minimatch glob pattern 规则的过滤器, 键值是 prompts 中用户的输入值或者表达式，代码如下：

```json
filters: {
    "store/*": "vuex"
}
// 在上面的询问中，如果你vuex选项选择了no，你的store文件夹以及其下面的子文件将被删除，如果选的yes，store文件夹以及其下面的子文件将被保留。
```

<a name="helpers"></a>
### helpers

在hleplers中，你可以注册handlebars函数，注册后，在template里面的文件中可以使用你注册的辅助函数。vue自带的有if_eq(判断两个参数相等的)和unless_eq这连个辅助函数。

```json
helpers: {
    between(v, v1, v2, options) {
      if (v > v1 && v < v2) {
        return options.fn(this)
      }
      return options.inverse(this)
    }
}
```

<a name="8621bda1"></a>
### complete或completeMessage

complete为一个函数，completeMessage为一个字符串。<br />如果同时写了这两个，会调用complete函数，即complete的优先级高。

```json
"completeMessage": "请按以下步骤启动，耐心等待:\n\n  {{^inPlace}}cd {{destDirName}}\n  {{/inPlace}}npm i\n  npm run dev 或者 npm start"
```

