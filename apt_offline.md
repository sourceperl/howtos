# apt-offline howto

Apt-offline is an offline package manager for Debian-based Linux systems.

It's a tool that allows you to update, upgrade, and install packages and their dependencies on a disconnected machine 
by using a signature file generated on the offline machine and a different, internet-connected machine to download 
the required files.


## Install the tool

```bash
# on online machine (internet-connected)
sudo apt install -y apt-offline
```

```bash
# on offline machine
# from an online (internet-connected) machine download mandatory deb archives and transfert it to the offline machine
apt download apt-offline python3-magic libmagic1t64
# on offline machine, install from deb archive (apt will look on local dir for dependencies)
sudo apt install ./apt-offline_1.8.6-1_all.deb
```

## Upgrade an offline machine

### Step 1: apt update (update list of packages)

```bash
# on offline machine: build a signature file
sudo apt-offline set --update apt-offline.sig

# offline machine "apt-offline.sig" --> online machine

# on online machine: download package index from debian mirrors
apt-offline get apt-offline.sig --threads 4 --bundle offline-updates.zip

# online machine "offline-updates.zip" --> offline machine

# on offline machine: apply zip offline-updates.zip to local machine, this sync the packages list
sudo apt-offline install offline-updates.zip
```

### Step 2. apt upgrade (update packages)

```bash
# on offline machine: build a signature file
sudo apt-offline set --upgrade apt-offline.sig

# offline machine "apt-offline.sig" --> online machine

# on online machine: download package(s) from debian mirrors
apt-offline get apt-offline.sig --threads 4 --bundle offline-upgrades.zip

# online machine "offline-upgrades.zip" --> offline machine

# on offline machine: apply zip offline-updates.zip to local machine  sync the packages list (as apt update)
sudo apt-offline install offline-upgrades.zip
```

## Install package(s) on offline machine

```bash
# on offline machine: install curl, pv and cowsay
sudo apt-offline set apt-packages.sig --install-packages curl pv cowsay

# offline machine "apt-packages.sig" --> online machine

# on online machine: download package(s) from debian mirrors
apt-offline get apt-packages.sig --threads 4 --bundle offline-packages.zip

# online machine "offline-packages.zip" --> offline machine

# on offline machine
sudo apt-offline install offline-packages.zip
```
