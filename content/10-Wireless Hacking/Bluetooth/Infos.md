- https://github.com/zedxpace/bluetooth-hacking

# Basics

```bash
# scan peripherals
service bluetooth start
hcitool scan
# or
bluetoothctl scan on

# connect
rfcomm bind 0 $mac $channel_nb
rfcomm bind 0 $mac 6
# connect to the peripheral on channel 6

# AT terminal 
AT*ESIL=1 # activates silence mode
AT+CMGS="+336875648" # sends SMS
```

# Tools

- https://github.com/fO-000/bluescan
- https://github.com/kimbo/bluesnarfer
