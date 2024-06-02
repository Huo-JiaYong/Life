## 搭建 DevOps 环境

### 服务器

目标：要有一套自己非常熟悉的环境，覆盖整个开发和部署的流程。类似于生产系统

- [x] CentOS 7 环境搭建，用 wsl2 搞相较于原生的 CentOS 7 使用一下不同工具（有可能会有坑）
- [x] Gitlab 搭建 https://zhuanlan.zhihu.com/p/534072989 
- [ ] 多个环境隔离

### CICD

- [ ] 完成 AgileBoot-Backend 的 gitlab 的自动部署脚本编写
  - [ ] 提交后自动打包并部署到 docker 的功能
  - [ ] 不同分支直接部署到不同环境
- [ ] 以 git flow 的流程模拟开发提交
- [ ] 类似于网思的多版本分支迭代的流程，主版本需要发布，其他版本只打包 OR 多环境
- [ ] 使用一下 jenkins 进行多个存储点（gitlab、gitee）的部署
- [ ] 使用 docker 部署的微服务 日志怎么搜集呢 ？



## 脚手架的应用

目标：熟悉主流脚手架的使用和部署等，达到能快速搭建一个项目的水平

选择源码和文档都开源的 ruoyi-vue-plus

- [ ] 授权模块
- [ ] Redis 集成
- [ ] RocketMQ 集成
- [ ] Elasticsearch 集成
- [ ] MinIO 的集成



## 自动化测试