# proxmox-notes
This document holds all of my experience so far with Proxmox, with a focus on retro OSes. Windows 98, Windows XP, Windows Vista.

- Proxmox version: 8.1
- CPU: i7 6700K
- iGPU: Intel HD 530
- GPU1: RTX 3090
- GPU2: AMD R5 340X/R7 250

## Pre-requisites
### Hardware
In order for any of this to work well, the hardware needs to support the OS. This means we need a GPU from the era that has drivers for the specific OS we need. For example:
- Nvidia FX 5500 for Windows 98
- Nvidia GTX 750 Ti for Windows XP

For consumer Nvidia cards on Windows XP up to Windows 7, there is an issue with official drivers which disables itself when it detects that we are running in a VM. The affected drivers are roughly from version 337.88, 340.52 and above.
To get pass this issue, we need to put this to our vm arguments `-cpu host,kvm=off` and use driver version 340.52.

Professional Nvidia (i.e. Quadro) and AMD cards do not have this issue.

For the rest of the components such as CPU, motherboard, it is not as important as long as it is x86, as they can be virtualised or emulated by KVM/Proxmox with decent speeds. Since retro OSes don't consume much CPU cycles, we should get decent speeds regardless. Not sure how this would work on an ARM based system as I have not tried.

As for sound, for XP and up we can get by with the GPU's HDMI/DP Audio output. I have not tried emulated sound hardware in Proxmox. 

### Proxmox settings
DO NOT follow any guide for GPU passthrough! Add all those boot flags and blacklisting the drivers gave me all sorts of issues.
The only thing we need to do is enabling virtio modules (we may even not need to do this, I haven't tested)
```
nano /etc/modules
```
Add the following (copy/paste) to the /etc/modules file:
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

That's it.

If we do anything else, it will prevent our GPUs from outputing the VM's boot sequence, which means we get a black screen until the OS inside the VM loads the graphics driver.

Another thing we should do is setting up our BIOS so that the IGPU is used to boot the computer and run Proxmox. That way all of our discreet GPUs will be able to see the VM's boot sequence and we won't get the above issue.

We can dump the GPU's vbios rom to supposedly preserve this ability to see the VM's boot sequence in case it breaks for whatever reason, but I have not confirmed whether this works. 
In all the different guides online, we'll only find these steps:
```
cd /sys/devices/[device-id-here]
echo 1 > rom
cat rom > /tmp/romfile
echo 0 > rom
```
However if we do this while the IGPU is used for booting, we'll get this error `cat: rom: Input/output error`.
In order to dump this, here are the steps that worked for me:
```
find /sys/devices -name rom
```
This lists all the paths to a possible rom in all of our devices. Take note of the device that we want the rom from (we will need to know our device id for this, but it can be easily seen from Proxmox's PCI Passthrough menu). 
For example, this could be
```
/sys/devices/pci0000:16/0000:16:00.0/0000:17:00.0/0000:18:08.0/0000:19:00.0/0000:1a:10.0/0000:22:00.0/0000:23:00.0/0000:24:00.0/rom
```
Copy the whole thing.

Run this to somehow allow us to read the rom. I will not pretend that I understand what it does, but it works.
```
setpci -s [device id] COMMAND=2:2 (manually manipulate the memory enable bit with setpci)
echo 1 > [paste the thing that we copied from above]
cat [paste the thing] > /tmp/romfile
echo 0 > [paste the thing]
setpci -s [device id] COMMAND=0:2
```

Once we have our romfile, for example if we put it at `~/gpuroms/rtx3090.rom`, here's how to add it to the VM.
```
nano /etc/pve/qemu-server/[vmid].conf
```
Go to the GPU device, the line should start with `pci`. Add this to the comma separated list
`romfile=../../../root/gpuroms/rtx3090.rom`

The 3 backlashes are needed because for some reason, Proxmox tries to find the rom in the wrong location even if we put in the full path (in this case it is `/root/gpuroms/rtx3090.rom`), so we need to get back to the root for the path to work.

I have tried passthrough a Soundblaster XFi but for some reason that freezes the entire physical machine. Not sure what's up with that.

## Windows XP

Needs q35-2.10 to stop the VM from crashing.
If installing XP 32 bit, we need to press F6 when the installer begins and install a AHCI driver. We'll need to attach `xp-satadrivers-ich9-flp.img` as a floppy for the installer to see it. To do this, upload the img to proxmox, and add this line to the vm's conf
```
args: -fda /var/lib/vz/template/iso/xp-satadrivers-ich9-flp.img
```
After pressing F6, pick the one that says **Intel(R) ICH9R/DO/DH SATA AHCI Controller**

Virtual hard drive should be SATA. We can enable Discard and SSD emulation and possibly use SSD Tweaker Pro to run Trim.

Great resources:
- https://forum.mattkc.com/viewtopic.php?t=206

## Windows Vista

Can run perfectly with q35 latest, no additional AHCI drivers required.
Virtual hard drive should be SATA. We can enable Discard and SSD emulation and possibly use SSD Tweaker Pro to run Trim.

If we run the VM with an emulated GPU such as by using the VMWare compatible display, Windows Vista might not load the drivers for our passed through GPU correctly, giving a code 12 error. In this case, we should try setting Display to None and ticking the Primary GPU checkbox for the GPU passthrough.

## Windows 98

Can run with the latest i440 machine type, but according to [this Vogons post](https://www.vogons.org/viewtopic.php?t=94012), we can get SB16 emulation if we pick 2.11.

I have not been able to get GPU Passthrough to work for this.

Great resources:
- https://www.vogons.org/viewtopic.php?t=94012
