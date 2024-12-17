## The device tree overlay for the MAX98090 audio codec is avaliable for RPi4 model B Rev 1.5, armv7l (32-bit) architecture.
**Note**: 
* The driver files for this codec are obtained from [this](https://github.com/raspberrypi/linux/blob/rpi-6.6.y/sound/soc/codecs/max98090.c) repository.
* Before adding the DTS file, make sure that the module for the drivers to your device /(audio codec/) is available, else first complete [these steps](#Steps to re-configure the kernel for module generation)

To edit this DTS file,<br>
 `$ sudo nano max98090.dts`<br>
 
To compile it to .dtbo file,<br>
 `$ dtc -@ -I dts -O dtb -o max98090.dtbo max98090.dts`<br>
 
To copy it to the overlays directory,<br>
 `$ sudo cp max98090.dtbo /boot/firmware/overlays/`<br>
 
Ensure to add it in config.txt file,<br>
 `$ sudo nano /boot/firmware/config.txt`<br>
 
Then add,<br>
 `dtoverlay = max98090` before that make sure if `dtparam=i2s=on` is uncommented in config.txt, otherwise the external I2S interface in RPi'S GPIO will be disabled.<br>
 
Then reboot your RPi with the following command,<br>
 `$ sudo reboot`<be>

## Steps to re-configure the kernel for module generation
For the driver module of the max98090, you should have reconfigured the kernel image. The following steps are mentioned for the [cross-compilation](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#cross-compile-the-kernel),<br>
**"clone git [repo](git clone https://github.com/raspberrypi/linux)"**<br>
```
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
$ sudo env PATH=$PATH make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=mnt/root modules_install   #(sudo env PATH=$PATH make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=media/poojaa/rootfs modules_install) [run on host sys]
$ sudo cp mnt/boot/$KERNEL.img mnt/boot/$KERNEL-backup.img 
$ cp arch/arm/boot/zImage /boot/kernel7l.img    (sudo cp /home/poojaa/rpi-kernel2/linux/arch/arm/boot/zImage /media/poojaa/bootfs/$KERNEL.img)
$ sudo cp arch/arm/boot/dts/broadcom/*.dtb mnt/boot/
$ sudo cp arch/arm/boot/dts/overlays/*.dtb* mnt/boot/overlays/
$ sudo cp arch/arm/boot/dts/overlays/README mnt/boot/overlays/
$ sudo umount mnt/boot
$ sudo umount mnt/root
```
**TIP**: I faced an issue with the header of the `ls -l /usr/lib/modules/build` build directory was pointing to the host system /(used for cross compilation/) instead of pointing to the source directory /(linux-header/) from `/usr/src/` so I copied the cloned directory (after reconfiguration) to /usr/src/ which helped me resolve the issue.<br>

Then in your /boot/firmware/config.txt, add<br>
`kernel=kernel7l.img`<br>
Also, if the architecture of your OS is armv7l but it shows aarch64 when `uname -a`, then add this in config.txt<br>
`arm_64bit=0`<br>
but if it is the countercase, then add<br>
`arm_64bit=1`<br>
