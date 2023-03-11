## 设置声卡
`aplay -l` 查看声卡  
`aplay -D hw:2,0 /usr/share/sounds/alsa/audio.wav`选择声卡进行播放  
`alsamixer` 调节音量大小 f6切换声卡，m改变设置  
* 使用 ~/.asoundrc配置默认声卡
用户配置  
https://alsa.opensrc.org/Asoundrc

在home目录添加 .asoundrc文件:

vi .asoundrc # 【测试可用】
```
defaults.ctl.card 2 
defaults.pcm.card 2
defaults.pcm.device 0
```
4.使用/etc/asound.conf配置默认声卡
全局配置  
```
pcm.!default {
        type plug
        slave {
                pcm "hw:tegrasndt210ref,0"
                channels 2
                rate 48000
        }
        hint.description "Tegra APE Soundcard (tegrasndt210ref,0)"
}

ctl.!default {
        type hw
        card tegrasndt210ref
}

pcm.music {
        type plug
        slave {
                pcm "hw:tegrasndt210ref,0"
                channels 2
                rate 48000
        }
}

pcm.demixer {
        type plug
        slave {
                pcm "dmix:tegrasndt210ref"
                channels 2
                rate 48000
        }
}

pcm.aux {
        type hw
        card "Tegra"
        device 3
}

pcm.voice {
        type plug
        slave {
                pcm "hw:tegrasndt210ref,2"
                channels 1
                rate 8000
        }
}

pcm.aux_plug {
        type plug
        slave.pcm "aux"
}

pcm.music_and_voice {
        type asym

        playback.pcm {
                type plug

                slave.pcm {
                        type multi

                        slaves.a.pcm music
                        slaves.b.pcm voice
                        slaves.a.channels 2
                        slaves.b.channels 1

                        bindings.0.slave a
                        bindings.0.channel 0
                        bindings.1.slave a
                        bindings.1.channel 1
                        bindings.2.slave b
                        bindings.2.channel 0
                }
                route_policy duplicate
        }
        capture.pcm "voice"
}
```
在文件最后添加一下内容

```
$ sudo vim /etc/asound.conf   
defaults.pcm.card 2  
defaults.ctl.card 2  
  ```