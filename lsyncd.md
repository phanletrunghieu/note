## Log

```
sudo mkdir /var/log/lsyncd
touch /var/log/lsyncd/lsyncd.{log,status}
```

## config file `/etc/lsyncd/lsyncd.conf.lua`

config file in source server

```
settings {
        logfile = "/var/log/lsyncd/lsyncd.log",
        statusFile = "/var/log/lsyncd/lsyncd.status"
}

sync {
        default.rsyncssh,
        delete = false,
        source = "/root/sync_source",
        host = "103.56.156.217",
        targetdir = "/root/sync_backup"
}
```
