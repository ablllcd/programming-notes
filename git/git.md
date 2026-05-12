# Git 理论知识

## GIT 官方文档

https://git-scm.com/book/zh/v2/%e8%b5%b7%e6%ad%a5-%e5%85%b3%e4%ba%8e%e7%89%88%e6%9c%ac%e6%8e%a7%e5%88%b6

其实官方文档介绍地很详细，有些内容可能直接查看官方文档而不进行记录了。

## 介绍

git 是用作版本控制的，先通过 init 创建一个仓库，仓库里的文件都将被 git 进行管理。

版本相当于对所有文件的当前状态拍一张照，它记录了所有文件当前的状态。而 git 来进行版本控制就是在当前文件夹下创建一个.git 文件，该文件保存了要管理的文件夹的所有版本，每次 commit 就是将一个版本提交到仓库中去。（注意：每次提交的版本是当前所有文件的状体，而不是和上次版本的差异）

文件区域可以分为三个：工作区(working Directory)，暂存区(Staging Area)和本地仓库(Local Repository)。

工作区：文件存在的区域，也就是.gti 存在的目录。

暂存区：存放即将提交的文件，位于.git/index。

本地仓库：存储文件和版本信息，位于.git/objects。

![Alt text](pic/workArea.png)

对应的，文件也可以被分为四个状态：

未跟踪：没有被 git 管理的文件, 处于工作区。

未修改：跟踪后但是文件没有变化，处于工作区。

已修改：跟踪的文件并且发生了修改，处于工作区。

已缓存：被放进缓存区的文件。

![Alt text](pic/fileStatus.png)

由图可知文件的状态并非单向转变的。

# 常用命令

## 全局操作

### 配置

```
# 全局配置
git config --global user.name "cian"
git config --global user.email a1401702664@gmail.com

# 本地配置（配置写在.git/config文件中）
git config user.name "cian"
git config user.email a1401702664@gmail.com

# 查看配置(默认先查看本地配置，如果没有则查看全局配置)
git config user.name
git config user.email
git config --global --list

# 删除配置
git config --global --unset user.name
git config --unset user.name
```

### 帮助

```
git command -h
git help command
```

## 基本操作

### 创建仓库

创建仓库其实就是创建一个.git 文件夹来管理当前的目录。

```
git init    // 将当前目录创建为仓库
git clone url   // 下载远程的仓库
```

也可以不进行 init，直接通过 git clone 来获取远程仓库。

### 查看仓库

```
git status  //查看状态

git log //查看历史提交
git log --oneline   // 简略信息
git log -p  // 详细信息

git ls-files //查看暂存区文件
```

### 增加文件

```
git add filename

git add -all    // 添加所有文件
git add .
```

### 提交文件

```
// 会进入message编辑器，查看vim操作指令来编辑
git commit

git commit -m "message"
git commit -a -m "message"  // 提交全部文件
```

### 修改文件名称

```
git mv from.txt to.txt  // 在工作区和暂存区更改名称
git commit      // 在仓库更改名称
```

### 删除文件

```
rm file1.txt    // 工作区中删除文件
git add file1.txt //通知暂存区删除文件
git commit //通知本地仓库删除文件

git rm file1.txt // 工作区和暂存区中删除文件
git commit //通知本地仓库删除文件

// 当你不想再跟踪file.txt或者忘记将file.txt加入到.ignore中时
git rm --cached file.txt    // 在暂存区中删除，工作区保留
git commit  //在仓库中删除
```

### 清除 untracked 文件

```
// 对于untracked，可以用git clean移除
git clean

// 参数
git clean -f //强制删除文件
git clean -df //强制删除文件夹
git clean -n //查看要被删除的文件
git clean -dff  //强制删除被其它git管理的文件夹
```

注意：git clean 作用于当前目录下的 untracked 文件，如果要清除父目录下的 untracked 文件，需要在父目录下执行 git clean 命令。

### 忽略文件

创建.gitignore，在文件中声明要忽略的文件，可以使用正则表达式。

### 放弃修改
```bash
git restore .    // 放弃工作区的修改
git restore --staged .    // 放弃暂存区的修改
```

## 暂存操作

### Stash
当我们在工作区修改了一些文件，但是这些修改还不想提交，而此时又需要切换分支或者 pull 远程仓库的更新时，可以使用 stash 将当前的修改保存起来，等到需要的时候再恢复。

```
git stash          // 保存当前修改并恢复到上次提交的状态
git stash save "message"  // 保存当前修改并添加描述信息
git stash list    // 查看所有保存的修改
git stash apply   // 恢复最近一次保存的修改，但不删除 stash 记录
git stash apply stash@{n}  // 恢复指定的修改
git stash pop     // 恢复最近一次保存的修改，并删除该 stash 记录
git stash drop stash@{n}  // 删除指定的 stash 记录
git stash clear   // 删除所有的 stash 记录
```

## 远程仓库

### git clone

如果是通过 git clone 创建的 git 仓库，那么自带一个 origion 远程仓库。

git clone 默认只检出一个分支（通常是远程仓库的默认分支），但会下载所有分支的信息。

具体表现为：

* 所有分支的历史记录和元数据都会被下载到本地的 .git 目录
* 但工作区只显示默认分支（通常是 main 或 master）的内容


### 增删改查操作

```
git remote：列出当前仓库中已配置的远程仓库。
git remote -v：列出当前仓库中已配置的远程仓库，并显示它们的 URL。
git remote add <remote_name> <remote_url>：添加一个新的远程仓库。
git remote rename <old_name> <new_name>：将已配置的远程仓库重命名。
git remote remove <remote_name>：从当前仓库中删除指定的远程仓库。
git remote set-url <remote_name> <new_url>：修改指定远程仓库的 URL。
git remote show <remote_name>：显示指定远程仓库的详细信息，包括 URL 和跟踪分支。
```

### 获取权限

在尝试 git push 来修改远程仓库时需要访问权限，由于 2021 年后不允许使用账号密码登录，这里使用 personal access token 来进行授权。

1. https://github.com/settings/tokens 在这里创建 token
2. 用户认证时，用 token 作为密码

### 远程仓库密码存储

由于 http 协议的限制，每次 push 或者 pull 都需要输入账号密码，为此 git 提供了存储功能来记录认证信息，避免每次操作都需要手动输入账号密码：

```
// 缓存账号密码,15分钟后过期
git config --global credential.helper cache

// 磁盘存储账号密码，明文存储但不会过期
git config --global credential.helper store
```

### 同步仓库

```
// 获取远程仓库的内容，但不进行合并操作
git fetch <remote>

// 等同于git fetch + git merge
git pull

// 将branch分支推送到remote仓库的branch分支
git push <remote> <branch>
```


## 工作流操作

### 撤销操作
```
// 修改上一次提交（快照）
git commit --amend

// 撤销暂存区中对某个文件的修改
git restore --staged <file>

// 撤销工作目录中对某个文件的修改
git restore <file>

// 清除未跟踪的文件
git clean -fd <file>

// 如果已经add但是未提交
git reset        # 取消已暂存
git restore .    # 还原改动
git clean -fd    # 删除未追踪文件和目录
```

### Tag 管理

```
// 查看当前tag list
git tag

// 查看tag对应的提交信息
git show <tagname>

// 为最近一次提交添加tag
git tag <tagname>

// 为之前某一次提交添加tag
git tag <tagname> <hashcode>

// 删除tag
git tag -d <tagname>
```

### 版本控制

参考文献：https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86.html

```
git log     //查看提交记录的版本号
git reset versionNumber
```

reset 命令会以特定的顺序重写 HEAD，暂存区和工作目录。在你指定以下选项时停止：

1. 移动 HEAD 分支的指向 （若指定了 --soft，则到此停止）

2. 使索引看起来像 HEAD （若未指定 --hard，则到此停止）

3. 使工作目录看起来像索引

![Alt text](pic/gitReset.png)

但要注意的是，就是是使用了--hard 参数，也是有办法撤回的，可以使用 git --reflog 命令查看被删除的版本好，然后 reset 到那个版本。

### 比较差异

我们可以使用 git diff 命令来比较不同版本之间的差异。

![Alt text](pic/gitDIff.png)



## 分支操作

### 基本操作

```
// 查看分支
git branch
git branch -a  //查看所有分支，包括远程分支

// 创建分支
git branch <branchname>

// 切换分支
git checkout <branchname>

// 删除分支
git branch -d <branchname>
```

### 合并分支

```
// 将当前分支与<branchname>合并
git merge <branchname>
```

### 远程分支
在拉取仓库或者克隆仓库后，git clone 下载了所有分支的全部数据，但只给你创建一个可修改的本地分支。其他分支必须手动 checkout 才能在本地上修改。

**只拉取远程分支**

```
git clone -b <分支名> <仓库地址>
```

**切换到远程分支：**
```
git checkout -b <branchname> <remote>/<branchname>
```

这是创建一个新的本地分支，并将其设置为跟踪远程分支。之后就可以在本地分支上进行修改和提交了。

如果直接切换到远程分支：

```
git checkout <remote>/<branchname>
```

这是切换到一个分离头指针（detached HEAD）状态，在这个状态下你可以查看远程分支的内容，但不能直接修改和提交。要修改和提交，必须先创建一个新的本地分支并切换到它。

**操作远程分支：**

```
// 将本地branch分支推送到remote仓库的branch分支
git push <remote> <branch>

// 删除远程仓库中的分支
git push <remote> --delete <branch>
```

### 跟踪分支

由于每次 push 或者 pull 时都需要指出本地分支以及对应的远程分支，为了方便，可以将本地分支和远程分支进行绑定，从而成为跟踪分支。跟踪分支知道自己对应的远程分支，进行 push 和 pull 操作时更加简洁:

```
// 指定当前分支的远程分支为<remote>/<branch>
git branch -u <remote>/<branch>
git branch --set-upstream-to <remote>/<branch>

// 从远程分支创建跟踪分支
git checkout -b <branch> <remote>/<branch>

// 当前分支推送到对应的远程分支
git push

// 当前分支拉去对应的远程分支
git pull

// 查看分支对应的远程分支
git branch -vv
```

### 拷贝分支

如果我们想要将一个分支的内容拷贝到另一个分支上，可以使用以下命令：

```
git checkout A -- .// 将分支A的内容拷贝到当前分支
```


# 第三方软件
## github
### 配置 SSH 登录

1. 生成 SSH 密码对
    ```
    ssh-keygen
    ```
    根据提示填写密钥名和存储位置即可

2. 将公钥添加到 github 上

3. 配置 ssh，使用生成的私钥来访问目标 github 仓库

    编辑 ~/.ssh/config 文件，添加如下内容：

    ```
    Host [Host别名]
        HostName github.com         # github 地址
        port 22                     # 端口号
        User git                    # git服务的固定用户名
        IdentityFile ~/.ssh/[私钥名称]  # 私钥路径
    ```

4. 测试连接

    ```
    ssh -T git@[Host别名]
    ```

    如果连接成功，会返回github的用户名

5. 使用 ssh 地址来 clone 仓库

    ```
    git clone git@[Host别名]:path/repository.git
    ```

### quick start

```
git init
git commit -m message
git branch -M main
git remote add origin https:url
git push -u origin main
```

### 更改远程仓库

首先在 github 上创建一个仓库，这个仓库会有一个 ssh 连接，根据这个连接，我们把仓库下载下来

```
git clone git@github.com:ablllcd/Note.git
```

然后把文件拷贝到克隆下来的仓库中并且 commit。

最后将本地仓库给 push 到 github 上

```
git push
```
