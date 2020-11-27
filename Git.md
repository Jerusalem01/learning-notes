## Git

#### 1.1 安装

[下载地址](https://git-scm.com/downloads)

#### 1.2 简化

**TortoiseGit 和 语言包**

[下载地址](https://tortoisegit.org/download/)

#### 1.3 基本操作

- **获取与创建项目命令**

```
# 创建新的 Git仓库
git init

# 拷贝一个 Git仓库到本地
git clone [url]
```

- **基本快照**

```
# 添加文件到缓存
git add <filename>
# 查看上次提交之后是否有修改
git status
git status -s
# 查看执行 git status的结果的详细信息
git diff 
（显示已写入缓存与已修改但尚未写入缓存的改动的区别）

git diff 有两个主要的应用场景：

尚未缓存的改动：git diff
查看已缓存的改动： git diff --cached
查看已缓存的与未缓存的所有改动：git diff HEAD
显示摘要而非整个 diff：git diff --stat
# git commit（将缓存区内容添加到仓库）

第一步，配置用户名和邮箱地址
git config --global user.name 'yourname'
git config --global user.email youremail

第二步，将文件写入缓存区并提供提交注释
git commit -m 'update message'
# 取消已缓存的内容
git reset HEAD -- <filename>
```

- **拉取与推送**

```
# git pull
用于从另一个存储库或本地分支获取并集成(整合)。
作用是：取回远程主机某个分支的更新，再与本地的指定分支合并。

git pull <远程主机名> <远程分支名>:<本地分支名>

在默认模式下，git pull 是 git fetch 后跟 git merge FETCH_HEAD 的缩写。
准确地说，git pull使用给定的参数运行 git fetch，并调用 git merge 将检索到的分支头合并到当前分支中。
# git push
用于将本地分支的更新，推送到远程主机。

git push <远程主机名> <本地分支名>:<远程分支名>
```

- **标签**

```
# git tag

如果希望永远记住某个特别的提交快照，可以使用 git tag 打上标签。

比如说，我们想为我们的 商城 项目发布一个"1.0.0"版本。 我们可以用 git tag -a v1.0.0 命令给最新一次提交打上（HEAD） "v1.0.0" 的标签。

-a 选项意为"创建一个带注解的标签" 
不用 -a 选项也可以执行
但不会记录标签是啥时候打的，谁打的，也不会让你添加个标签的注解

推荐创建带注解的标签：
git tag -a v1.0.0

查看所有标签：
git tag
```

#### 1.4 Git 过滤文件

- **.gitattributes**

```
# Windows-specific files that require CRLF:
*.bat       eol=crlf
*.txt       eol=crlf

# Unix-specific files that require LF:
*.java      eol=lf
*.sh        eol=lf
```

- **.gitignore**

```
target/
!.mvn/wrapper/maven-wrapper.jar

## STS ##
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans

## IntelliJ IDEA ##
.idea
*.iws
*.iml
*.ipr

## JRebel ##
rebel.xml

## MAC ##
.DS_Store

## Other ##
logs/
temp/
```