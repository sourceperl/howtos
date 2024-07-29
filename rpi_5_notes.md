# Raspberry Pi 5 notes


## RTC setup

### Turn on charge of external lithium-manganese battery

add this to /boot/firmware/config.txt

```ini
# only for pi5 (set charge to 3v = 3_000_000 microvolts)
[pi5]
dtparam=rtc_bbat_vchg=3000000
```

Reboot to activate this dtparam.

Some checks :

```bash
# read current charging voltage in microvolts (return 0 if dtparam "rtc_bbat_vchg" is unset)
cat /sys/devices/platform/soc/soc\:rpi_rtc/rtc/rtc0/charging_voltage
# read current battery voltage in microvolts (return 2964099 for my test)
cat /sys/devices/platform/soc/soc\:rpi_rtc/rtc/rtc0/battery_voltage 
```

## CPU and GPU overclock

**Experiment with different values ​​for the settings below to find a stable overclock. Here is just an example that works on my card.**

add this to /boot/firmware/config.txt

```ini
[pi5]
# overclock (add by LLe)
over_voltage_delta=30000
arm_freq=2800
gpu_freq=1000
```
