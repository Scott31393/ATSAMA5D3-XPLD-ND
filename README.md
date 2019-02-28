# ATSAMA5D3-XPLD-ND
Bring up Linux kernel 4.19.9 on ATSAMA5D3-XPLD-ND


## Board Connections

![alt text](./imgs/board.jpg "Logo Title Text 1")


![alt text](./imgs/board.jpeg "Logo Title Text 1")


## Booting Phase (Bootstrap - U-Boot)

![alt text](./imgs/booting_phase.jpeg)

# Board Datasheet

## Memory Mapping

![alt text](./imgs/memory_mapping.png)

# Commands Cheatsheet


##LS COMMAND
export PATH=/usr/bin:/bin


###GUIDE AND SLIDES

https://bootlin.com/doc/training/linux-kernel/
wget https://bootlin.com/doc/training/embedded-linux/embedded-linux-labs.tar.xz

###NEEDED PACKAGES

sudo apt install build-essential git autoconf bison flex texinfo help2man gawk libtool-bin libncurses5-dev

###CROSSTOOL-NG
git clone https://github.com/crosstool-ng/crosstool-ng.git

###INSTALL-CROSSTOOL-NG

./bootstrap
./configure --enable-local
make

./ct-ng help [help]

###CONFIGURE TOOLCHAIN TO PRODUCE MANY TOOLCHAIN FOR DIFFERENT ARCHITECTURE

./ct-ng arm-cortexa5-linux-uclibcgnueabihf
./ct-ng menuconfig

##SET OPTIONS
Path and misc options: --> Change Maximum log level to see to DEBUG
Toolchain options: --> Set Tuple's alias to "arm-linux"
C-library: --> Enable IPv6 support
Debug facilities: --> Only enable strace support


###SETTING PATH ENV VARIABLE
export  PATH=$HOME/x-tools/arm-cortexa5-linux-uclibcgnueabihf/bin/

###CLEAN
./ct-ng clean

###TO COMPILE
arm-linux-gcc hello.c


###DOWNLOAD MICROCHIP FLASHING-TOOL [sam-ba]
wget http://ww1.microchip.com/downloads/en/DeviceDoc/sam-ba_2.15.zip


###SETTING UP SERIAL COMMUNICATION WITH THE BOARD
install picocom --> sudo apt install picocom
sudo adduser $USER dialout
picocom -b 115200 /dev/ttyUSB0


###AT91BOOTSTRAP SETUP BOOTSTRAP IN SRAM THAT INITIALIZE DRAM WITH U-BOOT INSIDE THAT LOAD THE LINUX KERNEL
git clone https://github.com/linux4sam/at91bootstrap.git
cd at91bootstrap
git checkout v3.8.9


###BUILD THE TOOLCHAIN
./ct-ng build  (BUILT IN  $HOME/x-tools/)
make sama5d3_xplainednf_uboot_defconfig

#SET ENVIRONMENT VARIABLE
export CROSS_COMPILE=arm-linux

###AT91Bootstrap SETUP
remove the NAND CS jumper on the board
press the RESET button
Put the jumper back
running samba64 
./sam-ba_64 in his folder
Select the ttyACM0 connection, and the at91sama5d3x-xplained board. Hit Connect.
Hit the NANDFlash tab
In the Scripts choices, select Enable NandFlash and hit Execute
Select Erase All, and execute the command
Then, select and execute Enable OS PMECC parameters in order to change the NAND ECC parameters to what RomBOOT expects.
Finally, send the image we just compiled using the command Send Boot File

keep sam-ba open!!!!



###U-BOOT SETUP
download u-boot-->wget ftp://ftp.denx.de/pub/u-boot/u-boot-2017.09.tar.bz2 
make <NAME>_defconfig   NAME = sama5d3_xplained_nandflash
sudo apt install device-tree-compiler
make that build u-boot
Look at the size of the u-boot.bin --> ls -l <FILE-NAME> --> too large
make menuconfig, look for and disable the below options
ext4 options
• nfs options
• USB options
• SPL options
• XIMG support
• FIT (Flattened Image Tree) support
• CMD_ELF option
• dhcp command support
• Regular expression support (REGEX)
• loadb support
• CMD_MII option

recompile --> make


###FLASHING U-BOOT
In sam-ba, in the Send File Name field, set the path to the u-boot.bin that was just
compiled, and set the address to 0x40000. Click on the Send File button.


###TESTING U-BOOT AND AT91Bootstrap
Reset the board and check that it boots your new bootloaders

--->help to see u-boot commands


####Setting up Ethernet communication (Using tftp)

sudo apt-get install tftp

https://www.poftut.com/install-configure-run-linux-tftp-client/
http://wiki.r1soft.com/display/ServerBackupManager/Configure+a+TFTP+server+on+Linux
https://www.linuxquestions.org/questions/linux-embedded-and-single-board-computer-78/tftp-retry-count-exceeds-4175430611/

->folder-> /var/lib/tftpboot/ -> Create file.txt


sudo apt-get install xinetd

->folder ->create  /etc/xinetd.d/tftp

To set up create tftp file and write:

service tftp
{
protocol = udp
port = 69
socket_type = dgram
wait = yes
user = nobody
server = /usr/sbin/in.tftpd
server_args = -s /var/lib/tftpboot [FOR CHANGE THE DEFAULT TFTP FOLDER]
disable = no
}


$ifconfig -a

->enp0s31f6 Link encap:Ethernet  HWaddr 70:85:c2:57:82:ae  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:49 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:6175 (6.1 KB)
          Interrupt:16 Memory:df100000-df120000 

Configuring the host(My PC) IP address from command line (ip address = 192.168.0.1).

$nmcli con add type ethernet ifname enp0s31f6 ip4 192.168.0.1/24



Configure the network on the board in U-Boot by setting the ipaddr and serverip envi-ronment variables:

setenv ipaddr 192.168.0.100 (Target IP)
setenv serverip 192.168.0.1 (My PC/Host IP)

set the MAC address in U-boot:

setenv ethaddr 12:34:56:ab:cd:ef (Target Mac address)

saveenv

reset

To check if all working try to ping the Host(Server) from uBoot:

ping 192.168.0.1

and try to write prova.txt with something inside using :

tftp 0x22000000 textfile.txt

The tftp command should have downloaded the textfile.txt file from your development workstation into the board’s memory at location 0x22000000.

Verify with:

md 0x22000000

####Kernel sources

Get the kernel sources

wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.17.6.tar.xz (CHANGE THE VERSION)

EG.
-> 4.17.1
-> 4.18.2

##### Kernel - Cross-compilingKernel

Go to the $HOME/embedded-linux-labs/kernel directory
