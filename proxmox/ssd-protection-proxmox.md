[Back to Proxmox](README.md)

# SSD Protection in Proxmox

> [!WARNING]  
> These are only info that I gathered, I have not tested them


Proxmox by default writes frequent logs to the drive that you installed it in. There's a high chance you used an SSD for this. So here are some relevant info on how to limit frequent writing to the drive.

## Use zram for swap
https://pve.proxmox.com/wiki/Zram

To turn off physical SSD swap, comment out the other swap entry that is not the `zram` one in `/etc/fstab`

## Use log2ram to avoid writing logs to disk 
https://github.com/azlux/log2ram

## Limit the number of days that logs are stored 
Check out the comments in this topic
https://forum.proxmox.com/threads/log2ram-getting-full-frequently-and-proxmox-becomes-unusable.98647/

Edit the files in `/etc/logrotate.d` and change the `rotate` value to 3.
For example:
```
~# cat /etc/logrotate.d/pve-firewall
/var/log/pve-firewall.log {
    rotate 7
    daily
    missingok
    notifempty
    delaycompress
    compress
    sharedscripts
    create 640 root adm
    postrotate
        invoke-rc.d pvefw-logger restart 2>/dev/null >/dev/null || true
    endscript
}
```

## Proxmox logging to RAM
The below is a comment from a Proxmox forum thread, I'm not sure if it works

If are running Proxmox for personal use, we don't need persistence in logging that much. Here's a few ways I found on reddit to reduce logging written to disk

If you're not running in cluster mode, you can 
```
systemctl disable --now pve-ha-lrm.service pve-ha-crm.service
```
These two seem to be responsible for lots of low end drive deaths.

You can also
```
nano /etc/systemd/journald.conf
```
```
Storage=volatile
ForwardToSyslog=no
```
## Disable all logging
https://forum.proxmox.com/threads/is-it-possible-to-disable-all-logs-local-completly.52067/
