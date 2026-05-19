## 密钥对连接服务器

ssh可以通过账号密码或者密钥对来连接服务器。这里讲一下密钥对的连接方式。

### 1. 生成密钥对
```
ssh-keygen
```

### 2. 将公钥添加到服务器
```
ssh-copy-id -i ~/.ssh/id_rsa.pub user@hostname
```
如果是亚马逊等云服务商提供的服务器，根据其提供的方式将公钥添加到服务器上。


### 3. 连接服务器
```
ssh -i /path/to/private_key.pem user@hostname
```

这是连接服务器时手动输入private key路径的方式。为了方便，可以配置 ssh：

* 编辑 ~/.ssh/config 文件，添加如下内容：
  ```
  Host ElderlyCare
    HostName 20.2.19.2
    User caoyuwei
    IdentityFile C:\Users\Cc\.ssh\ElderlyCareServer-key.pem
  ```

  之后再连接只需要再命令行输入

  ```
  ssh ElderlyCare
  ```

* 将ssh private key放在默认位置（~/.ssh/id_rsa），这样ssh会自动使用默认的私钥进行连接，就不需要在命令行中指定私钥路径了：

  ```
  ssh user@hostname
  ```

  注意：私钥名称必须是 id_rsa等特定名称，或者在 ssh config 中指定 IdentityFile 的路径和名称，否则 ssh 不会自动使用这个私钥进行连接。

## 文件传输
### scp

```
# 拷贝单个文件到远程服务器
scp local_file user@hostname:/remote/directory

# 从远程服务器拷贝单个文件到本地
scp user@hostname:/remote/file local_directory

# 拷贝整个目录到远程服务器
scp -r local_directory user@hostname:/remote/directory

# 从远程服务器拷贝整个目录到本地
scp -r user@hostname:/remote/directory local_directory

# 通配符批量拷贝
scp user@hostname:/remote/directory/*.txt local_directory

# 多文件拷贝
scp file1.txt file2.txt user@hostname:/remote/directory
```

## Tunnel转发

SSH Tunnel转发可以将本地端口转发到远程服务器，或者将远程服务器的端口转发到本地，从而实现安全的访问。

例如， A-B-C的网络结构，A和B可以ssh连接，B和C可以ssh连接，但A和C不能直接ssh连接。此时可以在A上使用SSH Tunnel转发，将A的本地端口转发到C的SSH端口，这样就可以通过A访问C了。

```
# A通过B访问C的端口
ssh -L <local_port>:<IP_C>:<Port_C> ubuntu@<IP_B>
```

也可以取消转发：

```
exit  # 退出SSH连接，取消转发
```