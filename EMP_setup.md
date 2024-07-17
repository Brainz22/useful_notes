# Setting up EMP FWK

### Prequisites:
How to build the framework: https://serenity.web.cern.ch/serenity/emp-fwk/firmware/instructions.html.

You want to make sure you install the prerequisites:

* Xilinx Vivado: 2021.2
* Python 2.7 - available on most linux distributions, natively or as miniconda distribution.
* ipbb: dev/2022f pre-release or greater - the IPbus Builder Tool. Note: a single ipbb installation is not work area specific and suffices for any number of projects. Check EMP repo.

## Building FWK
### 1. Download ippb command files:
   `curl -L https://github.com/ipbus/ipbb/archive/dev/2023a.tar.gz | tar xvz`.

### 2. Source the following file as follows: `source ipbb-dev-2023a/env.sh`.

### 3. Run the `ipbb` commands specified on the link, which I also put inside the `EMP_setup.sh`. So, do `bash EMP_setup.sh`. My `EMP_setup.sh` has the following content:
```bash
ipbb init LLPtag-work
cd LLPtag-work
#For the EMP framework
ipbb add git ssh://git@gitlab.cern.ch:7999/p2-xware/firmware/emp-fwk.git -r v0.8.1
ipbb add git ssh://git@gitlab.cern.ch:7999/ttc/legacy_ttc.git -b v2.1
ipbb add git ssh://git@gitlab.cern.ch:7999/cms-tcds/cms-tcds2-firmware.git -b v0_1_1
ipbb add git ssh://git@gitlab.cern.ch:7999/HPTD/tclink.git -r fda0bcf
ipbb add git ssh://git@gitlab.cern.ch:7999/dth_p1-v2/slinkrocket_ips.git -b v03.12
ipbb add git ssh://git@gitlab.cern.ch:7999/dth_p1-v2/slinkrocket.git -b v03.12
ipbb add git git@github.com:ipbus/ipbus-firmware.git -b v1.9

#For the Jet setup
ipbb add git ssh://git@gitlab.cern.ch:8443/rufl/RuflCore.git -r d3ddf86f
ipbb add git ssh://git@gitlab.cern.ch:8443/cms-cactus/phase2/firmware/correlator-common.git
ipbb add git ssh://git@gitlab.cern.ch:8443/cms-cactus/phase2/firmware/correlator-layer2.git
```
You might get errors when adding some of the above repos. You need to add yourself to the e-groups `emp-fwk-users` and `cms-tcds2-users` using this [link](https://e-groups.cern.ch/e-groups/EgroupsSearchForm.do).

### 4. We need to add a repo containing the "payload", I think this refers to the firmaware (all connections to the board and everything. I didn't have this, so I had to create one and documented the steps below:
   * Run `mkdir -p src/my-algo-repo/an-algo/firmware/cfg` and `mkdir -p src/my-algo-repo/an-algo/firmware/hdl`. This will create such directory trees.
   * Rather than start from scratch, I started with `null algo payload`. Run:
```bash
cp algo-work/src/emp-fwk/components/payload/firmware/hdl/emp_payload.vhd src/my-algo-repo/an-algo/firmware/hdl/
```
   * Create a top-level IPBB dependency file. This file will directly/indirectly specify all of the files required to create a bitfile. For now, let's reference the `emp_payload` we created in in the last bullet. So, 
```bash
echo 'src emp_payload.vhd' >> src/my-algo-repo/an-algo/firmware/cfg/top.dep
```
   * Reference the default payload address table from our new depfile:
```bash
echo 'addrtab -c emp-fwk:components/payload emp_payload.xml' >> src/my-algo-repo/an-algo/firmware/cfg/top.dep
```
   * Add default constraints to the dep file:
```bash
echo 'src -c emp-fwk:components/payload ../ucf/emp_simple_payload.tcl' >> src/my-algo-repo/an-algo/firmware/cfg/top.dep
```

### 5. We need to choose a board. Here in our lab I think we use `VCU118`.
We need to include the specific board dependency file. For `VCU118`, run:
```bash
echo `include -c emp-fwk:boards/vcu118' >> src/my-algo-repo/an-algo/firmware/cfg/top.dep`
```

### 6. Create the declaration `emp_project_decl` package:
We need to implement a VHDL package named `emp_project_decl` that defines things like clock frequencies, input buffers, output buffers, etc.. Rather than creating one from scratch, we can start with an example package for our specific board. For `VCU118`, run the lines:
```bash
cp algo-work/src/emp-fwk/projects/examples/vcu118/firmware/hdl/emp_project_decl_full.vhd src/my-algo-repo/an-algo/firmware/hdl/.
echo 'emp_project_decl_full.vhd' >> src/my-algo-repo/an-algo/firmware/cfg/top.dep
```

### 7. Create a Vivado Project (error... Skipping this)

Issue here when creating my own repo folder. Documentation says:
Assuming that your top-level `.dep` file is located in source area `my-algo-repo`, at path `an-algo/firmware/cfg/top.dep`, you can create a Vivado project area (under directory `proj/my_algo`) by running:
```bash
ipbb proj create vivado my_algo my-algo-repo:an-algo top.dep
cd proj/my_algo
```
But I get the following error shown below. Basically, there I tried to specify a directory myself since `my-algo-repo` is in `/home/users/russelld/EMP/src/my-algo-repo` and `proj/my_algo` will be in `/home/users/russelld/EMP/algo-work/proj`?

```bash
ipbb proj create vivado my_algo ../src/my-algo-repo/an-algo/firmware/cfg/top.dep
Usage: ipbb proj create [OPTIONS] [vivado|sim|vitis-hls] PROJNAME COMPONENT
                        [TOPDEP]

Error: Invalid value for 'COMPONENT': Malformed component name : ../src/my-algo-repo/an-algo/firmware/cfg/top.dep. Expected <package>:<component>
```

### Building the Firmware on EMP
I am starting from here: `/home/users/russelld/EMP/algo-work/src`, then going into seeded cone in `/home/users/russelld/EMP/algo-work/src/correlator-common/jetmet/seededcone` when needed.

1. In `/seededcone/`, we need to run `vivado_hls xx.tcl` on several `.tcl` files on `correlator-common.git`. First, we need to install CMSSW in `/correlator-common/` .
   * Run (this installation is only needed once):
     ```bash
     source /cvmfs/cms.cern.ch/cmsset_default.sh
     ./utils/setup_cmssw.sh -run CMSSW_12_5_5_patch1 p2l1pfp:L1PF_12_5_X l1ct-125x-v1.15
     ```
     **Note** that on Scully, the file `./utils/setup_cmssw.sh` did not exist. I had to copy it from the `correlator 4`.

   * Set the path by running `export CMSSW_VERSION=CMSSW_12_5_5_patch1`
  
   * Source `vivado 2019.1` (for now) on Scully as follows: 
   ```bash
   source /home/xilinx/Vivado/2019.1/.settings64-Vivado.sh  vivado
   ```

2. Run the `.tcl` files in `/correlator-common/jetmet/seededcone/`. We can use a bash for loop as follows:
```bash
command="vivado_hls"

for file in run_Jet*.tcl; do
    $command "$file"
done
```
So, put that code inside a `<file_name>.sh` file and run `bash <file_name>.sh`. This should generate folders for each file and `.vhd` files inside `/firmware/` (DONE).


