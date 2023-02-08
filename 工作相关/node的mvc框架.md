---
title: node的mvc框架
url: https://www.yuque.com/endday/blog/vsz9ul
---

<a name="GLqlt"></a>

# 框架对比

| 名称 | 底层框架 | 优点 | 缺点 |
| --- | --- | --- | --- |
| koa2 |&#x20;
&#x20;| express的进化版，async/await 语法，小而美 | 只提供基础的网络请求和中间件功能，其他功能需要搭配插件 |
| nest | express或fastify | 基于Express.js的全功能框架，TypeScript，良好的类型提示，IoC、依赖注入、AOC，更新快，社区力量强 | 新的概念较多，写起来比较繁琐， 限制也很多 |
| egg.js | koa2 | 生成 web 框架的框架，约定大于配置，多进程启动，开发时的热更新 | 对ts的支持较弱，阿里的重心移交到midway上 |
| midway | egg.js、koa2、express | egg的TypeScript版，文档友好，支持注解，面向切面编程，阿里项目，可使用egg的生态 | 项目较新，社区力量不足
新的概念较多 |
| think.js | koa2 | 类似thinkphp的路由解析方式，TypeScript，360项目 |&#x20;
&#x20;|

<a name="GxGvj"></a>

# 开发和发布

| 前端 | 本地开发 |&#x20;
1\. 本地启动应用
2\. 使用fiddler代理请求到本地
&#x20;|
| --- | --- | --- |
|  | 应用发布 |&#x20;
1\. 前端合并代码到功能分支
2\. 通过前端发布系统发布
3\. 获取页面
&#x20;|
| 后端 | 本地开发 | 本地启动应用，在localhost进行开发 |
|  | 应用发布 | 前端页面和后端代码统一在一个代码仓库管理
1\. 后端合并代码到功能分支
2\. 前端提交页面到功能分支
3\. 合并功能分支到主干
4\. 将分支发布到测试环境或线上（可选择单独发布或一起发布）
&#x20;  1\. 页面发布（后端发布系统）
&#x20;  2\. 后端发布（微服务发布系统）
&#x20;     1\. 构建docker
&#x20;     2\. 发布镜像
&#x20;|

<a name="OEJTK"></a>

# 回滚

1. 找出综合提交记录和docker镜像的记录，找出合适的commitId
2. 根据commitId来确定回滚的位置

后端的docker镜像和commitId相关联，可以根据commitId来统一前端页面和后端代码回滚的位置

| 前端 | 根据代码提交记录，回滚页面 |
| --- | --- |
| 后端 | 根据docker镜像的记录回滚 |

| **多机部署，平滑重启** |  |  |
| :---: | --- | --- |
| egg.js和midway提供优雅关闭的功能，会自动回收 worker 等进程，等待请求结束后才结束。
其他框架待调查 |  |  |
|  | 方案1 | 方案2 |
| 概述 | 使用docker进行管理
1\. 提供性能监控和日志插件
2\. 已有可使用的发布平台
3\. 可服务化
&#x20;| 实现一套切换流量的方案 |

[
eggjs + docker 最佳实践](https://github.com/eggjs/egg/issues/1431#)
启动参数为

    "scripts": {
        "start": "egg-scripts start --workers=1 --title=egg-server-some-server"
    }

`docker-compose.yml`大概是这样:

    version: '3'
    services:
      some-server:
        image: node:10.16.0
        ports:
          - "7001"
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - .:/usr/src/app
        working_dir: /usr/src/app
        command: sh -c "npm run tsc && npm start"
        # 或者
        # command: npm start
        logging:
          driver: json-file
          options:
            max-file: '3'
            max-size: 10m
        restart: always

主要就是不要后台启动就行了，其他没啥了，标准的Node服务。
![](..\assets\vsz9ul\75753fc624fedd8f6beb774dcc77050f.svg)![](..\assets\vsz9ul\c6f34aa845727c28076242dacdc36ce9.svg)![](..\assets\vsz9ul\6b008efc50fd27f3dc7be646f93ce755.svg)![](..\assets\vsz9ul\051e1168c46dcd6d5ac7b1c902874b40.svg)![](..\assets\vsz9ul\bb379dd5185039e8e6c3da9eea201412.svg)
