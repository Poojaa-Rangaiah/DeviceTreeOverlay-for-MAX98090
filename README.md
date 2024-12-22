## The device tree overlay for the MAX98090 audio codec.
\- Raspberry Pi 4 Model B Rev 1.5, armv7l architecture.

**Note**: 
* The driver files for this codec are obtained from [this](https://github.com/raspberrypi/linux/blob/rpi-6.6.y/sound/soc/codecs/max98090.c) repository.
* The device-tree binding is referred from [max98090](https://github.com/raspberrypi/linux/blob/rpi-6.6.y/Documentation/devicetree/bindings/sound/maxim%2Cmax98090.yaml).
* Before adding the DTS file, make sure that the module for the drivers to your device (audio codec) is available, else first complete [these steps](#Steps-to-re-configure-the-kernel-for-module-generation).

To edit this DTS file,<br>
 ```$ sudo nano max98090.dts```<br>
 
To compile it to .dtbo file,<br>
 ```$ dtc -@ -I dts -O dtb -o max98090.dtbo max98090.dts```<br>
 
To copy it to the overlays directory,<br>
 ```$ sudo cp max98090.dtbo /boot/firmware/overlays/```<br>
 
Ensure to add it in config.txt file,<br>
 ```$ sudo nano /boot/firmware/config.txt```<br>
 
Then add,<br>
 ```dtoverlay = max98090``` before that make sure if ```dtparam=i2s=on``` is uncommented in config.txt, otherwise the external I2S interface in RPi'S GPIO will be disabled.<br>
 
Then reboot your RPi with the following command,<br>
 ```$ sudo reboot```


## Steps to re-configure the kernel for module generation.
For the driver module of the max98090, you should have reconfigured the kernel image. The following steps are mentioned for the [cross-compilation](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#cross-compile-the-kernel),<br>
```
$ git clone https://github.com/raspberrypi/linux
$ cd linux/
$ sudo apt install bc bison flex libssl-dev make libc6-dev libncurses5-dev
$ sudo apt install crossbuild-essential-armhf
$ export KERNEL=kernel7l
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2711_defconfig #(2711 for RPi4)
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
$ make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage
$ make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- dtbs
$ make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- modules
```
Then, [find your boot media](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#find-your-boot-media).<br>
Once all these are done, you must include the modules in your root media.<br>
```
$ sudo env PATH=$PATH make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=mnt/root modules_install #[run on host sys]
$ sudo cp mnt/boot/$KERNEL.img mnt/boot/$KERNEL-backup.img 
$ cp arch/arm/boot/zImage /boot/kernel7l.img
$ sudo cp arch/arm/boot/dts/broadcom/*.dtb mnt/boot/
$ sudo cp arch/arm/boot/dts/overlays/*.dtb* mnt/boot/overlays/
$ sudo cp arch/arm/boot/dts/overlays/README mnt/boot/overlays/
$ sudo umount mnt/boot
$ sudo umount mnt/root
```
**TIP**: 
- I faced an issue with the header of the `ls -l /usr/lib/modules/build` build directory was pointing to the host system (used for cross compilation) instead of pointing to the source directory (linux-header) from `/usr/src/` so I copied the cloned directory (after reconfiguration) to `/usr/src/` which helped me resolve the issue.<br>

Then in your /boot/firmware/config.txt, add<br>
```kernel=kernel7l.img```<br>
Also, if the architecture of your OS is armv7l but it shows aarch64 when `uname -a`, then add this in config.txt<br>
```arm_64bit=0```<br>
but if it is the countercase, then add<br>
```arm_64bit=1```<br>


## Verification commands.
To check if the driver and overlay is loaded.
```
$ sudo vclog -m | grep max98090 (or) sudo vcdbg log msg | grep max98090
$ dmesg | grep -i max98090
$ lsmod | grep max98090
```
To check the sound card created for playback and capture.
```
$ aplay -l
$ arecord -l
```
> Example:
> ```
> pi@raspberrypi:~ $ aplay -l
> **** List of PLAYBACK Hardware Devices ****
> card 0: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
>   Subdevices: 8/8
>   Subdevice #0: subdevice #0
>   Subdevice #1: subdevice #1
>   Subdevice #2: subdevice #2
>   Subdevice #3: subdevice #3
>   Subdevice #4: subdevice #4
>   Subdevice #5: subdevice #5
>   Subdevice #6: subdevice #6
>   Subdevice #7: subdevice #7
> card 1: MAX98090AudioCo [MAX98090-Audio-Codec], device 0: fe203000.i2s-HiFi HiFi-0 [fe203000.i2s-HiFi HiFi-0]
>   Subdevices: 1/1
>   Subdevice #0: subdevice #0
> card 2: vc4hdmi0 [vc4-hdmi-0], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
>   Subdevices: 1/1
>   Subdevice #0: subdevice #0
> card 3: vc4hdmi1 [vc4-hdmi-1], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
>   Subdevices: 1/1
>   Subdevice #0: subdevice #0
> pi@raspberrypi:~ $ arecord -l
> **** List of CAPTURE Hardware Devices ****
> card 1: MAX98090AudioCo [MAX98090-Audio-Codec], device 0: fe203000.i2s-HiFi HiFi-0 [fe203000.i2s-HiFi HiFi-0]
>   Subdevices: 1/1
>   Subdevice #0: subdevice #0
> ```
Other commands for more details,
```
$ cat /proc/interrupts
$ sudo cat /sys/kernel/debug/clk/clk_summary
$ cat /proc/asound/modules
$ cat /proc/asound/cards
```
