## 简介

Node.js 是javascript的运行环境。它包含：
* chrome v8引擎：能够解释执行js代码，类似于java虚拟机
* 基本包：提供一些基本API来让JS调用
* 开发工具：提供一些辅助工具，例如npm


## NPM常见命令

### 查看

```
npm list -g //查看全局安装
```

# NVM

NVM（Node Version Manager）是一个用于管理多个 Node.js 版本的工具。它允许你在同一台机器上安装和切换不同版本的 Node.js，非常适合开发者在不同项目中使用不同版本的 Node.js。

## 安装NVM

```bash
# 安装 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# 重启终端或执行下面命令使 nvm 生效
source ~/.bashrc
```

## 常用命令

```bash
# 查看已安装的 Node.js 版本
nvm ls
# 查看可用的 Node.js 版本
nvm ls-remote
# 安装指定版本的 Node.js
nvm install <version>
# 使用指定版本的 Node.js
nvm use <version>
# 设置默认版本
nvm alias default <version>
# 卸载指定版本的 Node.js
nvm uninstall <version>
```
