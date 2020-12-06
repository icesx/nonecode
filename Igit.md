git
==================
# command 

### clone
```
$git clone http://xxx
```
### commit
```
$git commit .
$git commit -a .
```
### push
```
$git push .
```
### push delete file
```
$git add -A
$git commit -m ""
$git push
```

## git client

```
git clone http://server/spring-cloud-config.git

cd spring-cloud-config
vi .git/config

#add passowd and use to url as 
	url = http://i:i@solar27/spring-cloud-config.git
#没有修改这句话的时候会出现错误

git push origin master
touch readme.md
git commit -m "init master"
git push origin

error: Cannot access URL http://solar27/spring-cloud-config.git/, return code 22
fatal: git-http-push failed
error: failed to push some refs to 'http://solar27/spring-cloud-config.git'
```
## Config

### 保持密码

```
git config --global credential.helper cache
git config --global credential.helper 'cache --timeout=360000'
```

### 换行符^M忽略
`git config --global core.whitespace cr-at-eol`

## branch
### 比较本地和远程
`git diff master origin/master`

### 中文文件名
`git config --global core.quotepath false`

# git 的正确用法
## branch
远程master分支
本地dev分支
## 正常工作流程
1. 从服务器checkout master分支
	
	`git clone http:///.xx.git`
	`git checkout master`
	
2. 创建本地 dev 分支
    `git branch dev`

3. 切换到dev分支开发代码
    `git checkout dev`

4. 提交之前从切换到master分支
    `git checkout master`

5. 将本地dev代码merge到master
    `git merge dev`

6. 提交代码
    `git push`
## 异常情况
### merge冲突
如果在从dev merge到master后，提交的时候，发现master remote已经更新，此时可以将master还原，再pull之后在进行merge
`git reset --hard`
	如果要reset到一个指定版本，可以使用 git log查看版本后，使用命令
`	git reset --hard 436d03844f742bdb077dbfeb9e42c2cb687ae384`
`	git pull`
`	git merge dev`

### 删除Untracked files
	git clean -f
### 强制更新
	git fetch --all    //只是下载代码到本地，不进行合并操
	git reset --hard origin/master

### 同时修改
error: Your local changes to the following files would be overwritten by merge:
Please commit your changes or stash them before you merge.


解决办法：

1、服务器代码合并本地代码
```
$ git stash     //暂存当前正在进行的工作。
$ git pull   origin master //拉取服务器的代码

$ git stash pop //合并暂存的代码
```

### 恢复版本

```
git revert -n 8b89621019c9adc6fc4d242cd41daeb13aeb9861
git commit -m "revert add text.txt" 
git push
```

### 删除远程分支

```
git push origin --delete IDEV
```



### 找到文件删除记录

```
git log --diff-filter=D --summary | grep delete|more
#找到所有被删除的文件
git log --full-history -1 devel/cbd-board/board-common/.gitignore
```

```
commit 0dec2c00b99c36b6a43543d0de068482415b6934
Merge: d5135553 e411c94e
Author: guowei <363578642@qq.com>
Date:   Wed Dec 18 01:13:23 2019 +0800

    commit 冲突
------------------
最后一次删除动作从 e411c94e merge 到了 d5135553
之后提交，commitid 为 0dec2c00b99c36b6a43543d0de068482415b6934
```

```
git log
commit 0dec2c00b99c36b6a43543d0de068482415b6934
Merge: d5135553 e411c94e
Author: guowei <363578642@qq.com>
Date:   Wed Dec 18 01:13:23 2019 +0800

    commit 冲突

commit d5135553f1d643f2cf589197ae98fda740f4b709
Author: guowei <363578642@qq.com>
Date:   Wed Dec 18 01:01:36 2019 +0800

    commit
-----------------------
在commit short id 为 d5135553 的提交动作中删除了代码
d5135553=d5135553f1d643f2cf589197ae98fda740f4b709
```

### rebase

> 如果需要本地合并提交（比如提交了一个大文件后又将其删除了，那么可以合并提交后，就不用提交了）
>
> ```
> git rebase -i HEAD~3
> #从HEAD版本开始往过去数3个版本
> git rebase -i 3a4226b
> #.指名要合并的版本之前的版本号
> 
> ```
>
> 1.执行了rebase命令之后，会弹出一个窗口，头几行如下：
>
> ```
> pick 3ca6ec3   '注释**********'
> 
> pick 1b40566   '注释*********'
> 
> pick 53f244a   '注释**********'
> ```
>
> 2.将pick改为squash或者s,之后保存并关闭文本编辑窗口即可。改完之后文本内容如下：
>
> ```
> pick 3ca6ec3   '注释**********'
> 
> s 1b40566   '注释*********'
> 
> s 53f244a   '注释**********'
> ```
>
> 3.然后保存退出，Git会压缩提交历史，如果有冲突，需要修改，修改的时候要注意，保留最新的历史，不然我们的修改就丢弃了。修改以后要记得敲下面的命令：
>
> ```
> git add .  
> 
> git rebase --continue  
> ```
>
> 如果你想放弃这次压缩的话，执行以下命令：
>
> ```
> git rebase --abort  
> ```

### 迁移

将一个project迁移到另外一个project下

```
git clone <old_url>
cd <repo_dir_name>
git remote add new_remote <new_url>
git push --all new_remote
```

### 删除历史大文件

https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/removing-sensitive-data-from-a-repository

1. 查找大文件

   ```sh
   git verify-pack -v .git/objects/pack/pack-*.idx | sort -k 3 -g | tail -10
   git rev-list --objects --all |grep 8ddb959457d756df414737e5e95181a01148722d
   ```

2. 删除目录

   check 分支

   ```sh
   for i in `git branch  -r|grep -v HEAD` ; do git checkout --track $i; done
   ```

   

   删除`doc`目录
   
   ```sh
   git filter-branch --force --index-filter 'git rm -rf --cached --ignore-unmatch --ignore-unmatch doc' --prune-empty --tag-name-filter cat -- --all
```
   
删除本地缓存
   
   ```sh
   git reflog expire --expire=now --all && git gc --prune=now --aggressive
   rm -Rf .git/logs .git/refs/original
```
   
   提交
   
   ```sh
   git push origin --all --force
   git push origin --tags --force
   ```
   
   