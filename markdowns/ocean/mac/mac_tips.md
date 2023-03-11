# mac tips
### mac connect tty usb debugger
`brew install minicom`  
get address of your USB to Serial adapter:

ls /dev/tty.*
/dev/tty.Bluetooth-Incoming-Port    /dev/tty.usbserial-1440
and set it up: minicom -s
```
        +-----[configuration]------+
        | Filenames and paths      |
        | File transfer protocols  |
        | Serial port setup        |
        | Modem and dialing        |
        | Screen and keyboard      |
        | Save setup as dfl        |
        | Save setup as..          |
        | Exit                     |
        | Exit from Minicom        |
        +--------------------------+
```
Choose Serial port setup

Press A to setup you USB to Serial device

Press F to disable Hardware flow control

So it would look like this:
```

+-----------------------------------------------------------------------+
| A -    Serial Device      : /dev/tty.usbserial-1440                   |
| B - Lockfile Location     : /usr/local/Cellar/minicom/2.7.1/var       |
| C -   Callin Program      :                                           |
| D -  Callout Program      :                                           |
| E -    Bps/Par/Bits       : 115200 8N1                                |
| F - Hardware Flow Control : No                                        |
| G - Software Flow Control : No                                        |
|                                                                       |
|    Change which setting?                                              |
+-----------------------------------------------------------------------+
```
Hardware flow control must be disabled for you to be able to send inputs to terminal in typical PL2303 USB to Serial cables.

Esc key is the Meta key for this program. Esc and arrow down to exit menu. Do not forget to save default!

    | Save setup as dfl        |
and Exit from Minicom

Next time you start it, it expects defaults you just configured and in my case ready to go from second 1.