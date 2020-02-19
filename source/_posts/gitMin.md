---
title: git的一些命令
date: 2019-11-13 14:34:13
tags:
- Show All
- git
header-img: "Demo.png"
---

- git与SVN的最核心的区别：git本地保存了一份完整的历史版本的代码，而SVN只要服务端保留着历史版本；

- git的几个仓储概念： workspace、stagingArea、localRepository、remoteRepository
1. workspace： 自己本地的工作空间，也就是直接在文件夹里看到的内容；
2. stagingArea： 暂存区,存在于.git文件夹中，每次add之后会在这里记录；
3. localRepository： 本地仓库，存在于.git文件夹中，每次的commit都提交到这里；
4. remoteReposiroty： 远程仓库的本地副本，存在于.git文件夹中，每次的push成功后在这里有记录；

- git的几个文件状态概念：untracked、origin、modified、staged、committed、pushed
1. untracked： 文件没有add过，不受git控制；
2. origin： 使用ckeckout后的文件状态，与本地的remoteReposiroty保持一致，自己没有修改过；
3. modified： origin状态的文件修改后的状态即为modified；
4. staged： modified状态的文件add之后的状态即为staged；
5. committed： staged状态的文件commit之后的状态即为committed；
6. pushed： comitted状态的文件push之后的状态即为pushed；

##### 安装git：
1. 下载git文件 https://git-scm.com/downloads ；
2. 设置自己的名字（对应每一次的提交者的名字）：
git config —global user.name "your name"
git config –global user.email "email@youngyedu.com"
3. 生成密钥对
ssh-keygen -t rsa -C "email@youngyedu.com"
如果是一路回车的话，在 C://User/.ssh文件夹中，找到id_rsa.pub，发给管理员

##### 与本地git相关的命令：
1. 暂存单个文件：（新增文件让git受控）
git add <fileName>

2. 暂存所有modifed的文件：
git add .

3. 提交所有staged的文件：
git commit -m 'message'

4. 状态检查
git status
git status -s (--short)

5. 查看提交记录
git log
git reflog （简要信息）
git log --graph (可以看到分支的生成与合并)
git log --stat (统计每次提交的每个文件有多少处修改)
git log -p -2 (-p表示每次提交的内容差异，-2表示最近两次)

6. 文件比较
git diff （当前文件与已暂存的文件进行比较）

##### 与git分支相关的命令：
1. 在当前分支上新建分支
git branch <branch>
2. 切换分支（workspace中的代码变动）
git checkout <branch>

3. git branch -b <branch> 等价上边的两条命令

4. 分支代码合并：将<branch>分支的代码合并到当前分支
git merge <branch>
注意：合并分支之前，一定要git status确保working tree clean。因为git merge合并的是当前workspace中的代码。

##### 与远程git相关的命令：
1. 克隆远程代码：
git clone <url>

2. 查看远程分支
git remote -v
git branch -a
git ls-remote

3. 添加并追踪远程分支
git remote add <name> <url>
git branch -u <name>/<branch> (-u 等价于 —set-upstream-to)
注意： git里默认的远程name是origin，这个origin没有什么特殊的，就是一个name而已。

4. 切换远程分支
git checkout <name>/<branch>

5. 拉取远程分支并关联
git checkout —track <name>/<branch>

6. 查看本地分支和远程分支的关联关系
git branch -vv

7. 执行 git clone 之后默认拉的是master分支。本地创建并跟踪dev分支
git checkout -b dev origin/dev

8. 拉取远程代码
git fetch <name>  （如果已关联远程分支，可以省略<name>）
git merge <name>/<branch>
git pull <name >  （如果已关联远程分支，可以省略<name>）
注意：第三条命令等价于上面两条，建议使用git fetch后自行merge；

9. 查看远程分支日志
git log <name>/<branch>

10. 创建远程分支
首先本地需要现有一个分支<local_branch>
git push <name> <local_branch>:<remote_branch>

11. 提交代码到远程分支
提交之前，请先确保本地分支是最新版本。（即提交之前必须先fetch + merge）
git push <name>
git push <name> -f (--fore) 强推~~，
注意：强推前须与团队成员沟通，否则别人也强推，你就尴尬了。

12. 远程分支被删除后，删除本地的远程分支
远程分支删除后，本地执行git branch -a 依然会显示被删除掉的远程分支。
执行 git remote show origin 可以看到对应的分支处于stale状态，其他正常的处于tracked状态。
执行 git remote prune origin 可以清除掉本地显示的远程其实已经不存在的分支。


#### 一些常用场景：
##### 取消修改：
1. origin 已变为 modified 尚未add，想恢复origin：(stagingArea -> workspace)
git ckeckout -- <file>
2. 已add的文件，想撤销add，文件还原为modifed状态：
git reset HEAD <file>
3. 已commit的文件想恢复为上一个版本，即对应svn的版本回退：
- 1. 使用git revert：
git revert HEAD 回退到上一次提交
git revert HEAD~2 回退到上两次提交
注意： revert实际上是创建了一个与上一次提交完全反操作的新提交；
注意： 连续两次 git revert HEAD 等于啥也没做
- 2. 使用git reset：
git reset --hard HEAD^ 回退上一次
git reset --hard HEAD^^ 回退上两次
git reset --hard commit-id 回退到指定的commit-id；
注意： 使用了--hard会丢失掉回退之前的那个版本，慎用；
gie reset --soft 之后的提交会退回到暂存区；
git reset --hard <name>/<branch> 可以强制回退，与远程代码保持一致

- 3. 使用git rebase
注意： git rebase类似黑魔法，分支merge的时候某些情况下使用，不懂慎用；

- 4. 在svn中删除文件再update可以取消工作空间的修改，但是在git中，切记不可，删除文件算是一次文件改动；

##### 文件比较
1. 比较 workspace中的文件 与 stagingArea中的文件：
git diff <file>

2. 比较 stagingArea中的文件 与 localRepository中的文件：
git diff --staged <file>

3. 比较 workspace中的文件 与 localReposiroty中的文件：
git diff HEAD <file>

4. 比较 workspace中的文件 与 任意commit-id的文件：
git diff commmit-id <file>

5. 比较 stagingArea中的文件 与 任意commit-id的文件：
git diff --stage commit-id <file>

6. 比较任意两个commit-id的文件
git diff commit-id1 commit-id2 <file>

##### 冲突修改
- 没有界面操作，请手动在编辑器内进行代码层面的代码合并，然后再提交

#### 备注
- <font color='red'>'- -'（连续的两个'-'）在showdoc的markdown渲染后会成为'--'，不知什么原因；如果直接复制以上命令使用的话，请注意此问题。 </font>