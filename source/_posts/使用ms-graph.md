---
title: 使用Microsoft Graph，操作ToDo任务清单。
time: 2022-4-27
---

[TOC]

# 前言

教程目的是本人的频道BOT——RuaRuaBot 的原理基础。
关于获取用户MicrosoftTodoApi以便提醒用户相关计划。

## 传送门

- RUARUABOT企划!
- RuaRuaBot-Node:使用频道机器人+云开发系列教程



# 准备

## 开发环境与框架准备

你需要准备好下面的东西。
- koa / koa-router
- Node.Js
- TypeScript
- ts-node
- msal-node

当然，你也可以准备这些东西
- WebStorm 希望是正版哦
- 一个能放音游曲的播放器。
- qq-guild-bot 来做一个频道bot！

## 注册Azure Ad应用

1. 添加一个应用
2. 增加一个密钥并复制。

## 引入相关包

### msal 
> MSAL Node enables applications to authenticate users using Azure AD
```typescript
import * as msal from "@azure/msal-node";
```
### 其他
其他koa，qq-guild-bot相关使用方法不多说了。

# 开始

## 修改AzureAD应用权限

### 修改受支持的帐户类型
进入清单页面
修改清单文件下面的属性。
> 鉴于受支持功能的暂时性差异，建议不要为现有应用程序注册启用个人 Microsoft 帐户。如需启用个人帐户，可使用清单编辑器。 
```json
"signInAudience": "AzureADandPersonalMicrosoftAccount",
```
- AzureADMyOrg：仅应用已注册的组织目录中的帐户（单租户）。
- AzureADMultipleOrgs：任何组织目录中的帐户（多租户）。
- AzureADandPersonalMicrosoftAccount：任何组织目录（多租户）中的帐户和个人 Microsoft 帐户（例如，Skype、Xbox 和 Outlook.com）。

同时
accessTokenAcceptedVersion要改为2(测试版API)
### 修改回调地址
对这个应用添加回调地址，包括`https://localhost:3000/auth`
在上线后，传递的重定向url改变后，需要包含在这里的列表中。



## 构建一个ConfidentialClientApplication

`ConfidentialClientApplication`用来生成用于授权验证的用户登入URL。

```typescript
const msalClient = new msal.ConfidentialClientApplication({
    auth: {
        clientId: "xxxx",
        clientSecret: "xxxxxxxxs",
        authority: "https://login.microsoftonline.com/common/"
    },
    system: {
        loggerOptions: {
            piiLoggingEnabled: false,
            logLevel: msal.LogLevel.Warning,
        }
    }
})
```

## 创建获取AccessToken相关的登入路由
```typescript

let code: string = '';
let token: string = '';
```

```typescript
router.get("/login", async (ctx) => {
    ctx.body = "准备跳转中..."
    let url = await msalClient.getAuthCodeUrl({
        redirectUri: "http://localhost:3000/auth",
        scopes: ['user.read', 'Tasks.Read', 'Tasks.ReadWrite', 'Tasks.Read.Shared', 'Tasks.ReadWrite.Shared'],
    });
    ctx.redirect(url)
})
```


```typescript
router.get("/auth", async (ctx) => {
    code = ctx.query.code as string;
    let url = await msalClient.acquireTokenByCode({
        code: code,
        redirectUri: 'http://localhost:3000/auth',
        scopes: ['user.read', 'Tasks.Read', 'Tasks.ReadWrite', 'Tasks.Read.Shared', 'Tasks.ReadWrite.Shared'],
    })
    ctx.body = "获取token中..."
    console.log("access: " + url!.accessToken)
    token = url!.accessToken;
    ctx.body = "完成!可以关闭了"

})
```
对于`scopes`中的权限，请查看[官方文档](https://docs.microsoft.com/zh-cn/graph/permissions-reference?view=graph-rest-1.0)

## 编写API操作类
我们先以TaskList为列
https://docs.microsoft.com/zh-cn/graph/api/todo-list-lists?view=graph-rest-1.0

```ts
import axios, {AxiosInstance} from "axios";

export class TaskApiUtil {

    axios: AxiosInstance;

    constructor(accessToken: string) {
        this.axios = axios.create({
            baseURL: 'https://graph.microsoft.com/v1.0',
            headers: {
                Authorization: "Bearer " + accessToken
            }
        })
    }

    async getLists() {
        return (await this.axios.get("/me/todo/lists")).data;
    }
}
```

## 编写获取列表路由

```ts
router.get("/task/list", async (ctx) => {
    let u = new TaskApiUtil(token)
    ctx.body = await u.getLists()

})
```
没问题了。

# 后续
至于如何运用到Bot中，请等待后续教程。