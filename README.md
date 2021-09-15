# Git Skills
- [Git Skills](#git-skills)
  - [创建版本库](#创建版本库)
  - [时光机穿梭](#时光机穿梭)
      - [版本回退](#版本回退)
      - [工作区和暂存区](#工作区和暂存区)
      - [管理修改](#管理修改)
      - [撤销修改](#撤销修改)
        - [注意：](#注意)
      - [删除文件](#删除文件)
        - [注意：](#注意-1)
  - [远程仓库](#远程仓库)
      - [添加远程库](#添加远程库)
        - [删除远程库](#删除远程库)
      - [从远程库克隆](#从远程库克隆)
  - [分支管理](#分支管理)
      - [创建与合并分支](#创建与合并分支)
        - [小结](#小结)
      - [解决冲突](#解决冲突)
      - [分支管理策略](#分支管理策略)
        - [小结](#小结-1)
      - [BUG分支](#bug分支)
      - [Feature分支](#feature分支)
      - [多人协作](#多人协作)
        - [推送分支](#推送分支)
        - [抓取分支](#抓取分支)
      - [Rebase](#rebase)
        - [分支管理模型](#分支管理模型)
        - [更多](#更多)
  - [标签管理](#标签管理)
      - [创建标签](#创建标签)
      - [操作标签](#操作标签)
        - [更多：](#更多-1)
  - [自定义Git](#自定义git)
      - [忽略特殊文件](#忽略特殊文件)
      - [配置别名](#配置别名)
      - [搭建Git服务器](#搭建git服务器)
## 创建版本库

首先，选择一个合适的地方，创建一个空目录：

```shell
$ mkdir learn-git
$ cd learn-git
$ pwd
/home/yxs/learn-git
```

第二步，通过`git init`命令把这个目录变成Git可以管理的仓库：

```shell
$ git init
```

当前目录下多了一个`.git`的目录，这个目录是Git来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了。

把一个文件放到Git仓库只需要两步：

1. 添加文件：`git add <filename>`

2. 提交文件：`git commit -m “提交说明”`

**注意：**

1. Git命令必须在Git仓库目录内执行（`git init`除外），在仓库目录外执行是没有意义的。
2. 可以一次添加多个文件

## 时光机穿梭

`git status`命令可以让我们时刻掌握仓库当前的状态。

`git diff <filename>`这个命令查看文件做了什么修改。

#### 版本回退

用`git log`命令查看提交历史：

```shell
$ git log
$ git log --graph --pretty=oneline --abbrev-commit
```

一大串类似`1094adb...`的是`commit id`（版本号），和SVN不一样，Git的`commit id`不是1，2，3……递增的数字，而是一个SHA1计算出来的一个非常大的数字，用十六进制表示，而且你看到的`commit id`和我的肯定不一样，以你自己的为准。为什么`commit id`需要用这么一大串数字表示呢？因为Git是分布式的版本控制系统，后面我们还要研究多人在同一个版本库里工作，如果大家都用1，2，3……作为版本号，那肯定就冲突了。

每提交一个新版本，实际上Git就会把它们自动串成一条时间线。

在Git中，用`HEAD`表示当前版本，也就是最新的提交，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

把当前版本回退到上一个版本，就可以使用`git reset`命令：

```shell
$ git reset --hard HEAD^
```

这时候`log`中已经没有之前那个最新提交了，如果想要恢复，需要知道当时的`commit id`：

```shell
$ git reset --hard 1094a
```

版本号没必要写全，前几位就可以了，Git会自动去找。

Git的版本回退速度非常快，因为Git在内部有个指向当前版本的`HEAD`指针，当你回退版本的时候，Git仅仅是把HEAD从指向`append GPL`：

```asciiarmor
┌────┐
│HEAD│
└────┘
   │
   └──> ○ append GPL
        │
        ○ add distributed
        │
        ○ wrote a readme file
```

改为指向`add distributed`：

```asciiarmor
┌────┐
│HEAD│
└────┘
   │
   │    ○ append GPL
   │    │
   └──> ○ add distributed
        │
        ○ wrote a readme file
```

然后顺便把工作区的文件更新了。所以你让`HEAD`指向哪个版本号，你就把当前版本定位在哪。

现在，你回退到了某个版本，想恢复到新版本，找不到新版本的`commit id`怎么办？

Git提供了一个命令`git reflog`用来记录你的每一次命令：

```shell
$ git reflog
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
e475afc HEAD@{3}: commit: add distributed
eaadf4e HEAD@{4}: commit (initial): wrote a readme file
```

从输出可知，`append GPL`的commit id是`1094adb`。

#### 工作区和暂存区

工作区：就是你在电脑里能看到的目录

版本库：工作区里的`.git`目录，版本库里最重要的是被称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的第一个指针——`HEAD`：

![git-repo](https://www.liaoxuefeng.com/files/attachments/919020037470528/0)

`git add`命令实际上就是把要提交的所有修改放到暂存区（Stage），然后，执行`git commit`就可以一次性把暂存区的所有修改提交到分支。

一旦提交后，如果你又没有对工作区做任何修改，那么工作区就是“干净”的：

![git-stage-after-commit](https://www.liaoxuefeng.com/files/attachments/919020100829536/0)

#### 管理修改

Git跟踪并管理的是修改，而非文件。

当你用`git add`命令后，在工作区的第一次修改被放入暂存区，准备提交，但是，在工作区的第二次修改并没有放入暂存区，所以，`git commit`只负责把暂存区的修改提交了，也就是第一次的修改被提交了，第二次的修改不会被提交。

提交后，用`git diff HEAD -- readme.txt`命令可以查看工作区和版本库里面最新版本的区别。

#### 撤销修改

`git checkout -- file`可以**丢弃**工作区的修改：

```shell
$ git checkout -- readme.txt
```

该命令表示把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

1. `readme.txt`自修改后还没有被放到暂存区，现在撤销修改就回到和版本库一模一样的状态；
2. `readme.txt`已经添加到暂存区后，又做了修改，现在，撤销修改就**回到添加到暂存区后的状态**。

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

用命令`git reset HEAD <file>`可以把**暂存区的修改撤销掉**（unstage），**重新放回**工作区：

```shell
$ git reset HEAD readme.txt
```

`git reset`命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用`HEAD`时，表示最新的版本。

所以，如果文件添加到暂存区后又做了修改，想要清除暂存区和工作区的所有修改，可以：

```shell
# 回退版本，清除暂存区
$ git reset HEAD readme.txt
# 丢弃工作区的修改
$ git checkout -- readme.txt
```

如果提交了不合适的修改到版本库时，想要撤销本次提交，参考[版本回退]()一节，前提是没有推送到远程库。

##### 注意：

`git checkout -- file`命令中的`--`很重要，没有`--`，就变成了“切换到另一个分支”的命令。

#### 删除文件

在Git中，删除也是一个修改操作。

一般情况下，你通常直接在文件管理器中把没用的文件删了，或者用`rm`命令删了：

```shell
$ rm test.txt
```

这个时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，`git status`命令会立刻告诉你哪些文件被删除了。

这时有两个选择：

1. 确实要从版本库中删除该文件，那就用`git rm`删掉，再执行一次`git commit`；

   ```shell
   $ git rm test.txt$ git commit -m "remove test.txt"
   ```

   现在文件就从版本库中删除了。

2. 删错了，这时需要从版本库中把误删的文件恢复到最新的版本：

   ```shell
   $ git checkout -- test.txt
   ```

   `git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

   **从来没有被添加到版本库就被删除的文件，是无法恢复的！**

##### 注意：

命令`git rm`用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失**最近一次提交后你修改的内容**。

## 远程仓库

设置SSH加密连接Github仓库：

1. 创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

   ```shell
   $ ssh-keygen -t rsa -C "youremail@example.com"
   ```

   一路回车，使用默认值即可。如果一切顺利的话，可以在用户主目录里找到`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件，这两个就是SSH Key的秘钥对，`id_rsa`是私钥，不能泄露出去，`id_rsa.pub`是公钥，可以放心地告诉任何人。

2. 登陆GitHub，打开“Account settings”，“SSH Keys”页面，然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容，点“Add Key”，你就应该看到已经添加的Key。

**注意：**

GitHub允许你添加多个Key。假定你有若干电脑，你一会儿在公司提交，一会儿在家里提交，只要把每台电脑的Key都添加到GitHub，就可以在每台电脑上往GitHub推送了。

#### 添加远程库

先有本地库，将本地Git仓库与Github上的仓库远程同步：

1. 在Github新建一个仓库，尽量保持相同的仓库名

2. 根据GitHub的提示，在本地的`learn-git`仓库下运行命令：

   ```shell
   $ git remote add origin git@github.com:yangxinsheng523/learn-git.git
   ```

   添加后，远程库的名字就是`origin`，这是Git默认的叫法，也可以改成别的，但是`origin`这个名字一看就知道是远程库。

3. 把本地库的所有内容推送到远程库上:

   ```shell
   $ git push -u origin master
   ```

   把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支`master`推送到远程。

   由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

##### 删除远程库

如果添加的时候地址写错了，或者就是想删除远程库，可以用`git remote rm <name>`命令。使用前，建议先用`git remote -v`查看远程库信息：

```shell
$ git remote -v
origin	git@github.com:yangxinsheng523/learn-git.git (fetch)
origin	git@github.com:yangxinsheng523/learn-git.git (push)
```

然后，根据名字删除，比如删除`origin`：

```shell
$ git remote rm origin
```

此处的“删除”其实是解除了本地和远程的绑定关系，并不是物理上删除了远程库。远程库本身并没有任何改动。要真正删除远程库，需要登录到GitHub，在后台页面找到删除按钮再删除。

#### 从远程库克隆

远程库已经准备好，用命令`git clone`克隆一个本地库：

```shell
$ git clone git@github.com:yangxinsheng523/gitskills.git
```

默认的`git://`使用`SSH`，但也可以使用`https`等其他协议。

使用`https`除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放`https`端口的公司内部就无法使用`SSH`协议而只能用`https`。

## 分支管理

#### 创建与合并分支

```shell
$ git checkout -b dev
```

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：

```shell
$git branch devgit checkout dev
```

用`git branch`查看当前分支，当前分支前会标`*`号。
在`dev`分支上完成工作，切换回`master`分支，再把`dev`分支的工作成果合并到`master`分支上：

```shell
$ git merge dev
更新 5395a5c..2fa06b9Fast-forward GitSkills.md | 0
1 file changed, 0 insertions(+), 0 deletions(-) create mode 100644 GitSkills.md
```

`git merge`命令用于合并指定分支到当前分支。
`Fast-forward`表示这次合并是“快进模式”，就是直接把`master`指向`dev`的当前提交，所以合并速度非常快。
不是每次合并都能`Fast-forward`
合并完成后，就可以放心地删除`dev`分支了：

```shell
$ git branch -d dev
```

因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在`master`分支上工作效果是一样的，但过程更安全。

##### 小结

Git鼓励大量使用分支：

查看分支：`git branch`

查看远程分支 `git branch -r`

查看所有分支 `git branch -a`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`或者`git switch <name>`

创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`

删除远程分支 `git push 远程仓库名 :分支名`   比如 git push origin :main

#### 解决冲突

新建分支

```shell
$ git switch -c feature1
```

修改文件内容并提交

切回`master`分支，修改统一文件的内容，使之与之前修改产生冲突，进行提交。

合并分支到`master`:

```shell
$ git merge feature1
```

产生冲突，无法合并，修改后再提交，两个分支就会自动合并，最后删除`feature1`分支。

查看分支合并情况：

```shell
$ git log --graph --pretty=oneline --abbrev-commit
```

#### 分支管理策略

`Fast forward`模式，删除分支前，分支历史信息：

```shell
$ git log --graph --pretty=oneline --abbrev-commit
* a1cc25f (HEAD -> master, dev) dev update* b16bc29 wrote a readme file
```

删除分支后，分支历史信息：

```shell
$ git log --graph --pretty=oneline --abbrev-commit
* a1cc25f (HEAD -> master) dev update* b16bc29 wrote a readme file
```

强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

创建并切换`dev`分支：

```shell
$ git switch -c dev
```

修改文件并提交，切换回`master`：

```shell
$ git switch master
```

准备合并`dev`分支，请注意`--no-ff`参数，表示禁用`Fast forward`：

```shell
$ git merge --no-ff -m "merge with no-ff" dev
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
$ git switch master$ git switch -c issue-101Switched to a new branch 'issue-101'
```

修复bug提交后，切换到`master`分支，合并，删除`issue-101`分支：

```shell
$ git add readme.txt 
$ git commit -m "fix bug 101"
[issue-101 4c805e2] fix bug 101 1 file changed, 1 insertion(+), 1 deletion(-)
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
$ git branch* dev  master
$ git cherry-pick 4c805e2
[master 1d4b803] fix bug 101 1 file changed, 1 insertion(+), 1 deletion(-)
```

注意，使用`git cherry-pick`前，需要先用`git stash`保存现场。

```shell
$ git cherry-pick 4c805e2
error: 您的本地修改将被拣选覆盖。提示：提交您的修改或贮藏后再继续。fatal: 拣选失败
```

#### Feature分支

添加一个新功能时，你肯定不希望因为一些实验性质的代码，把主分支搞乱了，所以，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

`feature`分支还没有被合并，如果删除，将丢失掉修改，如果要强行删除，需要使用大写的`-D`参数。

#### 多人协作

查看远程库的信息，用`git remote`：

```shell
$ git remote$ git remote -v  # 显示详细信息
origin	git@github.com:yangxinsheng523/learn-git.git (fetch)
origin	git@github.com:yangxinsheng523/learn-git.git (push)
```

上面显示了可以抓取和推送的`origin`的地址。如果没有推送权限，就看不到push的地址。

##### 推送分支

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：

```shell
$ git push origin master
```

##### 抓取分支

在另一台电脑（注意要把SSH Key添加到GitHub）或者同一台电脑的另一个目录下克隆：

```shell
$ git clone git@github.com:yangxinsheng523/learn-git.git
```

当你的小伙伴从远程库clone时，默认情况下，你的小伙伴只能看到本地的`master`分支。

现在，你的小伙伴要在`dev`分支上开发，就必须创建远程`origin`的`dev`分支到本地，于是他用这个命令创建本地`dev`分支：

```shell
$ git checkout -b dev origin/dev
```

你的小伙伴已经向`origin/dev`分支推送了他的提交，而碰巧你也对同样的文件作了修改，并试图推送，

推送失败，因为你的小伙伴的最新提交和你试图推送的提交有冲突，解决办法也很简单，Git已经提示我们，先用`git pull`把最新的提交从`origin/dev`抓下来，然后，在本地合并，解决冲突，再推送，

```shell
$ git pull
当前分支没有跟踪信息。请指定您要合并哪一个分支。详见 git-pull(1)。
$ git pull <远程> <分支>
如果您想要为此分支创建跟踪信息，您可以执行：
$ git branch --set-upstream-to=origin/<分支> dev
```

`git pull`也失败了，原因是没有指定本地`dev`分支与远程`origin/dev`分支的链接，根据提示，设置`dev`和`origin/dev`的链接：

```shell
$ git branch --set-upstream-to=origin/dev dev
$ git pull
```

这回`git pull`成功，但是合并有冲突，需要手动解决，解决的方法和分支管理中的解决冲突完全一样。解决后，提交，再push即可。

因此，多人协作的工作模式通常是这样：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

#### Rebase

##### 分支管理模型

<img src="http://s3.51cto.com/wyfs02/M02/12/44/wKiom1MA0v-horoSAAS4v41ef_U068.jpg" alt="img" style="zoom:150%;" />



##### 更多

[Git-变基](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)

[廖雪峰-Rebase](https://www.liaoxuefeng.com/wiki/896043488029600/1216289527823648)

## 标签管理

发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。

Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针（跟分支很像对不对？但是分支可以移动，标签不能移动），所以，创建和删除标签都是瞬间完成的。

标签是指向commit的死指针，分支是指向commit的活指针

#### 创建标签

切换到下需要打标签的分支上：

```shell
$ git branch$ git switch master
```

打一个新标签`git tag <name>`:

```shell
$ git tag v1.0
```

查看所有标签：

```shell
$ git tag
```

默认标签是打在最新提交的commit上的，也可以根据commit id在需要的地方打标签：

```shell
$ git log --pretty=oneline --abbrev-commita6bbfbf 
new file477e41e day day day
```

如果要对`day day day`这次提交打标签：

```shell
$ git tag v0.9 477e41e
```

注意，标签不是按时间顺序列出，而是按字母排序的。可以用`git show <tagname>`查看标签信息.

创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字：

```shell
$ git tag -a v0.1 -m "version 0.1 released" 477e41e
```

 注意：标签总是和某个commit挂钩。如果这个commit既出现在`master`分支，又出现在`dev`分支，那么在这两个分支上都可以看到这个标签。

#### 操作标签

删除标签：

```shell
$ git tag -d v0.1
```

推送某个标签到远程，使用命令`git push origin <tagname>`：

```shell
$ git push origin v1.0
```

一次性推送所有未推送到远程的本地标签：

```shell
$ git push origin --tags
```

要删除远程标签，首先删除本地标签：

```shell
$ git tag -d v0.1
```

然后从远程删除：

```shell
$ git push origin :refs/tags/v0.9$ git push origin :v0.9
```

##### 更多：

[Git-打标签](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)

## 自定义Git

#### 忽略特殊文件

在Git工作区的根目录下创建一个特殊的`.gitignore`文件，然后把要忽略的文件名或正则表达式填进去，Git就会自动忽略这些文件。

忽略文件的原则是：

1. 忽略操作系统自动生成的文件，比如缩略图等；
2. 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的`.class`文件；
3. 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

[`gitignore`模板](https://github.com/github/gitignore)

强制添加：`git add -f`

把指定文件排除在`.gitignore`规则外的写法就是`!`+文件名

**注意：**

1. `.gitignore`文件本身要放到版本库里，并且可以对`.gitignore`做版本管理

2. `.gitignore`只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.`gitignore`是无效的。那么解决方法就是先把本地缓存删除（改变成未track状态），然后再提交：

   ```shell
   $ git rm -r --cached .$ git add .$ git commit -m 'update .gitignore'
   ```


#### 配置别名

告诉Git，以后`st`就表示`status`：

```shell
$ git config --global alias.st status
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
```

`--global`参数是全局参数，也就是这些命令在这台电脑的所有Git仓库下都有用。

把暂存区的修改撤销掉：

```shell
$ git config --global alias.unstage 'reset HEAD'
$ git unstage test.py
$ git reset HEAD test.py
```

配置Git的时候，加上`--global`是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用。

**注意：**

1. 每个仓库的Git配置文件都放在`.git/config`文件中，
2. 当前用户的Git配置文件放在用户主目录下的一个隐藏文件`.gitconfig`中。

#### 搭建Git服务器

[廖雪峰-搭建Git服务器](https://www.liaoxuefeng.com/wiki/896043488029600/899998870925664)

