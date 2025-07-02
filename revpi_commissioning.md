# RevPi commissioning instructions


## Start from a fresh image

### step 1: download an image

Go to https://revolutionpi.com/en/support/downloads#c1069 and download as zip archive.

```bash
# Take a look at checksum with md5sum
md5sum -c  <<< "426e1f69cd1929127dc966c15c7d6fd2 250528-revpi-bookworm-arm64-lite.zip"
```

### step2: write image to revpi eMMC

To do it, we need a Raspberry Pi with rpiboot tool on it.

Tool rpiboot is a command-line utility primarily used for booting a Raspberry Pi (especially Compute Modules) in USB device mode. This allows your computer (the "host") to interact with the Raspberry Pi as if it were a USB mass storage device, even if the Pi itself doesn't have an SD card or is having boot issues.

```bash
# install rpiboot if not available
sudo apt install rpiboot
```
Connect the RPI to USB port in front of RevPi.

```bash
# Then, launch command
sudo rpiboot
```

Plug 24VDC connector to RevPi to start it.

Now eMMC, will be mounted as a USB mass storage.

Use a program like "Raspberry Pi Imager" or dd to write image on the eMMC USB mass storage.

Once writing is complete, remove the USB connector and cycle power to reboot the RevPi.


## Connect and update OS

```bash
# default password is on sticker at left side of RevPi (default factory is raspberry)
ssh pi@revpi123456.local
# update OS
sudo apt update
sudo apt dist-upgrade -y
```

## Fix locales

```bash
# add fr_FR.UTF8
sudo dpkg-reconfigure locales
# set server timezone
sudo timedatectl set-timezone Europe/Paris
```

## Add mandatory debian packages

```bash
# misc
sudo apt install -y ack apt-offline git hexedit htop minicom netcat-traditional nmap openssl pv supervisor tree jq vim
# python tools + packages
sudo apt install -y ipython3 pipx python3-pip python3-schedule python3-serial python3-venv python3-wheel
```

## Security setup

```bash
# add packages
sudo apt install -y ufw fail2ban
```

```bash
# add rules
sudo ufw allow proto tcp from 0.0.0.0/0 to any port 22
sudo ufw allow proto tcp from 192.168.0.0/24 to any port 80
sudo ufw allow proto tcp from 192.168.0.0/24 to any port 443
sudo ufw allow proto tcp from 192.168.0.0/24 to any port 41443
# turn UFW on
sudo ufw enable
```

```bash
# fail2ban setup
# see https://github.com/fail2ban/fail2ban/issues/3292#issuecomment-1142503461
# add sshd jail conf
echo -e "[sshd]\nbackend=systemd\nenabled=true" | sudo tee /etc/fail2ban/jail.d/sshd.conf
# remove default conf
sudo rm /etc/fail2ban/jail.d/defaults-debian.conf
# reload
sudo systemctl restart fail2ban.service 
# request status of sshd jail
sudo fail2ban-client status sshd
```
