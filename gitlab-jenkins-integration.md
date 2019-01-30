# GitLab与Jenkins集成配置

## 背景介绍
GitLab自己有CI/CD能力，但是目前项目中的发版工作使用Jenkins来打包并部署到测试服务器。  
当GitLab中有新的merge request完成时，需要再额外打开Jenkins界面，点击一次构建发版。  
本着能省就省的原则，我们是不是可以在GitLab中Merge Request完成时，自动触发jenkins构建呢？  
答案当然是肯定的

## 配置步骤
1. 查阅[官方配置文档](https://docs.gitlab.com/ee/integration/jenkins.html)
2. 在Jenkins服务中安装插件[Jenkins GitLab](https://wiki.jenkins.io/display/JENKINS/GitLab+Plugin)和[Jenkins Git](https://wiki.jenkins.io/display/JENKINS/Git+Plugin)
3. 参考[此文](https://github.com/jenkinsci/gitlab-plugin/wiki/Setup-Example)在Jenkins中进行API Token、GitLab插件配置
4. 在Jenkins指定项目-“构建触发器”下"Build When a change is pushed to GitLab"中配置构建策略
5. 在GitLab指定项目中Settings -> Integrations中增加jenkins webhook，webhook url在Jenkins指定项目-“构建触发器”-"Build When a change is pushed to GitLab"中有提供
6. 验证是否成功

## 参考资料
- [官方配置文档](https://docs.gitlab.com/ee/integration/jenkins.html)
- [Jenkins GitLab Plugin Setup Example](https://github.com/jenkinsci/gitlab-plugin/wiki/Setup-Example)
- [Jenkins GitLab Plugin 配置文档](https://github.com/jenkinsci/gitlab-plugin#global-plugin-configuration)
- [Webhook fails](https://github.com/jenkinsci/gitlab-plugin/issues/375)
