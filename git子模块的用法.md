# git仓库共享文件夹

> 场景：公司有两个代码仓库，一个是pc端前端代码，一个是mobile端的。这两个仓库中，有很多公共库及其相似，但是在两个仓库中各自持有。有没有办法将存放公共库的文件夹在两个仓库中共享，同时，对公共代码库的修改可以同步到两个仓库的文件夹中？使用git submodule 就可以实现。

## git submodule

> 建立一个公共代码库，存放共享代码。

```
// 假设仓库地址为 http://example.com/public-repo.git
mkdir public-repo
cd public-repo 
git init 
```



> 进入pc端代码仓库。

```
cd pc-project
git submodule add http://example.com/public-repo.git
git status 
// 将会显示刚刚添加的子模块里的文件
Your branch is up-to-date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   .gitmodules
	new file:   ...

git add . // 你需要将新文件的变化保存到当前项目中

git commit -m 'add submodule'

git push origin master // 将改变推送到远程仓库
```

> 当别人克隆这个项目时，子模块文件夹将被克隆下来，但是里面没有任何文件。你需要进入子模块文件夹，初始化并拉取代码。当子模块较多时，这样做费时费力，容易出错。所以我们一般使用第二种方法，拉取包含子模块的仓库时，传递一个额外的参数:—recursive,它会自动初始化并更新仓库中的每一个子模块。

```
// 第一种方法，仅做了解。
cd pc-project
cd public-repo
git submodule init
git submodule update
```



```
// 第二种方法，推荐使用。
git clone --recursive http://example.com/pc-project.git
```

> 如果公共代码库的代码修改了，如何同步到本地？

```
// 第一种方法：

cd pc-project
cd public-repo
git fetch
git merge

// 第二种方法

cd pc-project
git submodule update --remote public-repo

// 第三种方法
cd pc-project
git submodule update --remote // 更新所有子模块
```

> 子模块的更新和普通git仓库代码更新是一样的，你也可以切换分支，没有区别。

```
cd pc-project
cd public-repo
git checkout stable // 假设子模块有这样一个分支

// do something ...
```

> 如果你想在推送pc-project的时候，将子模块的改变也一起推送（而不是进入每个子模块单独进行推送），只需要加入参数。

```
git push --recurse-submodules=on-demand
// git 会进入子模块并推送子模块所作的提交，如果子模块推送失败，那么主项目也会推送失败。
```



在本篇文章中介绍了git子模块的核心用法，更加详细深入的了解，请参考：

git 子模块：(https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

