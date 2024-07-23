# Setting up EMP FWK

### Prequisites:
How to build the framework: https://serenity.web.cern.ch/serenity/emp-fwk/firmware/instructions.html.

You want to make sure you install the prerequisites:

* Xilinx Vivado: 2021.2
* Python 2.7 - available on most linux distributions, natively or as miniconda distribution.
* ipbb: dev/2022f pre-release or greater - the IPbus Builder Tool. Note: a single ipbb installation is not work area specific and suffices for any number of projects. Check EMP repo.

## Building FWK

### 1. Create and activate a new python environment with `python=2.7`
```bash
mamba create --name jetID python=2.7
mamba activate jetID
```

### 1.1 Download ippb command files:
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
ipbb add git ssh://git@gitlab.cern.ch:7999/rufl/RuflCore.git -r d3ddf86f
ipbb add git ssh://git@gitlab.cern.ch:7999/cms-cactus/phase2/firmware/correlator-common.git
ipbb add git ssh://git@gitlab.cern.ch:7999/cms-cactus/phase2/firmware/correlator-layer2.git
```
You might get errors when adding some of the above repos. You need to add yourself to the e-groups `emp-fwk-users` and `cms-tcds2-users` using this [link](https://e-groups.cern.ch/e-groups/EgroupsSearchForm.do). Additionally, you might need `gitlab` and `github` keys. I put instructions on the notes [here](https://github.com/Brainz22/useful_notes/blob/main/Workflow%40corr4_APxV1.md).

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

# Building Project and Running a Quick Simulation

After cloning the correct repos specified 1-4, and assuming I cloned the master branch of `correlator layer 2`:

*   Create a project via:
```bash
source /data/software/xilinx/Vivado/2020.1/settings64.sh # Mulder or Scully
ipbb proj create vivado jet-sim correlator-layer2:jet_seededcone/board/serenity top_sim.dep
```
This will add the `jet-sim` project in `../proj`.

*   Run simulation and open vivado GUI:
```bash
cd ../proj/jet-sim
ipbb vivado generate-project
```
Locate the `.xpr` file that will be generated and run:
```bash
vivado jet-sim.xpr
```
Make sure GUI can be activated, i.e. use `ssh -Y ...`.

# Deploying the Full Jet Project with the LLP Tagger:

The project is built using the following two gitlab repos (which are added in step 3 in the beginning): [correlator-layer2](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-layer2/-/tree/master/jet_seededcone?ref_type=heads) and [correlator-common](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-common).

1.    Set up CMSSW
```bash
source /cvmfs/cms.cern.ch/cmsset_default.sh

cd src/correlator-common/
./utils/setup_cmssw.sh -run CMSSW_14_0_0_pre3 cms-l1t-offline:phase2-l1t-integration-14_0_0_pre3 phase2-l1t-1400pre3_v9 # found in .gitlab-ci.yml
```
   Some errors may arise, but ignore:
```bash
/cvmfs/cms.cern.ch/el8_amd64_gcc11/external/gcc/11.4.1-30ebdc301ebd200f2ae0e3d880258e65/bin/../lib/gcc/x86_64-redhat-linux-gnu/11.4.1/../../../../x86_64-redhat-linux-gnu/bin/ld.bfd: cannot find -lssl: No such file or directory
/cvmfs/cms.cern.ch/el8_amd64_gcc11/external/gcc/11.4.1-30ebdc301ebd200f2ae0e3d880258e65/bin/../lib/gcc/x86_64-redhat-linux-gnu/11.4.1/../../../../x86_64-redhat-linux-gnu/bin/ld.bfd: cannot find -lcrypto: No such file or directory
collect2: error: ld returned 1 exit status
gmake: *** [config/SCRAM/GMake/Makefile.rules:1793: tmp/el8_amd64_gcc11/src/L1Trigger/L1CaloTrigger/plugins/L1TriggerL1CaloTriggerAuto/libL1TriggerL1CaloTriggerAuto.so] Error 1
gmake: *** [There are compilation/build errors. Please see the detail log above.] Error 2
```
   Export to environment `export CMSSW_VERSION=CMSSW_14_0_0_pre3`.


2.   Add Vivado variables: `source /data/software/xilinx/Vivado/2020.1/settings64.sh` (might change depending on your local server).

      Every time I log out and log back in, I need to run:
   ```bash
   source /cvmfs/cms.cern.ch/cmsset_default.sh
   export CMSSW_VERSION=CMSSW_14_0_0_pre3
   source /data/software/xilinx/Vivado/2020.1/settings64.sh
   #License
   export XILINXD_LICENSE_FILE=2100@cselm2.ucsd.edu
   export LM_LICENSE_FILE=2100@cselm2.ucsd.edu
   ```

3.   To get rid of errors because of definitions in `CMSSW` only defining the `btagger`, I made changes as follows:
Because I need `llpTagScore` instead of `btag_Score`, I thought of changing things in my `CMSSW`. Thus, I manually changed things that were defined as `b_tag` to `llp_tag` and `Btag` to `LLPtag` in the files `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/jets.h`, `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/dataformats.h`, and `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/gt_datatypes.h`.

4.    Run `vivado_hls -f *.tcl`. Currently failing `vivado_hls -f run_Sim.tcl`run because:
```bash
WARNING: [HLS 200-40] Cannot find test bench file '../../dumpfiles/TTbar_PU200_Barrel.dump'
WARNING: [HLS 200-40] Cannot find test bench file '../../dumpfiles/TTbar_PU200_HGCal.dump'
WARNING: [HLS 200-40] Cannot find test bench file '../../dumpfiles/TTbar_PU200_HGCalNoTK.dump'
WARNING: [HLS 200-40] Cannot find test bench file 'JetsOut.txt'
WARNING: [HLS 200-40] Cannot find test bench file 'JetsOut_lr.txt'
```
You can download the files from [here](https://cactus.web.cern.ch/cactus/phase2/firmware/correlator-common/tags/1.5.10/dumpfiles/). Or,
If working on `lxplus`, you should not run into this issue. Because I was working locally, I `git clone`d the `correlator-common` branch and built the `CMSSW_14...` there. Then, I used `scp russelld@lxplus.cern.ch:/afs/cern.ch/user/r/russelld/EMP/correlator-common/dumpfiles/* dump/` after making `dump/`. 

   -   Next, I ran zip the folder: `tar -czvf dump.tar.gz dump`.
   -   `scp` to Mulder: `scp dump.tar.gz russelld@mulder.t2.ucsd.edu:/home/users/russelld/EMP/Serenity/work/LLPtag-work/src/correlator-common/dumpfiles/`.
   -    mulder, untar the folder: `tar -xzvf dump.tar.gz`.
   -    Run `vivado_hls -f run_Sim.tcl`

5.   Add the `LLPtag` folder with the hls files, similar to [here](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-common/-/tree/btag_nn_token/jetmet/seededcone/btag?ref_type=heads).

6.   I add and change accordingly following the branch in the attached in step 5. `firmware_hls` is the folder resulting from converting our model using `hls4ml`. The new chages there need to be included in the `data.h` and add the `synchronizer.h`. Some new variables may need to be defined here and there Check errors.

7.   The main changes will be in the `LLPag` folder (`cd ..` from `firmware_hls`). We need the `.tcl` to `C synthesize` the wrapper `.cpp` files. One of the `.tcl` files is for the `NN`, while the other is for the `synchronizer`. For me, `firmware_hls/algo_LLP.cpp` is the wrapper, which also contains the `synchronizer`. Once we have those things, we can run everything with the following `.sh` file and run it using the `bash` command:
```bash
#Use vivado2022.1 for compilation
command="vitis_hls -f"

rm -rf proj_llp/ #made by run_hls_LLP.tcl
rm -rf proj_llp_sync/ #made by run_llptag_sync.tcl
rm -rf firmware/

$command run_llptag_sync.tcl
$command run_hls_LLP.tcl
```

