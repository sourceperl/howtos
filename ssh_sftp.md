# SSH cookbook


## Create a user for SFTP purposes only

```bash
# create a random strong password (secure it with tool like keepass)
sudo apt install -y pwgen
pwgen -B -s 64 1
# add harden config file to server conf directory
sudo useradd -m -s /sbin/nologin sftpuser
# set the password defined above
sudo passwd sftpuser
```

## Configure home directory permissions for chroot

This is a critical step for ChrootDirectory to work securely. The directory one level above the user's home directory 
must be owned by root and have very strict permissions.

```bash
# make root own the user's home directory initially
sudo chown root:root /home/sftpuser
# set permissions for the home directory
sudo chmod 755 /home/sftpuser
```

## Create a writable directory inside the chroot jail

```bash
sudo mkdir /home/sftpuser/data
sudo chown sftpuser:sftpuser /home/sftpuser/data
sudo chmod 755 /home/sftpuser/data
```

## Verify Essential SSHd Configuration

Ensure the following critical directives in your main `/etc/ssh/sshd_config` file are **uncommented** and correctly 
configured:

* `Port 22`: This line defines the default port for standard SSH connections.
* `Subsystem sftp /usr/lib/openssh/sftp-server`: This directive specifies the SFTP subsystem used by OpenSSH. 
(The path might vary slightly depending on your Linux distribution, e.g., `/usr/libexec/openssh/sftp-server` 
on some RHEL/CentOS systems.)

## Add SFTP configuration file

```bash
# create an SFTP-specific SSH configuration file
# -> adds a new SFTP-only service on port 8022 with secure settings
sudo tee /etc/ssh/sshd_config.d/sftp.conf > /dev/null <<'EOF'
# --- SFTP Service on Port 8022 ---
Port 8022

# --- Restrict access on port 8022 ---
Match LocalPort 8022

    # Force internal SFTP (no shell access)
    ForceCommand internal-sftp

    # Restrict users to their home directories (requires proper permissions)
    ChrootDirectory %h

    # Allow only the specific SFTP user
    AllowUsers sftpuser

    # Hardened security options
    PermitRootLogin no
    PermitTunnel no
    AllowAgentForwarding no
    AllowTcpForwarding no
    X11Forwarding no

    # Authentication settings
    PasswordAuthentication yes    # Set to 'no' to enforce key-only authentication
    PubkeyAuthentication yes
EOF

# test SSHD configuration
sudo sshd -t || { echo "sshd config test failed"; exit 1; }

# restart SSH service to apply changes
sudo systemctl restart sshd && echo "SFTP service on port 8022 configured and SSH restarted"
```

## Customize fail2ban to protect SFTP port

    **WORK IN PROGRESS**

## Adjust ufw rules

```bash
sudo ufw allow proto tcp from 192.168.0.0/16 to any port 8022
```