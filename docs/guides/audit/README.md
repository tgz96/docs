# 审计综述

安全性和合规性对一个企业或者组织的运作至关重要，是企业和组织运转的基石。想要实现安全与合规并不简单，Authing 提供操作审计功能以帮助企业和组织在业务不断增长的同时保持安全与合规。

## 日志分类

Authing 提供的审计功能分为两个方面：

- [管理员行为的操作日志](./administrator-action.md)：你可以得到所有管理员用户通过身份管理平台的行为
- [用户行为的日志](./user-action.md)：可以清晰得还原出用户在平台中的行为，以支持企业的合规管理，同时可用作事件发生后的回溯和定责。

## 日志内容

不管管理员行为日志还是用户行为日志，主要都是由以下 5 个部分组成，来记录操作人在什么时间操作了什么资源：

- 资源: 受影响的用户或资源。
- 操作: 采取什么行动，无论是访问资源还是发放证书。
- 相关信息: 发生此操作的项目或服务器。
- 操作人: 实施动作的人。
- 日期: 采取行动的时间。
