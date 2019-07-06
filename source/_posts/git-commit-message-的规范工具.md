---
title: git commit-message 的规范工具
date: 2019-06-12 14:40:13
tags: git 工具
---
## 背景
某一天下午，温度和湿度都很适宜，写代码也写的很来劲，心情也很好,兴致勃勃的。突然钉钉群响了，点开一看，隔壁同事发了一个关于提交代码的 `commit message` 范本。

```javaScript
// commit 信息规范

feat: 新功能 (feature)
fix: 修补bug
docs: 补充文档
ref: 重构代码 (refactor)
chore: 一些配置的变动
style: ui，样式调整
test: 测试内容
```
我一看，很新颖啊，说的很有道理，值得推广，于是立马在下午的提交代码的过程中，用上了。
但是种类太多了，又拿出了便利贴，把所有类别都抄在了上面，方便实时查看，又觉得确实是种规范，立马又推给了之前公司的同事。（可见我多么认可）

## 问题
一开始还可以，我很认真的严格按照规范执行，但是我写去写去就觉得 `fix: 修复ios12微信键盘失焦页面不下来的问题`，按照小学的作业来看，这个明明是个病句啊。 `fix` 就是 `修复` 啊，我为什么要多此一举。
渐渐地就开始懒了，便利贴也脱落了，然后我的画风就变成了这样混搭风
![commit-message](/images/article/commitMessage/commit-message.png)
并且我也觉得完全没毛病。写成什么样的形式，最终目的是可读性好。我已经做到了。其他同事做 `code review` 的时候也是没有问题的。

## 转变
周会了，主题是规范，其中就扯到了这个 `commit message` 的规范、形式、和执行。
当场我就google了，查阅相关资料。发现原来这个规范最初是 `Angular 规范`，大致长这样
![angular-commit-message](/images/article/commitMessage/angular-commit-message.png)

它是有好处的。
1. 提供更多的历史信息，方便快速浏览。

2. 可以过滤某些commit（比如文档改动），便于快速查找信息。
`git log <last release> HEAD --grep feature`

3. 可以直接从commit生成 `Change log`

## 执行规范
鉴于我之前慢慢的就不规范，并且没有约束力，但是确实用上了这个规范是有意义的。
于是想着能不能有像 `EsLint.js` 这样的强制工具，强制约束我。（果然人还是犯贱的...）

## commit message 的格式
`commit message` 包括三个部分： `Header` 、 `Body` 、 `Footer`。
```
<type>(<scope>): <subject> // 必填
// 空一行
<body> // 选填
// 空一行
<footer> // 选填
```
不管是哪一个部分，任何一行都不得超过72个字符（或100个字符）。这是为了避免自动换行影响美观。

### Header
Header部分只有一行，包括三个字段：type（必需）、scope（可选）和subject（必需）。

#### Type

> feat：新功能（feature）

> fix：修补bug

> docs：文档（documentation）

> style： 格式（不影响代码运行的变动）

> refactor：重构（即不是新增功能，也不是修改bug的代码变动）

> test：增加测试

> chore：构建过程或辅助工具的变动


tip:
如果 `type` 为 `feat` 和 `fix`，则该 `commit` 将肯定出现在 `Change log` 之中。

其他情况（`docs、chore、style、refactor、test`）由你决定，要不要放入 `Change log`，建议是不要。

#### scope
用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

#### subject
> 以动词开头，使用第一人称现在时，比如change，而不是changed或changes
> 第一个字母小写
> 结尾不加句号（.）

### Body
对本次 commit 的详细描述，可以分成多行。

### Footer

1. 关闭 `Issue`
如果当前 `commit` 针对某个`issue`，那么可以在 `Footer` 部分关闭这个 `issue` 。
`Closes #123, #245, #992`

2. 不兼容变动
如果当前代码与上一个版本不兼容，则 `Footer` 部分以 `BREAKING CHANGE` 开头，后面是对变动的描述、以及变动理由和迁移方法。
```
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

## commitlint
`commitlint` 是一个工具，会检查我们的 `commit message` 是否满足规范。

通常上，我们这个规范是 `type(scope?): subject  #scope is optional`，
这个type，常见类型根据 ` commitlint-config-conventional (based on the the Angular convention)`

```
build：主要目的是修改项目构建系统(例如 glup，webpack，rollup 的配置等)的提交
ci：主要目的是修改项目继续集成流程(例如 Travis，Jenkins，GitLab CI，Circle等)的提交
docs：文档更新
feat：新增功能
merge：分支合并 Merge branch ? of ?
fix：bug 修复
perf：性能, 体验优化
refactor：重构代码(既没有新增功能，也没有修复 bug)
style：不影响程序逻辑的代码修改(修改空白字符，格式缩进，补全缺失的分号等，没有改变代码逻辑)
test：新增测试用例或是更新现有测试
revert：回滚某个更早之前的提交
chore：不属于以上类型的其他类型
```
所以第一步，下载两个npm包
```
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```
第二步，根目录新建一个js文件 `commitlint.config.js` ，
然后内容是 `module.exports = {extends: ['@commitlint/config-conventional']}`
```
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

当然除了这个规范 `@commitlint/config-conventional` 还有其他的规范

> @commitlint/config-angular

> @commitlint/config-conventional

> @commitlint/config-lerna-scopes

> @commitlint/config-patternplate

> conventional-changelog-lint-config-atom

> conventional-changelog-lint-config-canonical

当然还可以自定义配置，但是不介绍了。

## git hooks
上面介绍的是一个检查 `commit message` 的工具，要想达到 `git commit -m 'xxxx'` 的时候触发这个检查动作，
就要利用另外一个工具 `git hooks`。

钩子(hooks)是一些在 `"$GIT-DIR/hooks"` 目录的脚本, 在被特定的事件(certain points)触发后被调用。当 `"git init"` 命令被调用后, 一些非常有用的示例钩子文件(hooks)被拷到新仓库的hooks目录中;
但是在默认情况下这些钩子(hooks)是不生效的。 把这些钩子文件(hooks)的".sample"文件名后缀去掉就可以使它们生效了。

也就是在我们的git仓库下会有一个.git配置文件夹，它里面包含git操作相关的一系列 `预处理／后处理` 脚本。如 `pre-commit` 、 `post-push` 之类的。
分为客户端钩子的和服务器端钩子。

### 如何安装钩子

钩子都被存储在 `Git` 目录下的 `hooks` 子目录中。 也即绝大部分项目中的 `.git/hooks` 。 当我们用 `git init` 初始化一个新版本库时，`Git` 默认会在这个目录中放置一些示例脚本。这些脚本除了本身可以被调用外，它们还透露了被触发时所传入的参数。
所有的示例都是 `shell` 脚本，其中一些还混杂了 `Perl` 代码，不过，任何正确命名的可执行脚本都可以正常使用 —— 你可以用 `Ruby` 或 `Python`，或其它语言编写它们。 这些示例的名字都是以 `.sample` 结尾，如果你想启用它们，得先移除这个后缀。

把一个正确命名且可执行的文件放入 `Git` 目录下的 `hooks` 子目录中，即可激活该钩子脚本。 这样一来，它就能被 `Git` 调用。

### 客户端钩子
提交工作流钩子、电子邮件工作流钩子和其它钩子

#### 提交工作流钩子
1. pre-commit

钩子在键入提交信息前运行。
它用于检查即将提交的快照，例如，检查是否有所遗漏，确保测试运行，以及核查代码。
可以利用该钩子，来检查代码风格是否一致（运行类似 lint 的程序）、尾随空白字符是否存在（自带的钩子就是这么做的），或新方法的文档是否适当。

2. prepare-commit-msg

钩子在启动提交信息编辑器之前，默认信息被创建之后运行。

它允许你编辑提交者所看到的默认信息。 该钩子接收一些选项：存有当前提交信息的文件的路径、提交类型和修补提交的提交的 SHA-1 校验。

3. commit-msg

钩子接收一个参数，此参数即上文提到的，存有当前提交信息的临时文件的路径。

可以用来在提交通过前验证项目状态或提交信息。
我们使用该钩子来核对提交信息是否遵循指定的模板。

4. post-commit

钩子在整个提交过程完成后运行。

它不接收任何参数，但你可以很容易地通过运行 git log -1 HEAD 来获得最后一次的提交信息。 该钩子一般用于通知之类的事情。

### 服务端钩子
放在 `Git Server` 上的 `Hooks` 脚本，作为管理员，可以利用这些服务端的脚本来强制确保项目的任何规范。
这些运行在服务端的脚本，会在push命令发生的前后执行。pre系列的脚本可以在任何时候返回非零值来终止某次push，并向push方返回一个错误。
可以比如监听 `git push` 来做消息通知，对接钉钉机器人，发送push的详细信息。


## husky(哈士奇)
`husky` 是一个对 `git hooks` 的封装，专门用来处理这种git提交时的自动化处理的插件。不过它只支持客户端钩子。
```
npm install --save-dev husky
```

原理：

实现的机制大致是，在你install该插件时，它会自动往git hooks里埋入很多相应的钩子脚本(git hook能识别的)，
这些钩子函数会去读区package.json中的配置信息，当用户做某些git操作触发相应钩子时会自然去调用配置好的任务，从而实现我们的自动化需求，
包括代码检查跟单元测试验证以及更多的自动化任务。


```
// package.json

"husky": {
    "hooks": {
        "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
}
// 这样的话，每次commit的时候会执行 `commitlint -E HUSKY_GIT_PARAMS` 命令，使用 `commitlint` 的能力
```
小小提一嘴，还有个 `git钩子` 插件，叫 `pre-commit`。

## 步骤汇总

1. 在自己的项目的 `package.json` 里面的 `devDependencies` 中新增插件：
```javascript
npm install --save-dev @commitlint/config-conventional @commitlint/cli husky
```

2. 在项目根目录新建js文件 `commitlint.config.js` 

```javascript
module.exports = {
  extends: ['@commitlint/config-conventional']
}
```

3. 在自己的项目的 `package.json` 里面新增配置

```javascript
{
  "dependencies": {...},
  "devDependencies": {...},
	"husky": {
		"hooks": {
			"commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
		}
	}
}
```

## 参考资料
1. [commitlint.js](https://commitlint.js.org/#/guides-local-setup)
2. [Git 钩子](https://www.git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)
3. [husky](https://github.com/typicode/husky)
4. [Git Hooks简介](https://www.cnblogs.com/StitchSun/articles/4712287.html)
