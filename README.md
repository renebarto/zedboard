# zedboard
Information on Agilent Zedboard and tutorials on using Vivado and writing firmware

This documents explains how to build an operating system and how to create firmware / software for this board.

We will use Vivado 2024.2 and PetaLinux 2024.2 for this.
This folder contains a lot of material, however some is quite outdated, as the board was create in 2012/2014.

## Install Vivado

Go to the [AMD / Xilinx site](https://www.xilinx.com/support/download.html), and log in (or create a new account).
Select 2024.2, and download [Vivado™ Edition - 2024.2  Full Product Installation](https://www.xilinx.com/member/forms/download/xef.html?filename=FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001_Win64.exe) for Windows.

Follow the installation guide. If desired, you can also install DocNav and Vitis.

## Install PetaLinux

For reference, use the following web page on PetaLinux: [UG1144](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Generate-MCS-Image?tocId=eyVYonb3Ea_anF_qmLeH~A)

Go to the [AMD / Xilinx site](https://www.xilinx.com/support/download.html), and log in (or create a new account).
Select 2024.2, and download [PetaLinux Tools - Installer](https://www.xilinx.com/member/forms/download/xef.html?filename=petalinux-v2024.2-11062026-installer.run).

You will either need a PC running Ubuntu 20.04 or 22.04, or you can use WSL 2 on Windows (10/11).

For WSL:

Open a Powershell prompt as administrator. Get a list of distributions available:

```cmd
wsl --list --online
```

```text
NAME                            FRIENDLY NAME
AlmaLinux-8                     AlmaLinux OS 8
AlmaLinux-9                     AlmaLinux OS 9
AlmaLinux-Kitten-10             AlmaLinux OS Kitten 10
AlmaLinux-10                    AlmaLinux OS 10
Debian                          Debian GNU/Linux
FedoraLinux-42                  Fedora Linux 42
SUSE-Linux-Enterprise-15-SP6    SUSE Linux Enterprise 15 SP6
SUSE-Linux-Enterprise-15-SP7    SUSE Linux Enterprise 15 SP7
Ubuntu                          Ubuntu
Ubuntu-24.04                    Ubuntu 24.04 LTS
archlinux                       Arch Linux
kali-linux                      Kali Linux Rolling
openSUSE-Tumbleweed             openSUSE Tumbleweed
openSUSE-Leap-16.0              openSUSE Leap 16.0
Ubuntu-20.04                    Ubuntu 20.04 LTS
Ubuntu-22.04                    Ubuntu 22.04 LTS
OracleLinux_7_9                 Oracle Linux 7.9
OracleLinux_8_10                Oracle Linux 8.10
OracleLinux_9_5                 Oracle Linux 9.5
openSUSE-Leap-15.6              openSUSE Leap 15.6
```

Select the Ubuntu 22.04 version name `Ubuntu-22.04`. Install Ubuntu:

```cmd
wsl --install Ubuntu-22.04
```

To run Ubuntu in WSL2:

```cmd
wsl --distribution Ubuntu-22.04
```

The first time it is run it will ask you for an account, which is automatically logged in next time. This is not a root acount, so you will need sudo in certain cases.

Create a directory for PetaLinux file:
```
mkdir petalinux
```

Create a directory for petalinux installation:
```
sudo mkdir /opt/petalinux
sudo chown <user>:<user> /opt/petalinux
```

For '\<user>' fill in your username (twice). This will create a directory with write permissions for you.

Install required packages for PetaLinux:

```bash
cd ~/petalinux
chmod +x plnx-env-setup.sh
./plnx-env-setup.sh
```

Install PetaLinux:
```
cd ~/petalinux
chmod +x petalinux-v2024.2-11062026-installer.run
./petalinux-v2024.2-11062026-installer.run --dir /opt/petalinux/2024.2
```

When requested to agree to licenses press Enter, then you will be asked twice to agree.
Each time, press q to get out of the viewer, and then y and Enter to agree.
PetaLinux will then install itself.

```
source /opt/petalinux/2024.2/settings.sh
    2  petalinux-package boot --u-boot
    3  cd
    4  cd petalinux/zedboard/
    5  petalinux-package boot --u-boot
    6  ls images/
    7  ls images/linux/
    8  ls -al images/linux/
    9  tar -tvf images/linux/rootfs.tar.gz
   10  ls /dev
   11  ls -al images/linux/
   12  petalinux-package boot --u-boot --fpga
   13  petalinux-package boot --u-boot --fpga --force
   14  ls -al images/linux/
   15  ip addr
   16  petalinux-package boot --u-boot --fpga --fsbl --force
   17  ls
   18  cd project-spec/
   19  ls
   20  cd configs/
   21  ls
   22  grep -i boot.scr *
   23  grep -ir boot.scr *
   24  cat config
   25  nano config
   26  petalinux-build
   27  cd ../../images/linux/
   28  ls -al
   29  tar tvf rootfs.tar.gz | grep boot
   30  tar tvf rootfs.tar.gz | grep system.bit
   31  tar tvf rootfs.tar.gz | grep fpgamanager
   32  cd ../../project-spec/configs/
   33  grep -ir fpgamanager *
   34  grep -ir fpga *
   35  petalinux-config
   36  cd ../../images/linux/
   37  petalinux-build
   38  cd ../..
   39  find . -name fpgamanager
   40  petalinux-config
   41  find . -name fpga_manager
   42  find . -name fpga-manager
   43  find . -name `fpga*`
   44  find . -name 'fpga*'
   45  find . -name 'fpga-manager*'
   46  cat
   47  cat ./components/yocto/layers/meta-xilinx/meta-xilinx-core/recipes-bsp/fpga-manager-script/fpga-manager-script_1.0.bb
   48  grep -ir system.bit *
   49  find . -name 'system.bit'
   50  cd images/linux/
   51  tar tvf rootfs.tar.gz | grep system
   52  petalinux-package --force --boot --fsbl ~/petalinux/basic-os/images/linux/zynq_fsbl.elf --fpga ~/petalinux/basic-os/images/linux/system.bit --u-boot
   53  petalinux-package --force --boot --fsbl ~/petalinux/zedboard/images/linux/zynq_fsbl.elf --fpga ~/petalinux/zedboard/images/linux/system.bit --u-boot
   54  nano system.bif
   55  bootgen -image system.bif -arch zynq -process_bitstream bin
   56  ls -al
   57  nano ../../startSystem.sh
   58  cd /sys/class/gpio
   59  ls
   ```