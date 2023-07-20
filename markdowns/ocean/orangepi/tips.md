### open proxy cmd
```bash
cd workspace/SmarGate/server/ &&
nohup ./proxy_server -i1000 -o1000 -w10 >/dev/null &
```

### 查看硬件温度
```bash
paste <(cat /sys/class/thermal/thermal_zone*/type) <(cat /sys/class/thermal/thermal_zone*/temp) | column -s $'\t' -t | sed 's/\(.\)..$/.\1°C/'
```