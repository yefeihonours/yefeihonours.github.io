## 一、准备环境

> 首先，安装好Git。不同的操作系统，安装的方式有所区别

```
$ git --version
git version 2.11.0 (Apple Git-81)

# 配置用户信息
$ git config --global user.name "name"
$ git config --global user.email "name@mail.com"
```
<!--more-->

## 二、Git基本操作

#### 初始化版本库

```
$ git init
Initialized empty Git repository in ~/work/gitbook/git_study/.git/

$ touch 1.txt 2.txt
.
|____.git/      # Git的项目历史信息会保存在.git文件夹中
|____1.txt
|____2.txt
```

#### 首次提交

将 `1.txt`, `2.txt` 添加到版本库的控制系统中。1，`add`命令确定了应该包含在下次提交中的文件。2，`commit`命令将修改传送到版本库中，并赋予一个散列值来标识这次新提交。
```
$ git add 1.txt 2.txt
$ git commit --message "first commit"
[master (root-commit) c86971d] first commit
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 1.txt
 create mode 100644 2.txt
```

#### 首次提交

将 `1.txt`, `2.txt` 添加到版本库的控制系统中。1，`add`命令确定了应该包含在下次提交中的文件。2，`commit`命令将修改传送到版本库中，并赋予一个散列值来标识这次新提交。
```
$ git add 1.txt 2.txt
$ git commit --message "first commit"
[master (root-commit) c86971d] first commit
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 1.txt
 create mode 100644 2.txt
```

#### 检查状态

现在修改版本库中的文件后，使用 `status` 命令查看都有那些文件被修改。

状态信息中: 
* `modified: 1.txt`: 文件内容有改动
* `deleted: 2.txt`: 文件被移除
* `new.txt`: 新添加的文件，需要使用 `git add <file>...` 命令将文件添加到版本库

```
$ touch new.txt         # 创建新文件
$ rm 2.txt              # 删除文件
$ git status            # 查看状态
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   1.txt
	deleted:    2.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	new.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

如果想看到更多文件被修改的细节，可以使用 `diff` 命令来显示文件被修改的每一行信息。`diff` 命令的简单使用如下
```
$ git diff 1.txt

diff --git a/1.txt b/1.txt
index be996b8..7655a69 100644
--- a/1.txt
+++ b/1.txt
@@ -1,1 +1,1 @@
-111111111
\ No newline at end of file
+222222222
\ No newline at end of file
```

#### 提交修改

版本库中的文件被修改后，下面就要提交本次修改。首先对修改过的文件和新添加的文件执行 `add` 命令，再对要删除的文件执行 `rm` 命令。(使用 `git add .` 表示添加对所有文件的修改。)
```
$ git add 1.txt new.txt
$ git rm 2.txt
rm '2.txt'
```

使用 `status` 查看现在文件的状态
```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   new.txt
	modified:   1.txt
	deleted:    2.txt
```

使用 `commit` 提交当前状态的修改。`--message` 参数指定本次提交的描述信息，可以使用 `-m` 缩写形式代替。
```
$ git commit --message "Some changes."
[master 6a1c221] Some changes.
 3 files changed, 2 insertions(+), 2 deletions(-)
 create mode 100644 new.txt
 dalete mode 100644 2.txt
```

#### 查看提交历史

`log` 命令可以查看项目的提交历史信息，所有的提交都会按照时间顺序被降序排列出来。
```
$ git log

commit 6a1c22103d2f5ac097bae73d999c9c3a86951359 (HEAD -> master)
Author: qidunwei <dunwei_qi@sina.com>
Date:   Wed Nov 22 14:11:52 2017 +0800

    Some changes.

commit 69ef56f8d2949531f1d40d1072b7fa6205e0a1f1
Author: qidunwei <dunwei_qi@sina.com>
Date:   Fri Nov 3 16:44:15 2017 +0800

    update 1.txt

commit c86971d60fcdd4f3dc8d71ad7df1c1fdb8c91bfd
Author: qidunwei <dunwei_qi@sina.com>
Date:   Thu Nov 2 18:26:06 2017 +0800

    first commit
```

#### 链接远程仓库

```
# 查看现有远程仓库地址 
$ git remote -v

# 删除已有远程地址
$ git remote rm origin

# 添加新的远程地址
$ git remote add origin git@github.com:username/repositorie.git 

# 直接修改已有远程地址
$ git remote set-url origin git@github.com:username/new.git

```
