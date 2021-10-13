# SSH cookbook


## Connect without password

Create ssh key and copy it to remote host, after this we can connect without manual authenticate.

```bash
# create ssh key
ssh-keygen
# copy it to remote
ssh-copy-id [USERNAME]@[REMOTE_HOST]
```


## Local port forward 

Run on client side, open CLIENT_PORT on localhost interface and forward it to existing SERVER_IP:SERVER_PORT.

```bash
# -L option indicates local
# -N option to only forward ports (don't open shell)
# -f option to run in the background
ssh -L CLIENT_PORT:127.0.0.1:SERVER_PORT -N [USERNAME]@[SERVER_IP]
```


## Remote port forward 

Run on server side, forward local existing SERVER_PORT to remote CLIENT_PORT (open it). 

```bash
# -R option indicates remote
# -N option to only forward ports (don't open shell)
# -f option to run in the background
ssh -R CLIENT_PORT:127.0.0.1:SERVER_PORT -N [USERNAME]@[CLIENT_IP]
```

