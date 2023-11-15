# Getting Started with PYNQ-Z2

Pynq is the Pythonic interface and infrastructure developed by Xilinx to interact with the Zynq (PYNQ board) System on Chip (SoC) or FPGA board. Several other FPGA boards support this feature and the number will grow, given the increasing numbers of Python programmers. For this tutorial, we focus on working with the PYNQ board.
Here is the link for the [PYNQ-Z2 Setup Guide](https://pynq.readthedocs.io/en/latest/getting_started/pynq_z2_setup.html). A lot of information is used from this website, and they have a video!

### Prequisites

* PYNQ-Z2 Board, normally sold by AMD Xilinx.
* Linux or Windows (easiest) OS Computer with compatible browser (Chrome or Firefox preferred). Also, an ethernet port is required, unless you can get an adapter for USB or USB-C.
* Micro-USB cable. One end (micro) connects to power up the board and the USB end connects to your computer.
* Micro-SD card with preloaded image, or blank card (Minimum 8GB recommended). If blank, an appropriate needs to be downloaded store here. You can use this [link](http://www.pynq.io/board.html).
* Vivado and/or Vitis desig and development tools.

## Vitis HLS (or Vivado HLS)
### Tutorial

1. Download the **AMD Unified Installer** for your respective OS using the link [here](https://www.xilinx.com/support/download.html).
We need Vitis HLS for C synthesis and generating the RTL, and Vivado for FPGA prototyping and generating the bitstream. Vitis is primarily for firmware design, thus not required. However, the unified installer allows you to install both tools. When you open the installer:
  *  `Select Vitis or Vivado`. Vitis is the newest software, but I think Vivado has more support at the moment.
  *  Select `Standard edition`.
  *  If you want to save disk space, you do not need to install simulation dependencies for all supported devices. You can just do `SoCs.`
  *  After hitting `next` for all previous steps, installation should begin.

2. Now, you should have a Vitis and a Vivado app on that you can just click on your computer. Alteratively, you can type `vitis_hls` on your terminal and it should open the GUI from there, too. 

3. Let's write a C/C++ to RTL application using Vitis. Open `Vitis HLS` by clicking on the app or typing `vitis_hls` on the terminal. Then:
   * Hit `create project`, and name it `pynq_mul`.
   * Do not add any files, hit `next` to part selection.
   * Select part `xc7z020clg400-1`.
  
4. Now that you are in, right click on `Source`, select `New file`, and create `mul_test.cpp`.

5. Complete the body of `mul_test.cpp` with the following code block: 
```c++

//this program creates a quick f(x)=2x multiplier.
void mul_test(int* out, int in){
      *out = 2*in;
}
```
6. Create a test bench file by right clicking on `Test Bench` in the `Explorer` and `create a new file` named `mul_tb.cpp`. Complete the body of this file as following:
```c++
#include <iostream>

using namespace std;

void mul_test(int* out, int in);

int main(int argc, char *argv[]){
      int x=5;
      int y;
      mul_test(&y, x);
      if(y!=2*x){
              cout << "Test Failed: output(" << y << ") is not equal to 2x" << x << endl;
      }else{
              cout << "Test Passed" << endl;
      }
      return 0;
}
```
7. Test your code by running C simulation. You should find a button maybe on `Flow navigator`. Make sure it passes the test bench.

8. We have to set up ports for variables and we can use the GUI for that. However, it easiest through `#pragma` directives. Change the code in `mul_test.cpp` to add the following `#pragmas`'s:

```c++
void mul_test(int* out, int in){

#pragma HLS INTERFACE mode=s_axilite port=return
#pragma HLS INTERFACE mode=s_axilite port=out
#pragma HLS INTERFACE mode=s_axilite port=in

      *out = 2*in;
}
```
We have successfully added port for our function (`return`) and for our variables.

9. Run C Synthesis by finding and clicking the correct botton. When this is done, it should open a `Synthesis Summary` report.

10. Click on `export RTL` to export your design and you can choose an address you want, or keep the default one, and hit `ok`. I usually use the default one. At this point, we can exit Vitis HLS.

## Vivado: RTL to bitstream
### Tutorial

1. Open `Vivado` application and create a new project. Name it `pynq_mul`. Then:
  * Choose a preferred location and hit `next`.
  * Select `RTL Project` and check `Do not specify sources a this time`. Hit `next`.
  * Set the default part to `xc7z020clg400-1`.
  * You should be in.

2. Under `IP Integrator`, click on `Create Block Design`.

3. Under `Project Manager`, click on `IP Catalog`.
  * Right click inside the newly open `IP Catalog` tab and select `Add Repository`.
  * In the open window, navigate to your Vivado HLS project folder and select `<path_to_vivado_hls_folder>/solution1/impl/ip`.
  * You should be able to search for and find `mul_test` under the `IP Catalog` (don't do anything with it, just check).

4. On the `Flow Navigator` and under `IP Integrator`, click on `open block design`. Then, click on the `+` button on the project manager. 
  * Find and click `mul_test`.
  * On that same window (or clikc the `+` button again), search and click `ZYNQ7 Processing System` to add it to your block design.
  * On the `green tab`, click `Run Block Automation` and then `Run Connection Automation` with default settings. Your diagram should change and show different connections and other IPs.

5. In `Sources`, right click `design_1` and select `Create HDL Wrapper`. Then, select `Let Vivado manage wrapper and auto-update` if a window pops up.

6. Under`Program and Debug`, click on `Generate Bitstream` and follow instructions to complete synthesis, implementation, and bitstream generation.

7. You can also check your `implemented design` by click on a button wiht that name. It should open a window where you can see the area of your implemented design on the board.

9. Before closing Vivado, we need to note our IP and its port addresses. Under `Sources`, find and open `mul_test_control_s_axi.v` (the exact name may vary across different versions of Vivado), scroll down and note addresses for in and out ports. We need these addresses for our host program.
  * Note the addresses of `control bus ap_ctrl` (possibly `0x00` here), output (possibly `0x10` here), and input (possibly `0x20` here).
  * Under `Address Editor`, note the IP's address (possibly `0x4000_0000` here).

10. Now, we can begin using the PYNQ Board.

   

   
## Connecting to a Jupyter Notebook on the PYNQ Board

Here is the link for the [PYNQ-Z2 Setup Guide](https://pynq.readthedocs.io/en/latest/getting_started/pynq_z2_setup.html). In case the three steps below do not work for you, you can follow all the steps on the tutorial and it should work.

0. You may need to download an `image` to the SD card that will be attached your PYNQ Board. **I need to add a link here...**, but really, the link above is your best guide. Once you know an image has been loaded, go to step 1.

1. Connect the pink board via microUSB and Ethernet cable to computer. Wait for yellow/green `done` light. Make sure **Power** jumper is set to USB since we are powering the board via USB.

2. On Linux, navigate to `Settings` and find `Network`. There, you have to choose `manual` under `IPv4`. Then, assign a static IP address and network address under `Adresses`. The IP address is `199.168.2.1` and the network address `255.255.255.0` as specified by the setup guide.

3. You can now open a jupyter notebook by navigating to [http://192.168.2.99](http://192.168.2.99/). If a password is asked, it should be `xilinx`.

Hopefully, you have opened a jupyter notebook window by this time.

## Running your program on the board via Jupyter Notebooks

1. Create a new folder and notebook. Upload `design_1_wrapper.bit` from `<vivado_project_path>/pynq_mul.runs/impl1` and copy `design_1.hwh` from `<vivado_project_path>/pynq_mul.gen/sources_1/bd/design_1/hw_handoff` to the folder you just created in Jupyter.

2. Make sure the .bit file and the .hwh file have the same name. In this case, they are named “design_1_wrapper.bit” and “design_1_wrapper.hwh”.

3. Open a notebook, and run the following code to test your IP:
```python
from pynq import Overlay
from pynq import MMIO

ol = Overlay("./design_1_wrapper.bit") # designate a bitstream to be flashed to the FPGA
ol.download() # flash the FPGA

mul_ip = MMIO(0x40000000, 0x10000) # (IP_BASE_ADDRESS, ADDRESS_RANGE), told to us in Vivado
inp = 5 # number we want to double

mul_ip.write(0x20, inp) # write input value to input address in fabric
print("input:", mul_ip.read(0x20)) # confirm that our value was written correctly to the fabric
mul_ip.write(0x00, 1) # set ap_start to 1 which initiates the process we wrote to the fabric
print("output:", mul_ip.read(0x10)) # read corresponding output value from the output address of the fabric
```
Congrats! You have finished the tutorial.

