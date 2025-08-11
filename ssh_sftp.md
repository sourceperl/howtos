# SSH cookbook


## Create a user for SFTP purposes only

```bash
# create a random strong password (secure it with tool like keepass)
sudo apt install -y pwgen
pwgen -B -s 64 1
# add user (don't copy default in /etc/skell to home dir)
sudo useradd --create-home --skel /dev/null --shell /sbin/nologin sftpuser
# set the password defined above
sudo passwd sftpuser
```

## Configure a directory for sftp chroot

This is a critical step for ChrootDirectory to work securely. The root directory must be owned by root 
and have very strict permissions.

```bash
# create root dir
sudo mkdir /srv/sftp
sudo chown root:root /srv/sftp
sudo chmod 755 /srv/sftp
# create a writable directory inside sftp chroot
sudo mkdir /srv/sftp/data
sudo chown sftpuser:sftpuser /srv/sftp/data
sudo chmod 755 /srv/sftp/data
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
sudo tee /etc/ssh/sshd_config.d/sftp.conf > /dev/null <<'EOF'
Match User sftpuser
    # Force internal SFTP (no shell access)
    ForceCommand internal-sftp

    # Restrict users to their home directories (requires proper permissions)
    ChrootDirectory /srv/sftp/

    # Authentication settings
    PasswordAuthentication yes
    PubkeyAuthentication yes
EOF

# test SSHD configuration
sudo sshd -t

# restart SSH service to apply changes
sudo systemctl restart sshd
```
