## 参考教程

https://www.runoob.com/linux/linux-shell.html

# 常用操作

## 系统操作

### 常用操作
```bash
#关机
shutdown -h now  # 立即关机
shutdown -h +10  # 10分钟后关机
reboot  # 重启系统
```

### 查看系统信息
1. 查看操作系统版本
   ```bash
   cat /etc/os-release  # 显示操作系统版本信息
   lsb_release -a        # 显示Linux标准基础信息
   hostnamectl           # 显示主机信息，包括操作系统版本
   ```

### 时间相关
1. 查看当前时间
   ```bash
   date  # 显示当前日期和时间
   date +%Y-%m-%d  # 显示当前日期
   date +%H:%M:%S  # 显示当前时间
   ```
2. 查看当前时间配置
   ```bash
   timedatectl  # 显示当前时间、时区和NTP状态
   ```
3. 列出可用时区
   ```bash
    timedatectl list-timezones  # 列出所有可用的时区
   ```
4. 修改时区
   ```bash
    sudo timedatectl set-timezone Asia/Shanghai  # 设置时区为上海
   ```
5. date进行时间格式转化
    ```bash
    # 将时间戳转换为UTC格式     
     date -u -d @<timestamp> +"%Y-%m-%dT%H:%M:%SZ"
    ```

### 查看系统日志
```bash
journalctl  # 显示系统日志
journalctl -u service_name  # 显示指定服务的日志
journalctl -f  # 实时跟踪日志输出
journalctl --since "2024-01-01" --until "2024-01-31"  # 显示指定日期范围内的日志
journalctl -n 100  # 显示最近的100条日志
journalctl -k # 显示内核日志
```

### 查看操作日志
```bash
last  # 显示最近的登录记录
last reboot  # 显示系统重启记录
history # 显示命令历史记录
```

### systemctl
systemctl 是 Linux 系统中用于 控制 systemd 系统和服务管理器 的命令行工具。而systemd 是一种初始化系统（init system），负责在系统启动时启动服务、挂载文件系统、管理日志等。systemctl 就是与 systemd 交互的工具。

通过 systemctl 管理的服务是系统服务（daemon），通常在后台运行，提供持续的功能，比如数据库、Web 服务器、网络服务等。
其特点为：
* 后台运行
* 需要系统启动时自动启动
* 由 systemd 管理生命周期（启动、停止、重启等）

**常用命令**
```bash
sudo systemctl start service_name    # 启动服务
sudo systemctl stop service_name     # 停止服务
sudo systemctl restart service_name  # 重启服务
sudo systemctl enable service_name   # 设置服务开机自启
```

## 环境变量

### CRUD环境变量
```bash
export VAR_NAME=value  # 创建环境变量 VAR_NAME 并赋值为 value
echo $VAR_NAME         # 查看环境变量 VAR_NAME 的值
export VAR_NAME=new_value  # 更新环境变量 VAR_NAME 的值为 new_value
unset VAR_NAME         # 删除环境变量 VAR_NAME
```

### 查找环境变量
```bash
printenv | grep VAR_NAME  # 查找包含 VAR_NAME 的环境变量
env | grep VAR_NAME       # 查找包含 VAR_NAME 的环境变量
```

### 应用环境变量
`export` 命令用于在当前 shell 环境中设置环境变量，使其在当前 shell 以及其子进程中可用。所谓的子进程是指由当前 shell 启动的任何程序或脚本。
```
export VAR_NAME=value  # 设置环境变量 VAR_NAME 的值为 value
```

## 用户管理

### 查看用户信息
```bash
cat /etc/passwd  # 显示系统中的用户信息
cat /etc/group   # 显示系统中的用户组信息
id username      # 显示指定用户的UID、GID和所属组信息
groups username  # 显示指定用户所属的所有组
```

### 编辑用户
```bash
sudo adduser username  # 创建新用户并设置密码
sudo userdel username  # 删除用户
sudo passwd username   # 修改用户密码
```

### 修改用户权限
```bash
sudo usermod -aG groupname username  # 将用户添加到指定组
sudo usermod -G groupname username     # 将用户的组设置为指定组（会覆盖原有组）
sudo usermod -aG sudo username        # 将用户添加到 sudo 组，赋予管理员权限
```

### 切换用户
```bash
su - username  # 切换到指定用户并加载其环境变量
```


## 文本操作

### grep命令
`grep` 是 Linux 系统中用于在文本文件中搜索特定字符串或模式的命令行工具。它可以根据用户提供的模式（通常是正则表达式）在文件中查找匹配的行，并将这些行输出到终端。

**常用选项**
- `-i`：忽略大小写进行匹配。
- `-r`：递归搜索目录中的文件。

**正则表达式**
- `.`：匹配任意单个字符。
- `*`：匹配前一个字符零次或多次。
- `^`：匹配行的开头。

## 磁盘操作

### 分区表类型

1. MBR（Master Boot Record）：传统的分区表类型，支持最多4个主分区或3个主分区加1个扩展分区。每个分区最大支持2TB。

2. GPT（GUID Partition Table）：现代的分区表类型，支持更多的分区（理论上最多128个）和更大的磁盘（超过2TB）。GPT使用GUID来标识分区，提供更好的数据完整性和恢复能力。

### 查看分区表
```bash
sudo parted -l # 通过Partition Table 字段查看分区表类型
sudo fdisk -l  # 通过Disklabel type字段查看分区表类型
```

### 文件系统类型

1. ext4：Linux系统中`最常用`的文件系统，支持大文件和大分区，具有较好的性能和稳定性。但是Windows系统无法原生识别ext4文件系统，需要第三方工具才能访问。

2. NTFS：Windows系统中常用的文件系统，支持大文件和大分区，但在Linux上通常以只读方式挂载。

3. FAT32: 兼容性较好，适用于U盘和移动存储设备，可以被Linux系统和Windows系统读取，但不支持大于4GB的单个文件。

4. exFAT: 兼容性较好，适用于U盘和移动存储设备，可以被Linux系统和Windows系统读取，支持大于4GB的单个文件。


### 查看磁盘用量
1. `df` 命令 - 显示文件系统磁盘空间使用情况
   ```bash
   df -h  # 以人类可读的方式显示（GB、MB等）
   df -T  # 显示文件系统类型
   df -i  # 显示inode信息而非块使用量
   ```

2. `du` 命令 - 估算文件和目录的磁盘使用量
   ```bash
   du -h /path  # 以人类可读方式显示指定目录的大小
   du -sh *     # 显示当前目录下各文件和目录占用空间
   du -sh /path # 显示指定目录总大小
   du -h --max-depth=1 /path  # 仅显示第一级子目录的大小
   ```
### 查看磁盘以及分区信息
1. `fdisk` 命令 - 查看磁盘分区（较详细）
   ```bash
   sudo fdisk -l  # 列出所有磁盘的分区表
   ```

2. `lsblk` 命令 - 以树状列出所有块设备（较直观）
   ```bash
   lsblk         # 列出所有块设备
   lsblk -f      # 显示文件系统信息
   ```

3. `parted` 命令 - 查看磁盘分区
   ```bash
   sudo parted -l  # 列出所有磁盘的分区表
   ```

4. `blkid` 命令 - 显示块设备的UUID和文件系统类型
   ```bash
   sudo blkid     # 显示所有块设备的UUID和文件系统类型
   ```

### parted 用法

```bash
sudo parted /dev/sdX  # 指出要操作的磁盘

(parted) print  # 显示磁盘分区信息

(parted) mkpart primary ext4 1MiB 100%  # 创建一个新的主分区，使用ext4文件系统，从1MiB开始到磁盘末尾

(parted) rm 1  # 删除分区1

(parted) resizepart 1 1MiB 50%  # 将分区1的大小调整为从1MiB到磁盘的50%

(parted) resizepart 2 80GB # 将分区2的大小(结束位置）调整为80GB
```

### 缩小磁盘分区

1. 修改磁盘前，要保证磁盘没有被挂载/使用，所以通常以live CD的方式进入系统来修改磁盘分区。

2. 先缩小文件系统的大小 (防止文件系统超过分区大小)
    ```bash
    sudo e2fsck -f /dev/sdXn  # 检查文件系统完整性，构建空闲空间地图
    sudo resize2fs /dev/sdXn size  # 将文件系统缩小到指定大小（size可以是K、M、G等单位）
    ```

3. 再缩小分区的大小
    ```bash
    sudo parted /dev/sdX  # 使用parted工具调整分区大小
    ```

### 扩展磁盘分区

1. 无需卸载磁盘，可以在系统运行时直接扩展分区。

2. 先扩展分区的大小
    ```bash
    sudo parted /dev/sdX  # 使用parted工具调整分区大小
    ```

3. 再扩展文件系统的大小
    ```bash
    sudo resize2fs /dev/sdXn  # 将文件系统扩展到分区的最大大小
    ```

## 文件操作

### 文件权限
```bash
ll  # 显示文件权限和所有者信息
chmod 755 filename  # 设置文件权限为 rwxr-xr-x
chmod -r 755 directory  # 递归设置目录及其内容的权限
chmod u+x filename  # 给文件所有者添加执行权限
chmod g-w filename  # 移除文件所属组的写权限
chmod o+r filename  # 给其他用户添加读权限
chown user:group filename  # 更改文件所有者和所属组
```
* 权限分为三类：所有者（user）、所属组（group）和其他用户（others）。每类权限可以分别设置读（r）、写（w）和执行（x）权限。
* 权限的数值表示：读（r）、写（w）和执行（x）。每个权限对应一个数字：读=4，写=2，执行=1。权限的总和决定了文件的权限设置。例如，755表示所有者有读、写、执行权限（4+2+1=7），而组和其他用户只有读和执行权限（4+1=5）。
* chmod就是用来修改文件权限的命令，chown用来修改文件的所有者和所属组。

### 统计文件夹
```bash
find /path/to/directory -type f | wc -l  # 统计指定目录下的文件数量
# 显示文件数量 
```

### 移动和重命名文件
```bash
mv oldname newname  # 重命名文件或移动文件到新位置
mv filename /path/to/directory/  # 移动文件到指定目录
```

### 复制文件
```bash
cp source_file destination_file  # 复制文件
cp -r source_directory destination_directory  # 递归复制目录
cp -r source_directory/* destination_directory/  # 复制目录下的所有内容到目标目录
```

### 查找文件
```bash
find /path/to/search -name "filename"  # 在指定路径下查找文件
find /path/to/search -type f -name "*.txt"  # 查找指定路径下的所有 .txt 文件
```

## 下载/安装/解压操作

### 压缩文件
```bash
tar -cvf file.tar /path/to/directory  # 将目录压缩为 .tar 文件
tar -czvf file.tar.gz /path/to/directory  # 将目录压缩为 .tar.gz 文件
tar -cjvf file.tar.bz2 /path/to/directory  # 将目录压缩为 .tar.bz2 文件
zip -r file.zip /path/to/directory  # 将目录压缩为 .zip 文件
rar a file.rar /path/to/directory  # 将目录压缩为 .rar 文件 
7z a file.7z /path/to/directory  # 将目录压缩为 .7z 文件
xz -z file  # 将文件压缩为 .xz 文件
gzip file  # 将文件压缩为 .gz 文件
```

### 解压文件

1. 解压 `.tar` 文件
    ```bash
    tar -xvf file.tar
    ```

2. 解压 `.tar.gz` 或 `.tgz` 文件
    ```bash
    tar -xzvf file.tar.gz
    ```

3. 解压 `.tar.bz2` 文件
    ```bash
    tar -xjvf file.tar.bz2
    ```

4. 解压 `.zip` 文件
    ```bash
    unzip file.zip
    unzip -d /path/to/destination file.zip  # 解压到指定目录
    ```

5. 解压 `.rar` 文件
    ```bash
    unrar x file.rar
    ```

6. 解压 `.7z` 文件
    ```bash
    7z x file.7z
    ```

7. 解压 `.xz` 文件
    ```bash
    xz -d file.xz
    ```

8. 解压 `.gz` 文件
    ```bash
    gunzip file.gz
    ```

### 使用 wget 下载文件
```bash
wget http://example.com/file.zip  # 下载指定URL的文件
wget -c http://example.com/file.zip  # 断点续传下载 
wget -r http://example.com/dir/  # 递归下载目录
```

### 使用apt-get下载软件包
```bash
sudo apt-get update  # 更新软件包列表
sudo apt-get install package_name  # 安装指定软件包
sudo apt-get remove package_name  # 卸载指定软件包
sudo apt-get upgrade  # 升级已安装的软件包
sudo apt-get dist-upgrade  # 升级系统，包括内核和依赖关系
```

### 安装软件包
```bash
sudo dpkg -i package.deb  # 安装 .deb 软件包
sudo rpm -i package.rpm  # 安装 .rpm 软件包
```


## 网络操作

### 配置网络IP
```bash
dhclient # 获取动态IP地址 & 临时生效
```

#### netplan配置静态IP
在 /etc/netplan/ 目录下创建一个 .yaml 文件，内容如下：
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:[8.8.8.8, 8.8.4.4]
```
然后运行以下命令应用配置：
```bash
sudo netplan apply
```

### 修改网口名称
```bash
# 1. 查看要修改的网口MAC地址
ip a

# 2. 编辑/创建配置文件
nano /etc/udev/rules.d/10-persistent-net.rules

# 3. 添加规则，格式如下：
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="<MAC地址>", NAME="<新网口名称>"

# 4. 重载udev规则
sudo udevadm control --reload-rules && sudo udevadm trigger

# 5. 重启
reboot
```

### 查看网络配置
```bash
ifconfig          # 显示网络接口配置
ip addr show      # 显示网络接口的详细信息
ip link show      # 显示网络接口的状态
ip a              # 显示所有网络接口信息
ip route show      # 显示路由表
ip route | grep default  # 显示默认路由
```

### 监控网络变化
```bash
watch -n 1 "ip addr show"  # 每秒刷新显示网络接口信息
watch -n 1 "ip route show"  # 每秒刷新显示路由表
ip monitor link      # 实时监控网络接口状态变化
```

### 临时添加/删除IP地址
```bash
sudo ip addr add 192.168.1.100/24 dev eth0  # 临时添加IP地址
sudo ip addr del 192.168.1.100/24 dev eth0  # 临时删除IP地址
```

### 临时修改网关
```bash
ip route del default    # 删除当前默认路由
ip route add default via <新网关IP> dev <网卡名称>  # 添加新的默认路由
```

### 查看端口占用

1. `netstat` 命令 - 显示网络连接、路由表、接口统计等
    ```bash
    netstat -tulpn  # 显示所有监听的端口
    ```
    - 参数说明:
        - `-t`: 显示TCP连接
        - `-u`: 显示UDP连接
        - `-l`: 显示监听状态的端口
        - `-p`: 显示进程ID和名称
        - `-n`: 显示数字地址而不解析主机


2. `ss` 命令 - 更快的网络状态查看工具
    ```bash
    ss -tuln       # 显示所有监听的端口
    ss -anp        # 显示所有端口及其对应的进程
    ```

3. `lsof` 命令 - 列出打开的文件，包括网络端口
    ```bash
    lsof -i :port  # 查看指定端口的占用情况
    lsof -i        # 查看所有网络连接
    ```

4. `fuser` 命令 - 显示使用指定端口的进程
    ```bash
    fuser -n tcp port  # 查看指定TCP端口的占用情况
    ```

### 查看网卡信息
```bash
ip a # 显示所有网络接口信息
ip a | grep TARGET_IP # 查找包含指定IP地址的网络接口信息
```

### 网络抓包
```bash
sudo tcpdump -i eth0  -w capture.pcap  # 在eth0接口上抓包并保存到capture.pcap文件

# 抓目的为 xxx、端口为 yy 的包，并把 payload 也打印（适合 HTTP）
sudo tcpdump -i any -nn -s0 -A "dst host xxx and dst port yy"
```

## 进程操作

### 查看进程以及资源占用

1. `ps` 命令 - 显示当前进程信息
    ```bash
    ps            # 显示当前用户的进程
    ps aux        # 显示所有进程的详细信息
    ps -ef        # 显示所有进程的完整格式
    ```

2. `top` 命令 - 实时显示系统资源使用情况
    ```bash
    top           # 显示实时进程和资源使用情况
    htop          # 更友好的交互式界面（需安装：sudo apt-get install htop）
    ```

3. `pidstat` 命令 - 显示进程的资源使用情况
    ```bash
    pidstat -u    # 显示进程的CPU使用情况
    pidstat -r    # 显示进程的内存使用情况
    pidstat -d    # 显示进程的I/O使用情况
    ```

4. `vmstat` 命令 - 显示系统性能统计
    ```bash
    vmstat 1      # 每秒更新一次系统性能统计
    ```

5. `iostat` 命令 - 显示CPU和磁盘I/O使用情况
    ```bash
    iostat -x     # 显示详细的CPU和磁盘I/O使用情况
    ```

6. `sar` 命令 - 收集、报告系统活动
    ```bash
    sar -u 1      # 每秒显示一次CPU使用情况
    sar -r 1      # 每秒显示一次内存使用情况
    ```

7. `pmap` 命令 - 显示进程的内存映射
    ```bash
    pmap pid      # 查看指定进程的内存使用情况
    ```

### 查找进程
1. 根据端口号查找进程
    ```bash
    lsof -i :port  # 查找占用指定端口的进程
    netstat -tulpn | grep :port  # 查找占用指定端口的进程
    ss -tuln | grep :port  # 查找占用指定端口的进程
    ```

### 终止进程
1. 使用 `kill` 命令终止进程
    ```bash
    kill pid              # 发送默认的TERM信号终止进程
    kill -9 pid           # 强制终止进程
    ```
2. 使用`pkill`命令根据进程名称终止进程
    ```bash
    pkill process_name     # 根据进程名称终止进程
    pkill -9 process_name  # 强制根据进程名称终止进程
    ```

## GPU相关操作

### 查看GPU信息
```bash
nvidia-smi  # 显示NVIDIA GPU的状态和使用情况
```
openebs-3.3.1
## 文本编辑器

### nano

#### 文件操作
```bash
Ctrl + O  # 保存文件
Ctrl + X  # 退出nano
```

#### 编辑操作
```bash
Ctrl + K  # 剪切当前行
alt +6    # 拷贝选中的内容
Ctrl + U  # 粘贴剪切的行
```

#### 查找操作
```
Ctrl + W  # 查找关键词
alt + G  #  输入行号，跳转到指定行
```

# Shell相关

## 环境变量
### CRUD环境变量
```bash
export VAR_NAME=value  # 创建环境变量 VAR_NAME 并赋值为 value
echo $VAR_NAME         # 查看环境变量 VAR_NAME 的值
export VAR_NAME=new_value  # 更新环境变量 VAR_NAME 的值为 new_value
unset VAR_NAME         # 删除环境变量 VAR_NAME
```

这些只是在当前 shell 会话中生效，如果想要永久生效，需要将 export 命令添加到用户的 shell 配置文件中，如 ~/.bashrc。

## 快捷操作

### 执行历史命令
```bash
!!  # 执行上一条命令
!n  # 执行历史记录中第 n 条命令
!string  # 执行历史记录中以 string 开头的命令
```

### 编辑文本
```bash
Ctrl + A  # 移动光标到行首
Ctrl + E  # 移动光标到行尾
Ctrl + U  # 删除光标前的文本 (U: "Up")
Ctrl + K  # 删除光标后的文本 (K: "Kill")
```

# 概念讲解

## 下载

### APT

APT（Advanced Package Tool）是Debian及其衍生发行版（如Ubuntu）中用于管理软件包的工具。它提供了一套命令行工具，用于安装、升级、删除和管理软件包。简单来说，APT是ubuntu系统中用来管理软件的软件包管理器,方便用户从`可靠来源`下载和安装软件。

APT的主要功能包括：
1. 软件包安装：通过APT，用户可以轻松地从软件仓库中安装所需的软件包。
2. 软件包升级：APT可以自动检查并升级已安装的软件包，确保系统保持最新状态。
3. 依赖管理：APT会自动处理软件包之间的依赖关系，确保安装的软件包能够正常运行。

#### apt的软件从哪来的？

APT从软件仓库（repository）中获取软件包。软件仓库是一个集中存储和分发软件包的服务器，通常由操作系统的维护者或第三方组织管理。APT通过配置文件（如`/etc/apt/sources.list`）指定要使用的软件仓库地址。

#### apt的软件下载/安装到哪？

APT下载的软件包通常存储在本地的缓存目录中，默认情况下，这个目录是`/var/cache/apt/archives/`。当用户使用APT安装软件包时，APT会先从指定的软件仓库下载软件包并将其存储在这个缓存目录中，然后再进行安装。安装位置则取决于软件包的类型和配置，通常会安装在系统的标准目录中，如`/usr/bin/`、`/usr/lib/`等。

# 网络配置深入理解

## linux中域名到IP的解析过程

### 一句话总览

程序自己既不读 `/etc/hosts`，也不直接发 DNS 查询；它调用 glibc（Linux 上最常用的标准 C 库，**NSS 就是它提供的机制**）的解析函数，由 **NSS（Name Service Switch）** 按 `/etc/nsswitch.conf` 里 `hosts:` 一行的顺序，依次去问各个“名字服务”，谁先给出结果就用谁。

```
应用程序（curl / ping / ssh / getent / 浏览器…）
      │ 调用 glibc：getaddrinfo() / gethostbyname()（反查用 getnameinfo()）
      ▼
┌────────────────────────────────────────────────────────────────┐
│ glibc 解析入口：读 /etc/nsswitch.conf 的 hosts: 行，按顺序调用模块 │
└────────────────────────────────────────────────────────────────┘
      │
      ├─ files         → 读 /etc/hosts（本地静态映射）
      ├─ mdns4_minimal → Avahi，只解析 .local（局域网 mDNS）
      ├─ dns           → libnss_dns 按 /etc/resolv.conf 发 DNS 查询（细节见下文「DNS 深入」）
      ├─ resolve       → libnss_resolve 经 unix socket 问 systemd-resolved（同上）
      └─ myhostname    → 本机名 / localhost / _gateway / _outbound
```

### 1. 查询顺序由 /etc/nsswitch.conf 决定

```
hosts:          files mdns4_minimal [NOTFOUND=return] dns myhostname
```

* 从左到右依次尝试，每个模块返回一个状态：`success`（找到，默认 `return` 立即返回）、`notfound`（没这个条目，默认 `continue` 继续问下一个）、`unavail`（该服务不可用，默认 `continue`）、`tryagain`（临时失败，默认 `continue`）。
* 可以用 `[状态=动作]` 改写默认行为：`[NOTFOUND=return]` 表示“前一个模块说找不到就立刻停止，不要再问后面的”。`!` 是取反，例如 systemd 推荐的 `resolve [!UNAVAIL=return]`：resolved 在跑但查不到 → 停止；resolved 没跑（UNAVAIL）→ 继续问后面的 `dns`。
* 常见取值（各发行版不同，**以本机文件为准**）：
  * Debian/Ubuntu：`hosts: files mdns4_minimal [NOTFOUND=return] dns myhostname`
  * systemd 官方推荐：`hosts: mymachines resolve [!UNAVAIL=return] files myhostname dns`
  * 极简 / 容器：`hosts: files dns`
* 如果 `/etc/nsswitch.conf` 不存在，glibc 会退回内置默认（`hosts` 通常是 `files dns`）；glibc 2.33 起配置文件被修改会自动重读，更老的版本只在进程第一次查询时读一次。

### 2. 各 NSS 模块到底做了什么

| 模块 | 实现 | 数据来源 / 行为 |
| --- | --- | --- |
| `files` | libnss_files.so | 读 `/etc/hosts`（`IP 主机名 [别名…]`），改完立即生效 |
| `dns` | libnss_dns.so | 按 `/etc/resolv.conf` 的 `nameserver/search/options` 发真正的 DNS 查询；glibc 自身不缓存（详见下文「DNS 深入」） |
| `resolve` | libnss_resolve.so | 经 AF_UNIX socket `/run/systemd/resolve/io.systemd.Resolve` 问 systemd-resolved（**不经过 127.0.0.53**） |
| `mdns4_minimal` | nss-mdns + avahi-daemon | 只在 `.local` 域（且只有两个 label，`host.local` 行，`a.b.local` 不管）作主；对其它名字返回 UNAVAIL，所以不会挡住正常 DNS |
| `myhostname` | libnss_myhostname.so | 本机名 → 本机所有 IP（都没有则 127.0.0.2/::1）；`localhost`、`localhost.localdomain`、`*.localhost` → 127.0.0.1/::1；`_gateway` → 默认网关；`_outbound` → 对外通信的源地址 |
| `mymachines` | libnss_mymachines.so | 本机 systemd 容器 / 虚拟机名 |
| `nscd` | 不是 nsswitch 里的模块 | glibc 在走模块之前会先问的缓存守护进程（装了才有），典型的“改了 hosts 不生效”元凶 |
| 其他 | `wins`（NetBIOS）、`nis`/`nisplus`、`ldap`、`sssd` | 企业 / 遗留环境 |

`myhostname` 官方推荐位置是 **`files` 之后、`dns` 之前**（既能兜底本机名，又允许用 `/etc/hosts` 覆盖）；它同时也支持反向解析。

### 3. 反向解析（IP → 域名）走同一行

`getnameinfo()` / `gethostbyaddr()` 同样按 `hosts:` 的顺序查：`files` 查 `/etc/hosts`，`dns` 发 PTR 查询（`in-addr.arpa` / `ip6.arpa`），`myhostname` / `resolve` 会先认领本地 IP。所以日志里看到的反解结果可能来自 `/etc/hosts` 而根本不是 DNS。

### 4. 不是所有程序都走 NSS（最容易踩的坑）

| 程序 | 是否走 glibc NSS |
| --- | --- |
| curl / ping / ssh / wget / getent / 大多数程序 | ✅ 走 |
| nslookup / dig / host / drill | ❌ 自己按 `/etc/resolv.conf` 直接发 DNS 查询，**看不到 /etc/hosts**（nss-mdns 官方文档明确提醒：测试 `.local` 别用 nslookup/host） |
| resolvectl query | ❌ 直接问 systemd-resolved |
| musl libc 程序（Alpine） | ❌ musl 没有 NSS，自己读 `/etc/hosts` 和 `/etc/resolv.conf`，`nsswitch.conf` 无效 |
| Go 程序（`CGO_ENABLED=0`，多数静态镜像） | ❌ 自带纯 Go 解析器，自己读 `/etc/hosts`、`/etc/resolv.conf`，对 `nsswitch.conf` 支持非常有限 |
| 开启 DoH 的浏览器 | ⚠️ 可能完全绕过系统解析 |
| Java / JVM | ✅ 最终调 getaddrinfo，但自带缓存（`networkaddress.cache.ttl`） |

所以“nslookup 查不到 /etc/hosts 里的记录”是设计如此，不是故障。

### 5. 排查命令

```bash
getent hosts   example.com   # 完整走 NSS（hosts 数据库），与普通程序看到的结果一致
getent ahosts  example.com   # 走 getaddrinfo，能看到 v4/v6 地址与顺序
resolvectl query example.com # 直接问 systemd-resolved，不经过 NSS
resolvectl status            # 每块网卡的 DNS、split DNS、DNSSEC / DoT 状态
resolvectl statistics        # 缓存命中情况
dig / nslookup example.com   # 只测 DNS 服务器本身
cat /etc/nsswitch.conf; cat /etc/resolv.conf; cat /etc/hosts    # 查看配置文件
strace -f -e trace=openat,connect,sendto getent hosts example.com  # 看真实走了哪条路
```

## DNS 深入：resolv.conf、dns 模块与 systemd-resolved

上一节讲的是“**谁来决定问谁**”（NSS 按顺序挑模块），这一节讲“**DNS 本身是怎么被问出去的**”。

### 1. 一次公网域名解析里的三个角色

| 角色 | 例子 | 干什么 |
| --- | --- | --- |
| stub resolver（本机解析器） | NSS 的 `dns` 模块、systemd-resolved、musl | 按本机配置把查询转发给上游，**自己不做递归** |
| 递归解析器 / 缓存 DNS | 运营商 DNS、8.8.8.8、公司内网 DNS、dnsmasq | 代你从根开始逐级问，并缓存结果 |
| 权威服务器 | 根服务器 → `.com` 的 TLD 服务器 → `example.com` 的 NS | 保存并回答某个域的记录 |

实际查询是**递归解析器**干的：问根 → 拿到 `.com` 的 TLD 服务器 → 拿到 `example.com` 的权威服务器 → 拿到 `A` 记录，然后按 TTL 缓存。本机只负责“问上游”和“查本机缓存”，不逐级迭代。

>> 在netplan里配置的或者DHCP中拿到的 DNS 服务器地址就是递归解析器。

>> stub resolver就是本机负责进行DNS查询的起点，例如：NSS 的 `dns` 模块。

### 2. NSS 的 dns 模块读什么：/etc/resolv.conf

>> 下文说的「NSS 的 `dns` 模块」就是 `nsswitch.conf` 里 `hosts:` 行写的那个 `dns`，由 glibc 自带的 `libnss_dns` 实现——「NSS 的 dns 模块」和「glibc 的 dns 模块」是同一个东西。

```conf
nameserver 127.0.0.53                  # 最多写 3 个，按顺序尝试，超时换下一个
search corp.example.com example.com    # 短名会依次拼接这些后缀
options ndots:1 timeout:5 attempts:2 rotate edns0 trust-ad
```

* `nameserver`：最多 3 个（`MAXNS`），按列出顺序查询；都不写时默认查本机。
* `search`：当名字里的点**少于 `ndots`**（默认 1）时，先拼接每个 search 后缀试一遍，再按绝对名字试。`domain` 是 `search` 的过时同义词（只接受一个）。
* `options` 常用项：
  * `ndots:n`（默认 1，上限 15）：控制“先拼后缀还是先当绝对名字”。
  * `timeout:n`（默认 5 秒，上限 30）、`attempts:n`（默认 2 次，上限 5）。
  * `rotate`（轮询 nameserver）、`edns0`、`use-vc`（强制 TCP）。
  * `single-request`：把并行的 A / AAAA 查询改成串行，用于兼容老设备。
  * `trust-ad`：保留 DNSSEC 的 AD 位。**用 127.0.0.53 stub 时必须有它**，否则上游验证过的 DNSSEC 结果到应用手里就丢了。
* 可按进程覆盖：环境变量 `LOCALDOMAIN=...` 改 search，`RES_OPTIONS=...` 改 options。
* `/etc/resolv.conf` 不存在时只查本机；glibc 2.26 起 `search` 不再限制 6 个域名 / 256 字符。
* **NSS 的 `dns` 模块自己不缓存**：查一次就真发一次查询。缓存只可能出现在 nscd、systemd-resolved、dnsmasq 或应用里。

谁在写这个文件：systemd-resolved（软链）、NetworkManager、dhclient / resolvconf、netplan、Docker（容器内写 `127.0.0.11`）、Kubernetes（写 `options ndots:5`，这是 k8s 里短名解析“多绕几圈”的根源）。

### 3. systemd-resolved 是什么

#### 打个比方

DNS 查询就像寄信，程序只知道“把信投给谁”。systemd-resolved 就是 Ubuntu 装在你本机上的一个**中转站**：

* 它自己**不会**从根服务器开始逐级去问（不做递归），只负责把信转给上游 DNS（运营商、公司 DNS、8.8.8.8 这些）；
* 它有个小本本：同样的问题答过一次就记下来，下次直接回答（**缓存**）；
* 它顺便管着“本机该用哪些上游 DNS、哪个域名该问哪一台”。

一句话：它是**本机的 DNS 管家**，不是“另一台 DNS 服务器”。

#### 它为什么存在

1. **缓存**：同一个域名不用每次都去问上游，更快。
2. **统一配置**：所有程序共用一份上游 DNS 设置，不用各自去读 `/etc/resolv.conf`。
3. **按域名分流**（split DNS）：公司域名走 VPN 的 DNS，其它走公网 DNS。
4. **顺带支持**：加密查询（DoT）、防篡改校验（DNSSEC）、局域网名字（mDNS / LLMNR），以及 `localhost`、本机名、`_gateway` 这些特殊名字。

#### 程序怎么找到它：两个入口

| 入口 | 地址 | 谁从这里进 |
| --- | --- | --- |
| ① 装成一台普通 DNS 服务器 | `127.0.0.53:53` | NSS 的 `dns` 模块（因为 `/etc/resolv.conf` 里写的就是它）、`dig`/`nslookup`、Go 程序、浏览器 |
| ② systemd 的私有通道 | unix socket `/run/systemd/resolve/io.systemd.Resolve`（不走网络端口） | nsswitch 里配的 `resolve` 模块、`resolvectl` 命令 |

入口 ① 走的是**标准 DNS 协议**，从报文上看不出 resolved 的存在；入口 ② 是 systemd 自己的内部通道，能附带更多信息（结果来自哪块网卡、有没有校验过、是不是命中缓存）。

#### 那 `dns` 模块和 resolved 是什么关系

**没有直接关系**：`dns` 模块根本不知道有 resolved 这个东西，它只会照着 `/etc/resolv.conf` 里写的地址发查询。所谓“查询走了 resolved”，只是因为那个地址恰好是 resolved 的 `127.0.0.53`。

所以“`dns` 模块和 resolved 的关系”只有三种情况：

| nsswitch 的 `hosts:` 行 | /etc/resolv.conf | 实际路径 |
| --- | --- | --- |
| `... dns ...` | 指向 `127.0.0.53`（软链，Ubuntu 默认） | `dns` 模块 → 127.0.0.53:53 → resolved → 上游 |
| `... resolve ...` | 写什么无所谓 | `nss-resolve` → unix socket → resolved（**完全不看** resolv.conf 里的 nameserver） |
| `... dns ...` | 写真实 DNS 或 dnsmasq 的 `127.0.0.1` | `dns` 模块 → 直接问那台 DNS，resolved 不参与 |

>> 记住一句话：**看 `/etc/resolv.conf` 的是 `dns` 模块，看 resolved 的是 `resolve` 模块。**（nsswitch 里配了 `resolve` 时，resolv.conf 写什么都不影响 DNS 查询。）

#### /etc/resolv.conf 在 Ubuntu 上长什么样

**不要手改 `/etc/resolv.conf`**，它通常是个软链，改了也白改：

* `/etc/resolv.conf` → `/run/systemd/resolve/stub-resolv.conf`：里面就是 `nameserver 127.0.0.53`（外加 search 域），Ubuntu 默认，推荐。
* `/run/systemd/resolve/resolv.conf`：里面是 resolved 知道的**真实上游 DNS 地址**，给那些不认 `127.0.0.53` 的程序或容器用。
* 想知道当前是哪种：`readlink -f /etc/resolv.conf`。

#### 怎么改resolved的配置

改 `/etc/systemd/resolved.conf`（或者放一份 `resolved.conf.d/*.conf`），然后 `sudo systemctl restart systemd-resolved` 生效。

| 配置项 | 作用 |
| --- | --- |
| `DNS=` | 用哪些上游 DNS |
| `FallbackDNS=` | 谁都没给 DNS 时的兜底；留空表示禁用 |
| `Domains=` | 哪些域名归它管，用来做分流；`~.` 表示“其余的都归我” |
| `DNSOverTLS=` | 是否加密查询：`no` / `opportunistic` / `yes` |
| `DNSSEC=` | 是否做防篡改校验：`no` / `allow-downgrade` / `yes` |
| `LLMNR=` / `MulticastDNS=` | 是否解析局域网里的名字 |
| `Cache=` / `DNSStubListener=` | 是否缓存 / 是否监听 `127.0.0.53:53` |

（还有 `DNSStubListenerExtra=`、`ResolveUnicastSingleLabel=` 等冷门项，用到再查手册。）

例：全局使用加密 DNS（DoT）

```ini
# /etc/systemd/resolved.conf.d/dot.conf
[Resolve]
DNS=9.9.9.9#dns.quad9.net 149.112.112.112#dns.quad9.net
DNSOverTLS=true
Domains=~.
```

> 想知道它当前到底在用哪些 DNS、每块网卡分别是什么，用 `resolvectl status` 看。

#### netplan / DHCP 里的 DNS 是怎么进 resolved 的

resolved 里的 DNS 有**两个来源**，互不覆盖：

| 来源 | 在哪 | 谁写 |
| --- | --- | --- |
| 手写的全局设置 | `/etc/systemd/resolved.conf`（+ `resolved.conf.d/*.conf`） | 你自己 |
| 每块网卡的设置 | 不写进任何配置文件，只在 resolved 运行时内存里（重启后由网络程序重新推一次） | netplan / DHCP / VPN **自动**推给它 |

所以：**netplan 和 DHCP 里的 DNS 不会写进 `/etc/systemd/resolved.conf`**。网络管理程序（systemd-networkd 或 NetworkManager）是绕开配置文件、直接通过 D-Bus 告诉 resolved：“eth0 这块网卡用这些 DNS、管这些域名”。

路径大致是这样：

```
netplan 的 yaml
   │ netplan generate
   ▼
/run/systemd/network/*.network   （yaml 被翻译成后端配置，里面有 DNS= / Domains=）
   │ systemd-networkd（或 NetworkManager）
   │ D-Bus：SetLinkDNS() / SetLinkDomains()
   ▼
systemd-resolved（内存里的“每网卡 DNS 配置”）
   └─ 顺便生成 /run/systemd/resolve/stub-resolv.conf 给别的程序看
```

DHCP 走的是同一条路：网卡拿到的 IPv4 DNS（option 6）和 IPv6 DNS（RDNSS），由 networkd 或 NetworkManager 收下后，同样用 D-Bus 推给 resolved。

两者的分工：**每块网卡的 DNS** 负责它自己那条链路/域名的查询；**`resolved.conf` 里的全局 `DNS=`** 负责没匹配到任何网卡的查询（在那里写 `Domains=~.` 就变成“默认都走它”）。

怎么验证 —— `resolvectl status` 会把这两部分分开显示：

```bash
resolvectl status
#  ├─ Global：DNS Servers: ...        ← 这一段来自 /etc/systemd/resolved.conf（你手写的）
#  └─ Link 2 (eth0)：DNS Servers: 8.8.8.8
#                   DNS Domain: ...   ← 这一段来自 netplan / DHCP（D-Bus 推来的）
```

配套命令：

```bash
resolvectl dns                      # 每块网卡当前生效的 DNS
resolvectl domain                   # 每块网卡的域名路由（split DNS 表）
cat /run/systemd/network/*.network  # netplan(networkd) 生成的后端配置，看 yaml 里的 DNS 变成了什么
```

>> 如果系统根本没启用 resolved（例如直接由 dhclient 写 /etc/resolv.conf），那 netplan / DHCP 的 DNS 就直接落在 `/etc/resolv.conf` 里，不经过 resolved。

### 4. 把两条路合起来看

```
curl example.com
  └─ glibc getaddrinfo → NSS（按 hosts: 行）
       ├─ files → /etc/hosts                       命中就结束
       └─ dns   → /etc/resolv.conf（nameserver 127.0.0.53）
                   └─ systemd-resolved stub（先查自己的缓存 / /etc/hosts）
                        └─ 仍未命中 → 按 split DNS 选上游 → 递归解析器 → 根 / TLD / 权威
                             └─ 结果按 TTL 缓存在 resolved → 原路返回
（若 hosts: 行里配的是 resolve，则上面第 3 步换成 nss-resolve 经 unix socket，不经过 53 端口）
```

