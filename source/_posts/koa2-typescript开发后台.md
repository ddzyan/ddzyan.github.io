---
title: koa2-typescript开发后台
date: 2019-08-20 00:07:01
categories:
  - node
tags:
  - node
  - typescript
---

---

### 本文目的

- 学习 typescript 在后端中的应用。typescript 是 javascript 的超集，他提供了静态语言强类型的支持，可以在项目编译阶段提早发现类型的错误，避免在项目在运行阶段才出现，提高代码开发效率。
- 结合当前流行的 koa 框架，快速上手 typescript 在后端开发中的使用，使之有真正能输出的项目。
- 代码仓库：https://github.com/ddzyan/koa2-ts

### 如何创建 koa/typescript 项目

本文首选不采用脚手架的方式创建项目，目的在于了解项目的基础结构。

#### 创建基础文件

```shell
mkdir src dist

npm init

npm i koa --save

npm i --save-dev @types/koa tslint typescript
```

<!--more-->

tsconfig.json

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es2015",
    "noImplicitAny": true,
    "moduleResolution": "node",
    "sourceMap": true,
    "outDir": "dist",
    "baseUrl": ".",
    "paths": {
      "*": ["node_modules/*", "src/types/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

./src/server.ts

```ts
debugger;

import * as Koa from 'koa';
const app = new Koa();

app.use(ctx => {
  ctx.body = 'hello word';
});

app.listen(3000, () => {
  console.log('Server is running at http://localhost:3000');
  console.log('Press CTRL-C to stop \n');
});

module.exports = app;
```

修改 package.json

```json
"scripts": {
    "tsc": "tsc",
    "tsc:w": "tsc -w"
 }
```

.vscode 目录

- launch.json 添加 debug 配置

```json
{
  // 使用 IntelliSense 了解相关属性。
  // 悬停以查看现有属性的描述。
  // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Current JS File",
      "type": "node",
      "request": "launch",
      "program": "${workspaceRoot}/dist/server.js",
      "args": [],
      "cwd": "${workspaceRoot}",
      "protocol": "inspector"
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "program": "${workspaceFolder}/dist/server.js",
      "preLaunchTask": "npm: build",
      "outFiles": ["${workspaceFolder}/dist/**/*.js"]
    }
  ]
}
```

- tasks.json

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

点击 vscode 终端 ---> 运行任务 --->ts watch：typescript.json，之后每次编辑 ts 文件将自动转编译为 js 文件

#### 启动

Launch Program ，点击 F5 将直接运行转换后的 js 文件，调试转换后的 js 代码

#### 直接调试

ts-node 调试 ts 文件时，不会显式生成 js。假如你不想编译为 js 后 在调试，可以考虑这种方式。

##### 安装依赖

```shell
npm i ts-node --save-dev
```

添加 launch.json

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

在 server.ts 头部添加 debugger，上面代码已经存在

插入断点，选择指定的 debug 模式，F5 运行
