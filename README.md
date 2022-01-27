# AMD RX 5XXXX Series GPUs troubleshooting
Instruction for fixing problems with AMD RX 5xxxx series GPU on Windows and Arch Linux

Hello to all! I know that many people are expeirencing problems with AMD 5000 series graphics cards. I was no exception in this case. More than a year ago I bought an AMD 5600 XT video card from Gigabyte. Like many, in my case the problem lied in random crashes during games. I could play and suddenly see a green screen, after which the computer would restart. At the same time, the temperatures of the processor and video card were absolutely normal, and the resource consumption of the game had no effect on the frequency of crashes. Also the crashes were completely random in time. I could play for a week without problems or I could experience a crash 20 minutes after the start of the game. A couple of times the crash happened not in games even, but just while browsing Google Maps.

Of course, after first crashes I started to seek information about these crashes and there are a lot of information and advice on the Internet regarding such problems. Some people blame the processor, some blame the RAM, some blame the video card. I think that in most cases, with such problems, the video card settings should be fixed first of all. However, a year ago I did not know this and tried a lot of ways to solve the problem, none of which worked. I disabled D.O.C.P, C-states, processor overclocking, reinstalled BIOS and vBIOS, disabled game mode in Windows, changed processor power plans, changed PSI-E 4.0 to PSI-E 3.0, deleted Radeon software and used only driver without special Radeon software. Of course, among these methods there are a couple of useful actions that can help you because I can't guarantee that problems with my GPU and your GPU have the same source. Updating the BIOS, for example, eould not do harm in any case. However, if you, like me, have tried everything you can and nothing has worked, it's time to start adjusting the frequency and voltage of your video card.

**Windows**

Simply adjust the freqency in Radeon Software so it equals to recommended value by vendor of your GPU. If crashes continue then lower the frequency by 10 MGhz until crashes are disappeared.

**Arch Linux**

1) Check your values by
```
# nano /sys/class/drm/card0/device/pp_od_clk_voltage
```
Change "card0" to whatever your card is located. If the file contains only 2 or 3 rows then manual adjusting of your GPU settings isn't activated. You can change that by adding kernel paremeter in your bootloader settings
```
amdgpu.ppfeaturemask=0xffffffff
```
2) Check your maximum power cap limit
```
# nano /sys/class/drm/card0/device/hwmon/hwmon3/power1_cap_max
```

3) Create file amdgpu_settings.sh which contains next lines. Change !frequency, !voltage, !power_cap to your parameters
```
echo "manual" > /sys/class/drm/card0/device/power_dpm_force_performance_level
echo "vc 2 !frequency !voltage" > /sys/class/drm/card0/device/pp_od_clk_voltage
echo "c" > /sys/class/drm/card0/device/pp_od_clk_voltage
echo "s 1 !frequency" > /sys/class/drm/card0/device/pp_od_clk_voltage
echo "!power_cap" > /sys/class/drm/card0/device/hwmon/hwmon3/power1_cap
```
4) Copy that file to /usr/bin and make it executable
```
# cp amdgpu_settings.sh /usr/bin/amdgpu_settings.sh
# chmod +x /usr/bin/amdgpu_settings.sh
```
5) Create amdgpu_settings.service in /lib/systemd/system/ which contains next lines
```
[Unit]
Description=Service for setting amdgpu settings.
Wants=modprobe@amdgpu.service

[Service]
Type=simple
ExecStart=/bin/bash /usr/bin/amdgpu_settings.sh

[Install]
WantedBy=multi-user.target
```
6) Copy this file to /etc/systemd/system and change rights
```
# cp /lib/systemd/system/amdgpu_settings.service /etc/systemd/system/amdgpu_settings.service
# chmod 644 /etc/systemd/system/amdgpu_settings.service
```
7) Start and enable service in systemd
```
# systemctl start amdgpu_settings.service
# systemctl enable amdgpu_settings.service
```
