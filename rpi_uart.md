# Raspberry Pi UART


Official documentation:  https://www.raspberrypi.com/documentation/computers/configuration.html#configuring-uarts

## Raspberry Pi 3

### Turn on UART on GPIO 14 (Tx) and 15 (Rx)

* Avoid kernel use /dev/serial0 as a system console.

```bash
# remove "console=serial0,115200" from /boot/cmdline.txt
sudo nano /boot/cmdline.txt
```

* Disable bluetooth and link UART0 (/dev/ttyAMA0) to GPIO 14/15

```bash
# update device tree
echo "dtoverlay=disable-bt" | sudo tee -a /boot/config.txt
# turn off hciuart service
sudo systemctl disable hciuart
```

* Reboot

```bash
sudo reboot
```
