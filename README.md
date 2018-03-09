# remarkableflash

This repro includes tools and docs to flash a new image to a remarkable tablet.

WIP and be aware... Here be dragons
## Install the build chain
### 1. Preparation
Create a development folder for remarkable development
```bash
mkdir ~/remarkable_dev
cd remarkable_dev
```
### 1. Install the remarkable SDK
* Download the remarkable SDK and save it in your developer folder:
https://remarkable.engineering/deploy/sdk/
* Execute the shell file:
```bash
chmod +x poky-glibc-x86_64-meta-toolchain-qt5-cortexa9hf-neon-toolchain-2.1.3.sh
./poky-glibc-x86_64-meta-toolchain-qt5-cortexa9hf-neon-toolchain-2.1.3.sh
```
* You will be asked where to install it. Default will be `/opt/poky/2.1.3`. You might change it to a folder in your developer directory e.g. `~/remarkable_dev/poky`.
* Whenever, you want to compile something for the remarkable you have to source the set-up script. This will set all environmental parameters thus, that a build will be created for the remarkable and not the host PC. It is advisable to keep a terminal open for cross-compiling and another one to work on the host-machine. Note the dot in front it is important
```bash
. poky/environment-setup-cortexa9hf-neon-poky-linux-gnueabi
```
### 2. Build uboot
* Clone uboot from the remarkable github page. Configure it via make and build it. Make sure you sourced the poky environment in the very same terminal (see step 1). 
```bash
cd ~/remarkable_dev
git clone https://github.com/reMarkable/uboot
cd uboot
make zero-gravitas_defconfig
make
```
### 3. Build the linux kernel for remarkable
* Clone the linux kernel from the remarkable github page. Configure it via make.
```bash
cd ~/remarkable_dev
git clone https://github.com/reMarkable/linux
cd linux
make zero-gravitas_defconfig
```
* To get access via a serial terminal, modify the generated config file manually. Search in `.config` for the lines 
```bash
# CONFIG_USB_ACM is not set
# CONFIG_USB_CDC_COMPOSITE is not set
```
and change it to 
```bash
CONFIG_USB_ACM=y
CONFIG_USB_CDC_COMPOSITE=y
```
* Run the build process via make (good time to get a coffee or tea, as this can take some time...)
```bash
make 
```
### 4. Build the IMX_USB tool
* Clone the `imx_usb_loader` from the remarkable github page. Configure it via make and build it. This is a tool for the host machine. Compile it with your standard build set-up **NOT** the remarkable SDK (e.g. use another terminal shell).
```bash
cd ~/remarkable_dev
git clone https://github.com/reMarkable/imx_usb_loader.git
cd imx_usb_loader
make
```
### 5. Adapt the config_file for remarkable
Within the `imx_usb_loader` folder different configuration files can be found for different boards. The remarkable tablet will be configured via zero-gravitas.conf. Open it and adjust the paths according to your set-up. If you followed this manual it should be (all comments removed):
```bash
mx6_qsb
hid,1024,0x10000000,1G,0x00907000,0x31000
../uboot/u-boot.imx:dcd,plug
../linux/arch/arm/boot/zImage:load 0x82000000
../linux/arch/arm/boot/dts/zero-gravitas.dtb:load 0x88000000
initramfs.cpio.gz.uboot:load 0x89000000
../uboot/u-boot.imx:clear_dcd,plug,jump header
```
### 6. Start the kernel
* Attach the rM tablet via USB to the host machine
* Power-off the rM tablet
* Set it into upload mode, by pressing the middle button and then the power button. The display will not change, there will be no feedback! 
* To control if the mode was set type:
```bash
dmesg
```
One of the last messages should be something (usb address might be different) like:
```
 hid-generic 0003:15A2:0063.0008: hiddev1,hidraw3: USB HID v1.10 Device [Freescale SemiConductor Inc  SE Blank MEGREZ] on usb-0000:00:1a.0-1.3/input0
```
* Run the `imx_usb` command (might need root access)
```bash
sudo ./imx_usb
```
* In the shell,you should see the upload of files
```bash 
config file <./imx_usb.conf>
vid=0x15a2 pid=0x0063 file_name=zero-gravitas.conf
vid=0x15a2 pid=0x0061 file_name=zero-gravitas.conf
vid=0x066f pid=0x3780 file_name=mx23_usb_work.conf
vid=0x15a2 pid=0x004f file_name=mx28_usb_work.conf
vid=0x15a2 pid=0x0052 file_name=mx50_usb_work.conf
vid=0x15a2 pid=0x0054 file_name=mx6_usb_work.conf
vid=0x15a2 pid=0x0063 file_name=mx6_usb_work.conf
vid=0x15a2 pid=0x0071 file_name=mx6_usb_work.conf
vid=0x15a2 pid=0x007d file_name=mx6_usb_work.conf
vid=0x15a2 pid=0x0076 file_name=mx7_usb_work.conf
vid=0x15a2 pid=0x0041 file_name=mx51_usb_work.conf
vid=0x15a2 pid=0x004e file_name=mx53_usb_work.conf
vid=0x15a2 pid=0x006a file_name=vybrid_usb_work.conf
vid=0x066f pid=0x37ff file_name=linux_gadget.conf
config file <./zero-gravitas.conf>
parse ./zero-gravitas.conf
15a2:0063(mx6_qsb) bConfigurationValue =1
Interface 0 claimed
HAB security state: development mode (0x56787856)
== work item
filename ../uboot/u-boot.imx
load_size 0 bytes
load_addr 0x13f00000
dcd 1
clear_dcd 0
plug 1
jump_mode 0
jump_addr 0x00000000
== end work item

<<<0, 528 bytes>>>
succeeded (status 0x128a8a12)
HAB security state: development mode (0x56787856)
jump_mode 0 plug=0 err=0
== work item
filename ../linux/arch/arm/boot/zImage
load_size 0 bytes
load_addr 0x82000000
dcd 0
clear_dcd 0
plug 0
jump_mode 0
jump_addr 0x00000000
== end work item
load_addr=82000000

loading binary file(../linux/arch/arm/boot/zImage) to 82000000, skip=0, fsize=404380 type=0

<<<4211584, 4211584 bytes>>>
succeeded (status 0x88888888)
HAB security state: development mode (0x56787856)
jump_mode 0 plug=0 err=0
== work item
filename ../linux/arch/arm/boot/dts/zero-gravitas.dtb
load_size 0 bytes
load_addr 0x88000000
dcd 0
clear_dcd 0
plug 0
jump_mode 0
jump_addr 0x00000000
== end work item
load_addr=88000000

loading binary file(../linux/arch/arm/boot/dts/zero-gravitas.dtb) to 88000000, skip=0, fsize=7858 type=0

<<<30808, 30808 bytes>>>
succeeded (status 0x88888888)
HAB security state: development mode (0x56787856)
jump_mode 0 plug=0 err=0
== work item
filename initramfs.cpio.gz.uboot
load_size 0 bytes
load_addr 0x89000000
dcd 0
clear_dcd 0
plug 0
jump_mode 0
jump_addr 0x00000000
== end work item
load_addr=89000000

loading binary file(initramfs.cpio.gz.uboot) to 89000000, skip=0, fsize=9df6b2 type=0

<<<10352306, 10352306 bytes>>>
succeeded (status 0x88888888)
HAB security state: development mode (0x56787856)
jump_mode 0 plug=0 err=0
== work item
filename ../uboot/u-boot.imx
load_size 0 bytes
load_addr 0x13f00000
dcd 0
clear_dcd 1
plug 1
jump_mode 2
jump_addr 0x00000000
== end work item
clear dcd_ptr=0x877ff42c

loading binary file(../uboot/u-boot.imx) to 877ff400, skip=0, fsize=36c00 type=aa

<<<224256, 224256 bytes>>>
succeeded (status 0x88888888)
jumping to 0x877ff400
```
* The rM tablet display might refresh and the message `reMarkable is starting` might appear. However, the tablet remains unresponsive at this point.
### 7. Access the remarkable via the UART interface
* Check with `dmesg` the current output, there should be a line like:
```bash
cdc_acm 1-1.3:1.2: ttyACM0: USB ACM device
```
* Note the name of the serial device (it might differ from distro to distro). To access the tablet use a serial communication program like minicom, picocom or even screen (you might need sudo rights)
```bash
screen /dev/ttyACM0
```
* A login prompt will appear:
```bash 
Poky (Yocto Project Reference Distro) 1.8.1 imx6dlsabresd /dev/ttyGS0
imx6dlsabresd login: 
```
* Use `root` as password
### 8. Mount the internal flash memory partitions
* The entire visible system is the initramfs within the rM RAM. Thus, the flash memory partitions of the real system have to be mounted.
```bash 
mount /dev/mmcblk1p2 /mnt/
mount /dev/mmcblk1p7 /mnt/home
mount /dev/mmcblk1p1 /var/lib/uboot
```
* For convince, one can chroot into the real system.
```bash 
chroot /mnt
```
* You can now change settings or reset passwords, etc. After you finished, type 
```bash 
exit #if you used the chroot
reboot
``` 
to restart the rM tablet and boot into the normal operation mode.

