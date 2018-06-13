# Angular 创建新项目时报错

最近使用 ng new 创建新项目时报错了：

```shell
npm WARN deprecated istanbul-lib-hook@1.2.1: 1.2.0 should have been a major version bump
npm ERR! Unexpected end of JSON input while parsing near '...b9cf5a53d28616f911894'

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/yang/.npm/_logs/2018-06-13T01_34_37_009Z-debug.log
Package install failed, see above.
```

解决办法：

```shell
npm cache clean --force
```

然后重新运行 ng new 即可。