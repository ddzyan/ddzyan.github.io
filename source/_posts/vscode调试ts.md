---
title: vscode调试ts
date: 2019-04-22 00:54:12
categories: typescript
tags:
  - typescript
---

参考文章：

- https://segmentfault.com/a/1190000011935122
- https://medium.com/@rossbulat/typescript-introduction-with-nodejs-c160c4362746

## 编译后调试

### 安装依赖

```bash
npm install typescript --save-dev
```

### 配置任务

#### tsconfig.json

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "esModuleInterop": true,
    "target": "es6",
    "noImplicitAny": true,
    "moduleResolution": "node",
    "sourceMap": true,
    "outDir": "dist",
    "baseUrl": ".",
    "paths": {
      "*": ["node_modules/*", "src/types/*"]
    }
  },
  "include": ["src/**/*"]
}
```

<!--more-->

#### 参数解释

- "module": "commonjs"： 定义输出模块类型。commonjs 是 Node.js 中使用的标准模块格式。
- "esModuleInterop"：true：允许使用备用模块导入语法; 例如你可能会使用语法：import bar from 'foobar';
- "target": "es6"：输出 Javascript 语言规范。NodeJS 支持 ES6，因此我们将其设置为 ES6。
- "noImplicitAny": true：将此设置为 true 会在使用默认的 any 类型时抛出错误。
- "moduleResolution": "node"：Typescript 将模拟 Node 的模块解析策略。
- "sourceMap": true：Typescript 源码将与 Javascript 一起输出。 以便我们在进行断点调试时将运行的 Javascript 代码映射到 Typescript 上。
- "outDir": "dist"：编译生成的 .js 文件输出路径。通常使用 dist/ 目录。
- include：可以配置一组文件路径来指定编译哪些文件，这里我们添加了 src/ 下的所有文件。

#### package.json

```json
"tsc": "tsc",
"tsc:w": "tsc -w"
```

#### .vscode/tasks.json

```json
{
  // 有关 tasks.json 格式的文档，请参见
  // https://go.microsoft.com/fwlink/?LinkId=733558
  "version": "2.0.0",
  "tasks": [
    {
      "type": "typescript",
      "tsconfig": "tsconfig.json",
      "option": "watch",
      "problemMatcher": ["$tsc-watch"]
    }
  ]
}
```

点击 vscode 终端 ---> 运行任务 --->ts 监视：typescript.json，之后每次编辑 ts 文件将自动转编译为 js 文件

#### .vscode/launch.json

添加 debug 配置

```json
{
  "name": "Current JS File",
  "type": "node",
  "request": "launch",
  "program": "${workspaceRoot}/dist/index.js",
  "args": [],
  "cwd": "${workspaceRoot}",
  "protocol": "inspector"
}
```

- program :选择要执行的文件

记住在 F5 debug 之前，选择当前配置的 debug 模式，并且在对应文件的 ts 文件中插入断点

## 直接调试

ts-node 调试 ts 文件时，不会显式生成 js。假如你不想编译为 js 后 在调试，可以考虑这种方式。

### 安装依赖

```bash
npm i typescript --save-dev
npm i ts-node --save-dev
```

### 配置任务

#### .vscode/launch.json

```json
{
  "name": "Current TS File",
  "type": "node",
  "request": "launch",
  "program": "${workspaceRoot}/node_modules/ts-node/dist/_bin.js",
  "args": ["${relativeFile}"],
  "cwd": "${workspaceRoot}",
  "protocol": "inspector"
}
```

### 调试

在要调试的文件头部添加 debugger

```ts
debugger;

const message: string = 'hello word';
const abc: string = message;
const aaa: string = abc;
console.log(message);
```

插入断点，选择指定的 debug 模式，F5 运行
