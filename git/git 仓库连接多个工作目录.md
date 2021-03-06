# 如何同时查看同一个 git 仓库两个分支（或更多）的代码

## git worktree

```bash
git worktree add -b <新分支名> <新路径> <从此分支创建>
```

## 场景

目录 project 是项目仓库。在此仓库的 A 分支上开发，布署。现在创建一个 B 分支，对代码进行优化和重构，过程中可能要经常对两个分支的代码进行比对和修改。这时可以使用 git worktree 创建一个新目录，相当于 clone 了一份项目代码。这样就可以同时在编辑器中打开A分支和B分支的代码了。

那和 clone 有什么不同？

不同处在于，两个文件夹其实共享了 .git 目录。所以它们的分支状态，文件改动是实时同步的。不需要你 pull。

## 删除 worktree

删除之前记得将修改合并到对应分支上

```shell
git worktree list

git worktree remove [path]
```