# SSH keys with OpenSSH and PuTTY

## Setup

```bash
sudo apt install -y putty-tools
```

## File extensions
- OpenSSH private key:  .pk (default is nothing, like "id_rsa")
- OpenSSH public key:   .pub
- PuTTY private key:    .ppk

*For example id_rsa_myserver_shell.pk is a RSA private key for shell access to myserver with OpenSSH.*

## Some shell command

File naming: `id_<key-algorithm>_<servername>_<purpose>.<format-information>`

```bash
# produce OpenSSH private key(.pk) and public key (.pub)
ssh-keygen -t rsa -b 4096 -C '' -f id_rsa_srv1_shell
mv id_rsa_srv1_shell id_rsa_srv1_shell.pk

# OpenSSH private key -> PuTTY private key 
# debian package putty-tools provide puttygen tool
puttygen id_rsa_srv1_shell.pk -o id_rsa_srv1_shell.ppk

# PuTTY private key -> OpenSSH private key
puttygen id_rsa_srv1_shell.ppk -O private-openssh-new -o id_rsa_srv1_shell.pk

# extract OpenSSH public key (same format as ~/.ssh/authorized_keys) from OpenSSH private key 
ssh-keygen -y -f id_rsa_srv1_shell.pk > id_rsa_srv1_shell.pub

# add public key to trust list for current linux user
cat id_rsa_srv1_shell.pub >> ~/.ssh/authorized_keys

# get all currently authorized public key fingerprints
ssh-keygen -l -f - < ~/.ssh/authorized_keys
```
