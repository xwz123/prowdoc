## gitee & github issue and comment two-way synchronization

### 需求

1. 将指定仓库在Gitee上创建的issue以及在issue上添加的评论的内容经过汉译英后在GitHub上同步创建；
2. 将指定仓库在Github上创建的issue以及在issue上添加的评论的内容经过英译汉后再Gitee上同步创建；

### 解决方案

**系统架构图：**

![img](image\async-issue-comment.png)

- robot-gitee-courier：负责接收Gitee平台的webhook 并封装成对应的MQ消息并发布到MQ；
- MQ：kafka 消息队列中间件
- robot-gitee-access：订阅MQ中的gitee-webhook主题并将消息分发到对应的下游机器人服务；
- robot-gitee-synchronizer：处理gitee 平台 issue 和 comment的创建事件消息，验证消息的合法性，对内容进行翻译后并透传给sync-server；
- robot-github-courier：负责接收Github平台的webhook 并封装成对应的MQ消息并发布到MQ；
- robot-github-access：订阅MQ中的github-webhook主题并将消息分发到对应的下游机器人服务；
- robot-github-synchorizer：处理Github平台 issue 和 comment的创建事件消息，验证消息的合法性，对内容进行翻译后并透传给sync-server；
- tow-way-sync-server：负责响应robot-gitee-synchronizer和robot-github-synchronizer发起的同issue和comment的请求分别在gitee和github平台创建对应的issue和comment；

### 问题梳理

1. **需同步issue和comment的仓库对应关系怎么关联？**
   A：gitee 和 github 组织和仓库保证一一对应，并支持组织对应关系可配置；同步机器人只需关心需要同步的仓库，不用配置对应关系。
   B：组织/仓库名同步配置一一对应建立映射关系。
2. **同步创建的issue和comment标识，避免循环同步以及存量数据同步？**
   A：同步创建的issue的创建者应都为机器人所创建，并在创建的同时设定 `sync-from-platform` 标签；并以创建者+标签进行区分是否需要同步。comment 同步时在body中加隐藏域，并根据隐藏域+创建者判段是否需要同步。
   B：使用存储组件。
3. **各社区机器人评论是否需要同步？**
4. **翻译组件接入方？**
5. **存量issue和评论处理方案？**
   A：在tow-way-sync-server 服务内部中支持cron-job完成存量同步；
   B：分别开发同步gitee和github平台cron-job服务再调用tow-way-sync-server 的api接口完成存量同步；

### 开发计划

**03月07 至 03月11：**

确定方案

robot-gitee-synchorizer 开发（不包含翻译组件接入）

robot-github-courier、robot-github-access、robot-github-synchorizer 开发（不包含翻译组件接入）

**03月14 至 03月18：**

翻译组件调研以及各机器人接入，tow-way-sync-server服务开发，完善issue同步逻辑。

**03月21 至 3月25:**

完成comment同步，接入华为云kafka，以及测试和debug

**03月28 至 03月31：**

支持存量issue和comment同步
