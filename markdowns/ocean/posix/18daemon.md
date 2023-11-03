### init 进程退出daemon进程
因为 init 在发完 SIGTERM 信号的 5 秒之后会发送一个 SIGKILL 信号。（这并不意味着这个
daemon 能够执行 5 秒的 CPU 时间，因为 init 会同时向系统中的所有进程发送信号，而它们可
能都试图在 5 秒内完成清理工作。）

### daemon进程需要面对的两个问题
1. 热部署，当部署完成后，可以通过`killall -HUP [processName]`的方式向进程发送SIGUP信号，通过这个通知来更新配置文件
2. log过大，通过使用syslog(3)来调用系统log库