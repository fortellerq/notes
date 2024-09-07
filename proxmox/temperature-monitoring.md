[Back to Proxmox](README.md)

# Temperature Monitoring in Proxmox

Since the motherboard is not directly exposed to the VMs, we can't monitor temperatures of the CPU and motherboard directly in the VM.
However, we can do this on the Proxmox host.

First, install the lm-sensors package
```
apt install lm-sensors
```

Now we can see the current readings with
```
sensors
```

To continually monitor the sensors, do
```
watch -n 4 sensors
```
The above command will show the readings and refresh every 4 seconds.

There is a way to edit the Proxmox dashboard's template and manually add html and javascript to show the values. However, it's not a method I recommend, due to the below reasons
- If you make a mistake when editing the template, such as a typo, you won't be able to access the dashboard
- Any future update that changes the template will revert your changes

Good resources:
- https://gist.github.com/tinwhisker/53d77c887535129021a1f58359930935
- https://itsfoss.com/monitor-cpu-gpu-temp-linux/
