# 使用 pm2 管理你的node应用

> 上一篇文章介绍了如何使用 nginx 来代理你的node应用。在实际项目中，不建议直接使用 node index.js 这样的方式来启动服务。推荐使用 pm2。

```shell
npm install -g pm2
```

> 有些同学在全局安装后，发现命令行敲 pm2 ，命令不存在。那是因为没有把npm全局包的bin目录加入到PATH 中。

```shell
vi ~/.bash_profile
```

>在最后加入下面这行

```shell
PATH=$PATH:$(npm config get prefix)/bin
```

> npm config get prefix 是获取你全局包安装的目录，它下面的bin目录存放的就是可执行文件。
>
> 最后，还要刷新文件。

```shell
source ~/.bash_profile
```

> 再次运行 pm2

```shell
$ pm2 -v
2.10.3
```

> 先停掉正在运行的 nodejs 应用。 可能有的同学上次连接了服务器，开启node应用后，就从服务器登出了。这次连接后，不知道怎么停掉服务器。如果不先停掉运行着的node应用，再次开启就会报错，因为端口已经被占用了。可以通过命令行来查看并杀死进程。

```shell
$ netstat -lnp|grep 8080
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6  	0  	0	 :::8080     :::*      LISTEN      9750/node /home/user 
```

> 可以看到端口被 node 占用了，pid 是 9750，杀掉这个进程就可以了。

```shell
kill 9750
```

> 使用pm2运行你的 node 应用

```shell
pm2 start path/to/yourserver/app.js --name app1
```

> 使用pm2 查看你的应用列表

```shell
pm2 list 
```

> 停止你的应用，参数为应用名称，也就是 —name 传入的参数

```shell
pm2 stop app1
```

> 重启你的应用

```shell
pm2 restart app1
```

> 查看日志

```shell
pm2 logs
```

> 查看资源占用

```shell
pm2 monit
```

> 查看所有应用的列表

```shell
pm2 list
```



> pm2 还有监视模式，当服务器文件发生改变时，自动重启。更多详细信息请查看pm2官方文档。