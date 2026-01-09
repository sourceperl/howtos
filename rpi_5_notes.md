# Raspberry Pi 5 notes


## RTC setup

### Turn on charge of external lithium-manganese battery

Update /boot/firmware/config.txt :

```ini
# only for pi5 (set charge to 3 volts => 3_000_000 microvolts)
# turn on RTC battery charge(add by LLe)
[pi5]
dtparam=rtc_bbat_vchg=3000000
```

Reboot to activate this dtparam.

Some checks :

```bash
# read current charging voltage in microvolts (return 0 if dtparam "rtc_bbat_vchg" is unset)
cat /sys/devices/platform/soc@107c000000/soc@107c000000\:rpi_rtc/rtc/rtc0/charging_voltage
# read current battery voltage in microvolts (return 2964099 for my test)
cat /sys/devices/platform/soc@107c000000/soc@107c000000\:rpi_rtc/rtc/rtc0/battery_voltage
```

## CPU and GPU overclock

**Experiment with different values ​​for the settings below to find a stable overclock. Here is just an example that works on my card.**

Update /boot/firmware/config.txt :

```ini
[pi5]
# overclock (add by LLe)
over_voltage_delta=30000
arm_freq=2800
gpu_freq=1000
```

*More at https://www.raspberrypi.com/documentation/computers/config_txt.html#overclocking-options.*

## Fan

```bash
# get temp of the SoC
vcgencmd measure_temp
# read current FAN speed in RPM
cat /sys/devices/platform/cooling_fan/hwmon/hwmon2/fan1_input
# read PWM FAN output (at full FAN speed, PWM = 250)
cat /sys/class/hwmon/hwmon2/pwm1
```

FAN speed vs temperature at https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#fan-cases.


## Pi box Argon NEO 5 M.2 NVMe (test with SSD Crucial 1TB P3 Plus CT1000P3PSSD8)


### PCIe speed update (to increase SSD I/O access time)

Benchmark default speed of SSD reading:

```bash
# hdparm disk tool is mandatory
sudo apt install hdparm

# run a test 
sudo hdparm -t --direct /dev/nvme0n1
```

Command hdparm return :

```txt
/dev/nvme0n1:
 Timing O_DIRECT disk reads: 1310 MB in  3.00 seconds = 436.30 MB/sec
```


Update /boot/firmware/config.txt :

```ini
[pi5]
# enable PCIe Gen 3 Speeds 1,000 MB/s (add by LLe)
dtparam=pciex1_gen=3
```

After a reboot, relaunch hdparm and we get :

```txt
/dev/nvme0n1:
 Timing O_DIRECT disk reads: 2462 MB in  3.00 seconds = 820.66 MB/sec
```


### Increase the size of swap file (from default 200 MB to 2 GB)

```bash
# turn off swap
sudo dphys-swapfile swapoff

# edit swap configuration file
sudo nano /etc/dphys-swapfile
# update "CONF_SWAPSIZE=200" to "CONF_SWAPSIZE=2048" (200 MB to 2 GB)
# exit nano editor

# recreate swap file
sudo dphys-swapfile setup

# turn on swap
sudo dphys-swapfile swapon

# check result
free -h

# reboot
sudo reboot
```
