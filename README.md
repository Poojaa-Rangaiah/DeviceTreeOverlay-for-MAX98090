## MAX98090-Audio-Codec
#The device tree overlay for the MAX98090 audio codec is avaliable for RPi4 model B Rev 1.5, armv7l (32-bit) architecture.

To edit this DTS file,<br>
 `$sudo nano max98090.dts`<br>
To compile it to .dtbo file,<br>
 `$dtc -@ -I dts -O dtb -o max98090.dtbo max98090.dts`<br>
To copy it to the overlays directory,<br>
 `$sudo cp max98090.dtbo /boot/firmware/overlays/`<br>
Ensure to add it in config.txt file,<br>
 `$sudo nano /boot/firmware/config.txt`<br>
Then add,<br>
 `dtoverlay = max98090` before that make sure if `dtparam=i2s=on` is uncommented, otherwise the external I2S interface in RPi'S GPIO will be disabled.<br>
Then reboot your RPi with the following command,<br>
 `$sudo reboot`<br>
