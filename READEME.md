# PetaLinux Demonstration for Avnet ZedBoard Platform
This repository implements a simple proof of concept project demonstrating the use of PetaLinux in conjunction with Vivado to deploy a Linux environment alongside programable logic on a heterogeneous System on a Chip (SoC). This demonstration is based heavily on the excellent [YouTube tutorial](https://www.youtube.com/watch?v=PwYV2lQhpno) by Waqar Rashid.

## Technical Requirements
This project utilizes Visual Studio Code's [Remote-Containers](https://code.visualstudio.com/docs/remote/containers) functionality to create an easily reproducible development environment. This requires that you have VSCode installed on your computer, along with the Remote-Containers extension. It also requires Docker as a container management tool. The repository was authored using the WSL2 backend on Microsoft Windows, but should be equally functional in a true linux (or MacOS) environment. 

At the moment, the author has not integrated the Vivado portion of the build process into the container. Accordingly, the user will need to complete the Vivado portion of the build process below using a Vivado installation on their local machine, before transitioning into the container to complete the PetaLinux portion of the process.

The author gratefully acknowledges github user z4yx, whose [petalinux-docker](https://github.com/z4yx/petalinux-docker) repo served as the basis of this container.

### An important note 
Please see the [README for the devcontainer](./.devcontainer/README.md) for important notes on this process - Visual Studio Code should automatically detect the devcontainer and handle it appropriately, but you need to copy the PetaLinux installer into the .devcontainer folder. Since PetaLinux, like most Xilinx products, is hidden behind licensing and export agreements, it can't be included in the repository. (It's also a large file, and it wouldn't make sense to put it here).

Finally, this project utilizes a [Shared State Cache](https://wiki.yoctoproject.org/wiki/Enable_sstate_cache) to speed up builds. Although this isn't meaningful if you're only building this one project one time and then never touching PetaLinux again, it will lead to tremendous time savings on subsequent builds. To maximize this, the Shared State Cache is constructed on your local machine and shared into the container. Accordingly, you will need to make a folder at $HOME/SStateCache to hold this cache. __Failing to do so will cause the container build to fail!__

## Building the project
1. Launch the Vivado TCL shell, and navigate to the hw_zed_gpio directory using the `cd` command
1. Execute the command `source hw_zed_gpio.tcl` to build the vivado project. This script will generate the project, run the implementation and synthesis process, generate the bitstream, and package the project into a XSA file, which is ingested by PetaLinux. Optionally, the `--notrace` argument can be passed to skip printing out the script to the terminal as it is executing.
1. Launch the container by opening VSCode and selecting "Reopen in container" from the popup in the lower right. This will spawn a new VSCode window and start the container build process. If this is the first time the container is being built, it can take quite a while (especially the PetaLinux install). once it completes, the container will launch and the folder contents will populate the "Explorer" pane on the left side of the vscode interface.
1. Open a new bash prompt within vscode by clicking "Terminal" in the lower pane (if it isn't already selected), and clicking the plus in the upper right corner. Execute the command `cd zedboard-petalinux-gpio` to enter the PetaLinux project directory.
1. Within the directory, execute `petalinux-build` to build out the image.
1. Execute `petalinux-package --boot --fsbl images/linux/zynq_fsbl.elf --fpga images/linux/system.bit --u-boot` to construct the boot image for the platform.
1. Format an SD card for the zedboard (if you don't already have one prepared), per the [Xilinx Instructions](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842385/How+to+format+SD+card+for+SD+boot)
1. Copy the image files (`BOOT.bin` and `Image.ub`) from the `images/linux` directory onto the SD card.
1. Ensure the ZedBoard is configured for SD boot, per the [ZedBoard Hardware User's Guide](https://digilent.com/reference/_media/zedboard:zedboard_ug.pdf). Specifically, see Table 18. Connect a USB cable to the UART port of the ZedBoard, and connect to it via the serial console of your choice. The author utilized [PuTTY](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjbgPbPp4_0AhXvz4UKHaSuBOMQFnoECAMQAQ&url=https%3A%2F%2Fwww.putty.org%2F&usg=AOvVaw0iOGrunharr0YuZtN9wsn1).
1. Insert the SD Card and power on the ZedBoard. The system should boot up, displaying output in the serial console.
1. Log in with the username/password combination root/root
1. execute the command gpio-demo, which will print the instructions for reading and writing to the various GPIO ports connected to the switches, buttons, and LEDs on the ZedBoard.