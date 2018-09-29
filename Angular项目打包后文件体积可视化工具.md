# Angular应用打包后查看类库体积

> 在打包angular应用后，发现体积较大，但是又不知道是哪个类库导致的。本文介绍一种简单有效的方式，让你一目了然的看到各类库文件在打包文件中的体积。





> 首先，安装webpack的一个工具：

```
npm install webpack-bundle-analyzer --save-dev
```

> 配置package.json 脚本, 假设你的打包目录是 dist ，生成的 stats.json 会放置在打包目录下。

```js
"scripts":{
    "size":"webpack-bundle-analyzer dist/stats.json"
}
```

> 打包项目

```js
ng build --prod --stats-json
```

> 查看效果

```
npm run size
```

![屏幕快照 2018-04-25 下午3.17.58](/Users/yang/Desktop/docs/屏幕快照 2018-04-25 下午3.17.58.png)