---
title: git commit规范
tags:
  - git commit
categories: git
abbrlink: 76bd
---

> git 每次提交都需要写commit message，否则就不允许提交，一般来说，commit message应该清晰明了，说明本次提交的目的，具体做了什么操作，但是在日常开发中，大家的commit message千奇百怪，中英文混合使用，fix bug等各种笼统的message司空见怪，规范git commit message很重要

比较流行的规范整理如下。

## commit message 格式

建议每行不超过100个字符

`<type>(<scope>): <subject>`

```shell
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```



* type(必须)

  用于说明git commit的类别，只允许使用下面的标识。

  feat: 新功能（feature）

  fix: 修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG

  docs: 文档（document）

  style: 格式（不影响代码运行的变动, 比如删除多余的空行）

  refactor: 重构（即不是新增功能，也不是修改bug的代码运动）

  perf: 优化相关，比如提升性能、体验

  test: 增加测试

  chore: 构建过程或者辅助工具的变动

  revert: 回滚到上一个版本，需要在body中写上从哪一个版本revert的

  merge: 代码合并

  sync: 同步主线或者分布的bug

* scope（可选）

  scope用于说明影响的范围，比如数据层、控制层、视图层等等，视项目的不同而不同

  如果修改影响了不止一个socope，你可以使用*代替

* subject（必须）

  subject是commit目的的简短描述，不超过50字符。

  建议使用中文。

  结尾不加句号或者其他标点符号

```shell
# 举例
docs(api): 接口说明完善
```

```shell
feat($browser): onUrlChange event (popstate/hashchange/polling)

Added new event to $browser:
- forward popstate event if available
- forward hashchange event if popstate not available
- do polling when neither popstate nor hashchange available

Breaks $browser.onHashChange, which was removed (use onUrlChange instead)


```

```shell
fix($compile): couple of unit tests for IE9

Older IEs serialize html uppercased, but IE9 does not...
Would be better to expect case insensitive, unfortunately jasmine does
not allow to user regexps for throw expectations.

Closes #392
Breaks foo.bar api, foo.baz should be used instead

```

```shell
feat($compile): simplify isolate scope bindings

Changed the isolate scope binding options to:
  - @attr - attribute binding (including interpolation)
  - =model - by-directional model binding
  - &expr - expression execution binding

This change simplifies the terminology as well as
number of choices available to the developer. It
also supports local name aliasing from the parent.

BREAKING CHANGE: isolate scope bindings definition has changed and
the inject option for the directive controller injection was removed.

To migrate the code follow the example below:

Before:

scope: {
  myAttr: 'attribute',
  myBind: 'bind',
  myExpression: 'expression',
  myEval: 'evaluate',
  myAccessor: 'accessor'
}

After:

scope: {
  myAttr: '@',
  myBind: '@',
  myExpression: '&',
  // myEval - usually not useful, but in cases where the expression is assignable, you can use '='
  myAccessor: '=' // in directive's template change myAccessor() to myAccessor
}

The removed `inject` wasn't generaly useful for directives so there should be no code using it.

```



### 使用git commit 规范的好处

1. 便于追溯提交历史进行追溯
2. 一旦约束了commit message，就不能再一股脑的把各种各样的改动都放在一个git commit里面
3. 便于自动化输出change log(CHANGELOG.md)

[1] https://developer.aliyun.com/article/770277?accounttraceid=d4154093542c440aa51196b76d1ccbe3eyfz

[2] https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#