# SSH file memo

## File naming
```
id_<key-algorithm>_<servername>_<purpose>.<format-information>
```

## File extensions
- OpenSSH private key:  .pk
- OpenSSH public key:   .pub
- PuTTY private key:    .ppk
- PuTTy public key:     .ppubk

*For example id_rsa_myservername_shell.pk is a RSA private key for shell access to myserver with OpenSSH.*

## Some shell command
```bash
# produce OpenSSh private key and 
ssh-keygen -t rsa -b 4096 -C '' -f id_rsa_loos-sv1_shell.pk

# OpenSSH private key -> PuTTY private key 
# debian package putty-tools provide puttygen tool
puttygen id_rsa_loos-sv1_shell.pk -o id_rsa_loos-sv1_shell.ppk
```
