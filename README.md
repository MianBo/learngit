# Git 入门
基于[廖雪峰入门教程](https://liaoxuefeng.com/books/git/introduction/index.html)。
## 0.SSH秘钥
每一对SSH秘钥对应着一台主机，因此想要和github之间完成推送和克隆，就需要每台主机都通过git创建一个ssh并关联github。
## 1.创建目录
```
mkdir learngit // 创建文件夹，名为learngit
cd learngit    // 进入该文件夹
pwd            // 显示文件夹路径
```
## 2.仓库初始化
```
git init  // 将该文件夹改造为Git仓库
```
## 3.添加文件
```
git add test.txt                    // 将该文件夹下文件添加进仓库的暂存区，可以添加多个文件
git commit -m "wrote a reademe file"  // 将暂存区内的全部文件推送给仓库并备注
```
## 4.时光机穿梭
更改readme内容，并用`git status`观察状态，命令输出会告诉我们，test.txt被修改过了，但还没有提交修改。想要看这次修改具体更改了哪些内容，可以用`git diff`命令。

随后，可以继续通过`git add`和`git commit`完成新一轮修改后的提交。
### 4.1 版本回退
快照在Git中被称为commit，一旦把文件改乱了，或者误删了文件，还可以从最近的一个commit恢复，然后继续工作。

多次更改test.txt文档后，我们需要查看前几次都改了什么内容，这时候可以用`git log`命令，若是嫌输出信息琐碎，可以用`git log --pretty=oneline`命令。

在Git中，用`HEAD`表示当前版本，也就是最新的提交，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个\^比较容易数不过来，所以写成`HEAD~100`。

现在，我们要把当前版本回退到上一个版本:
```
git reset --hard HEAD^ 
```
这里，--hard会回退到上个版本的已提交状态，--soft会回退到上个版本的未提交状态，--mixed会回退到上个版本已添加但未提交的状态。

这样做确实可以回退到上个版本，但是新版本就没了！解决方法是在命令行窗口找到最新版本号的commit id，于是就可以指定回到未来的某个版本：
```
git reset --hard 1094a
```
版本号写前几位就可以了，Git会自动去找。

但是要是命令行窗口关了，咋办？这时候就可以用`git reflog`命令去回溯查找之前用过的命令，进而找到commit id。
### 4.2 工作区和暂存区
Git分为工作区和隐藏的.git文件夹，后者即版本库，版本库内存着暂存区和第一个分支`master`，指向master的一个指针名为`HEAD`。

即，需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。
### 4.3 管理修改
Git管理的是修改而并非文件：

第一次修改 -> git add -> 第二次修改 -> git commit (×)
第一次修改 -> git add -> 第二次修改 -> git add -> git commit (√)

每次修改，如果不用`git add`到暂存区，那就不会加入到`commit`中。
### 4.4 撤销修改
丢弃工作区的修改（还没add）：
```
cat test.txt // 查看文件内容
git checkout -- test.txt // 将文件在工作区的修改全部撤销，让这个文件回到最近一次git commit或git add时的状态
```
但是如果已经`git add`到暂存区了:
```
git reset HEAD test.txt // 撤销暂存区的修改
```
小结：
- 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。
- 场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD <file>，就回到了场景1，第二步按场景1操作。
- 场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。
### 4.5 删除文件
通常用`rm test.txt`命令就能删除文件，但此时工作区和版本库不一致，用`git status`会报提醒，所以：
```
git rm test.txt                 // 删除版本库中文件
git commit -m "remove test.txt" 
```
要是发现删错文件了：
```
git checkout -- test.txt // 用版本库的版本替换工作区的版本，从而实现还原
```
命令`git rm`用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失**最近一次提交后你修改的内容**。
## 5.远程仓库
### 5.1 添加远程库
现在的情景是，你已经在本地创建了一个Git仓库后，又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步，这样，GitHub上的仓库既可以作为备份，又可以让其他人通过该仓库来协作。

首先，在github上新建一个repo。

注意，因为github上默认的分支是`main`，而本地分支默认是`master`，如果直接按上述命令操作会在github的库中多出一个`master`分支，解决方案如下：
```
git branch    // 查看本地库当前分支
git branch -a // 查看所有分支

git branch -m master main // 重命名当前分支

git branch -d master // 删除本地master分支

git pull origin main --allow-unrelated-histories // 因为建库时不小心添加了readme文档，因此这里需要保证远程库和本地库的统一连贯性，即合并分支
```
然后，根据SSH去关联该仓库。
```
git remote add origin git@github.com:MianBo/learngit.git
```
此时，远程库的名字就是origin。然后，利用`git push`命令，将本地的main分支全部推送到Github的origin远程库中。
```
git push -u origin main
```
由于远程库是空的，第一次推送main分支时加上了-u参数，Git不但会把本地的main分支内容推送的远程新的main分支，还会把本地的main分支和远程的main分支关联起来，在以后的推送或者拉取时就可以简化命令。

从现在起，只要本地作了提交，就可以通过命令：
```
git push origin main
```
把本地main分支的最新修改推送至GitHub，现在，你就拥有了真正的分布式版本库。

当然如果想删除远程库，则：
```
git remote -v        // 查看远程库信息
git remote rm origin // 删除本地和远程库间的绑定关系
```
### 5.2 从远程库克隆
克隆库很简单：
```
git clone git@github.com:MianBo/learngit.git
```
显然，要克隆一个仓库，首先必须知道仓库的地址，然后使用git clone命令克隆。Git支持多种协议，包括https，但ssh协议速度最快。
## 6.分支管理
### 6.1 创建与合并分支
最开始时，`master`分支是一条线，git用`master`指向最新的提交，再用`HEAD`指向`master`，就能确定当前分支及其提交点。每次提交，`master`分支都会向前移动一步，线就会越来越长。

创建新的分支如`dev`时，Git会新建一个指针`dev`指向`master`相同的提交，并将`HEAD`指向`dev`，表示当前分支在`dev`上。
```
git branch dev   // 创建dev分支
git switch dev   // 切换到dev分支上
git branch       // 查看当前分支
```
此时，每新提交一次，`dev`指针往前移动一次，`master`指针不变。
```
git add test.txt          // 添加一行内容并add
git commit -m "branch test"

git switch master   // 切换到master分支上
cat test.txt      // 刚才添加的内容没了，因为上次提交是在dev分支上的
```
当`dev`上工作完成了，就可以让`master`指向`dev`的当前提交，这就完成了合并，还可以顺手删掉`dev`分支，也就是删掉`dev`指针:
```
git merge dev       // 将dev分支的工作成果合并到master分支上，此时test.txt内容是最新版本，也就是添加一行内容之后的
cat test.txt

git branch -d dev   // dev分支的任务已完成，删除dev分支

git branch          // 此时只剩下master分支
```
### 6.2 解决冲突 
有这么一个场景：我新建了一个`future`分支，并提交了新的修改后的test.txt，然后我切换到`master`分支，也提交了一个新的修改后的test.txt，那这两次提交就产生冲突了，强行`merge`是会报错的。`git status`也可以告诉我们冲突的文件。

解决方法就是手动在工作区把test.txt文档改了，然后再add和commit。
### 6.3 分支管理策略
通常，合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息：
```
git switch -c dev // 创建并切换到分支dev

// 修改test.txt
git add test.txt 
git commit -m "add merge"

git switch master // 切换到分支master
git merge --no-ff -m "merge with no-ff" dev // --no-ff参数，表示禁用Fast forward
```
**分支策略**：
首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布1.0版本；

团队中每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。
### 6.4 Bug分支
在Git中，每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。
```
git stash // 把当前工作现场“储藏”起来，等以后恢复现场后继续工作

git switch master // 确定要在哪个分支修复bug
git branch issue-101 // 创建bug分支
git switch issue-101

// 修改test.txt中的bug
git add test.txt 
git commit -m "fix bug 101"

[issue-101 4c805e2] fix bug 101
1 file changed, 1 insertion(+), 1 deletion(-)

git switch master
git merge --no-ff -m "merged bug fix 101" issue-101

git switch dev
git stash list // 查看刚才stash的工作现场保存位置
git stash pop // 恢复工作现场
```
在`master`分支上修复了bug后，想一想，`dev`分支是早期从`master`分支分出来的，所以，这个bug其实在当前`dev`分支上也存在。

同样的bug，要在`dev`上修复，我们只需要把`4c805e2 fix bug 101`这个提交所做的修改“复制”到`dev`分支。
```
git cherry-pick 4c805e2 // 复制一个特定的提交到当前分支
```
### 6.5 Feature分支
添加一个新功能时，肯定不希望因为一些实验性质的代码，把主分支搞乱了，所以，每添加一个新功能，最好新建一个`feature`分支，在上面开发，完成后，合并，最后，删除该`feature`分支。
```
git switch -c feature-vulcan
git add vulcan.py
git commit -m "add feature vulcan"

git switch dev // 准备合并
```
就在此时，接到上级命令，因经费不足，新功能必须取消！
```
git branch -D feature-vulcan // 强行销毁该分支
```
### 6.6 多人协作
当从远程仓库克隆时，实际上Git自动把本地的`master`分支和远程的`master`分支对应起来了，并且，远程仓库的默认名称是`origin`。
```
git remote -v // 查看远程库信息
git push origin master // 将master分支内所有内容推送到origin库中
git push origin dev
```
另一个团队成员此刻想参与进团队协作中，当他从远程库clone时，默认情况下，他只能看到本地的master分支，所以还要另外去clone掉dev分支：
```
git checkout -b dev origin/dev // 创建远程origin的dev分支到本地

// 本地修改env.txt文件
git add env.txt
git commit -m "add env"

git push origin dev

// 此时，如果有两个人在同时推送修改
git branch --set-upstream-to=origin/dev dev // 指定本地dev分支与远程origin/dev分支的链接
git pull // 抓取最新的提交
// 手动解决修改产生的冲突
git commit -m "fix env conflict"
git push origin dev
```
### 6.7 Rebase
rebase操作可以把本地未push的分叉提交历史整理成直线，目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。
```
git rebase
git log --graph --pretty=oneline --abbrev-commit
```
## 7.标签管理
### 7.1 创建标签
默认标签是打在最新提交的commit上的。
```
git switch master
git tag v1.0 // 为master打标签
git tag // 查看所有标签

git log --pretty=oneline --abbrev-commit
git tag v0.9 f52c633 // 对过往的commit打标签
git show v0.9 // 查看标签信息

git tag -a v0.1 -m "version 0.1 released" 1094adb // 在指定commit上打标签并加入注释

```
> 注：标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签。
### 7.2 操作标签
删除标签：
```
git tag -d v0.1
```
因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。如果要推送某个标签到远程：
```
git push origin v1.0
git push origin --tags // 推送全部尚未推送到远程的本地标签
```
如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除，然后，从远程删除：
```
git tag -d v0.9
git push origin :refs/tags/v0.9
```
