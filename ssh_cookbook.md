# SSH cookbook


## Harden default configuration on RPI

```bash
# add harden config file to server conf directory
cp data/sshd_rpi_harden.conf /etc/ssh/sshd_config.d/
# check syntax (return nothing = ok)
sudo sshd -t
# reload server conf.
sudo systemctl reload sshd
```


## View current server configuration

```bash
# read current server configuration
sudo sshd -T
```


## Connect without password

Create ssh key and copy it to remote host, after this we can connect without manual authenticate.

```bash
# create ssh key
ssh-keygen
# copy it to remote
ssh-copy-id [USERNAME]@[REMOTE_HOST]
```

To undo this, you have to manualy edit ~/.ssh/authorized_keys file on REMOTE_HOST and remove the line matching your SSH public key (available in local file ~/.ssh/id_rsa.pub).


## Local port forward

Open LOCAL_PORT on LOCAL_IP and forward it to remote at REMOTE_IP:REMOTE_PORT.

```bash
# -L option indicates local
# -N option to only forward ports (don't open shell)
# -f option to run in the background
ssh -L LOCAL_IP:LOCAL_PORT:REMOTE_IP:REMOTE_PORT -N [USERNAME]@[SERVER_IP]
```

### Example

Forward http (tcp/80) of host 172.18.40.16 to local port tcp/8080 (LISTEN on localhost).

```bash
ssh -L 127.0.0.1:8080:127.0.0.1:80 -N user@172.18.40.16
```


## Remote port forward

Open REMOTE_PORT on REMOTE_IP and forward it to LOCAL_IP:LOCAL_PORT app.

```bash
# -R option indicates remote
# -N option to only forward ports (don't open shell)
# -f option to run in the background
ssh -R REMOTE_IP:REMOTE_PORT:LOCAL_IP:LOCAL_PORT -N [USERNAME]@[CLIENT_IP]
```

### Example

Open port tcp/8080 (LISTEN on localhost) of 172.18.40.16, forward content
to local app at port tcp/80.

```bash
ssh -R 127.0.0.1:8080:127.0.0.1:80 -N user@172.18.40.16
```
