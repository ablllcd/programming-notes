## 参考教程

https://www.runoob.com/linux/linux-shell.html

# 常用操作

## 系统操作

### 查看系统信息
1. 查看操作系统版本
   ```bash
   cat /etc/os-release  # 显示操作系统版本信息
   lsb_release -a        # 显示Linux标准基础信息
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

## 磁盘操作

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

3. `fdisk` 命令 - 查看磁盘分区
   ```bash
   sudo fdisk -l  # 列出所有磁盘的分区表
   ```

4. `lsblk` 命令 - 以树状列出所有块设备
   ```bash
   lsblk         # 列出所有块设备
   lsblk -f      # 显示文件系统信息
   ```

5. `ncdu` 命令 - 交互式磁盘使用分析器
   ```bash
   # 安装：sudo apt-get install ncdu
   ncdu /path    # 分析指定路径的磁盘占用
   ```

## 文件操作

### 文件权限
```bash
chmod 755 filename  # 设置文件权限为 rwxr-xr-x

# 查看文件权限
ls -l filename  # 显示文件权限和所有者信息
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
```

## 解压操作

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


## 网络操作

### 查看网络配置
```bash
ifconfig          # 显示网络接口配置
ip addr show      # 显示网络接口的详细信息
ip link show      # 显示网络接口的状态
```

### 查看端口占用

1. `netstat` 命令 - 显示网络连接、路由表、接口统计等
    ```bash
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

## 进程操作

### 查看进程以及资源占用

1. `ps` 命令 - 显示当前进程信息
    ```bash
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

## 下载操作

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

## GPU相关操作

### 查看GPU信息
```bash
nvidia-smi  # 显示NVIDIA GPU的状态和使用情况
```

