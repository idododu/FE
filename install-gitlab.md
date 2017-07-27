# gitlab install memo
## 缘由
目前公司内部使用SVN进行版本控制，使用jekins进行打包部署，使用jira进行bug跟中，开发人员需要在不同的应用中切来切去。
之前有查资料时就对gitlab有点钟情，参加过TFC,张云龙分享的前端体系构建有感，更加想部署个gitlab玩一下，看能否推广到公司内部去。
于是开始

## install
跟随着官网上一步一步走下来，成功安装，没有什么太多好解释的。   
https://about.gitlab.com/installation/#centos-6

接下来安装ci的时候就有坑了   
- 安装runner, 参考[安装文档](https://docs.gitlab.com/runner/install/linux-repository.html), 一切正常   
- 配置`.gitlab-ci.yml`后提交，第一次跑CI报错：   
```
Using Docker executor with image pijzl/docker-node-karma-protractor-chrome ...   
ERROR: Preparation failed: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```
  问题原因： 服务器上没有装docker   
  解决方法: 安装docker后解决   
- 安装好docker后再次跑CI,仍然报错   
```
fatal: unable to access 'http://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@localhost/root/git-test.git/': Failed to connect to localhost port 80: Connection refused
```
  问题原因: Runner启动的docker容器里无法访问localhost:80   
  解决方法：第一步： 使用`ifconfig`命令检查docker容器的ip地址是多少  
  
```
docker0 Link encap:Ethernet  HWaddr 00:00:00:00:00:00
    inet addr:172.17.42.1  Bcast:0.0.0.0  Mask:255.255.0.0
    inet6 addr: fe80::54b8:e8ff:fe43:5554/64 Scope:Link
    UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
    RX packets:693658 errors:0 dropped:0 overruns:0 frame:0
    TX packets:1044150 errors:0 dropped:0 overruns:0 carrier:0
    collisions:0 txqueuelen:0 
    RX bytes:153277725 (146.1 MiB)  TX bytes:1495669968 (1.3 GiB)
```   
  
  第二步：修改`/etc/gitlab-runner/config.toml`配置文件，在runners.docker下增加extra_hosts(不需要重启gitlab-runner)   
```
[runners.docker]
  extra_hosts = ["localhost:172.17.42.1"] # 增加extra_hosts设置
  pull_policy = "never"
  ...
[runners.cache]
```
  
- 修改后再次执行CI，不由得“卧槽！终于跑到npm install了！”，但是紧接着又是一声“卧槽？怎么npm install这么慢，十几分钟了还没装好？”   
  原因： npm registry被墙   
  解决方法： 设置npm registry源`npm config set registry https://registry.npm.taobao.org`   
- 设置完cnpm registry后，唰唰唰npm install 命令跑的很快， 但是！但是还是报错！   
  问题：  
  
    ```
    npm ERR! phantomjs-prebuilt@2.1.14 install: `node install.js`
    npm ERR! Exit status 1
    npm ERR! 
    npm ERR! Failed at the phantomjs-prebuilt@2.1.14 install script 'node install.js'.
    npm ERR! Make sure you have the latest version of node.js and npm installed.
    npm ERR! If you do, this is most likely a problem with the phantomjs-prebuilt package,
    npm ERR! not with npm itself.
    npm ERR! Tell the author that this fails on your system:
    npm ERR!     node install.js
    npm ERR! You can get information on how to open an issue for this project with:
    npm ERR!     npm bugs phantomjs-prebuilt
    npm ERR! Or if that isn't available, you can get their info via:
    npm ERR!     npm owner ls phantomjs-prebuilt
    ```
  
    ```
    > node-sass@4.5.3 install /builds/root/git-test/node_modules/node-sass
    > node scripts/install.js
    
    Downloading binary from https://github.com/sass/node-sass/releases/download/v4.5.3/linux-x64-51_binding.node
    Cannot download "https://github.com/sass/node-sass/releases/download/v4.5.3/linux-x64-51_binding.node": 
    
    ESOCKETTIMEDOUT
    
    Hint: If github.com is not accessible in your location
        try setting a proxy via HTTP_PROXY, e.g. 
    
        export HTTP_PROXY=http://example.com:1234
    
    or configure npm proxy via
    
      npm config set proxy http://example.com:8080
    ```
  原因：感谢GFW! [npm常用mirror](https://segmentfault.com/a/1190000004690758)   
  解决方法： 修改npm配置文件，设置常用依赖包的cdn url   
  
   ```
   vi ~/.npmrc

   sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
   phantomjs_cdnurl=https://npm.taobao.org/mirrors/phantomjs
   CHROMEDRIVER_CDNURL=http://npm.taobao.org/mirrors/chromedriver
   electron_mirror=https://npm.taobao.org/mirrors/electron/
   fsevents_binary_host_mirror=https://npm.taobao.org/mirrors/fsevents/
   ```
  
- 设置完CDN URL变量之后，再跑CI, 好像可以装了，但是... 怎么它还是每次都要安装这些玩意呢？我不禁陷入了深深的沉思....
  问题： 每次install都要一堆的时间，很烦   
  解决方法： [自定义docker镜像](http://edu.cnzz.cn/201509/96952310.shtml), 将需要的npm依赖全部安装在global中，虽然每个项目可能需要的依赖不一样，但是考虑到有很多相同的依赖，就先这样吧   
- YEAH~ 我做了自己的docker镜像哎！修改，加载，运行CI... 哎， 怎么可能那么顺利呢？   
  问题：   
  
  ```
  Using Docker executor with image cn/ci-runner ...
  Using docker image caf323b112b9818d3116f3eb370aecafb9db2424512e53c982cdfbe5473aec9b for predefined container...
  ERROR: Preparation failed: Error: No such image: cn/ci-runner
  ```
  原因： 如果没有设置registry的话，docker默认会去hub.docker.com加载镜像文件，但是因为这个cn/ci-runner没有push到hub.docker.com,docker无法加载，就会报错   
  解决方法： 使用本地镜像   
  
    ```
    [runners.docker]
       extra_hosts = ["localhost:172.17.42.1"] # 增加extra_hosts设置
       pull_policy = "never" # 使用本地镜像，不从hub.docker.com读取
    ...
    [runners.cache]
    ```
- 还有bower的坑
  问题： bower install很慢，都是从github下载的
  解决方法： 现在docker中安装好Bower, ci执行`bower install --offline --allow-root`命令，来使用本地缓存   
- 设置nginx访问权限
  问题： 跟公司自身的管理规则相结合，有时候可能需要设置固定某些网段才可以访问源代码
  解决方法： 在`gitlab.rb`中增加`nginx['custom_gitlab_server_config']`设置
  ```
  nginx['custom_gitlab_server_config'] = "location ~* (.*) {\n allow 192.168.1.0/24; \n deny all;\n proxy_cache off;\n proxy_pass  http://gitlab-workhorse;\n root   html;\n index  index.html index.htm;}\n"
  ```
  注意：   
    1. 此处修改不可以设置`location /`规则,因为gitlab自己的`gitlab-http.conf`中已经有对应的配置；   
    2. `proxy_cache off;\n proxy_pass  http://gitlab-workhorse;\n`这两行一定要加，不然全部报404错误   
    3. `root   html;\n index  index.html index.htm;`这两行也要加，因为我们使用`location ~* (.*)`重置了所有的请求匹配   
    4. `192.168.1.0/24`是指`192.168.1.0` - `192.168.1.254`的ip区段， 如果想匹配`192.168.*.*`可以使用`192.168.0.0/16`   
    5. 以后应该会有对应的配置项，[Issue 2198](https://gitlab.com/gitlab-org/omnibus-gitlab/issues/2198)中有人提出一个proposal   
    6. 其实更好的ip权限控制应该是放在组里边或者跟`git command`相关，比如developer必须在`192.168.1.*`网段中才能`git clone/push/pull`源代码，gitlab的其他功能应该是全部可以访问的。参考[gitolite](http://gitolite.com/gitolite/cookbook/index.html#example-2-restricting-by-source-ip-address)和[github enterprise](https://help.github.com/enterprise/11.10.340/admin/articles/restricting-ssh-access-to-specific-hosts/)

## 其他资料
[gitlab官网](https://gitlab.com/)   
[gitlab中文网](https://docs.gitlab.com.cn)   
[www-gitlab-com源码](https://gitlab.com/gitlab-com/www-gitlab-com/tree/master)   
[gitlab 汉化](http://www.ywlinux.com/archives/166)   
[gitlab workflow overview](https://www.gitlab.com.cn/2016/10/25/gitlab-workflow-an-overview/)   
[centos7 下参考 官方说明 搭建gitlab服务](https://segmentfault.com/a/1190000008291730)   
[gitlab-ci 从安装到差点放弃](https://segmentfault.com/a/1190000007180257)   
[Gitlab 安装并修改http端口](https://low.bi/ubuntu-gitlab/)   
[gitlab ci - ng protractor - demo](https://gitlab.com/planet-innovation/gitlab-ci-angular-webapp)   
[gitlab ci - maven](https://gitlab.com/snippets/1665449)   
[gitlab ci安装步骤](http://www.tuicool.com/articles/iqUzMrq)   
[docker 修改已有镜像](http://edu.cnzz.cn/201509/96952310.shtml)
[gitlab nginx配置官方文档](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/nginx.md)   
[nginx配置location总结](https://segmentfault.com/a/1190000002797606)   
