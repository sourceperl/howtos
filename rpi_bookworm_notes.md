# Raspberry Pi Bookworm notes


## Packages setup

### Update system

```bash
sudo apt update
sudo apt upgrade -y
```

### Mandatory debian packages

```bash
# all
sudo apt install -y ack apt-offline git hexedit htop minicom netcat-traditional nmap openssl pv supervisor tree jq vim

# with gui
sudo apt install -y code putty python3-tk screenruler keepass2

# python
sudo apt install -y ipython3 pipx python3-pip python3-schedule python3-serial python3-venv python3-wheel
```

## Security setup

```bash
# add packages
sudo apt install -y ufw fail2ban
```

```bash
# UFW setup
sudo ufw enable
sudo ufw allow proto tcp from 192.168.0.0/24 to any port ssh
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


## SSHd setup

### Turn on service

```bash
# with raspi-config: choice 3 > I1 SSH > enable
sudo raspi-config
```
### Harden the default configuration

see [SSH cookbook](ssh_cookbook.md)