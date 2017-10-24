# 本地搭建angular-material-io

## 说明
[Angular Material](https://material.angular.io/)是Google官方遵循Material Design设计规范，使用Angular实现的UI组件库。  
想要将公司ng1的前端框架升级到ng2/4/5，觉得可以借鉴Angular Material的代码实现，更好的进行模块化、组件化、工程化。  
原先以为只是正常的"Download" -> `npm install` -> `ng serve`, 结果真的跑起来却又遇到一堆坑，特此记录  

## 搭建步骤
1. 从https://github.com/angular/material.angular.io下载源代码
2. `npm install`安装依赖包
3. `ng serve`启动本地服务
   - 此时cmd报错"Cannot find module "@angular/material-examples""
     一番搜索后再[这里](https://github.com/angular/material.angular.io/issues/264)找到了答案。 原来`material-examples`模块的代码是放在https://github.com/angular/material2/中的。
     需要将material2下载下来，`npm install`之后，再在material.angular.io中执行`npm run fetch-local`脚本。
     `fetch-local`脚本会做以下的事情：
     1) 在`material2`文件夹下执行`gulp docs`, `gulp material-examples:build-release`任务
     2) gulp任务执行完成后，将相关资源文件（Packages，Examples, Pluckers, APIs, Guides, Overview）copy到material.angular.io文件夹下, 有了这些资源文件之后再执行`ng serve`就可以成功运行了
4. 从[Material 2](https://github.com/angular/material2/)下载源代码，然后执行`npm install`
5. 在`material.angular.io`文件夹下执行`npm run fetch-local`脚本
   - 此时cmd报错"bash is not recogized as an internal or external command"  
     原来`fetch-local`的脚本是写在一个sh文件中，通过`bash`命令来调用的, 可是因为windows自身没有bash命令工具，所以报错。  
     解决方法： 安装[Cygwin](https://www.cygwin.com/), Cygwin是windows平台上运行的unix模拟环境，安装以后可以直接在cmd窗口运行unix命令。  
     [安装教程](是windows平台上运行的unix模拟环境)  
     也可以通过360电脑管家进行安装， 安装之后需要设置环境变量， 将`D:\Program Files\cygwin\bin`添加到环境变量中  
   - 装好Cygwin后，再执行`npm run fetch-local`, 此时cmd报错"Cannot find module "@angular/material-moment-adapter""  
     看报错应该是源码中export， import的问题，搜了一下issue, 没找到原因，因为只是demo页面，也没有细究  
     解决方法： 将`src/material-examples/`下的`datepicker-formats`, `datepicker-locale`, `datepicker-moment`删除，眼不见心不烦  
   - 再执行`npm run fetch-local`命令，gulp任务顺利通过，但是在Copy Overview环节再次报错'find parameter format not correct'  
     原因： Cygwin的环境变量是追加上去的，命令行执行时先找到了windows的find命令，而不是Cygwin的命令  
     解决方法： 再package.json，fetch-local脚本那里，先设置环境变量，再执行shell脚本,注意斜杠和引号  
               `PATH=\"D:/Program Files/cygwin/bin;%PATH%\" && bash ./tools/fetch-assets-local.sh`  
     参考资料： [Stackoverflow](https://stackoverflow.com/questions/3918341/find-parameter-format-not-correct)  
               [设置环境变量](https://www.91r.net/ask/4643117.html)  
