# Add Debian backports to Raspberry Pi OS

# Debian buster-backports

```bash
# add a new packages source
sudo sh -c 'echo "deb http://deb.debian.org/debian buster-backports main" > /etc/apt/sources.list.d/backports.list'
# add debian public keys
# add "Debian Archive Automatic Signing Key (10/buster)"
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 648ACFD622F3D138
# add "Debian Archive Automatic Signing Key (11/bullseye)"
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 0E98404D386FA1D9
# update packages database
sudo apt update 
# install a package from buster-backports
sudo apt install redis-tools/buster-backports
```

# Debian bullseye-backports

```bash
# add a new packages source
sudo sh -c 'echo "deb http://deb.debian.org/debian bullseye-backports main" > /etc/apt/sources.list.d/backports.list'
# add debian public keys
# add "Debian Archive Automatic Signing Key (10/buster)"
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 648ACFD622F3D138
# add "Debian Archive Automatic Signing Key (11/bullseye)"
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 0E98404D386FA1D9
# update packages database
sudo apt update 
# install a package from bullseye-backports
sudo apt install redis-tools/bullseye-backports
```