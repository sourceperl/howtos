# Unattended-upgrades

Automatically installs security updates on Debian-based systems, in the background.

## Setup

```bash
# add packages
sudo apt update
sudo apt install unattended-upgrades apt-listchanges
```

**On some cloud platforms, such as OVH VPS instances running Debian, unattended-upgrades appears to be pre-installed 
and active by default.**


## Configure package update origins

Modify the `Unattended-Upgrade::Origins-Pattern` section. This defines which repositories `unattended-upgrades` will 
consider for automatic updates. In the following example, we also add an element to update Grafana.

`/etc/apt/apt.conf.d/50unattended-upgrades` :

```c
Unattended-Upgrade::Origins-Pattern {
        // Codename based matching:
        // This will follow the migration of a release through different
        // archives (e.g. from testing to stable and later oldstable).
        // Software will be the latest available for the named release,
        // but the Debian release itself will not be automatically upgraded.
//      "origin=Debian,codename=${distro_codename}-updates";
//      "origin=Debian,codename=${distro_codename}-proposed-updates";
        "origin=Debian,codename=${distro_codename},label=Debian";
        "origin=Debian,codename=${distro_codename},label=Debian-Security";
        "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
        // add for grafana support
        "site=apt.grafana.com";
}
```


## Configure server auto reboot

To enable the automatic reboot of your server after an upgrade, you need to edit the `50unattended-upgrades` 
configuration file.

Edit `/etc/apt/apt.conf.d/50unattended-upgrades` :

*Near line 115, you will find the following section. **Uncomment** the two lines shown below by removing the `//` at 
the beginning of each.*

```c
// Automatically reboot *WITHOUT CONFIRMATION* if
//  the file /var/run/reboot-required is found after the upgrade
Unattended-Upgrade::Automatic-Reboot "true";

// If automatic reboot is enabled and needed, reboot at the specific
// time instead of immediately
//  Default: "now"
Unattended-Upgrade::Automatic-Reboot-Time "02:00";

```

* **`Unattended-Upgrade::Automatic-Reboot "true";`**: This setting activates the automatic reboot feature. If the system
 determines that a reboot is required after an unattended upgrade (signified by the presence of the 
 `/var/run/reboot-required` file), the server will restart automatically.

* **`Unattended-Upgrade::Automatic-Reboot-Time "02:00";`**: This line allows you to specify a precise time for 
the automatic reboot. In this example, the reboot is scheduled for 2:00 AM. You can adjust `"02:00"` to any time that 
best suits your needs to minimize any potential disruption. If this line remains commented out, or if the value is set 
to `"now"`, the server will reboot immediately upon detecting a required restart.

This configuration helps ensure your server remains secure and up-to-date by automating necessary reboots at a 
convenient time.


## Perform a Dry Run

This is the safest way to test your configuration without actually making any changes to your system. The 
`unattended-upgrade` command with the `--dry-run` and `--debug` options will simulate the upgrade process and show you 
what would happen.

```bash
sudo unattended-upgrade --dry-run --debug
```


## Check the Logs

`unattended-upgrades` keeps detailed logs of its operations. 
You can examine these logs to see past activities and ensure that the process is running without errors.

The main log file is:
`/var/log/unattended-upgrades/unattended-upgrades.log`

You can view the latest entries using `tail`:

```bash
tail /var/log/unattended-upgrades/unattended-upgrades.log
```


## Verify Systemd Timers (if applicable)
On modern Debian systems, `unattended-upgrades` is often triggered by systemd timers. 
You can check the status of these timers to ensure they are enabled and running.

```bash
systemctl status apt-daily.timer
systemctl status apt-daily-upgrade.timer
```

This will show you when they last ran and when they are scheduled to run next.


## More infos

> https://reintech.io/blog/configure-automatic-security-updates-debian-12

> https://wiki.debian.org/fr/unattended-upgrades
