# 使用 pm2 管理你的node应用

> 上一篇文章介绍了如何使用 nginx 来代理你的node应用。在实际项目中，不建议直接使用 node index.js 这样的方式来启动服务。推荐使用 pm2。

```shell
npm install -g pm2
```

> 注意，有的同学在安装全局包的时候，会遇到权限问题。这时候不要使用sudo来安装，会引发很多问题。一种办法是改变全局包所在目录的权限。一种是将npm全局安装目录设置为当前用户目录的 node_modules 。我选择的是后者。如果没有遇到问题，请忽略下面步骤。

```shell
npm config set prefix ~/node_modules
```

> 查看是否设置成功

```shell
npm config get prefix 

// /home/your_user_name/node_modules
```

> 在全局安装后，如果发现命令行敲 pm2 ，提示命令不存在，那说明没有把npm全局包的bin目录加入到PATH 中。如果已经正常输出 pm2 介绍，请忽略下面步骤。

```shell
vi ~/.bash_profile
```

> 在最后加入下面这行。

```shell
PATH=$PATH:~/node_modules/bin  
```

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