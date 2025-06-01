# APT cookbook


## Hold a package: prevent the current package from being upgraded

```bash
# here example is for code package (vscode IDE)
# hold it
sudo apt-mark hold code
# show all packages mark as "hold"
apt-mark showhold
# unhold it
sudo apt-mark unhold code
```

## Install a specific version directly from a pool/ directory

```bash
# get package
curl -O http://archive.raspberrypi.com/debian/pool/main/c/code/code_1.97.2-1739406006_arm64.deb
# install it
sudo dpkg -i code_1.97.2-1739406006_arm64.deb
# hold it
sudo apt-mark hold code
```

## Download a DEB package and all its dependencies

```bash
# an example for the "vim" package (download everything to the local directory)
apt-cache depends --recurse --no-recommends --no-suggests --no-conflicts --no-breaks --no-replaces --no-enhances vim | \
grep "^\w" | sort -u | xargs apt download
```
