# Getting Started with PYNQ-Z2

Pynq is the Pythonic interface and infrastructure developed by Xilinx to interact with the Zynq (PYNQ board) System on Chip (SoC) or FPGA board. Several other FPGA boards support this feature and the number will grow, given the increasing numbers of Python programmers. For this tutorial, we focus on working with the PYNQ board.
Here is the link for the [PYNQ-Z2 Setup Guide](https://pynq.readthedocs.io/en/latest/getting_started/pynq_z2_setup.html). A lot of information is used from this website, and they have a video!

### Prequisites

* PYNQ-Z2 Board, normally sold by AMD Xilinx.
* Linux or Windows (easiest) OS Computer with compatible browser (Chrome or Firefox preferred). Also, an ethernet port is required, unless you can get an adapter for USB or USB-C.
* Micro-USB cable. One end (micro) connects to power up the board and the USB end connects to your computer.
* Micro-SD card with preloaded image, or blank card (Minimum 8GB recommended). If blank, an appropriate needs to be downloaded store here. You can use this [link](http://www.pynq.io/board.html).
* Vivado and/or Vitis desig and development tools.

### Tutorial

1. Download the **AMD Unified Installer** for your respective OS using the link [here](https://www.xilinx.com/support/download.html).
We need Vitis HLS for C synthesis and generating the RTL, and Vivado for FPGA prototyping and generating the bitstream. Vitis is primarily for firmware design, thus not required. However, the unified installer allows you to install both tools.

2. 
  
## Connecting to a Jupyter Notebook on the PYNQ Board

Here is the link for the [PYNQ-Z2 Setup Guide](https://pynq.readthedocs.io/en/latest/getting_started/pynq_z2_setup.html).

1. Connect the pink board via microUSB and Ethernet cable to computer. Wait for yellow/green `done` light. Make sure **Power** jumper is set to USB since we are powering the board via USB.

2. On Linux, navigate to `Settings` and find `Network`. There, you have to choose `manual` under `IPv4`. Then, assign a static IP address and network address under `Adresses`. The IP address is `199.168.2.1` and the network address `255.255.255.0` as specified by the setup guide.

3. You can now open a jupyter notebook by navigating to [http://192.168.2.99](http://192.168.2.99/). If a password is asked, it should be `xilinx`.
