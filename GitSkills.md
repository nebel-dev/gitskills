## 分支管理

#### 创建与合并分支

```shell
git checkout -b dev
```

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：

```shell
git branch dev
git checkout dev
```

用`git branch`查看当前分支，当前分支前会标`*`号。
在`dev`分支上完成工作，切换回`master`分支，再把`dev`分支的工作成果合并到`master`分支上：

```shell
$ git merge dev
更新 5395a5c..2fa06b9
Fast-forward
 GitSkills.md | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 GitSkills.md
```

`git merge`命令用于合并指定分支到当前分支。
`Fast-forward`表示这次合并是“快进模式”，就是直接把`master`指向`dev`的当前提交，所以合并速度非常快。
不是每次合并都能`Fast-forward`
合并完成后，就可以放心地删除`dev`分支了：

```shell
git branch -d dev
```

因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在`master`分支上工作效果是一样的，但过程更安全。

##### 小结

Git鼓励大量使用分支：

查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`或者`git switch <name>`

创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`

#### 解决冲突

新建分支

```shell
git switch -c feature1
```

修改文件内容并提交

切回`master`分支，修改统一文件的内容，使之与之前修改产生冲突，进行提交。

合并分支到`master`:

```shell
git merge feature1
```

产生冲突，无法合并，修改后再提交，两个分支就会自动合并，最后删除`feature1`分支。

查看分支合并情况：

```shell
git log --graph --pretty=oneline --abbrev-commit
```

#### 分支管理策略





