# SSH hardening guide

Use `ssh-audit` python tool (https://github.com/jtesta/ssh-audit).

```bash
# pipx install app from python package (into isolated Virtual Environments)
pipx install ssh-audit
```

## Audit local server

```bash
ssh-audit localhost
```

## Hardening sshd default configuration

```bash
# add harden config file to server conf directory
cp data/sshd_hardening.conf /etc/ssh/sshd_config.d/
# check syntax (return nothing = ok)
sudo sshd -t
# reload server conf.
sudo systemctl restart sshd
```

## Hardening sshd for Debian Bookworm (Debian 12) from ssh-audit guide

Come from https://www.ssh-audit.com/hardening_guides.html#debian_12

```bash
# re-generate the RSA (to 4096 bits) and ED25519 keys (remove ecdsa)
sudo rm /etc/ssh/ssh_host_*
sudo ssh-keygen -t rsa -b 4096 -f /etc/ssh/ssh_host_rsa_key -N ""
sudo ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ""

# uncomment this line in sshd_config
sudo sed -i '/^#Port 22/s/^#//' /etc/ssh/sshd_config
sudo sed -i '/^#HostKey \/etc\/ssh\/ssh_host_rsa_key/s/^#//' /etc/ssh/sshd_config
sudo sed -i '/^#HostKey \/etc\/ssh\/ssh_host_ed25519_key/s/^#//' /etc/ssh/sshd_config

# remove small Diffie-Hellman moduli
sudo awk '$5 >= 3071' /etc/ssh/moduli | sudo tee /etc/ssh/moduli.safe
sudo mv /etc/ssh/moduli.safe /etc/ssh/moduli

# restart sshd daemon
sudo systemctl restart ssh
```