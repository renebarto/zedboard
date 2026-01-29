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

<img  src="images/Vivado-block-design-zynq-standard-ports.png" alt="Vivado block design Zynq after automation" width="500"/>

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

## Create a PetaLinux project for zedboard

Set up the PetaLinux environment (always needed when starting to work with PetaLinux after login):

```bash
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
[WARNING] This is not a supported OS
[INFO] Checking free disk space
[INFO] Checking installed tools
[INFO] Checking installed development libraries
[INFO] Checking network and other services
[WARNING] No tftp server found - please refer to "UG1144 2024.2 PetaLinux Tools Documentation Reference Guide" for its impact and solution
```

Create a project for zedboard:

```bash
petalinux-create project --template zynq --name zedboard
```

```
[INFO] Create project: zedboard
[INFO] New project successfully created in /home/rene/petalinux/zedboard
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