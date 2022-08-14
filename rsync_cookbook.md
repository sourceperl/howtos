# rsync cookbook


## Mirror two directories

```bash
# mirror with ssh
rsync -aALxXv --delete 192.168.0.60:/srv/remote_source/. /srv/local_destination/.
```
