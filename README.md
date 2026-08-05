# Allied Vision Alvium CSI driver for Jetpack 7.2

## Compatibility

### SoMs + Carrier Boards 
- **New** Jetson Thor T5000/T4000 + forecr DSBOARD-THRMAX carrier
- Jetson AGX Orin DevKit
- Jetson Orin Nano DevKit
- Jetson Orin NX + forecr DSBOARD-ORNX carrier
### Cameras
- All Alvium C cameras with Firmware 14 or newer

## Thor Adapter Board Information
> [!CAUTION]
> For using the Thor system the "Adapter Board for NVIDIA Jetson 
> AGX Orin/Xavier and TX2" needs to be modified. Otherwise the system won't boot
> with the adatper board attached. The required modification is to remove the two 
> resistors R13 and R14. Detailed design information about the adapter board
> can be found in the following Github repository: [adapter_Nvidia_Jetson_Xavier_TX2_VarOrin_DevKit](https://github.com/alliedvision/adapter_Nvidia_Jetson_Xavier_TX2_VarOrin_DevKit)


## Installation 
1. Download the debian package from the releases section to our target board
2. Install the packages by running:
    ```shell
    sudo apt install ./avt-nvidia-csi2-driver_<version>.deb
    ```
3. Configure the device tree
    - Orin
        1. Start the jetson-io tool
            ```shell
            sudo /opt/nvidia/jetson-io/jetson-io.py
            ```
        2. Select the CSI connector configuration
            - For AGX Orin: "Jetson AGX CSI Connector"
            - For Orin Nano / NX: "Jetson 22pin CSI Connector"
        3. Select "Configure for compatible hardware"
        4. Select the appropriate "* Alvium C Dual *" configuration
        5. Select "Save pin changes" 
        6. Select "Save and reboot to reconfigure pins"
    - Thor
        > **WARNING**
        > For the Thor system jetson-io is not supported and therefore the bootloader configuration must be adjusted manually, which can lead to an not booting system.


        1. Determine the name of the system device tree by running
            ```shell
            ls /boot/dtb/
            ```
        2. Open the bootloader configuration file "/boot/extlinux/extlinux.conf" as root using a text editor of our choice
        3. Create a copy of the "primary" configuration below the "primary" configuration. The relevant part of the configuration file should now look similar to this: 
            ```
            ...

            LABEL primary
                MENU LABEL primary kernel
                LINUX /boot/Image
                INITRD /boot/initrd
                APPEND ${cbootargs} root=PARTUUID=c13a4afc-3098-4389-971f-f5d1ee390c2f rw rootwait rootfstype=ext4 mminit_loglevel=4 earlycon=tegra_utc,mmio32,0xc5a0000 console=ttyUTC0,115200 firmware_class.path=/etc/firmware fbcon=map:0 efi=runtime audit=1 audit_backlog_limit=8192 swiotlb=2048 video=efifb:off console=tty0

            LABEL primary
                MENU LABEL primary kernel
                LINUX /boot/Image
                INITRD /boot/initrd
                APPEND ${cbootargs} root=PARTUUID=c13a4afc-3098-4389-971f-f5d1ee390c2f rw rootwait rootfstype=ext4 mminit_loglevel=4 earlycon=tegra_utc,mmio32,0xc5a0000 console=ttyUTC0,115200 firmware_class.path=/etc/firmware fbcon=map:0 efi=runtime audit=1 audit_backlog_limit=8192 swiotlb=2048 video=efifb:off console=tty0
            ```
        4. Rename the copied configuration to "AVT_CSI2" by changing the LABEL
            ```
            ...

            LABEL AVT_CSI2

            ...
            ```
        5. Add an FDT entry below INITRD with the system device tree file. The name must be changing according to the information from step 1.
            ```
            ...

            LINUX /boot/Image
            INITRD /boot/initrd
            FDT /boot/dtb/kernel_tegra264-p4071-0000+p3834-0000-nv.dtb
            
            ...
            ```
        6. Enable the device tree overlay by adding the OVERLAYS entry below FDT.
            ```
            ...
            
            FDT /boot/dtb/kernel_tegra264-p4071-0000+p3834-0000-nv.dtb
            OVERLAYS /boot/tegra264-p3834-camera-forecr-thrmax-dual-alvium-19616-2x4.dtbo
            
            ...
            ```
        7. As last step set the "AVT_CSI2" configuration as default.
            ```
            TIMEOUT 30
            DEFAULT AVT_CSI2
            
            ...
            ```
4. After the board has rebooted the camera can be accessed with V4L2 and Vimba X

## Building
1. Clone this repository including all submodules
2. Download the Jetson Linux driver package and cross compiler from: [Jetson Linux Downloads](https://developer.nvidia.com/embedded/jetson-linux)
3. Extract the driver package: 
    ```shell
        tar -xf jetson_linux_r36*.tbz2
    ```
4. Extract the kernel headers from the driver package:
    ```shell
        cd Linux_for_Tegra/kernel/
        tar -xf kernel_headers.tbz2
    ```
5. Extract the cross compiler
6. Build the modules:
    ```shell
        export ARCH=arm64
        export CROSS_COMPILE=<path to cross compiler>/aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-
        export KERNEL_SRC=Linux_for_Tegra/kernel/linux-headers-*-linux_x86_64/3rdparty/canonical/linux-noble
        make all 
    ```
7. Install the driver modules
    ```shell
        export INSTALL_MOD_PATH=<path to install directory>
        make install
    ```

## Known limitations

- When using external triggers the NVIDIA v4l2 control  ```override_capture_timeout_ms``` has to be set a suitable timeout value or -1 for a infinite timeout. Otherwise incomplete buffers with the error flag set might be returned due to a timeout while waiting for the image. 

## NVIDIA accelerated gstreamer examples

### nvv4l2camerasrc
To use the nvv4l2camerasrc gstreamer element with an Alvium camera the driver must be configured correctly.

The pixelformat must be set to UYVY:
```shell
v4l2-ctl -v pixelformat=UYVY
```
The bytesperline value must be a multiple of 256:
1. Read bytesperline: ```v4l2-ctl -v```
2. Calculate algined value: ```aligned_stride = ceil(bytesperline/256)```
3. Set preferred_stride v4l2 ctrl: ```v4l2-ctl –c preferred_stride=<aligned_stride>```

Example pipeline:
```
gst-launch-1.0 nvv4l2camerasrc ! 'video/x-raw(memory:NVMM), width=<width>, height=<height>' ! nvvidconv ! nveglglessink
```

### vmbsrc
It is also possible to connect the vmbstrc with the NVIDIA accelerated gstreamer elements by using the nvvidconv elements to transform the normal into NVMM image buffers.
Example pipeline:
```
gst-launch-1.0 vmbsrc camera=DEV_00012C00D323 ! 'video/x-raw,format=UYVY,' ! nvvidconv ! nveglglessink
```

# Beta Disclaimer

Please be aware that all code revisions not explicitly listed in the Github Release section are
considered a **Beta Version**.

For Beta Versions, the following applies in addition to the BSD 3-Clause License:

THE SOFTWARE IS PRELIMINARY AND STILL IN TESTING AND VERIFICATION PHASE AND IS PROVIDED ON AN “AS
IS” AND “AS AVAILABLE” BASIS AND IS BELIEVED TO CONTAIN DEFECTS. THE PRIMARY PURPOSE OF THIS EARLY
ACCESS IS TO OBTAIN FEEDBACK ON PERFORMANCE AND THE IDENTIFICATION OF DEFECTS IN THE SOFTWARE,
HARDWARE AND DOCUMENTATION.
