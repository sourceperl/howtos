# Raspberry Pi UART

## Turn on UART on GPIO 14 (Tx) and 15 (Rx)

Official documentation:  https://www.raspberrypi.com/documentation/computers/configuration.html#configuring-uarts

### for Raspberry Pi 3

* Remove "console=serial0,115200" from /boot/cmdline.txt

```bash
sudo nano /boot/cmdline.txt
```

*This avoid kernel use /dev/serial0 as a system console.*

* Disable bluetooth (device tree and hciuart service)

```bash
echo "dtoverlay=disable-bt" | sudo tee -a /boot/config.txt
sudo systemctl disable hciuart
```

* Reboot

```bash
sudo reboot
```
