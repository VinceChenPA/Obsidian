nohup (no hang up) 用于在系统后台不挂断运行命令，退出终端不会影响程序的运行。

例如某 Python 程序:

`nohup python -u main.py >> log.out 2>&1 &   `

`2>&1` 是一个重定向操作符，它使错误消息也会被写入 log.out 文件。