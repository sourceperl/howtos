## Listen on a port below 1024 without root

On Linux, the ports below 1024 are "privileged": by default only root can listen on them. **Don't run a server as root
(or with sudo) just to get a low port** (80, 443, 25...): any flaw in the program, in its plugins or in a library would
then give full control of the host to an attacker.

Pick one of the solutions below instead, from the most to the least recommended. Each of them only gives the right to
listen on a low port, nothing else.

In the examples, `myapp` is the service to run, `/opt/myapp/bin/myapp` its executable and `443` the port to listen on.
Replace them with your own values.

### 1. Run the server as a systemd service (recommended for a permanent server)

The server runs as a dedicated user without privileges, systemd gives it the `CAP_NET_BIND_SERVICE` capability (the
right to listen on a port below 1024) and nothing else.

```bash
# a system user without shell or home
sudo useradd --system --no-create-home --shell /usr/sbin/nologin myapp
sudo mkdir -p /opt/myapp/bin
sudo cp myapp /opt/myapp/bin/
```

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My application
After=network-online.target
Wants=network-online.target

[Service]
User=myapp
Group=myapp
ExecStart=/opt/myapp/bin/myapp --host 192.168.1.10 --port 443
# the right to listen on a port below 1024, and nothing else
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
NoNewPrivileges=true
# optional hardening: read-only system, no access to /home, private /tmp, network and kernel restrictions
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
PrivateDevices=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectKernelLogs=true
ProtectControlGroups=true
ProtectClock=true
ProtectHostname=true
ProtectProc=invisible
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
RestrictNamespaces=true
RestrictRealtime=true
RestrictSUIDSGID=true
LockPersonality=true
SystemCallArchitectures=native
UMask=0077
# if the service must write data, allow only its own directory (created in /var/lib/myapp)
#StateDirectory=myapp
# if the service needs a device (/dev/ttyUSB0...): replace "PrivateDevices=true" with these 2 lines
#DeviceAllow=/dev/ttyUSB0 rw
#SupplementaryGroups=dialout
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
journalctl -u myapp -f   # server logs
```

### 2. Redirect the port on the firewall

The server listens on a port above 1023 (like 8443) without any privilege, the firewall redirects the connections from
the low port to it.

```bash
./myapp --host 192.168.1.10 --port 8443

# nftables
sudo nft add table ip nat
sudo nft add chain ip nat prerouting '{ type nat hook prerouting priority dstnat; }'
sudo nft add rule ip nat prerouting tcp dport 443 redirect to :8443
# or iptables
sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-ports 8443
```

- The redirection only applies to the connections coming from the network: a client on the same host must use port
  8443.
- The server must listen on the address of a network interface (not on localhost).
- These rules only cover IPv4: add the same rules in an `ip6` table (or with `ip6tables`) for IPv6.
- Save the rules to keep them after a reboot (`/etc/nftables.conf`, `iptables-save`...).

#### With ufw

ufw has no command for redirections: add them to its rule files, which are loaded at every start (no need to save
them).

Add this block at the top of `/etc/ufw/before.rules`, before the `*filter` line:

```
# redirect port 443 to 8443
*nat
:PREROUTING ACCEPT [0:0]
-A PREROUTING -p tcp --dport 443 -j REDIRECT --to-ports 8443
COMMIT
```

For IPv6, add the same block at the top of `/etc/ufw/before6.rules`.

Then allow the **destination** port (the firewall sees the connections after the redirection, so on port 8443) and
reload:

```bash
sudo ufw allow 8443/tcp                                          # from any client
# or only from a network
sudo ufw allow from 192.168.1.0/24 to any port 8443 proto tcp
sudo ufw reload
sudo iptables -t nat -L PREROUTING -n -v                         # check: the REDIRECT rule is listed
```

- Allowing port 8443 also lets clients connect to it directly, which is harmless: it's the same server.
- To redirect only the connections from one interface, add `-i eth0` to the rule.
- The notes above (local clients, listening address) apply too.

### 3. Give the capability to the executable (tests, no systemd)

```bash
sudo setcap 'cap_net_bind_service=+ep' /opt/myapp/bin/myapp
getcap /opt/myapp/bin/myapp             # check: cap_net_bind_service=ep
/opt/myapp/bin/myapp --port 443
```

- The capability is attached to the file, so anyone who can run this file can listen on a low port: make sure only
  root can modify it.
- The capability is lost when the file is replaced (update, rebuild): run `setcap` again.
- It doesn't work on a symbolic link (set it on the real file) nor on a script (`#!` line): for an interpreted program
  (Python, Node.js...), it must be set on the interpreter itself. In that case use a copy dedicated to this program
  (for Python: `python3 -m venv --copies`) and **never** the system interpreter (`/usr/bin/python3`, `/usr/bin/node`),
  otherwise every script of every user could listen on a low port.

### Not recommended

- `sudo ./myapp`: the whole program runs as root.
- `sysctl net.ipv4.ip_unprivileged_port_start=443`: every user and every program of the host can then listen on port
  443 and above (IPv4 and IPv6 despite the name). Only acceptable on a host or in a container dedicated to the server.

### Other systems

- **Docker**: the server listens on a port above 1023 in the container, publish it with `-p 443:8443`.
- **Windows**: there are no privileged ports, a regular user can listen on any port (the firewall must still allow it).
- **macOS**: since macOS 10.14 (Mojave), a regular user can listen on ports below 1024.
