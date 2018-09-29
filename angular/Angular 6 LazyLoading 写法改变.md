# Angular 6 LazyLoading 变化

> 升级到 Angular6 以后，发现lazyloading 模块无法正常工作了。控制台报错：

```JS
core.js:1598 ERROR Error: Uncaught (in promise): Error: Cannot find module "app/optimization/optimization.module".
```

> 很奇怪，于是去官网看 issue，发现有变化，下面是 github 上显示的修改：

```JS
const routes: Routes = [
   {
     path: 'customers',
-    loadChildren: 'app/customers/customers.module#CustomersModule'
+    loadChildren: './customers/customers.module#CustomersModule'
   },
   {
     path: 'orders',
-    loadChildren: 'app/orders/orders.module#OrdersModule'
+    loadChildren: './orders/orders.module#OrdersModule'
   },
   {
     path: '',
```

> 原来 loadChildren 对应的路径发生了改变。需要将路径设置为与当前模块的相对路径。比如原来我的目录结构是这样的。

```JS
--src
  --app
    app.module.ts
    app-routing.module.ts
  --optimization
    -optimization.module.ts
    -optimization-routing.module.ts
```

> Angular5 中 AppRouting 的配置：

```JS
const routes: Routes = [
  {
      path: 'optimization',
      loadChildren: 'app/optimization/optimization.module#OptimizationModule'
  }
];
```

> 现在需要修改为：

```JS
const routes: Routes = [
  {
      path: 'optimization', 
   	  loadChildren: './optimization/optimization.module#OptimizationModule'
  }
];
```

> 也就是 optimization.module.ts 相对于 app-routing.module.ts 的路径。