# Screen lock solutions for Raspberry Pi OS

# dm-tool lock

Bad idea since "ctrl+alt+f7" just bypass this...

# xsreensaver

```bash
# setup
sudo apt install -y xscreensaver
```

Then configure timeout and lock timeout on setup GUI. Delay to lock is timeout + lock timeout delay.


```bash
# from command line, we can trig a screen lock like this
xscreensaver-command -lock
```

# xautolock + xtrlock

Displays a padlock on a black screen until the current user's password is entered (with enter at end).

```bash
# setup
sudo apt install -y xautolock xtrlock
# xautolock start lock utility xtrlock after 10 min of user inactivity under X server
xautolock -time 10 -locker 'xtrlock -b'
```

