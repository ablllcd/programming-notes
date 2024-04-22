## 连接服务器

ssh经常被用作服务器的登录认证，这里记录一下基本用法

### 命令行连接

```
ssh -i /path/to/private_key.pem user@hostname
```

### 配置文件

由于命令行每次都要写参数，很麻烦。为此可以创建"C:\Users\Cc\.ssh\config"文件，其中填写连接信息：

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
