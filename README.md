# zedboard
Information on Agilent Zedboard and tutorials on using Vivado and writing firmware

This documents explains how to build an operating system and how to create firmware / software for this board.

We will use Vivado 2024.2 and PetaLinux 2024.2 for this.
This folder contains a lot of material, however some is quite outdated, as the board was create in 2012/2014.

We are going to ultimately use PetaLinux 2024.2, but for now will stick with 2021.2, as the [tutorial](https://spacewire.co.uk/tutorial/setup_vitis_vivado_petalinux/) by Spacewire is complete and working.

## Try out board

To make sure the board is fine, create SD card (>=1Gb), and place the original contents on this.

1. Format the partion of the SD card as FAT32 (if there are more partitions, just the first one is important).

2. Unzip the file `zedboard_oob_design.zip`, and copy the contents of the folder `ZedBoard_OOB_Design\sd_image` to the SD card.

3. Eject the SD card correctly to make sure the contents are not corrupted.

4. Connect the UART port to a PC using the mini USB to USB cable.
5. Make sure the board is set to boot from SD:
- JP2 is shortcut
- JP3 is shortcut
- JP7 (MI02) is in the low position
- JP8 (MI03) is in the low position
- JP9 (MI04) is in the high position
- JP10 (MI05) is in the high position
- JP11 (MI06) is in the low position

6. Power on the board, and make sure the USB serial port is recognized. On Windows 11, it is quite tricky to find the right driver as the USB chip is quite old. On Linux, the USB port should be recognized. The driver that should be found for Windows 11 is shown below.
7. Install a terminal application, and attach it to the serial device at 115200 N81.

<img  src="images/cypress-driver.png" alt="Cypress-USB-to-serial-driver" width="400"/>

8. Power off the board again.
9. Place the SD card in the board.
10. Open the terminal application.
11. Power on the board and wait some time until the blue led lights up.
12. After this you should see Linux boot on the console.
13. Finally after booting you should see the display show the Digilent logo.

## Install Vivado

Go to the [AMD / Xilinx site](https://www.xilinx.com/support/download.html), and log in (or create a new account).
Select 2024.2, and download [Vivado™ Edition - 2024.2  Full Product Installation](https://www.xilinx.com/member/forms/download/xef.html?filename=FPGAs_AdaptiveSoCs_Unified_2024.2_1113_1001_Win64.exe) for Windows.

:bangbang: **Important note:**
Parts of Vitis / Vivado may not install correctly if the virus scanner is active. This may lead to confusing errors such as CMake not being able to configure. Consider temporarily switch off the virusscanner while installing.

Follow the installation guide. If desired, you can also install DocNav and Vitis.

## Install PetaLinux

For reference, use the following web page on PetaLinux: [UG1144](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Generate-MCS-Image?tocId=eyVYonb3Ea_anF_qmLeH~A)

Go to the [AMD / Xilinx site](https://www.xilinx.com/support/download.html), and log in (or create a new account).
Select 2024.2, and download [PetaLinux Tools - Installer](https://www.xilinx.com/member/forms/download/xef.html?filename=petalinux-v2024.2-11062026-installer.run).

You will either need a PC running Ubuntu 20.04 or 22.04, or you can use WSL 2 on Windows (10/11).

:bangbang: **Important note:**
The WSL build sometimes fails for unclear reasons. Often a rebuild fixes this. For a reproducible build, consider using a real Ubuntu installation, either on a physical machine or a VM.

### Using WSL

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

### Installation

Make sure the following packages are installed:

```
sudo apt install tofrodos iproute2 gawk xvfb git make net-tools libncurses5-dev tftpd zlib1g-dev:i386 libssl-dev flex bison libselinux1 gnupg wget diffstat chrpath socat xterm autoconf libtool tar unzip texinfo zlib1g-dev gcc-multilib build-essential libsdl1.2-dev libglib2.0-dev screen pax gzip libtinfo5
 ```

Create a directory for PetaLinux files:
```
mkdir petalinux
```

Create a directory for petalinux installation:
```
sudo mkdir /opt/petalinux
sudo chown <user>:<user> /opt/petalinux
```

For '\<user>' fill in your username (twice). This will create a directory with write permissions for you.

Copy `plnx-env-setup.sh` to the build system.

Install required packages for PetaLinux:

```bash
cd ~/petalinux
chmod +x plnx-env-setup.sh
./plnx-env-setup.sh
```

Install PetaLinux:
```bash
cd ~/petalinux
chmod +x petalinux-v2024.2-11062026-installer.run
./petalinux-v2024.2-11062026-installer.run --log petalinux-v2024.2-install.log --dir /opt/petalinux/2024.2
```

When requested to agree to licenses press Enter, then you will be asked twice to agree.
Each time, press q to get out of the viewer, and then y and Enter to agree.
PetaLinux will then install itself.

## Create a hardware platform for zedboard

In order to create a PetaLinux project, we need to create a hardware platform first.
This will be stored in a .xsa file.

Start Vivado, and create a new project.

<img  src="images/Vivado-start.png" alt="Vivado after startup" width="600"/>

Select `Create Project`

<img  src="images/Vivado-create-project.png" alt="Vivado create project" width="600"/>

Fill in the project location and name. Select `Next`.

<img  src="images/Vivado-create-project-type.png" alt="Vivado project type" width="600"/>

Leave the settings unchanged, select `Next`.

<img  src="images/Vivado-create-project-zedboard.png" alt="Vivado create project" width="600"/>

Select the tab `Boards` and in the search box type `zedboard`. Select the correct board revision (Rev. D in my case). Select `Next`.

<img  src="images/Vivado-create-project-zedboard.png" alt="Vivado select zedboard" width="600"/>

Select `Finish`.

<img  src="images/Vivado-main.png" alt="Vivado main window" width="600"/>

From the `Flow Navigator` select `Create Block Design`.

<img  src="images/Vivado-create-block-design.png" alt="Vivado create block design" width="300"/>

Leave the name unchanged, and select `OK`.

<img  src="images/Vivado-block-design-empty.png" alt="Vivado empty block design" width="1000"/>

In the `Diagram` panel, click `+` to add a component.

<img  src="images/Vivado-block-design-select-zynq.png" alt="Vivado empty block design" width="300"/>

In the search box, type `zynq`, select `Zynq Processing System`, and press Enter.

<img  src="images/Vivado-block-design-zynq.png" alt="Vivado block design Zynq" width="1000"/>

In the green bar at the top, select `Run Block Automation`.

<img  src="images/Vivado-block-design-zynq-block-automation.png" alt="Vivado Zynq block automation" width="1000"/>

Leave `Apply Board Preset` selected. This will configure the specifics for the selected board. Select `OK`.

<img  src="images/Vivado-block-design-zynq-standard-ports.png" alt="Vivado block design Zynq after automation" width="1000"/>

Select the `Board` tab in the `Block Design` panel (top left).

<img  src="images/Vivado-block-design-board-devices.png" alt="Vivado ZedBoard devices" width="500"/>

One after another, double-click the `DIP switches`, `LED` and `Push buttons` devices. For each select `OK` to add it.

<img  src="images/Vivado-block-design-zynq-dip-switches.png" alt="Vivado ZedBoard add DIP switches" width="400"/>

<img  src="images/Vivado-block-design-zynq-leds.png" alt="Vivado ZedBoard add LEDs" width="400"/>

<img  src="images/Vivado-block-design-zynq-push-buttons.png" alt="Vivado ZedBoard add push buttons" width="400"/>

<img  src="images/Vivado-block-design-zynq-with-devices.png" alt="Vivado empty block design" width="1000"/>

Again, in the green bar at the top, select `Run Block Automation`.

<img  src="images/Vivado-block-design-zynq-block-automation-dialog.png" alt="Vivado Zynq block automation dialog" width="700"/>

Select `All Automation`. Select `OK`.

<img  src="images/Vivado-block-design-zynq-block-automation2.png" alt="Vivado Zynq block automation after adding devices" width="1000"/>

You can drag the components around a little to make the block design look more neat.

<img  src="images/Vivado-block-design-reorganized.png" alt="Vivado Zynq block automation after reorganizing" width="1000"/>

Validate the design. Select the `Design` tab, right click on the top level design `design_1` and select `Validate Design`.

<img  src="images/Vivado-block-design-validated.png" alt="Vivado Zynq block automation after adding devices" width="400"/>

Add an HDL wrapper source to the design. In the `Sources` tab under `BLOCK DESIGN`, right lock the yellow block design entry, and select `Create HDL Wrapper...`

<img  src="images/Vivado-block-design-create-hdl-wrapper.png" alt="Vivado create HDL wrapper" width="400"/>

Leave the option `Let Vivado manger wrapper and auto-update` selected, select `OK`.

Run the linker. In the `Flow Navigator` panel under `RTL ANALYSIS` select `Run Linter`.

<img  src="images/Vivado-linter-progress.png" alt="Vivado run linter" width="400"/>

In the `Flow Navigator` panel, run synthesis, implementation and bitstream generation. Select `Generate Bitstream` under `PRGRAM AND DEBUG`. This will automaticall run the other steps if needed.

<img  src="images/Vivado-save-project.png" alt="Vivado save project" width="400"/>

Vivado may ask to save the project. If so, obviously select `Save`.

<img  src="images/Vivado-run-steps.png" alt="Vivado run steps dialog" width="400"/>

Select `OK` to start the run. The `Log` tab at the bottom will show progress and any warnings or errors.

<img  src="images/Vivado-steps-progress.png" alt="Vivado steps progress" width="400"/>

Once the bitstream is generated. A dialog will pop up.

<img  src="images/Vivado-bitstream-complete.png" alt="Vivado bitstream complete" width="400"/>

Select `Cancel` to finish.

Now export the hardware platform. In the top level menu select `File->Export->Export Hardware...`

<img  src="images/Vivado-export-hardware-platform.png" alt="Vivado export hardware platform" width="600"/>

Select `Next`.

<img  src="images/Vivado-export-hardware-platform-output.png" alt="Vivado export hardware platform output" width="600"/>

Do not include the bitstream, that can be exported separately later. Select `Next`.

<img  src="images/Vivado-export-hardware-platform-files.png" alt="Vivado export hardware platform files" width="600"/>

Give the output file a convenient name, and optionally select a desired directory. Select `Next`.

<img  src="images/Vivado-export-hardware-platform-finish.png" alt="Vivado export hardware platform finish" width="600"/>

Select `Finish`.

Now look for the .xsa file. In the case shown, there will be a file `zedboard.xsa` in the directory `H:/Projects/zedboard`.

Copy this file to the PetaLinux build environment.

## Prepare PetaLinux build

Go to the [AMD / Xilinx site](https://www.xilinx.com/support/download.html), and log in (or create a new account).
Select 2024.2, and download

[sstate_arm_2024.2_11061705.tar.gz](https://www.xilinx.com/member/forms/download/xef.html?filename=sstate_arm_2024.2_11061705.tar.gz)
[downloads_2024.2_11061705.tar.gz](https://www.xilinx.com/member/forms/download/xef.html?filename=downloads_2024.2_11061705.tar.gz)

And place the files in `/opt/xilinx/2024.2/downloads`. An easy way to do this is using WinSCP.

Unpack the sstate cache:

```bash
rm -rf /opt/petalinux/2024.2/sstate-cache
mkdir -p /opt/petalinux/2024.2/sstate-cache
tar xzf /opt/xilinx/2024.2/downloads/sstate_arm_2024.2_11061705.tar.gz -C /opt/petalinux/2024.2/sstate-cache/
```

Unpack the download mirror:

```bash
rm -rf /opt/petalinux/2024.2/downloads
mkdir -p /opt/petalinux/2024.2/downloads
tar xzf /opt/xilinx/2024.2/downloads/downloads_2024.2_11061705.tar.gz -C /opt/petalinux/2024.2/
```

Install a TFTP server

```bash
sudo apt install tftpd-hpa tftp-hpa
```

Set up a TFTP server and directory

```bash
sudo mkdir /tftpboot
sudo chown -R nobody:nogroup /tftpboot
chmod -R 777 /tftpboot
```

Configure TFTP

```bash
sudo nano /etc/default/tftpd-hpa
```

Edit the contents:
```text
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure"
```

```bash
sudo systemctl restart tftpd-hpa
sudo systemctl enable tftpd-hpa
```

## Create a PetaLinux project for zedboard

Set up the PetaLinux environment (always needed when starting to work with PetaLinux after login):

```bash
mkdir ~/petalinux
cd ~/petalinux
source /opt/petalinux/2024.2/settings.sh
```

```
*************************************************************************************************************************************************
The PetaLinux source code and images provided/generated are for demonstration purposes only.
Please refer to https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/2741928025/Moving+from+PetaLinux+to+Production+Deployment
 for more details
*************************************************************************************************************************************************
PetaLinux environment set to '/opt/petalinux/2024.2'
WARNING: /bin/sh is not bash! 
bash is PetaLinux recommended shell. Please set your default shell to bash.
[WARNING] This is not a supported OS
[INFO] Checking free disk space
[INFO] Checking installed tools
[INFO] Checking installed development libraries
[INFO] Checking network and other services
```

Create a project for zedboard:

```bash
petalinux-create project --template zynq --name zedboard
```

```
[INFO] Create project: zedboard
[INFO] New project successfully created in /home/rene/petalinux/zedboard
```

Now add the hardware platform to the project:

```bash
cd zedboard
petalinux-config --get-hw-description ../zedboard.xsa
```

```
[INFO] Getting hardware description
[INFO] Renaming zedboard.xsa to system.xsa
[INFO] Extracting yocto SDK to components/yocto. This may take time!
[INFO] Bitbake is not available, some functionality may be reduced.
[INFO] Using HW file: /home/rene/petalinux/zedboard/project-spec/hw-description/system.xsa
[INFO] Getting Platform info from HW file
```

The configuration menu will be shown. Change the following settings:

- DTG Settings  --->
  - MACHINE_NAME
    - zedboard
- FPGA Manager  --->
  - Select Fpga Manager
- Yocto Setting  --->
- - Add pre-mirror url  --->
    - file:///opt/petalinux/2024.2/downloads
  - Local sstate feeds settings  --->
    - file:///opt/petalinux/2024.2/sstate-cache/arm

```
[INFO] Generating Kconfig for project
[INFO] Menuconfig project
[INFO] Generating kconfig for rootfs
[INFO] Silentconfig rootfs
[INFO] Generating configuration files
[INFO] Adding user layers
[INFO] Generating machine conf file
[INFO] Generating plnxtool conf file
[INFO] Generating workspace directory
NOTE: Starting bitbake server...
NOTE: Started PRServer with DBfile: /home/rene/petalinux/zedboard/build/cache/prserv.sqlite3, Address: 127.0.0.1:42201, PID: 19703
INFO: Specified workspace already set up, leaving as-is
INFO: Enabling workspace layer in bblayers.conf
[INFO] Successfully configured project
```

## Build the petalinux project

Build the project:

```bash
petalinux-build
```

```
[INFO] Building project
[INFO] Bitbake is not available, some functionality may be reduced.
[INFO] Using HW file: /home/rene/petalinux/zedboard/project-spec/hw-description/system.xsa
[INFO] Getting Platform info from HW file
[INFO] Silentconfig project
[INFO] Silentconfig rootfs
[INFO] Generating configuration files
[INFO] Generating workspace directory
NOTE: Starting bitbake server...
NOTE: Started PRServer with DBfile: /home/rene/petalinux/zedboard/build/cache/prserv.sqlite3, Address: 127.0.0.1:42129, PID: 256989
INFO: Specified workspace already set up, leaving as-is
[INFO] bitbake petalinux-image-minimal
NOTE: Started PRServer with DBfile: /home/rene/petalinux/zedboard/build/cache/prserv.sqlite3, Address: 127.0.0.1:40045, PID: 257056
WARNING: XSCT has been deprecated. It will still be available for several releases. In the future, it's recommended to start new projects with SDT workflow.
Loading cache: 100% |                                                                                                                                                                                                                                         | ETA:  --:--:--
Loaded 0 entries from dependency cache.
Parsing recipes: 100% |########################################################################################################################################################################################################################################| Time: 0:00:45
Parsing of 5800 .bb files complete (0 cached, 5800 parsed). 8454 targets, 1106 skipped, 27 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
Checking sstate mirror object availability: 100% |#############################################################################################################################################################################################################| Time: 0:00:06
Sstate summary: Wanted 536 Local 10 Mirrors 467 Missed 59 Current 2177 (88% match, 97% complete)
Removing 52 stale sstate objects for arch zynq_generic_7z020: 100% |###########################################################################################################################################################################################| Time: 0:00:00
NOTE: Executing Tasks
NOTE: Tasks Summary: Attempted 5988 tasks of which 5852 didn't need to be rerun and all succeeded.

Summary: There was 1 WARNING message.
[INFO] Successfully copied built images to tftp dir: /tftpboot
[INFO] Successfully built project
```

Build the SDK:

```bash
petalinux-build --sdk
```

```
```

## Booting

```
setenv dtb_addr 0x01000000
setenv kernel_addr 0x02000000
setenv serverip 192.168.1.245
tftpboot ${dtb_addr} ${serverip}:system.dtb
tftpboot ${kernel_addr} ${serverip}:image.ub
bootm ${kernel_addr} - ${dtb_addr}
```


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