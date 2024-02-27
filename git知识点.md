# <center>git相关操作说明</center>

---
# 一, git命令:
**常规使用频繁的命令如下:**
- clone（克隆）: 从远程仓库中克隆代码到本地仓库
- checkout （检出）:从本地仓库中检出一个仓库分支然后进行修订
- add（添加）: 在提交前先将代码提交到暂存区
- commit（提交）: 提交到本地仓库。本地仓库中保存修改的各个历史版本
- fetch (抓取) ： 从远程库，抓取到本地仓库，不进行任何的合并动作，一般操作比较少。
- pull (拉取) ： 从远程库拉到本地库，自动进行合并(merge)，然后放到到工作区，相当于fetch+merge
- push（推送） : 修改完成后，需要和团队成员共享代码时，将代码推送到远程仓库

# 二, 常用命令:
- ls/ll 查看当前目录
- cat 查看文件内容
- touch 创建文件
- vi vi编辑器（使用vi编辑器是为了方便展示效果，可以使用记事本、editPlus、notPad++等其它编辑器）

# 三, 基础操作指令
- git add (工作区 --> 暂存区) 命令形式：git add 单个文件名/通配符, 将所有修改加入暂存区：git add .
- git commit (暂存区 --> 本地仓库) 命令形式：git commit -m '注释内容'
- git status *查看修改的状态（status）作用：查看的修改的状态（暂存区、工作区）
- git log 命令形式：git log [option] 作用:查看提交记录

options:
```text
--all 显示所有分支
--pretty=oneline 将提交信息显示为一行
--abbrev-commit 使得输出的commitId更简短
--graph 以图的形式显示
```

- 版本回退: git reset 

```
作用：版本切换

命令形式：git reset --hard commitID
(commitID 可以使用 git-log 或 git log 指令查看)
```
- 如何查看已经删除的记录:  git reflog(这个指令可以看到已经删除的提交记录)

# 四, .ignore文件
按项目规则实行分配

# 五, 分支常用指令
1、查看本地分支  命令：git branch

2、创建本地分支  命令：git branch 分支名

3、*切换分支(checkout)  命令：git checkout 分支名

- 还可以直接切换到一个不存在的分支（创建并切换） 命令：git checkout -b 分支名

4、*合并分支(merge)  命令：git merge 分支名称

- 一个分支上的提交可以合并到另一个分支

5、删除分支(不能删除当前分支，只能删除其他分支)
- git branch -d 分支名 删除分支时，需要做各种检查
- git branch -D 分支名 不做任何检查，强制删除

6、合并冲突(控制台界面显示)
```cmd
CONFLICT (content): Merge conflict in file01.txt           ---提示file01处有修改冲突
Automatic merge failed; fix conflicts and then commit the result.
```

7、解决冲突步骤如下：
- 处理文件中冲突的地方
git会在file01文件提示: 
```text
<<<<<<< HEAD
count=3
=======
count=2
>>>>>>> dev01
直接暴力修改留下count=2
```

- 将解决完冲突的文件加入暂存区(add)

- 提交到仓库(commit)


# 六, 远程仓库推送
1、把本地的 master 仓库名称修改为远端的 main重命名命令：
```cmd
git branch -m oldBranchName newBranchName
```
2, 只允许提交, 不允许覆盖

3, 添加远程仓库:  git remote add <远端名称> <仓库路径>

4, 推送到远程仓库:  git push [-f] [--set-upstream] [远端名称 [本地分支名][:远端分支名] ]
- 如果远程分支名和本地分支名称相同，则可以只写本地分支
```cmd
git push origin master
```
- -f 表示强制覆盖
- --set-upstream 推送到远端的同时并且建立起和远端分支的关联关系。
```cmd
git push --set-upstream origin master
```
- 前分支已经和远端分支关联，则可以省略分支名和远端名,将master分支推送到已关联的远端分支
```cmd
git push 
```

5, 从远程仓库中抓取和拉取
远程分支和本地的分支一样，我们可以进行merge操作，只是需要先把远端仓库里的更新都下载到本
地，再进行操作。

- 抓取命令：git fetch [remote name] [origin/master] (抓取指令就是将仓库里的更新都抓取到本地，不会进行合并)

- 忽略其他分支的合并:
```cmd
git merge origin/develop -allow-unrelated-histories
```

- 拉取命令：git pull [remote name] [branch name] (如果不指定远端名称和分支名，则抓取所有分支。
  拉取指令就是将远端仓库的修改拉到本地并自动进行合并，等同于fetch+merg，如果不指定远端名称和分支名，则抓取所有并更新当前分支)

# 附录: 
**git网络代理设置:**

- 使用http代理 
```cmd
	git config --global http.proxy http://127.0.0.1:58591
	git config --global https.proxy https://127.0.0.1:58591
```
- 使用socks5代理
```cmd
	git config --global http.proxy socks5://127.0.0.1:51837
	git config --global https.proxy socks5://127.0.0.1:51837
```


- 取消代理:
```cmd
	git config --global --unset http.proxy
	git config --global --unset https.proxy
```

- 查看代理: 
```cmd
	git config --global --get http.proxy
	git config --global --get https.proxy
```


