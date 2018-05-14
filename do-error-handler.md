# 前端错误日志捕捉及上报
## 前端错误类型
- Javascript 语法错误: 
如缺少括号，注释未结束，字符串未结束等。
此类型错误现在一般在开发时就已经有IDE及打包脚本处理解决
- Javascript 运行时错误
如变量未定义，数组下标越界，内存溢出，对象不支持该属性或方法等
- 资源加载错误
`<script>`, `<img>`, '<iframe>'等资源标签加载出错， 常见404， Access is denied, 500, 502等错误
该种类型错误一般通过标签自己的onerror处理函数来处理
- MVVM框架错误
流行的MVVM框架如Angular, Vue, React都封装了自己的错误处理机制，需要遵循其处理方式来进行异常处理

## 前端错误日志捕捉及上报思路
1. try, catch, and finally
	在原生JS中我们可以通过`try`, `catch`, `finally`以及`throw`来捕捉前端异常，并进行相应处理
	```javascript
	try { 
	    // a is not defined
	    alert(a)
	    // 自定义错误
	    throw '自定义错误信息'
	    // 自定义错误对象
	    throw new Error('自定义错误对象')
	}
	catch(err) {
	    if (typeof err === 'string') {
	    	console.log(err)
	    } else {
	    	console.log(err.message); // a is not defined
	    }
	}
	```

2. 使用window.onerror处理未try/catch的错误
	```javascript
	const orgError = window.onerror;
	window.onerror = (msg, url, line, col, error) => {
	  // deal with error here

	  // call original error handler
	  if (orgError) {
	    orgError.apply(window, [msg, url, line, col, error]);
	  }
	};
	```
3. MVVM框架中的error处理方式
	- Angular 2+
		```typescript
		class MyErrorHandler implements ErrorHandler {
		  handleError(error) {
		    // recover your app gracefully or report it to an error monitoring service
		  }
		}

		@NgModule({
		  providers: [{provide: ErrorHandler, useClass: MyErrorHandler}]
		})
		class MyModule {}
		```
	- Vue 2.2.0+
		```javascript
		Vue.config.errorHandler = function (err, vm, info) {
		  // recover your app gracefully or report it to an error monitoring service
		  // `info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
		  // 只在 2.2.0+ 可用
		}
		```
4. 跨域Script Error
	跨域引用的脚本报错之后，window.onerror是无法捕获异常信息的，所以统一返回Script error。我们可以通过微script标签配置 crossorigin="anonymous" 并且在服务器添加Access-Control-Allow-Origin来解决。
	```html
	<script src="http://cdn.xxx.com/index.js" crossorigin="anonymous" />
	```

## 干货：DoErrorHandler(埋坑待填)


## 参考资料
- [MDN Javascript Error Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error)
- [MDN Guide - Exception handling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling#Exception_handling_statements)
- [MDN - Global Event Handlers - onerror](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onerror)
- [MSDN - Error Object(Javascript)](https://docs.microsoft.com/en-us/scripting/javascript/reference/error-object-javascript)
- [MSDN - Javascript Run-tme Errors](https://docs.microsoft.com/en-us/scripting/javascript/reference/javascript-run-time-errors)
- [MSDN - Javascript Syntx Errors](https://docs.microsoft.com/en-us/scripting/javascript/reference/javascript-syntax-errors)
- [Sentry - Capture and report Javascript errors with window.onerror](https://blog.sentry.io/2016/01/04/client-javascript-reporting-window-onerror)
- [Loggly - Angular 2+ exception handling made simple with logging](https://www.loggly.com/blog/angular-exception-logging-made-simple/)
- [Dzone - Custom error handling for angular](https://dzone.com/articles/custom-error-handling-for-angular)
- [Github - Error Inspector](https://github.com/famanoder/ErrorInspector)
- [前端代码异常监控](http://rapheal.sinaapp.com/2014/11/06/javascript-error-monitor/)
- [Sentry异常监控方案部署](https://segmentfault.com/a/1190000014496409)
- [fundebug](https://www.fundebug.com/)
- [谈谈前端异常捕获与上报](https://segmentfault.com/a/1190000013983109)
- [前端魔法堂——异常不仅仅是try/catch](https://segmentfault.com/a/1190000011602203)
- [Bad JS Report](https://github.com/BetterJS/badjs-report)
- [TraceKit](https://github.com/csnover/TraceKit)
- [Improve Javascript error reporting with tracekit](https://zetafleet.com/blog/2010/06/improve-javascript-error-reporting-with-tracekit.html)