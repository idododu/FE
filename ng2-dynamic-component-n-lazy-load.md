# ng2 懒加载与动态编译
---

### 1. 约定
- *AppModule* 为根模块，同理 *AppRouting* 为根模块路由配置
- *FmModule* 为功能模块（也叫特性模块），*FmRouting*是其对应的路由配置
- 除特殊指明，示例都是使用 *angular-cli*（简称 *ngc*）构建
- 动态编译需要一个宿主组件（或者叫容器）
- 示例代码有删节

### 2. 结合webpack和ngc的懒加载路由配置
1. `ng new PROJECT_NAME --routing` 创建带有路由配置的ng项目
2. 进入工程目录
3. `ng generate module MODULE_NAME --routing` 创建带有路由配置的功能模块
4. `ng generate component [path]COMPONENT_NAME` 创建组件，path以 *app* 目录为始点

##### 有项目结构如下（忽略测试文件和配置文件）:

    |-- e2e/
    |-- src/
        |-- app/                                应用根目录 `ng generate`中路径的始点
            |-- app.module.ts                   根模块
            |-- app-routing.module.ts           根模块路由配置
            |-- app.component.ts                主视图（也叫根组件），设置在AppModule的bootstrap中
            |-- fm/
                |-- fm.module.ts                功能模块
                |-- fm-routing.module.ts        功能模块路由配置
                |-- list/
                    |-- list.component.ts       功能模块下的视图（组件）
        |-- assets/
        |-- index.html
        |-- main.ts
        |-- ···

##### *AppModule* 和 *AppRouting* 的配置:
未修改前的 *app-routing.module.ts* （删除诸如 `import` 的代码）：
```ts
const routes: Routes = [
  {
    path: '',
    children: []
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
```
注意到，在根模块中的 `imports` 属性使用的是 `RouterModule.forRoot()` 方法。
现在添加懒加载模块 *fm* ,在 *AppRouting* 的 `routes` 数组添加代码如下：
```ts
{
    path: 'fm',
    loadChildren: 'app/fm/fm.module#FmModule'  // path_to_module#ModuleName
}
```
##### *FmRouting* 的配置:
在功能模块的路由配置 *FmRouting* 中，需要注意 `imports` 属性使用的是 `RouterModule.forChild()` 方法,有配置如下：
```ts
const routes: Routes = [
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
```
为了展示效果，可在 *FmRouting* 的 `routes` 中添加视图配置：
```ts
{
    path: 'list',
    component: ListComponent
}
```
##### 小结：
至此，可知 ng2 的懒加载策略是在功能模块的路由配置中设置 `loadChildren` 属性（非 `children` ）。其背后的原理大致是通过 *ngtools* (webpack loader) 收集包含 `loadChildren` 属性的路由配置生成模块文件路径和模块的对照对照表（`path_to_module#ModuleName`），而后交由webpack做代码分割。

文档：惰性加载路由配置 ([zh](https://angular.cn/guide/router#里程碑6：异步路由) / [en](https://angular.io/guide/router#milestone-6-asynchronous-routing))

### 3. 运行时动态创建组件
1. 创建一个template字符串tpl
2. 使用tpl作为属性创建一个组件tmpCmp
3. 创建一个NgModule模块tmpModule并将tmpCmp声明为其组件
4. 编译tmpModule以及其组件
5. 获得组件的componentFactory,创建组件实例并获得引用cmpRef
6. 将cmpRef和视图容器绑定

```ts
// 获取容器引用
@ViewChild('dynamic', {read: ViewContainerRef}) vc: ViewContainerRef;
constructor(private _compiler: Compiler,
          private _injector: Injector,
          private _m: NgModuleRef<any>) { }
ngAfterViewInit() {
    const tpl = '<span>运行时动态生成组件: {{section}}</span>';
    
    // 定义一个组件类以及其template属性，并使用装饰器函数装饰
    const tmpCmp = Component({template: tpl})(class {
    });
    
    // 定义一个NgModule类以及其declarations属性，并使用装饰器函数装饰
    const tmpModule = NgModule({declarations: [tmpCmp]})(class {
    });
    
    // 编译模块 tmpModule
    this._compiler.compileModuleAndAllComponentsAsync(tmpModule)
      .then((factories) => {
        const f = factories.componentFactories[0];
        // 注入依赖并获得引用
        const cmpRef = f.create(this._injector, [], null, this._m);
        cmpRef.instance.section = '动态内容';
        // 将组件和视图容器绑定
        this.vc.insert(cmpRef.hostView);
      });
}
```

### 4. 使用 SystemJS 的动态模块加载与动态编译
SystemJs是一个可以在浏览器端动态加载ES模块和npm库的模块加载工具，ng2 为其编写了相应的服务。使得使用 SystemJS 可以很方便的动态加载 ng2 模块并进行动态编译。

```ts
import {AComponent} from './a.component';

@Component({
  // 依赖注入 NgModuleFactoryLoader 和 SystemJsNgModuleLoader 两个服务
  providers: [
    {
      provide: NgModuleFactoryLoader,
      useClass: SystemJsNgModuleLoader
    }
  ]
})
export class ModuleLoaderComponent {
  constructor(private _injector: Injector,
              private loader: NgModuleFactoryLoader) {
  }

  ngAfterViewInit() {
    // 加载 app/t.module
    this.loader.load('app/t.module#TModule').then((factory) => {
      const module = factory.create(this._injector);
      const r = module.componentFactoryResolver;
      const cmpFactory = r.resolveComponentFactory(AComponent);
      
      // 创建组件并与视图绑定
      const componentRef = cmpFactory.create(this._injector);
      this.container.insert(componentRef.hostView);
    })
  }
}
```
SystemJS作为一个模块加载工具并不合并压缩源码（但可以借助gulp或者build loader实现），允许在运行时加载任意模块，其模块加载路径和文件路径一致。即使进行了预编译处理（AOT）和 Tree-shaking， AOT时在AppComponent根组件引入@angular/compiler并注入，也能正确动态加载和动态编译。

### 5. 使用 webpack 的动态模块加载与动态编译
webpack 在其github上的简介有这样一句 **"webpack is a module bundler."** ，所以webpack并不是一个模块加载库，但有其自己的模块加载机制。所以，ng 提供了webpack插件以匹配ng2中的打包规则和实现其异步加载功能。

其中第2节中的 `loadChildren: 'path_to_module#ModuleName'` 就是ng2中配合webpack使用的配置规则。

另外有：
```ts
// app.component.ts 宿主组件
@ViewChild('container', {read: ViewContainerRef}) container: ViewContainerRef;

constructor(private loader: SystemJsNgModuleLoader, private inj: Injector) {}

ngOnInit() {
    this.loader.load('app/lazy/lazy.module#LazyModule')
    .then((factory: NgModuleFactory<any>) => {
        const cmp = (<any>factory.moduleType).entry;
        const moduleRef = factory.create(this.inj);

        const compFactory = moduleRef.componentFactoryResolver.resolveComponentFactory(cmp);
        this.container.createComponent(compFactory);
    });
}

// lazy.module.ts 懒加载组件
@Component({
    ...
})
export class LazyComponent {
    ...
}

@NgModule({
    imports: [CommonModule],
    declarations: [LazyComponent],
    entryComponents: [LazyComponent]
})
export class LazyModule {
    // 申明入口组件并导出
    static entry = LazyComponent;
}
```
需要注意的是，因为需要配合打包规则，需要在 *app.module.ts* 中 `providers` 配置如下：
```ts
import { ..., SystemJsNgModuleLoader } from '@angular/core';

import { AppComponent } from './app.component';
import { provideRoutes } from '@angular/router';

@NgModule({
  declarations: [
    ...
  ],
  imports: [
    ...
  ],
  providers: [
      SystemJsNgModuleLoader,
      provideRoutes([
          // 依旧是 loadChildren: 'path_to_module#ModuleName'，但是不需要配置path了
          { loadChildren: 'app/lazy/lazy.module#LazyModule' }
      ])
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
### 6. 参考
- 惰性加载路由配置 ( [zh](https://angular.cn/guide/router#里程碑6：异步路由) / [en](https://angular.io/guide/router#milestone-6-asynchronous-routing) )
- AoT 预编译 ( [zh](https://angular.cn/guide/aot-compiler) / [en](https://angular.io/guide/aot-compiler) )
- Here is what you need to know about dynamic components in Angular ( [blog](https://blog.angularindepth.com/here-is-what-you-need-to-know-about-dynamic-components-in-angular-ac1e96167f9e) / [git](https://github.com/maximusk/Here-is-what-you-need-to-know-about-dynamic-components-in-Angular) / [A&Q](https://medium.com/@maximus.koretskyi/systemjs-is-a-shim-for-upcoming-es6-modules-api-5e4efad22383) )
- webpack + ngc 的动态懒加载 ( [stackoverflow](https://stackoverflow.com/questions/40293240/how-to-manually-lazy-load-a-module) / [git](https://github.com/alexzuza/angular-cli-lazy) )
- Exploring Angular DOM manipulation techniques using ViewContainerRef ( [blog](https://blog.angularindepth.com/exploring-angular-dom-abstractions-80b3ebcfc02) )