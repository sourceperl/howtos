# HOWTO: Minimal graphical image for a Tk dashboard on Raspberry Pi (64-bit)

> Target: any 64-bit capable Raspberry Pi (3, 3B+, 3A+, 4, 400, 5, Zero 2W), Raspberry Pi OS Lite 64-bit (based on Debian 13 "Trixie"), minimal X11, Python/Tkinter application running full screen at boot.

## Contents

1. [Approach](#1-approach)
2. [Prerequisites](#2-prerequisites)
3. [Flash the base image](#3-flash-the-base-image)
4. [Install the minimal graphical layer](#4-install-the-minimal-graphical-layer)
5. [Auto-login and starting X](#5-auto-login-and-starting-x)
6. [Tk dashboard skeleton](#6-tk-dashboard-skeleton)
7. [Silent boot (optional)](#7-silent-boot-optional)
8. [French locale (optional)](#8-french-locale-optional)
9. [Turn the result into a reusable image](#9-turn-the-result-into-a-reusable-image)
10. [Alternative: "pure" Debian](#10-alternative-pure-debian)
11. [Sources](#11-sources)

---

## 1. Approach

Start from **Raspberry Pi OS Lite 64-bit** (no desktop environment) and add only what is strictly needed:

- a minimal X11 server, without a compositor (Tk is native to X11);
- a very lightweight window manager (optional) to handle full screen;
- Python 3 + Tkinter.

This keeps the official Pi hardware support (firmware, KMS, `raspi-config`) with a minimal footprint.

Raspberry Pi OS is based on Debian 13 "Trixie", the current Debian stable. Its 64-bit version runs on the Pi 3, 3B+, 3A+, 4, 400, 5 and Zero 2W.

## 2. Prerequisites

- A 64-bit capable Raspberry Pi with its official power supply
- microSD card (16 GB or more recommended)
- A display
- A computer with [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- Network access for configuration over SSH

## 3. Flash the base image

1. In Raspberry Pi Imager, choose **your Raspberry Pi model → Raspberry Pi OS (other) → Raspberry Pi OS Lite (64-bit)**.
2. In the customisation options, set:
   - the hostname;
   - the user (e.g. `pi`) and its password;
   - Wi-Fi (if needed);
   - **SSH** enabled.
3. Flash the card, boot the Pi and connect over SSH:

```bash
ssh pi@<hostname>.local
```

## 4. Install the minimal graphical layer

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install --no-install-recommends -y \
  xserver-xorg-core xserver-xorg-input-libinput xinit x11-xserver-utils \
  python3-tk fonts-dejavu-core unclutter matchbox-window-manager
```

| Package | Purpose |
|---|---|
| `xserver-xorg-core` | X server (includes the `modesetting` driver used with KMS) |
| `xserver-xorg-input-libinput` | Keyboard, mouse, touchscreen |
| `xinit` | Provides `startx` |
| `x11-xserver-utils` | `xset`, `xrandr` (blanking, rotation) |
| `python3-tk` | Tkinter |
| `fonts-dejavu-core` | Readable default font |
| `unclutter` | Hides the idle cursor |
| `matchbox-window-manager` | Minimal window manager (optional) |

> 💡 `--no-install-recommends` avoids pulling in hundreds of unnecessary packages.
>
> ℹ️ Trixie also provides official meta-packages (`rpd-x-core`, `rpd-wayland-core`) to turn a Lite image into a desktop. They are heavier than needed for a Tk kiosk.

## 5. Auto-login and starting X

### 5.1 Console auto-login

```bash
sudo raspi-config nonint do_boot_behaviour B2
```

### 5.2 Start X automatically on tty1

On Debian and Raspberry Pi OS, the file read at login is **`~/.profile`**. Append the block to it:

```bash
cat >> ~/.profile <<'EOF'

# Start X automatically on tty1
if [ -z "$DISPLAY" ] && [ "$(tty)" = "/dev/tty1" ]; then
  exec startx -- -nocursor
fi
EOF
```

> ⚠️ Don't create a `~/.bash_profile`: if it exists, bash stops reading `~/.profile` (PATH, `~/.bashrc`, etc.). If you create one anyway, its first line must be `. ~/.profile`.
>
> The `tty1` check ensures X only starts on the local boot console. SSH sessions remain plain shells.

### 5.3 The `~/.xinitrc` file

```bash
cat > ~/.xinitrc <<'EOF'
#!/bin/sh
xset s off
xset s noblank
xset -dpms
unclutter -idle 0.5 &
matchbox-window-manager -use_titlebar no &
exec python3 /home/pi/dashboard/app.py
EOF
chmod +x ~/.xinitrc
```

| Line | Effect |
|---|---|
| `xset s off` / `s noblank` / `-dpms` | Disables the screensaver and display power-off |
| `unclutter` | Hides the cursor after 0.5 s of inactivity |
| `matchbox-window-manager` | Clean full screen, no title bar |
| `exec python3 …` | Starts the dashboard; when the app exits, X stops |

## 6. Tk dashboard skeleton

Create `/home/pi/dashboard/app.py`:

```python
import tkinter as tk
import time

root = tk.Tk()
root.attributes("-fullscreen", True)
root.configure(bg="black")
root.bind("<Escape>", lambda e: root.destroy())

lbl = tk.Label(root, font=("DejaVu Sans", 64), fg="white", bg="black")
lbl.pack(expand=True)

def tick():
    lbl.config(text=time.strftime("%H:%M:%S"))
    root.after(1000, tick)

tick()
root.mainloop()
```

Test with `sudo reboot`: the Pi should boot straight into the full-screen clock.

## 7. Silent boot (optional)

On Trixie, the boot files live in `/boot/firmware/`.

- In **`/boot/firmware/config.txt`**, add:

  ```ini
  disable_splash=1
  ```

- In **`/boot/firmware/cmdline.txt`**, append to the end of the **single existing line**:

  ```text
  quiet loglevel=3 logo.nologo consoleblank=0 vt.global_cursor_default=0
  ```

## 8. French locale (optional)

If the SSH login shows:

```text
-bash: warning: setlocale: LC_ALL: cannot change locale (fr_FR.UTF-8): No such file or directory
```

your client is forwarding its locale, but that locale has not been generated on the Pi. Generate it:

```bash
sudo sed -i 's/^# *fr_FR.UTF-8 UTF-8/fr_FR.UTF-8 UTF-8/' /etc/locale.gen
sudo locale-gen
sudo update-locale LANG=fr_FR.UTF-8
```

Alternatively, stop forwarding the locale by commenting out `SendEnv LANG LC_*` in the client's `/etc/ssh/ssh_config`.

## 9. Turn the result into a reusable image

### Method A: clone the configured card (quick)

Insert the SD card into a Linux computer and identify its device with `lsblk`, then:

```bash
sudo dd if=/dev/sdX of=tk-dashboard.img bs=4M status=progress conv=fsync
sudo pishrink.sh -z tk-dashboard.img   # shrinks the image and re-enables auto-expansion
```

Then flash `tk-dashboard.img.gz` to other cards with Raspberry Pi Imager (**Use custom**).

> ⚠️ Double-check `/dev/sdX`: picking the wrong device overwrites the wrong disk.

### Method B: build the image from scratch (reproducible)

With Trixie, the Raspberry Pi image build process became more modular. The official **`rpi-image-gen`** tool (GitHub repository `raspberrypi/rpi-image-gen`) lets you describe in a configuration:

- the Lite 64-bit base;
- the package list from step 4;
- your files (`~/.profile`, `~/.xinitrc`, `app.py`, `config.txt`, `cmdline.txt`, locale).

The image is then built with a single command. This is the recommended approach if the image needs to be rebuilt regularly or kept under version control.

## 10. Alternative: "pure" Debian

Official Debian images for Raspberry Pi exist ([raspi.debian.net](https://raspi.debian.net/)). They don't include the Raspberry Pi packages, and hardware support (display, `raspi-config`, firmware) is less complete. **For a dashboard, Raspberry Pi OS Lite remains the recommended choice.**

## 11. Sources

- [Trixie — the new version of Raspberry Pi OS (raspberrypi.com)](https://www.raspberrypi.com/news/trixie-the-new-version-of-raspberry-pi-os/)
- [Notebookcheck – Raspberry Pi OS Trixie](https://www.notebookcheck.net/Raspberry-Pi-releases-free-major-Trixie-OS-update-for-Raspberry-Pi-single-board-computers.1130182.0.html)
- [Neowin – Raspberry Pi OS Trixie](https://www.neowin.net/amp/raspberry-pi-os-trixie-unveils-new-control-center-and-future-proof-design/)
- [RaspberryTips – Which Raspberry Pi OS version](https://raspberrytips.com/raspberry-pi-os-versions/)