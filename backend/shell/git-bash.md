```bash
# 查看git命令帮助文档, 命令后加-h

git remote set-url origin [url]

# 设置远程地址
git remote set-url origin https://github.com/cschenzz/springboot-demo.git

# 列出详细信息，在每一个名字后面列出其远程url
# 此时， -v 选项(译注:此为 –verbose 的简写,取首字母),显示对应的克隆地址
git remote -v

# 将本地新git库推送到远程仓库		
git remote add origin https://github.com/cs_chenzz/sb-api.git
git push -u origin master

# --------------------
git remote add mygit https://github.com/cschenzz/springboot-web-thymeleaf.git

# 将本地的master分支推送到mygit主机的master分支。如果master不存在，则会被新建。
git push mygit master

# 将本地的所有分支推送到origin主机。如果对应分支不存在，则会被新建。
git push origin --all

# 取消已经commit但是未push的操作
# 回退到上个版本
git reset --hard HEAD^

# 回退到前3次提交之前，以此类推，回退到n次提交之前
git reset --hard HEAD~3

# 回退到指定commit的版本
git reset --hard commit_id

# 该命令会在本地主机生成一个目录，与远程主机的版本库同名。如果要指定不同的目录名，可以将目录名作为git clone命令的第二个参数
git clone http://github.com/jquery/jquery.git

# 在当前目录创建一个git仓库
git init

# 显示工作目录和暂存区的状态
git status
位于分支 master
您的分支与上游分支 'origin/master' 一致。
尚未暂存以备提交的变更：
（使用 "git add <文件>..." 更新要提交的内容）
（使用 "git restore <文件>..." 丢弃工作区的改动）


# 添加当前目录下的所有文件(包括子目录)
git add .
# 添加指定目录下所有文件
git add ./src/*

# commit提交
git commit -m "the commit message"

# 取回origin主机的master分支，与本地的当前分支合并
git pull origin master
# 上面命令表示，取回origin/master分支，再与当前分支合并。实质上，这等同于先做git fetch，再执行git merge。
git fetch origin
git merge origin/master

# 下载远程仓库最新内容，不做合并
git fetch --all

# 撤销本地所有未提交的修改
git checkout .
# 撤销本地指定文件的修改
git checkout ./src/test.java
# ---------------------

比如，要取回origin主机的next分支，与本地的master分支合并，需要写成下面这样
git pull origin next:master

如果远程分支(next)要与当前分支合并，则冒号后面的部分可以省略。上面命令可以简写为：
git pull origin next

上面命令表示，取回origin/next分支，再与当前分支合并。实质上，这等同于先做git fetch，再执行git merge。
git fetch origin
git merge origin/next

git fetch和git pull的区别 git fetch：相当于是从远程获取最新版本到本地，不会自动合并。
```

## 日志差异
```bash
# 显示整个提交历史记录，但跳过合并
git log --no-merges

# -p 选项展开显示每次提交的内容差异，用 -2 则仅显示最近的两次更新
git log -p -2

# compares the changes in your working directory with the staging area
git diff

# If you provide a commit hash or branch name, this command will compare the working directory with the specified commit. It's useful for seeing changes since a particular point in history
git diff HEAD~1

# HEAD~{n}
# ~ 是用来在当前提交路径上回溯的修饰符
# HEAD~{n} 表示当前所在的提交路径上的前 n 个提交（n >= 0）：
# HEAD = HEAD~0
# HEAD~ = HEAD~1

# HEAD^n
# ^ 是用来切换父级提交路径的修饰符
# HEAD^ 第1个父级
# HEAD^2 第2个父级


# 查看相邻2个版本修改了那些文件及内容(good), 后面的commit是要显示的具体修改提交
git diff HEAD~2 HEAD~1
git diff 6c8ae2^ 6c8ae2
# 查看最近一次修改了那些文件及内容
git diff HEAD~1 HEAD
# ---------------------------------------

# 查看两个版本之间修改了哪些文件内容
git diff a637dd 6c8ae2 --stat

#查看两个版本库状态差异的汇总
git diff --stat 5fc9a6 0c76d45

# 查看两个版本之间(和最新版本对比)的差异汇总
git diff --stat eaa12d HEAD

# numstat表示以表格的形式展示改动文件，并且文件路径是完整路径
git diff --numstat 67df97d5 HEAD

#了解谁在什么时候对my_file做了什么样的改动
git blame my_file

#显示本地代码库 HEAD 的更改日志。这个命令很适合查找丢失的工作
git reflog
```


## 分支
```bash
# 列出现有的分支; 当前分支将以星号突出显示。选项-a显示本地和远程分支
git branch --list -a
git branch -a

# checkout一个远程分支
git checkout jdk-17-temp-dev
# 分支 'jdk-17-temp-dev' 设置为跟踪 'origin/jdk-17-temp-dev'。
# 切换到一个新分支 'jdk-17-temp-dev'
```

## 清理重命名
```bash
# 删除当前目录下没有被track过的文件和文件夹
git clean -df

# 删除当前目录下所有没有track过的文件. 不管他是否是.gitignore文件里面指定的文件夹和文件
git clean -xf

# 删除当前目录中的所有文件(包括子目录)
git rm -rf ./

# 将json目录的login.js移动到static/js/目录
git mv json/login.js static/js/

# 将README.md改名为MyREADME.md
git mv README.md MyREADME.md

# 清理多余的文件和目录
git clean -xfd
```

## 高级用法
```bash
# 对当前分支apply已有的commits
git cherry-pick
特性：

对于给定的一个或多个已有的commits，在当前工作分支上，再次apply并生成新的commits
当前工作分支必须是干净的，即HEAD不包含本地commits
使用：

git cherry-pick [--edit] [-n | --no-commit] [-m parent-number] [-s] [-x] [--ff]

git cherry-pick --continue | --quit | --abort

选项说明：

--edit，编辑commit message
-n, --no-commit，只是在当前分支上apply这些commits的改变，但是不提交到当前分支
示例：

git cherry-pick -n f2c9e399c

# 导出代码, 建议在代码库的根目录下进行
# 导出master分支到当前目录
git archive --output "./code-mm-01.tar.gz" master

# 导出指定commit
git archive --format tar.gz --output "/home/chenzz/tmp-src-001.tar.gz" ca16ac0d
```


## Git设置
```bash
git config --global user.name "chenzz"
git config --global user.email "cs.chenzz@qq.com"

# 查看Git设置
git config --list

# 设置命令别名lg, 这样输入git lg就能看到漂亮的git log
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```
