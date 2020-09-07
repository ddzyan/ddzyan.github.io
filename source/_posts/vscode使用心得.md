---
title: vscode使用心得
date: 2019-06-06 00:53:42
categories:
  - nodejs
tags:
  - nodejs
---

最近看了一些 vscode 官方对编写 js 代码的介绍文章，在此记录一些心得

## 参考资料

- https://code.visualstudio.com/docs/languages/javascript
- https://code.visualstudio.com/docs/languages/jsconfig

## 内容

### jsconfig.json

目的解决 vsCode 编写 js 项目语法不提示，参数类型无法检查和引用的包方法不提示问题

目录中存在 jsconfig.json 文件表示该目录是 JavaScript 项目的根目录。jsconfig.json 文件指定根文件和 JavaScript 语言服务提供的功能选项。

推荐 vsCode 编译器下的 js 项目都创建在项目根目录下创建 jsconfig.json。

#### 作用

实现 js 语法提示，代码类型检查，部分引用包的方法提示等

#### jsconfig.json 常用配置

```json
{
  "compilerOptions": {
    "target": "es6",
    "checkJs": true
  },
  "exclude": ["node_modules"]
}
```

<!--more-->

- target：默认的语法提示库
- checkJs：启用 js 文件类型检查
- exclude：不包含的文件

在单个文件开头设置 // @ts-nocheck ，则不进行类型检查

```js
// @ts-nocheck
let easy = 'abc';
easy = 123; // no error
```

### 方法注释

```js
/**
 * 计算两个数字之和
 * @param {number} a 参数A
 * @param {number} b 参数B
 * @returns {number} 返回两数之和
 */
function sumAB(a, b) {
  return a + b;
}

sumAB(1, 2);
```

### vsCode 快捷键

- 在语法提示时，按 Tab 键插入最佳匹配
- F9 在当前行插入断点
- F2 方法改名

### vsCode

#### "editor.renderIndentGuides"

- true：控制编辑器呈现缩进参考线。

#### "editor.wordSeparators"

- "./\\()\"':,.;<>~!@#\$%^&\*|+=[]{}`~?"：双击选中词语（包含下划线、中横线等分割的词语）。

#### "javascript.updateImportsOnFileMove.enabled"

- "prompt" - 默认值。询问是否应为每个文件移动更新路径。
- "always" - 始终自动更新路径。
- "never" - 不要自动更新路径，也不要提示。

#### "javascript.format.insertSpaceBeforeFunctionParenthesis"

- false：js 代码保存时不在函数括号前添加一个空格

#### "javascript.referencesCodeLens.enabled"

- true：显示类的方法，属性和导出对象的内联引用计数，单击引用计数以快速浏览引用列表
  ![image](https://code.visualstudio.com/assets/docs/languages/javascript/references-codelens.png)
  ![image](https://code.visualstudio.com/assets/docs/languages/javascript/references-codelens-peek.png)
