# Allied Vision Alvium CSI driver for Jetpack 6.2

## Compatibility
> [!IMPORTANT]
> ### GenICam for CSI2 access
> To use GenICam for CSI2 access a beta version of the CSI Transport Layer is required. It can be downloaded here: https://www.alliedvision.com/de/products/software/beta-early-adaptor/

### SoMs + Carrier Boards 
- Jetson AGX Orin DevKit
- Jetson Orin Nano DevKit
- Jetson Orin NX + Jetson Orin Nano DevKit carrier
### Cameras
- All Alvium C cameras with Firmware 13 or newer

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

# Beta Disclaimer

Please be aware that all code revisions not explicitly listed in the Github Release section are
considered a **Beta Version**.

For Beta Versions, the following applies in addition to the GPLv2 License:

THE SOFTWARE IS PRELIMINARY AND STILL IN TESTING AND VERIFICATION PHASE AND IS PROVIDED ON AN “AS
IS” AND “AS AVAILABLE” BASIS AND IS BELIEVED TO CONTAIN DEFECTS. THE PRIMARY PURPOSE OF THIS EARLY
ACCESS IS TO OBTAIN FEEDBACK ON PERFORMANCE AND THE IDENTIFICATION OF DEFECTS IN THE SOFTWARE,
HARDWARE AND DOCUMENTATION.
