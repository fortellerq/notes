[Back to main page](README.md)

# Protect your SSD when running legacy OSes
## Windows XP and Vista
### Disable autolayout
- Browse to [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\OptimalLayout]
- Add a new DWORD, call EnableAutoLayout, leave value as 0

### Disable background defragmentation
- Browse to [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Dfrg\BootOptimizeFunction]
- Change "Enable" to "N"

### Disable prefetch
- Browse to [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters]
- Change "EnablePrefetcher" to 0
- (Vista only) Change "EnableSuperfetch" to 0
- Delete all files in C:\Windows\Prefetch

### Disable pagefile on the SSD
- Right click on My Computer, Properties, Advanced, Performance Settings, Advanced

### (Vista only) Turn off ReadyBoot
- Open Control Panel, go to Reliability and Performance Monitor
- In the left panel, open the "Data Collection Sets" and click on "Startup Event Trace Sessions"
- In the right panel, double-click on "Readyboot"
- Open the "Trace Session" tab
- Remove the check from the "Enabled" checkbox

### Install SSD Tweaker Pro version 4.0.1
I won't post a link but you can search on the internet for a download of this tool, which will allow you to run the TRIM command manually. Version 4.0.1 should be compatible with both XP and Vista.
