# Xilinx Software Installation and UCSD License Server

Credits to Ryan Kastner. Link to original: https://kastner.ucsd.edu/ryan/vivado-installation/.

Steps for installing and running the Xilinx software on your own machine.

### Xilinx Software Installation

1. Go to: http://www.xilinx.com/support/download/.
2. Download Vitis Core Development Kit using the Xilinx Unified Installer.
3. You will need to make a username and password at the Xilinx website.
4. Both Linux or Windows versions are available.
5. Make certain to note support Linux versions. Other versions may not provide full functionality.

### License Settings 

1. Open the program “Manage Xilinx Licenses.”
2. Open Vivado 2019.1 -> Manage Xilinx License.
3. Open Manage Xilinx License Search Paths.
4. Set the XILINXD_LICENSE_FILE to 2100@cselm2.ucsd.edu.
5. The license will only work if you are using the UCSD network. One can use VPN off-campus to access the license server.

### Notes

1. Installation takes a long time and requires a lot of free space.
2. I had some issues when installing in Ubuntu with the last step not finishing (very frustrating). It was very likely due to a library issue, so I would make sure that is installed.
3. Some visualization features do not seem to work in Windows 10 (Control Flow, Function Call Graph, etc.). A smaller subset of those features did not work on Ubuntu 21.04 (which is not Xilinx supported). I got them all to work on Ubuntu 20.04 (which is Xilinx supported).
