# 使用grunt搭建前后端分离的开发环境
喊了很长时间的前后端分离，这两天终于花时间实现，在这里总结一下。


## 为什么前后端分离
先介绍我们这边的开发模式，我们后端采用的Java，开发IDE使用的eclipse。因为整个系统比较庞大，又切分为多个小工程， 通过maven管理工程之间的依赖关系。开发/调试时启动eclipse的tomcat服务。
在这种模式下开发，只做前端页面的小伙伴时不时会遇到以下问题： 
- Eclipse编译报错，tomcat服务无法启动。小伙伴们需要更新maven, 删除以前的java包，重新编译启动服务。 这些操作，在快的情况下可能半个小时内可以搞定，在不顺利的时候可能大半天都用在启动服务商了
- 每次保存代码时，Eclipse都会重新编译java文件，需要等编译发布完成之后才可以刷新页面，看到修改后的效果，导致效率低下

## 方案
google/bing/baidu大法过后，打算用以下思路来处理前后端分离：
1. 前端开发人员不启动eclipse，而是启动grunt-connect提供的静态web服务
2. 在grunt-connect配置中增加一个插件，用来转发请求，将所有的后台服务接口转发到另外一台后台服务器上
示例代码如下
```javascript
// gruntfile.js
// ...

// 引入connect-modrewrite-do插件，用来转发请求
var proxyRewrite = require('connect-modrewrite-do');

grunt.initConfig({
  connect: {
    options: {
      base: [
        // 数组的形式配置不同子工程的地址
        'D:/work/projects/projectA/src',
        'D:/work/projects/projectB/src',
        '.'
      ],
      port: 9000,
      hostname: '*',
      livereload: true
    },
    websocket: {
                proxies: [
                    {
                        context: '/websocket',
                        host: proxyServer.split(':')[0],
                        port: proxyServer.split(':')[1],
                        ws: true
                    }
                ]
            },
    livereload: {
      options: {
        open: false;
        middleware: function(connect, options, middlewares) {
          // 设置websocket转发
          middlewares.unshift(require('grunt-connect-proxy/lib/utils').proxyRequest);
          
          // CORS middleware, 设置通用的头部信息
          middlewares.unshift(function(req, res, next) {
            res.setHeader('Access-Control-Allow-Origin', '*');
            res.setHeader('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE,OPTIONS');
            res.setHeader('Access-Control-Allow-Headers', 'authorization, Origin, X-Requested-With, Content-Type, Accept');
            res.setHeader('Access-Control-Allow-Credentials', true);
            
            next();
          });
          
          // 使用connect-modrewrite插件
          middlewares.unshift(proxyRewrite([
            '^/api/ http://10.0.3.3/api/ [P]',
            '^/files/ http://file.server.ip.address/files/ [P]'
          ]));
        }
      }
    }
  }
})
grunt.loadNpmTasks('grunt-contrib-connect');

grunt.registerTask('server', [
    'configureProxies:websocket',
    'connect:livereload',
    'watch'
]);
// ...
```
在命令行中执行`grunt server`， 然后在浏览器中打开10.0.3.ip:9999就可以访问了。
## 遇到的问题

1. 原先是打算用grunt-connect-proxy插件来转发请求的， 但是grunt-connect-proxy不支持post请求, 搜索后改用connect-modrewrite插件
2. Cookie path的重写
   假设前端grunt-connect服务为http://10.0.3.1:9999
   
   后端提供的服务因为tomcat设置的缘故，有时是以工程名为path的，如http://10.0.3.3/projectA. 直接访问10.0.3.3是访问不了系统的， 需要访问10.0.3.3/projectA才可以。
   这样，后端服务设置的JSESSIONID cookie在前端服务器不生效。
   
   后端设置的cookie为“Set-Cookie: JSESSIONID=404C538DA2FEC00E165D3B844AF2A06D; Path=/projectA/; HttpOnly”
   
   前端服务需要的cookie"Set-Cookie: JSESSIONID=404C538DA2FEC00E165D3B844AF2A06D; Path=/; HttpOnly"
   
   因为这个原因，修改了connect-modrewrite插件，在代理返回结果中将cookie的path属性强制设定为Path=/.然后提交了一个新的npm包。(很简单粗暴，更优雅的方式应该是为connect-modrewrite插件扩展一个事件/回调函数，可以让使用者在grunt中处理自己的逻辑)
3. Updated on 20170411: 增加了grunt-connect-proxy来进行websocket的转发
   
## 参考资料
http://www.cnblogs.com/lyzg/p/5224063.html

https://github.com/gruntjs/grunt-contrib-connect

https://github.com/drewzboto/grunt-connect-proxy/

https://github.com/tinganho/connect-modrewrite

https://www.npmjs.com/package/connect-modrewrite-do

http://stackoverflow.com/questions/12755865/rewrite-response-headers-with-node-http-proxy

