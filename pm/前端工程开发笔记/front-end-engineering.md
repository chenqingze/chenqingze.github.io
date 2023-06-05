# 前端工程开发笔记（Angular）

## 一. 初始化工程

```
npx @angular/cli@latest new your-project-name
```

note:推荐使用npx的方式初始化项目便于版本管理

## 二. 增加依赖（配置略）

1. 安装angular material组件库和cdk
   
   ```
   ng add @angular/material
   ```

2. 安装语言国际化依赖
   
   ```
   ng add @angular/localize
   ```

## 三. 配置环境

```
ng generate environments
```



## 四. 代理到后端服务器

1. Create a file proxy.conf.json in your project's src/ folder.

2. Add the following content to the new proxy file:
   
   ```
    {
      "/api": {
       "target": "http://localhost:8080",
       "secure": false,
       "pathRewrite": {
         "^/api": ""
       },
       "changeOrigin": true,
       "logLevel": "info"
      }
      }
   ```

3. In the CLI configuration file, angular.json, add the proxyConfig option to the serve target:
   
   ```
    …
      "architect": {
       "serve": {
         "builder": "@angular-devkit/build-angular:dev-server",
         "options": {
           "browserTarget": "your-application-name:build",
           "proxyConfig": "src/proxy.conf.json"
         },
      …
   ```
