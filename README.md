# Allied Vision Alvium CSI driver for Jetpack 6.2.1

## Compatibility

### SoMs + Carrier Boards 
- Jetson AGX Orin DevKit
- Jetson Orin Nano DevKit
- Jetson Orin NX + forecr DSBOARD-ORNX carrier
### Cameras
- All Alvium C cameras with Firmware 14 or newer

## Installation 
1. Download the debian package from the releases section to our target board
2. Install the packages by running:
    ```shell
    sudo apt install ./avt-nvidia-csi2-driver_<version>.deb
    ```
3. Configure the device tree
    1. Start the jetson-io tool
        ```shell
        sudo /opt/nvidia/jetson-io/jetson-io.py
        ```
    2. Select the CSI connector configuration
        - For AGX Orin: "Jetson AGX CSI Connector"
        - For Orin Nano / NX: "Jetson 24pin CSI Connector"
    3. Select "Configure for compatible hardware"
    4. Select the appropriate "* Alvium C Dual *" configuration
    5. Select "Save pin changes" 
    6. Select "Save and reboot to reconfigure pins"
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
        export CROSS_COMPILE=<path to cross compiler>/bin/aarch64-buildroot-linux-gnu-
        export KERNEL_SRC=Linux_for_Tegra/kernel/linux-headers-*-linux_x86_64/3rdparty/canonical/linux-jammy/kernel-source/
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
It is also possible to connect the vmbstrc with the NVIDIA accelerated gstreamer elements by using the nvvidconv elements. To supports transforms the normal into NVMM image buffers.
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
