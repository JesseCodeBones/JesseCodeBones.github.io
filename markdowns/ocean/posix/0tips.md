### SSH 长期转发port

1. `sudo nano /etc/ssh/ssh_config`  
2. 
```
    ServerAliveInterval 20
    ServerAliveCountMax 999
    ClientAliveInterval 60
    ClientAliveCountMax 3
```
3. 
通过tmux后台链接
`tmux`  
4. ssh 命令 
`ssh -n  -R 1221:127.0.0.1:22 ubuntu@[remote ip]`