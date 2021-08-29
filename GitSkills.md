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

`Fast forward`模式，删除分支前，分支历史信息：

```shell
$ git log --graph --pretty=oneline --abbrev-commit
* a1cc25f (HEAD -> master, dev) dev update
* b16bc29 wrote a readme file
```

删除分支后，分支历史信息：

```shell
$ git log --graph --pretty=oneline --abbrev-commit
* a1cc25f (HEAD -> master) dev update
* b16bc29 wrote a readme file
```

强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

创建并切换`dev`分支：

```shell
git switch -c dev
```

修改文件并提交，切换回`master`：

```shell
git switch master
```

准备合并`dev`分支，请注意`--no-ff`参数，表示禁用`Fast forward`：

```shell
git merge --no-ff -m "merge with no-ff" dev
```

因为本次合并要创建一个新的commit，所以加上`-m`参数，把commit描述写进去。

合并后，我们用`git log`看看删除分支前的分支历史：

```shell
$ git log --graph --pretty=oneline --abbrev-commit
*   907026a (HEAD -> master) no-ff-merge
|\  
| * ae71770 (dev) dev update
|/  
* b16bc29 wrote a readme file
```

删除分支后的分支历史：

```shell
$ git log --graph --pretty=oneline --abbrev-commit
*   907026a (HEAD -> master) no-ff-merge
|\  
| * ae71770 dev update
|/  
* b16bc29 wrote a readme file
```

##### 小结

合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。

#### BUG分支

工作只进行到一半，**没法提交**，预计完成还需1天时间。但是，必须在两个小时内修复该bug，怎么办？

Git提供一个`stash`功能，可以把当前工作现场**“储藏”**起来，等以后恢复现场后继续工作，注意要先`git add`：

```shell
$ git stash
```

现在，用`git status`查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。

首先确定要在哪个分支上修复bug，假定需要在`master`分支上修复，就从`master`创建临时分支：

```shell
$ git switch master
$ git switch -c issue-101
Switched to a new branch 'issue-101'
```

修复bug提交后，切换到`master`分支，合并，删除`issue-101`分支：

```shell
$ git add readme.txt 
$ git commit -m "fix bug 101"
[issue-101 4c805e2] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git switch master
$ git merge --no-ff -m "merged bug fix 101" issue-101
```

恢复`dev`工作区状态：

1. 使用`git stash apply`恢复后，stash内容并不删除，需要用`git stash drop`来删除。

   多次stash，恢复的时候可以先用`git stash list`查看，然后恢复指定的stash：

   ```shell
   $ git stash apply stash@{0}
   ```

2. 使用`git stash pop`，恢复的同时也把stash内容删了

如果`dev`上有从`master`复制过来的bug，可以把`[issue-101 4c805e2] fix bug 101`这个提交“**复制**”到`dev`上：

```shell
$ git branch
* dev
  master
$ git cherry-pick 4c805e2
[master 1d4b803] fix bug 101
 1 file changed, 1 insertion(+), 1 deletion(-)
```

注意，使用`git cherry-pick`前，需要先用`git stash`保存现场。

```shell
$ g cherry-pick 4c805e2
error: 您的本地修改将被拣选覆盖。
提示：提交您的修改或贮藏后再继续。
fatal: 拣选失败
```

#### Feature分支

添加一个新功能时，你肯定不希望因为一些实验性质的代码，把主分支搞乱了，所以，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

`feature`分支还没有被合并，如果删除，将丢失掉修改，如果要强行删除，需要使用大写的`-D`参数。

#### 多人协作

#### Rebase

