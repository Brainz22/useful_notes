## Connecting to a Jupyter Notebook on the PYNQ Board

Here is the link for the [PYNQ-Z2 Setup Guide](https://pynq.readthedocs.io/en/latest/getting_started/pynq_z2_setup.html).

1. Connect the pink board via microUSB and Ethernet cable to computer. Wait for yellow/green `done` light. Make sure **Power** jumper is set to USB since we are powering the board via USB.

2. On Linux, navigate to `Settings` and find `Network`. There, you have to choose `manual` under `IPv4`. Then, assign a static IP address and network address under `Adresses`. The IP address is `199.168.2.1` and the network address `255.255.255.0` as specified by the setup guide.

3. You can now open a jupyter notebook by navigating to [http://192.168.2.99](http://192.168.2.99/). If a password is asked, it should be `xilinx`.
