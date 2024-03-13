# Raspberry Pi Bookworm notes


## SSHd  setup

### Turn on service

```bash
# with raspi-config: choice 3 > I1 SSH > enable
sudo raspi-config
```

### Add a basic layer of security

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

### Harden the default configuration

see [SSH cookbook](ssh_cookbook.md)