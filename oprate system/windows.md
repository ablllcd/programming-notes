## 查找端口被哪个程序占用

教程： https://www.runoob.com/w3cnote/windows-finds-port-usage.html

1. 查看端口号对应的PID：`netstat -aon|findstr "8081"`
2. 查看PID对应的任务：` tasklist|findstr "7156"`

