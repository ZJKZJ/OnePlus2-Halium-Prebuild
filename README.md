# OnePlus2-Halium-Prebuild

## what works what don't work?
 * [x] boot
  * [x] graphics
  * [ ] calls
  * [x] sound
  * [x] gps
  * [x] wifi (FIXED)
  * [ ] Bluetooth (WIP+)
  * [ ] Camera
  * [x] anbox


### For Ubuntu Touch

just use the prebuild image just as  a normal compiled image.
install it with the JBB's halium-install script [here](https://github.com/JBBgameich/halium-install)
and get the ubports edge rootfs from [here](https://ci.ubports.com/job/xenial-rootfs-armhf/lastSuccessfulBuild/artifact/out/ubports-touch.rootfs-xenial-armhf.tar.gz)
```./halium-install -p ut ubports-touch.rootfs-xenial-edge-armhf.tar.gz system.img```
```sudo fastboot flash boot halium-boot.img```

then while in TWRP
```adb shell 'touch /data/.writable_image; mkdir /a; mount /data/rootfs.img /a; echo manual | tee /a/etc/init/rsyslog.override;  touch /a/.writable_device_image; umount /a; sync'```


some command are needed in order to get a working UT device (run as root).
```
chmod 666 /dev/kgsl-3d0

cat /var/lib/lxc/android/rootfs/ueventd*.rc|grep ^/dev|sed -e 's/^\/dev\///'|awk '{printf "ACTION==\"add\", KERNEL==\"%s\", OWNER=\"%s\", GROUP=\"%s\", MODE=\"%s\"\n",$1,$3,$4,$2}' | sed -e 's/\r//' > /etc/udev/rules.d/70-oneplus2.rules

chmod 4777 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
chown root:messagebus /usr/lib/dbus-1.0/dbus-daemon-launch-helper
chmod u+s /usr/lib/dbus-1.0/dbus-daemon-launch-helper

adduser --force-badname --system --home /nonexistent --no-create-home --quiet _apt

mkdir -p /etc/system-image/config.d
mkdir /lib/firmware
ln -s /system/etc/firmware/* /lib/firmware/
# ln -s /firmware/image/*  /lib/firmware/
echo manual | tee /etc/init/apparmor.override

cd /tmp
wget https://ci.ubports.com/job/pulseaudio-modules-droid/job/PR-1/3/artifact/pulseaudio-modules-droid2_0.1+0~20181121174750.3~1.gbp54cf4b_armhf.deb --no-check-certificate
dpkg -i pulseaudio-modules-droid2_0.1+0~20181121174750.3~1.gbp54cf4b_armhf.deb
sed -i -e "s/load-module module-droid-discover voice_virtual_stream=true/load-module module-droid-card rate=48000 /" /etc/pulse/touch.pa

```

### For Ubuntu Touch Anbox (WIP)

follow the instruction above, then follow this.

reflash the boot

```sudo fastboot flash boot halium-boot-anbox.img```

check if exist
/dev/anbox-binder


if it doesn't, you are not using the right ```halium-boot.img```

then
```
sudo apt install anbox-ubuntu-touch
sudo wget -q --show-progress -O /home/phablet/anbox-data/android.img http://cdimage.ubports.com/anbox-images/android-armhf-64binder.img
touch /home/phablet/anbox-data/.enable
```
<br />
some permition fixes are needed.

```
sudo chmod -R o+wrx /home/phablet/anbox-data/data
sudo start -q anbox-container
#personnaly this one fail
start -q anbox-session
```

reboot, wait 2 minutes then reboot again
