## 介绍
git是用作版本控制的，先通过init创建一个仓库，仓库里的文件都将被git进行管理。

版本相当于对所有文件的当前状态拍一张照，它记录了所有文件当前的状态。而git来进行版本控制就是在当前文件夹下创建一个.git文件，该文件保存了要管理的文件夹的所有版本，每次commit就是将一个版本提交到仓库中去。（注意：每次提交的版本是当前所有文件的状体，而不是和上次版本的差异）

文件区域可以分为三个：工作区(working Directory)，暂存区(Staging Area)和本地仓库(Local Repository)。

工作区：文件存在的区域，也就是.gti存在的目录。

暂存区：存放即将提交的文件，位于.git/index。

本地仓库：存储文件和版本信息，位于.git/objects。

![Alt text](pic/workArea.png)

对应的，文件也可以被分为四个状态：

未跟踪：没有被git管理的文件。

未修改：跟踪后但是文件没有变化。

已修改：跟踪的文件并且发生了修改。

已缓存：被放进缓存区的文件。

![Alt text](pic/fileStatus.png)

由图可知文件的状态并非单向转变的。

## 配置
````
git config --global user.name "cian"
git config --global user.email a1401702664@gmail.com

//查看配置
git config user.name
git config user.email
git config --global --list
````

## 帮助
````
git command -h
git help command
````

## 仓库
#### 创建仓库
创建仓库其实就是创建一个.git文件夹来管理当前的目录。
````
git init    // 将当前目录创建为仓库
git clone url   // 下载远程的仓库
````
#### 查看仓库
````
git status  //查看状态

git log //查看历史提交
git log --oneline   // 简略信息
git log -p  // 详细信息

git ls-files //查看暂存区文件
````
#### 增加文件
````
git add filename

git add -all    // 添加所有文件
git add .

// 此时并未提交，只是在缓冲区，可以撤销
git rm --cached filename
````

#### 提交文件
````
// 会进入message编辑器，查看vim操作指令来编辑
git commit

git commit -m "message"
git commit -a -m "message"  // 提交全部文件
````

#### 删除文件
````
// 文件夹中删除文件
git add deletedFile //通知暂存区删除文件
git commit //通知本地仓库删除文件
````


#### 忽略文件
创建.gitignore，在文件中声明要忽略的文件，可以使用正则表达式。

## 版本控制
````
// git log 可以查看版本号
git reset versionNumber
````

git rest 可以将本地仓库回退到某次版本，而根据参数不同，对于工作区和暂存区的处理也不同。

git rest --soft: 并不会删除工作区和暂存区的内容，也就是文件夹中的内容并没有被删除。（这个操作主要是用来合并之前的多次版本）

git rest --mixed：这个是默认参数，删除缓存区的内容但保留工作区的内容。（也是用于合并版本，和soft无什么差别）

git rest --hard：这个会删除工作区和暂存区的内容，它旨在真的将文件回退到某个版本。


![Alt text](pic/gitReset.png)

但要注意的是，就是是使用了--hard参数，也是有办法撤回的，可以使用 git --reflog命令查看被删除的版本好，然后reset到那个版本。

## 比较差异
我们可以使用 git diff 命令来比较不同版本之间的差异。

![Alt text](pic/gitDIff.png)

## github
### 配置SSH
1. 首先生成ssh密钥对
````
C:\Users\Cc>ssh-keygen -t rsa -b 4096

// 选择密钥保存的位置
Enter file in which to save the key (C:\Users\Cc/.ssh/id_rsa): D:\Programming\Git\github-ssh-key
````
2. 将生成的公钥（*.pub）文件拷贝到github设置里的`SSH and GPG Keys`，选择new SSH Key。

3. 配置私钥来匹配github

    首先查看ssh-agent服务是否开启：

    1. cmd + services.msc后看OpenSSH Authentication Agent，右击开启

    2. 或者在powershell管理者模式中（不是命令行）
    ````
    Get-Service ssh*    //查看ssh-agent状态
    Start-Service ssh-agent  //开启服务
    ````

    其次添加私钥到agent里
    ````
    ssh-add "D:\Programming\Git\github-SSHkey\github-ssh-key"

    ssh-add -l  //查看已添加的私钥
    ````

    补充内容：配置.ssh文件无用，以及`ssh -i "C:\Users\john\.ssh\id_rsa" git@github.com`可以暂时连接密钥和网站，关闭命令行后失效。

### 同步仓库

首先在github上创建一个仓库，这个仓库会有一个ssh连接，根据这个连接，我们把仓库下载下来
````
git clone git@github.com:ablllcd/Note.git
````
然后把文件拷贝到克隆下来的仓库中并且commit。

最后将本地仓库给push到github上
````
git push
````