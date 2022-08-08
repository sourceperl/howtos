# Create a 16MB RAM disk (with tmpfs)

## add a mountpoint for tmpfs: /media/ramdisk/

```bash
sudo mkdir -p /media/ramdisk/
```

## update fstab for mount RAMDISK at system startup

```bash
# define env vars
LINE='tmpfs           /media/ramdisk  tmpfs   defaults,size=16M 0       0'
FILE='/etc/fstab'
# add content to fstab if it not already exist
sudo grep -q "$LINE" "$FILE" || echo "$LINE" | sudo tee -a "$FILE"
```

## reboot to take effect

```bash
sudo reboot
```