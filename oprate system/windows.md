## 查找端口被哪个程序占用

教程： https://www.runoob.com/w3cnote/windows-finds-port-usage.html

1. 查看端口号对应的PID：`netstat -aon|findstr "8081"`
2. 查看PID对应的任务：` tasklist|findstr "7156"`

## 添加用户环境变量

1. 通过```reg query "HKCU\Environment" /v Path```来查看当前用户的环境变量。

2. 通过 ```setx PATH "添加了新路径的总路径"```