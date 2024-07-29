# Raspberry Pi 5 notes


## RTC setup

### Turn on charge of external lithium-manganese battery

```txt
# only for pi5 (set charge to 3v = 3_000_000 microvolts)
[pi5]
dtparam=rtc_bbat_vchg=3000000
```

## CPU and GPU overclock

 add this to /boot/firmware/config.txt
```txt
[pi5]
# overclock (add by LLe)
over_voltage_delta=30000
arm_freq=2800
gpu_freq=1000
```
