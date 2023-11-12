# Raspberry Pi UART


Official documentation:  https://www.raspberrypi.com/documentation/computers/configuration.html#configuring-uarts

## Raspberry Pi 3

### Expose UART0 (PL011) on GPIO 14 (Tx) and 15 (Rx) 

This permit to use /dev/ttyAMA0 for user purpose application (but will turn off on-board bluetooth).

*By default /dev/serial0 is a symbolic link to /dev/ttyS0 (mini-UART) and /dev/serial1 is a symbolic link to /dev/ttyAMA0 (UART0 PL011).*

* Avoid kernel use /dev/serial0 (primary uart) as a system console.

```bash
# remove "console=serial0,115200" from /boot/cmdline.txt
sudo nano /boot/cmdline.txt
```

* Disable bluetooth HCI service which use /dev/serial1 (secondary uart)

```bash
# turn off hciuart service
sudo systemctl disable hciuart
```

* Link UART0 (/dev/ttyAMA0) to GPIO 14/15

```bash
# update device tree
# after reboot:
# - /dev/serial0 -> /dev/ttyAMA0 (PL011) and /dev/serial1 -> /dev/ttySC0 (mini-uart)
# - /dev/ttyAMA0 will be expose on GPIO 14 and 15
echo "dtoverlay=disable-bt" | sudo tee -a /boot/config.txt
```

* Reboot

```bash
sudo reboot
```

## Raspberry Pi 4B and 400

*Raspberry Pi 4B and 400 have an additional four PL011s (UART2 to 5), which are disabled by default.*

| Name  |   Type    |    Device    |       GPIOs (Tx & Rx)        |                         Comment                             |
|-------|-----------|--------------|:----------------------------:|-------------------------------------------------------------|
| UART0 | PL011     | /dev/ttyAMA0 | 32 & 33 (default) or 14 & 15 | by default used by bluetooth HCI service (via /dev/serial1) |
| UART1 | mini UART | /dev/ttyS0   | 14 & 15 (default) or 32 & 33 | by default used by system console (via /dev/serial0)        |
| UART2 | PL011     | /dev/ttyAMA1 | 0 & 1                        | add "dtoverlay=uart2" to /boot/config.txt to enable         |
| UART3 | PL011     | /dev/ttyAMA2 | 4 & 5                        | add "dtoverlay=uart3" to /boot/config.txt to enable         |
| UART4 | PL011     | /dev/ttyAMA3 | 8 & 9                        | add "dtoverlay=uart4" to /boot/config.txt to enable         |
| UART5 | PL011     | /dev/ttyAMA4 | 12 & 13                      | add "dtoverlay=uart5" to /boot/config.txt to enable         |

