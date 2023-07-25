### open proxy cmd
```bash
cd workspace/SmarGate/server/ &&
nohup ./proxy_server -i1000 -o1000 -w10 >/dev/null &
```

### 查看硬件温度
```bash
paste <(cat /sys/class/thermal/thermal_zone*/type) <(cat /sys/class/thermal/thermal_zone*/temp) | column -s $'\t' -t | sed 's/\(.\)..$/.\1°C/'
```

### 格式化分区
```bash
  ls -al /dev/sd*
  sudo fdisk /dev/sda
  d //删除分区
  w //删除分区表
  n //新建分区
  w //写入并推出
  sudo mkfs.ext3 /dev/sda1 //格式化分区
```