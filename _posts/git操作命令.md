---
title: git操作命令
date: 2019-08-14 09:44:48
tags:
categories: tools
---



# 沉浸式学习Git

[参考地址]( http://igit.linuxtoy.org/index.html )

# 创建仓库

__git clone [url] [dir]__
__git init__

git@github.com:bzadhere/myim.git ssh协议
https://github.com/bzadhere/myim.git https协议

__git生成SSH公钥__
~/.ssh 目录下有公钥, 或ssh-keygen创建, 生成公钥id_rsa.pub和私钥id_rsa 

```
$ ssh-keygen -t rsa -C "youremail@example.com"
// github上，进入 Account Settings（账户配置），左边选择SSH Keys， 
// Add SSH Key,title随便填，粘贴在你电脑上生成的key
$ ssh -T git@github.com
Hi xxxx! You've successfully authenticated, but GitHub does not provide shell access.
```
<!-- more -->
-----
# 基本概念
* __working tree:__ 能看到的目录
* __HEAD:__ 是当前分支引用的指针，它总是指向该分支上的最后一次提交(commit)
* __Index:__ 索引是你的“预期的下一次提交”–“暂存区域”，运行git add后，代码就进入“暂存区域”

git add 索引就记录
git commit 暂存区目录树写到对象库中, master分支相应更新。master指向的目录树就是提交时暂存区的目录树
git reset HEAD 暂存区的目录树会被重写, 被 master 分支指向的目录树所替换, 但是工作区不受影响
git rm --cached <file> 从暂存区删文件, 工作区不变
git checkout HEAD(.) 或git checkout HEAD <file> 会用 HEAD 指向的分支中的全部或部分文件替换暂存区和以及工作区中的文件

-----
# 配置说明
__依次加载配置, 变量依次覆盖__
/etc/gitconfig git config --system
~/.gitconfig git config --global
.git/config git config --list 一般就用这个

```
[imdev@localhost .git]$ cat config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        quotepath=false #避免中文文件名显示乱码
[remote "origin"]
        url = git@github.com:bzadhere/myim.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
[user]
        name = bzadhere
        email = bzadhere@gmail.com
```

__别名__
```
git config --global alias.st status;
git config --global alias.ci commit;
git config --global alias.co checkout;
git config --global alias.br branch;
git config --global alias.last 'log -1 HEAD';
```
-----
# 分支操作

```
# 创建分支
git branch (branchname)
# 切换分支
git checkout (branchname)
# 列出分支
git branch
# 创建并切换到该分支
git checkout -b (branchname)
# 删除分支
git branch -d (branchname)
# 合并指定分支到当前分支
git merge <branchname>
```
-----
# 标签
```
# 查看标签
git tag
# 添加标签
git tag -a v1.0
# 指定版本添加标签
git tag -a v1.0 c3d2d07
// 删除
git tag -d v1.0
// 标签传送到远端服务器上
git push <remote> [tagname]
git push <remote> --tags
```
-----

# 查看日志和文件状态

```
# 查看那些文件修改了
$ git status -s

# 查看帮助
[imdev@localhost ~/myim]$ git help log

# 查看修改日志
[imdev@localhost ~/myim]$ git log
commit 6ee178bf83f1996568401cfb23411b3c046922e7
Merge: ebe63a1 c3d2d07
Author: bzadhere <bzadhere@gmail.com>
Date:   Mon Aug 19 23:22:30 2019 -0400

    Merge branch 'tmpim'
    
    Conflicts:
        test.txt

commit c3d2d07be4add9fb1eba57b49fc3fe0202615dc3
Author: bzadhere <bzadhere@gmail.com>
Date:   Mon Aug 19 23:20:10 2019 -0400

    modify from tmpim

commit ebe63a1782656804e002a35ec372195ba72ebdfd
Author: bzadhere <bzadhere@gmail.com>
Date:   Mon Aug 19 23:17:07 2019 -0400

    modify from master
......

# 简洁版本--oneline, 分支合并--graph
[imdev@localhost ~/myim]$ git log --oneline --graph
*   6ee178b Merge branch 'tmpim'
|\  
| * c3d2d07 modify from tmpim
* | ebe63a1 modify from master
|/  
* 5b824c1 add test.txt
* 7706505 add dir
* 7d4976b add dir
* 8782412 add file

# --reverse 逆向, --author=zhangbb 指定用户, --no-merges 隐藏合并提交
# --before={1.weeks.ago} 一周前, --after={2010-04-18}, 4.18号之后; --since, --util

# 分支还没合并进主干的修改
git log master..branch
```
------
# 远程仓库

```
// 查看当前的远程仓库
git remote -v
// 查看远程仓库版本库信息
git remote show [shortname]
// shortname 别名
git remote add [shortname] [url]
// 重命名
git remote rename old new
// 删除
git remote rm [shortname]

```

------
# 修改提交
```
// 文件改名
git mv <old> <new>
// 彻底删除文件,  包括工作区和暂存区
git rm <file>
// 暂存区和上一个版本差异
git diff --stage
// 比较指定版本差异
git diff (id1) (id2) --binary --(path) > 目标文件路径
// 增加全部文件 或 指定文件夹
git add .
git add --all
git add <dir>
// 提交指定文件或多个文件
git commit <file> -m "comment"
// 提交了所有 暂存区 的文件
git commit -m "commit"
// 提交分别来自不同地方的文件，比如 工作区的 和 暂存区的
git commit -o a.txt b.txt -m "comment"
// 忽略文件或文件夹，在项目根目录下面 添加 .gitignore文件 
// 只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的
vim .gitignore
# 忽略*.o和*.a文件
 *.[oa]
# 忽略*.b和*.B文件，my.b除外
*.[bB]
!my.b
# 忽略dbg文件和dbg目录
dbg
# 只忽略dbg目录，不忽略dbg文件
dbg/
# 只忽略dbg文件，不忽略dbg目录
dbg
!dbg/
# 忽略.gitignore本身
!.gitignore
# 只忽略当前目录下的dbg文件和目录，子目录的dbg不在忽略范围内
/dbg
# 以'#'开始的行，被视为注释.
 * ？：代表任意的一个字符
    * ＊：代表任意数目的字符
    * {!ab}：必须不是此类型
    * {ab,bb,cx}：代表ab,bb,cx中任一类型即可
    * [abc]：代表a,b,c中任一字符即可
    * [ ^abc]：代表必须不是a,b,c中任一字符

```
------
# 版本合并(更新)
```
// 下载更新到本地
git fetch [remote-name]
// 合并到任意分支
git merge [shortname]
// 推送本地数据到远程仓库, 如果从分支push 会产生合并请求
git push [remote-name] [branch-name]
// 从远程仓库更新合并到本地
git pull [remote-name] [branch-name]

// 合并一个分支上改动的部分文件到master
git checkeout master
git checkeout --path branch file

// 签出指定文件
git checkout [<options>] [<branch>] -- <file>
// 从上一次提交中签出指定文件
git checkout -- a.txt
// 从指定的提交历史中签出指定文件
git checkout 830cf95f56ef9a7d6838f6894796dac8385643b7 -- a.txt
// 从其他分支签出指定文件
git checkout master -- a.txt
// 签出某个后缀的文件 或 指定目录
git checkout -- *.txt
git checkout -- css/

```
------

# 回退commit
最后三个版本 HEAD~3包含(HEAD, HEAD^, HEAD~2)

```
// 本地进行了多次git commit操作，现在想撤销到其中某次Commit
// soft 回退到某个版本，只回退了commit的信息，不会恢复到index file一级
// mixed 此为默认方式, 回退到某个版本，只保留源码，回退commit和index信息
// hard 彻底回退到某个版本，本地的源码也会变为上一个版本的内容
git reset [--soft | --mixed | --hard | --merge | --keep] [-q] [<commit>]
// 回退到指定版本
git reset --hard 406b5944e3

// 文件被修改了，但未执行git add操作(working tree内撤销) 
git checkout fileName 
git checkout .

// 同时对多个文件执行了git add操作，但本次只想提交其中一部分文件
git add *
git status
git reset HEAD <filename> # 取消暂存

// 文件执行了git add操作，但想撤销对其的修改（index内回滚）
git reset HEAD fileName # 取消暂存
git checkout fileName # 撤销修改

// 修改的文件已被git commit，但想再次修改不再产生新的Commit
git add sample.txt
git commit --amend -m "说明" # 修改最后一次提交 

// 修改最近三次提交信息, 删除某次提交, 编辑删除即可
git rebase -i HEAD~3
```

__已经git push推送到远程仓库中__
```
// 若有tag
git checkout <tag>

// 回到当前HEAD指向
git checkout <branch_name>

// 撤销指定文件到指定版本
git log <filename>  
git checkout <commitID> <filename> # 回滚到指定commitID

// 删除最后一次远程提交
git revert HEAD          # 放弃指定提交的修改，生成一次新的提交, 以前的历史记录都在
git push origin master

git reset --hard HEAD^   # 将HEAD指针指到指定提交，历史记录中不会出现放弃的提交记录
git push origin master -f # -f 将旧版本强制推送更新到远程仓库

// 回滚某次提交
git revert commitID

```

# 保存修改
```
// 暂存修改, 工作目录恢复到修改前, save [message]
$ git stash
// 查看暂存修改
git stash list
// 查看最近一个修改的diff, 
git stash show
// 取出和删除指定修改
git stash apply stash@{2}
git stash drop stash@{2}
// 取出最近一个修改
git stash apply

// 取出并删除最近一个修改
git stash pop
// 从暂存区创建一个分支
$ git stash branch testchanges
// 删除全部stash
git stash clear

```

# 命令总结

[官方地址](https://git-scm.com/book/zh/v2)

![](git操作命令/git-cheat-sheet.png)

# 练习

```
1. 配置
2. 创建分支，切换
3. 文件修改，状态查看，日志查看(指定时间条件)，提交(批量或单个), 忽略文件或文件夹，回退
4. 同步分支，合并，冲突解决，撤销，更新某些指定文件
5. 暂存当前工作
6. 推送到远程仓库，同步远程仓库到本地
7. 文档查看, git cmd --help
```



```
git init
git add README.md
git commit -m "first commit"
git remote add origin git@*************.git
git push -u origin master

# tmpim分支创建切换
[imdev@localhost ~/myim]$ git branch
* master
[imdev@localhost ~/myim]$ git branch tmpim
[imdev@localhost ~/myim]$ git checkout tmpim
Switched to branch 'tmpim'
[imdev@localhost ~/myim]$ git branch
  master
* tmpim

# master 合并 tmpim修改
[imdev@localhost ~/myim]$ echo "hello world!" > test.txt
[imdev@localhost ~/myim]$ ls
ob_rel  Readme  source  test.txt
[imdev@localhost ~/myim]$ git add test.txt 
[imdev@localhost ~/myim]$ git commit test.txt -m 'add test.txt'
[tmpim 5b824c1] add test.txt
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
[imdev@localhost ~/myim]$ git checkout master
Switched to branch 'master'
[imdev@localhost ~/myim]$ ls
ob_rel  Readme  source
[imdev@localhost ~/myim]$ git merge tmpim
Updating 7706505..5b824c1
Fast-forward
 test.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 test.txt
[imdev@localhost ~/myim]$ ls
ob_rel  Readme  source  test.txt

# 分支合并冲突
[imdev@localhost ~/myim]$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)
[imdev@localhost ~/myim]$ git merge tmpim
Auto-merging test.txt
CONFLICT (content): Merge conflict in test.txt
Automatic merge failed; fix conflicts and then commit the result.
[imdev@localhost ~/myim]$ cat test.txt 
hello world!
<<<<<<< HEAD
by zhangbb
=======
by tmpim
>>>>>>> tmpim
[imdev@localhost ~/myim]$ vi test.txt 
.......
[imdev@localhost ~/myim]$ git status -s
UU test.txt
[imdev@localhost ~/myim]$ git add test.txt 
[imdev@localhost ~/myim]$ git status -s
M  test.txt
[imdev@localhost ~/myim]$ git commit

```

