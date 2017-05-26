# Angular Animate 使用注意事项

标签（空格分隔）： 未分类

---

Angular 提供了ngAnimate模块，在项目中引入该模块依赖后，ng会在内置指令（ng-hide, ng-if, ng-repeat ...）中为元素增加对应的css动画钩子类，触发自定义的css动画。

## 使用示例
例如我们想要在元素隐藏显示时增加一个过渡动画效果可以使用以下代码：
```css
.box {
    transition: all linear 0.5s;
    background-color: lightblue;
    height: 50px;
	margin-bottom: 10px;
}
.box.ng-hide {
    height: 0;
}
```
当`ng-show`或`ng-hide`表达式结果为false时，元素会增加`ng-hide`css类,box高度从50px减小到0，并伴随平滑过渡效果；当表达式结果为true时，元素会移除`ng-hide`css类, box高度从0增加到50px

ng-repeat中在增加或者删除元素时也有对应的动画效果,不过是另外的`ng-enter`,`ng-leave`模式
```css
.repeat-item {
    -webkit-transition:0.5s linear all;
    transition:0.5s linear all;
}
/* 元素被追加到DOM中时，ng会追加ng-enter类到元素上 */
.repeat-item.ng-enter { height: 0; }
/* ng-enter中动画效果执行后，ng向元素追加ng-enter-active类 */
.repeat-item.ng-enter-active { height: 50px } 
/* 元素将要从DOM中移除时，ng追加ng-leave类到元素上 */
.repeat-item.ng-leave { opacity: 0 }
/* ng-leave中动画效果执行后，ng向元素追加ng-leave-active类 */
.repeat-item.ng-leave-active { }
/* ng-leave-active中动画效果执行完成后，ng将元素从DOM中移除 */
```

## 注意事项
动画效果只应当在需要的时候加，否则当有动画的元素很多时，会严重影响性能。
我们在实际项目就遇到这样的问题：
一个页面上拥有4个页签，页签时repeat出来的
每个页签下拥有一个50行*20列的表格， 列与行都是repeat出来的
在这种情况下，切换页签时非常卡，需要3-5秒钟才能完成页签切换的功能。
在chrome面板中分析得出ngEnter方法存在性能瓶颈，于是想办法移除动画效果。

只在指定元素中执行动画效果可以使用`$animateProvider`的`classNameFilter`方法，示例代码如下：
```javascript
var myApp = angular.module("MyApp", ["ngAnimate"]);
myApp.config(function($animateProvider) {
    // /angular-animate/为正则表达式
    $animateProvider.classNameFilter(/angular-animate/);
})
```

## 参考资料
- http://docs.ngnice.com/api/ngAnimate 
- http://www.runoob.com/angularjs/angularjs-animations.html 
- http://www.cnblogs.com/linchen1987/p/3707239.html?utm_source=tuicool&utm_medium=referral 
- http://www.cnblogs.com/leosx/p/4054818.html 
- https://stackoverflow.com/questions/21249441/disable-nganimate-for-some-elements 
