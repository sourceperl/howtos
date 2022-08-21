# stunnel cookbook


## Setup

```bash
# install stunnel
sudo apt install stunnel
# launch stunnel at system startup
sudo systemctl enable stunnel4.service
# create certificates directories
sudo mkdir -p /etc/stunnel/certs/
```

## Add an SSL front-end to an MQTT server

```bash
# create trusted certificates directory for clients
sudo mkdir -p /etc/stunnel/certs/mqtt-srv.trusted-cli.d/
# add configuration file for MQTT server 
cat <<EOF | sudo tee -a /etc/stunnel/mqtt-srv.conf
; avoid stunnel run with root privileges
setuid = stunnel4
setgid = stunnel4
pid = /run/stunnel4/stunnel-mqtt-srv.pid

[mqtt-srv]
client = no
verifyPeer = yes
cert = /etc/stunnel/certs/mqtt-srv.crt
key = /etc/stunnel/certs/mqtt-srv.key
CApath = /etc/stunnel/certs/mqtt-srv.trusted-cli.d/
accept  = 0.0.0.0:18830
connect = 127.0.0.1:1883
EOF
```

```bash
# create private key and self-signed certificate for server
target_prefix=/etc/stunnel/certs/mqtt-srv
sudo openssl req -x509 -newkey rsa:4096 -days 7305 -nodes \
                 -subj "/C=FR/CN=mqtt-srv" \
                 -keyout ${target_prefix}.key \
                 -out ${target_prefix}.crt
```

```bash
# load this configuration
sudo systemctl restart stunnel4.service
```

### Create cert/key for client and trust it

```bash
# create private key and self-signed certificate for server
target_prefix=/etc/stunnel/certs/mqtt-cli
sudo openssl req -x509 -newkey rsa:4096 -days 7305 -nodes \
                 -subj "/C=FR/CN=mqtt-cli" \
                 -keyout ${target_prefix}.key \
                 -out ${target_prefix}.crt
# copy cert to trusted directory
sudo cp mqtt-cli.crt /etc/stunnel/certs/mqtt-srv.trusted-cli.d/
# add symbolic links to certs as hash values (need by stunnel CApath)
sudo c_rehash /etc/stunnel/certs/mqtt-srv.trusted-cli.d/
```

