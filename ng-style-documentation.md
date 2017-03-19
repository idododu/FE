# 浅谈angular的代码注释以及如何自动生成组件文档

## 背景描述
前端组虽然产出了很多组件，也做了专门的demo页面，但总还是有其他业务组的同事过来询问如何使用组件，不得不停下手头工作协助业务组向同事进行讲解。 
另外，在处理其他同事负责的组件也发现有些代码不好理解，维护起来不够方便。
刚好本周又轮到我做技术分享，不由得想是否有什么更好的方式可以解决这几个问题呢？ 因为没有处理过类似的事情，于是把目光转向google. 输入 angular comment style关键字后，翻了几篇，终于找到有用的， 是github上angular wiki的一篇文章[Writing AngularJS Documentation](https://github.com/angular/angular.js/wiki/Writing-AngularJS-Documentation) 

在这篇文章中提到angular源代码的注释采用jsdoc的格式，使用ngdoc工具来生成api文档，所有的注释、例子、说明，全部都存放在源代码中。
顺着这篇文章又抓到一些其他信息，现在把链接都放在这里，大家可以参考查阅
- [API Docs Sytax](https://github.com/idanush/ngdocs/wiki/API-Docs-Syntax): 为服务、指令、过滤器编写文档注释的格式
- [grunt-ngdocs](https://github.com/m7r/grunt-ngdocs): grunt生成ngdocs的工具
- [grunt-ngdocs-example](https://github.com/m7r/grunt-ngdocs-example): grunt-ngdocs的使用实例

## 其他想说的话
其实现在流行的框架都有很完整，API文档，自动化工具，官方cli, 插件库， 配到ui框架，提问论坛，教育课程等等应有尽有。
如果我们仅仅是使用这些框架去完成工作任务，而不去体系化的了解这个框架，那么我们掌握的可能只是那几个api方法函数而已。

想起朋友圈中的那个笑话：
经理召开部门会议，会上说：A君做的这个项目很不错！为公司带来了100万的收入。
A君窃喜，这是升职加薪发奖金的节奏啊。
但是经理接下来说，那么A君值100万么？
不值。值100万的那个人是我。
假如这个项目不给A君做，而是花10万请其他的人来做，培训一下业务知识，同样也可以做的很好。
所以，大家不要以为自己程序写的好就有多厉害。 重要的是思维，知道么。

Be Creative!
