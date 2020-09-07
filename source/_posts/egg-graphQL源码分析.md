---
title: egg-graphQL源码分析
date: 2020-07-14 18:06:44
categories:
  - nodejs
tag:
  - egg
  - nodejs
  - graphQL
---

### 简介

egg-graphQL 是 egg 的 graphQL 插件，规范化 GraphQL API 实现，自动创建 schema ，挂载 koa middleware 处理 GraphQL 请求，并且提供创建 GraphQL 开发者工具

### 源码分析

#### 项目依赖模块分析

```json
{
  "dependencies": {
    "apollo-server-koa": "2.0.4",
    "apollo-server-module-graphiql": "1.4.0",
    "graphql": "0.13.2",
    "graphql-tools": "3.1.1",
    "lodash": "^4.17.10",
    "lru-cache": "^4.1.2"
  }
}
```

apollo-server-koa：apollo 提供的服务端 GraphQL 开源模块种 nodejs-koa 的实现方式

apollo-server-module-graphiql ：apollo 提供的 GraphQL 开发者工具

graphql-tools：提供更好的创建 GraphQL schema 的方法

#### 项目核心代码结构

```
.
├── agent.js
├── app
│   ├── extend
│   │   └── context.js
│   ├── middleware
│   │   └── graphql.js
│   └── service
│       └── graphql.js
├── app.js
├── config
│   └── config.default.js
├── lib
│   ├── graphql-tag.js
│   ├── load_connector.js
│   ├── load_schema.js
│   └── util.js
├── package.json

```

<!--more-->

#### 核心代码

执行属顺序：load config.js ,extends --> load agent.js ---> load start app.js --> load app/service ---> load middleware

以上步骤略有省略，去除了源代码种未涉及的内容

app.js 和 agent.js 都只做了 两件相同的事情，在项目启动时挂载如下内容

1. 挂载 schema 属性到 app 对象
2. 挂载 connectorClass 属性到 app 对象

```js
'use strict';

module.exports = (app) => {
  require('./lib/load_schema')(app);
  require('./lib/load_connector')(app);
};
```

load_schema.js 核心代码，遍历 Graphql 目录下文件，获取指定参数属性，执行 makeExecutableSchema 返回

```js
Object.defineProperty(app, 'schema', {
  get() {
    if (!this[SYMBOL_SCHEMA]) {
      resolverFactories.forEach((resolverFactory) =>
        _.merge(resolverMap, resolverFactory(app))
      );

      this[SYMBOL_SCHEMA] = makeExecutableSchema({
        typeDefs: schemas,
        resolvers: resolverMap,
        directiveResolvers: directiveMap,
        schemaDirectives: schemaDirectivesProps,
      });
    }
    return this[SYMBOL_SCHEMA];
  },
});
```

load_connector.js 遍历 Graphql 目录下文件所有 connector.js，创建 Map 对象返回，key 为 connector.js 上级目录，value 为 connector.js 导出的方法

```js
Object.defineProperty(app, 'connectorClass', {
  get() {
    if (!this[SYMBOL_CONNECTOR_CLASS]) {
      const classes = new Map();

      types.forEach((type) => {
        const connectorFile = path.join(basePath, type, 'connector.js');
        /* istanbul ignore else */
        if (fs.existsSync(connectorFile)) {
          const Connector = require(connectorFile);
          classes.set(path.basename(type), Connector);
        }
      });

      this[SYMBOL_CONNECTOR_CLASS] = classes;
    }
    return this[SYMBOL_CONNECTOR_CLASS];
  },
});
```

根据如上加载规则，则项目文件格式则有特殊的含义：

1. schema.graphql ：采用 SDL 编写的 schema
2. resolver.js ：解析器，对应 schema 的字段，一般在此调用 connector
3. connector.js : 连接器，可以直接调用 ctx 对象，一般在这调用 service 或者逻辑处理
4. directive：指令，在解析器处理之前运行，一般处理一些通用操作，例如参数验证，权限验证等

app/extend/context.js 为 ctx 对象添加方法：

1. 获取 service.graphql 类的所有方法
2. 获得所有在 app.connectorClass 对象

```js
module.exports = {
  // 获取所有连接器，app.connectorClass 已经在 app.js/agent.js挂载
  get connector() {
    // 单例模式，判断是否已经存在，避免重复遍历
    if (!this[SYMBOL_CONNECTOR]) {
      const connectors = {};
      for (const [type, Class] of this.app.connectorClass) {
        // 实例化 connector 对象，并且传入 ctx
        connectors[type] = new Class(this);
      }
      this[SYMBOL_CONNECTOR] = connectors;
    }
    return this[SYMBOL_CONNECTOR];
  },

  get graphql() {
    return this.service.graphql;
  },
};
```

app/middleware/graphql.js 使用 apollo-server-koa 模块的中间件 处理 GraphQL API 路径下的请求

```js
const { graphqlKoa } = require('apollo-server-koa/dist/koaApollo');
const { resolveGraphiQLString } = require('apollo-server-module-graphiql');

function graphiqlKoa(options) {
  return (ctx) => {
    const query = ctx.request.query;
    return resolveGraphiQLString(query, options, ctx).then((graphiqlString) => {
      ctx.set('Content-Type', 'text/html');
      ctx.body = graphiqlString;
    });
  };
}

module.exports = (_, app) => {
  const options = app.config.graphql;
  const graphQLRouter = options.router;
  let graphiql = true;

  if (options.graphiql === false) {
    graphiql = false;
  }
  return async (ctx, next) => {
    // 捕获在config.graphql.router 设定的路由，因为所有 GraphQL 接口都走一个路由
    if (ctx.path === graphQLRouter) {
      // ....
      const { onPreGraphiQL, onPreGraphQL, apolloServerOptions } = options;
      // 执行配置的GraphiQL钩子函数
      if (ctx.request.accepts(['json', 'html']) === 'html' && graphiql) {
        if (onPreGraphiQL) {
          await onPreGraphiQL(ctx);
        }
        // 返回开发者平台路径
        return graphiqlKoa({
          endpointURL: graphQLRouter,
        })(ctx);
      }

      // 执行配置的GraphQL钩子函数
      if (onPreGraphQL) {
        await onPreGraphQL(ctx);
      }

      const serverOptions = Object.assign({}, apolloServerOptions, {
        schema: app.schema,
        context: ctx,
      });
      return graphqlKoa(serverOptions)(ctx);
    }
    await next();
  };
};
```

apollo-server-koa 涉及的 graphqlKoa 源代码，主要为获取 ctx 中的属性和根据 GraphQL 的配置，处理请求并且返回

```js
export interface KoaHandler {
    (ctx: Koa.Context, next: any): void;
}
export declare function graphqlKoa(options: GraphQLOptions | KoaGraphQLOptionsFunction): KoaHandler;
```

```js
export function graphqlKoa(
  options: GraphQLOptions | KoaGraphQLOptionsFunction,
): KoaHandler {
  //...
  const graphqlHandler = (ctx: Koa.Context): Promise<void> => {
    return runHttpQuery([ctx], {
      method: ctx.request.method,
      options: options,
      query:
        ctx.request.method === 'POST'
          ? // fallback to ctx.req.body for koa-multer support
            ctx.request.body || (ctx.req as any).body
          : ctx.request.query,
      request: convertNodeHttpToRequest(ctx.req),
    }).then(
      ({ graphqlResponse, responseInit }) => {
        Object.keys(responseInit.headers).forEach(key =>
          ctx.set(key, responseInit.headers[key]),
        );
        ctx.body = graphqlResponse;
      },
      (error: HttpQueryError) => {
        if ('HttpQueryError' !== error.name) {
          throw error;
        }

        if (error.headers) {
          Object.keys(error.headers).forEach(header => {
            ctx.set(header, error.headers[header]);
          });
        }

        ctx.status = error.statusCode;
        ctx.body = error.message;
      },
    );
  };

  return graphqlHandler;
}
```
