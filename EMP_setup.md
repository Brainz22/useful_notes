# Setting up EMP FWK

### Notes:

* `csynth` of the tagger fails with `Vivado2019.2` and `Vitis2020.1`, so I moved to `Vitis2020.2`.

### Prequisites:
How to build the framework: https://serenity.web.cern.ch/serenity/emp-fwk/firmware/instructions.html.

You want to make sure you install the prerequisites:

* Xilinx vitis2022.2, vivado2019.2. I found that different versions give different results sometimes. I noted on the steps whenever I changed versions.
* Python 2.7 - available on most linux distributions, natively or as miniconda distribution.
* ipbb: dev/2022f pre-release or greater - the IPbus Builder Tool. Note: a single ipbb installation is not work area specific and suffices for any number of projects. Check EMP repo.

### Useful commands to help debug:

* `cp /home/rmarroqu/EMP/btag-work/proj/Btagging/Btagging/Btagging.sim/sim_1/behav/xsim/source.txt /home/rmarroqu/EMP/LLPtag-work/proj/LLPtagging/LLPtagging/LLPtagging.sim/sim_1/behav/xsim` and `cp /home/rmarroqu/EMP/btag-work/proj/Btagging/Btagging/Btagging.sim/sim_1/behav/xsim/source.txt /home/rmarroqu/EMP/LLPtag-work/src/emp-fwk/components/testbench/firmware/hdl` (last one is for synthesis).

* `ipbb dep report` after `ipbb proj create ...`.
* `lsof | grep <.nfs0000000358311012000009a8>`, then `kill PID`. The PID will be shown by the List of Open Files `lsof` with input file given.
* `git rev-parse HEAD`, returns the commit hash of the current repo. I use it to check I cloned correct commit versions when needed.

## Building FWK

0. You can work with `mamba` or `micromamba`. Here is a quick way to install `mamba`:
```bash
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

1. Create and activate a new python environment with `python=2.7`
```bash
mamba create --name jetID python=2.7
mamba activate jetID
python --version #should return a 2.7 version
mamba install git
which git # Git should be in conda path now
```

1.1 Download ippb command files:
   `curl -L https://github.com/ipbus/ipbb/archive/dev/2023a.tar.gz | tar xvz`.

2. Source the following file as follows: `source ipbb-dev-2023a/env.sh`.

3. Run the `ipbb` commands specified on the link, which I also put inside the `EMP_setup.sh`. So, do `bash EMP_setup.sh`. My `EMP_setup.sh` has the following content:
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
ipbb add git ssh://git@gitlab.cern.ch:7999/cms-cactus/phase2/firmware/correlator-common -b llptag_nn
ipbb add git ssh://git@gitlab.cern.ch:7999/cms-cactus/phase2/firmware/correlator-layer2 -b llptag_nn
```
For Correlator 2 on FNAL, I had to use `kinit username@CERN.CH` and add repos the following way for some reason:

```bash
ipbb add git https://:@gitlab.cern.ch:8443/p2-xware/firmware/emp-fwk.git -r v0.8.1
ipbb add git https://gitlab.cern.ch/ttc/legacy_ttc.git -b v2.1
ipbb add git https://:@gitlab.cern.ch:8443/cms-tcds/cms-tcds2-firmware.git -b v0_1_1
ipbb add git https://gitlab.cern.ch/HPTD/tclink.git -r fda0bcf
ipbb add git https://gitlab.cern.ch/dth_p1-v2/slinkrocket_ips.git -b v03.12
ipbb add git https://:@gitlab.cern.ch:8443/dth_p1-v2/slinkrocket.git -b v03.12
ipbb add git https://github.com/ipbus/ipbus-firmware -b v1.9

#For the Jet & MET setup
ipbb add git https://:@gitlab.cern.ch:8443/rufl/RuflCore.git -r d3ddf86f
ipbb add git https://:@gitlab.cern.ch:8443/cms-cactus/phase2/firmware/correlator-common -b llptag_nn
ipbb add git https://:@gitlab.cern.ch:8443/cms-cactus/phase2/firmware/correlator-layer2 -b llptag_nn
```
Alternative (and probably easier way) is using the file from Continuous Integration (CI). Note that the branch `-b` flag for the `correlator-common.git` can be removed and it will clone the master branch. When you in the folder `LLPtag-work/src` that we made above, run: 
```bash

kinit YOURUSERNAME@CERN.CH
ipbb add git https://:@gitlab.cern.ch:8443/cms-cactus/phase2/firmware/correlator-layer2 -b llptag_nn
cd correlator-layer2/ci
source add_ipbb_source_areas.sh

```

You might get errors when adding some of the above repos. You need to add yourself to the e-groups `emp-fwk-users` and `cms-tcds2-users` using this [link](https://e-groups.cern.ch/e-groups/EgroupsSearchForm.do). Additionally, you might need `gitlab` and `github` keys. I put instructions on the notes [here](https://github.com/Brainz22/useful_notes/blob/main/Workflow%40corr4_APxV1.md).



# Building Project and Running Simulation and Synthesis

### Simulation: 

After cloning the correct repos specified 1-4, and assuming I cloned the master branch of `correlator layer 2`:

*   Create a project via:
```bash
source /data/software/xilinx/Vivado/2022.2/settings64.sh # Mulder or Scully
ipbb proj create vivado LLPtagging correlator-layer2:jet_seededcone/board/serenity top_sim.dep
```
This will add the `LLPtagging` project in `../proj`.

*   Run simulation and open vivado GUI:
```bash
cd ../proj/LLPtagging
ipbb vivado generate-project --enable-ip-cache 
```
Locate the `.xpr` file that will be generated and run:
```bash
vivado LLPtagging.xpr
```
Make sure GUI can be activated, i.e. use `ssh -Y ...`.

### Synthesis:

Similar to the above, but make the following changes to the commands (note the different `.dep` file):

* `ipbb proj create vivado LLPtagging_syn correlator-layer2:jet_seededcone/board/serenity top_serenity.dep`. This creates a whole new folder in `/proj/` to run synthesis.
* Generate project and start synthesis
```bash
cd proj/LLPtagging_syn
ipbb vivado generate-project --enable-ip-cache -1
ipbb vivado synth -j8 impl -j8
```
Synthesis might take a minute, so the last command can be run in the background in a `nohup` command as follows:
`nohup ipbb vivado synth -j8 impl -j8 &`. You can even close the terminal. Output is stored in `nohup.out`.
You can check that it's running using the command `jobs` or `ps -p <jobID>`. The job ID was an output directly on the terminal from running `nohup`. 

# Deploying the Full Jet Project with the LLP Tagger:

The project is built using the following two gitlab repos (which are added in step 3 in the beginning): [correlator-layer2](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-layer2/-/tree/master/jet_seededcone?ref_type=heads) and [correlator-common](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-common).

0. We need to download and synthesize the IPs, as stated on the instructions [here](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-layer2/-/tree/master/jet_seededcone?ref_type=heads). After running the two commands there, move on to step 1.

   *   Working on FNAL Correlator 4, the second command fails at the moment with the error:
```bash
COPY core "JetLoop" from local correlator-common instance
Traceback (most recent call last):
File "correlator-layer2/util/hls_cores.py", line 205, in <module>
 get_correlator_common_hls(os.path.join(opt.CCPATH, cr[0]), cr[1], ci_url_base)
File "correlator-layer2/util/hls_cores.py", line 98, in get_correlator_common_hls
 for vhd_file in os.listdir(local_build_dir):
FileNotFoundError: [Errno 2] No such file or directory: 'correlator-common/jetmet/seededcone/JetLoop/solution/impl/vhdl'
```
Maybe I need to run thsi after running `vivado_hls -f` on the respective files, since it looks for a folder create after that.
   

2.    Set up CMSSW
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


2.   Add Vivado variables: `source /data/software/xilinx/Vivado/2020.1/settings64.sh` (might change depending on your local server). Preferably, use `Vivado2019.2` to run produce wrappers, i.e. to run `vivado_hls -f`.

      Every time I log out and log back in, I need to run:
   ```bash
   source /cvmfs/cms.cern.ch/cmsset_default.sh
   export CMSSW_VERSION=CMSSW_14_0_0_pre3
   #source /data/software/xilinx/Vivado/2020.1/settings64.sh #for opening GUI and LLPtagger synth
   source /home/xilinx/Vivado/2019.2/settings64.sh #For synthesis of everything else
   #License
   export XILINXD_LICENSE_FILE=2100@cselm2.ucsd.edu
   export LM_LICENSE_FILE=2100@cselm2.ucsd.edu
   #license @corr4 and @Corr2
   export XILINXD_LICENSE_FILE=2100@xilinx-lic.fnal.gov
   export LM_LICENSE_FILE=2100@xilinx-lic.fnal.gov
   ```

3.   To get rid of errors because of definitions in `CMSSW` only defining the `btagger`, I made changes as follows:
Because I need `llpTagScore` instead of `btag_Score`, I thought of changing things in my `CMSSW`. Thus, I manually changed things that were defined as `b_tag` to `llp_tag` and `Btag` to `LLPtag` in the files `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/jets.h`, `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/dataformats.h`, and `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/gt_datatypes.h`. The amount of the bits we'll use comes from `struct Jet` in the file `correlator-common/CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/jets.h`. The specific objects there are defined in `gt_datatypes.h` and `datatypes.h`. However, the bits we use are taken from `datatypes.h`.
Also, add `llptag` wherever we see `btag` in `correlator-common/jetmet/seededcone/RUFL/Jet/firmware/hdl/PkgJet.vhd` and include the correct number of bits there, too.

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

5.   Add the `LLPtag` folder with the hls files, similar to [here](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-common/-/tree/btag_nn_token/jetmet/seededcone/btag?ref_type=heads). I used vivado2020.1 to convert the model to hls because it optimized the latency (versus vitis2022.2, for example).
      * Following the link, add the files `algo_llp.cpp` and `algo_llp.h` similar to the `btagger` on the link. Make sure you pay attention to the changes in `make_inputs`.
      * Add `synchronizer.h`.
      * Add `data.h` and make changes to the respective file following the one for the btagger file [here](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-common/-/blob/btag_nn_token/jetmet/seededcone/firmware/data.h?ref_type=heads). Basically, we need to define `lllp_tag_score_t`, a `clear()` function, and some constants similar to the file on the link.
      * Add the respective `.tcl` files in one directory above `../`.
      * Remember that wrappers need to be put in the default library. Thus, edit the `.tcl` files above, accordingly.
      * Add the wrappers (similar to the ones from the btagger) in `seededcone/firmware/`. Change the bits number to be 57 + output bits of `LLPtagger`. Right now, I am trying with 6 because total number must be 64.

7.  Add variables accordingly in `src/correlator-layer2/jet_seededcone/firmware/hdl/PkgConstants.vhd`, similar to the `btag_nn_token` branch.

8.  Make changes to the file `correlator-common/jetmet/seededcone/JetControl/firmware/hdl/JetControl.vhd` following the [btagger](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-common/-/blob/btag_nn_token/jetmet/seededcone/JetControl/firmware/hdl/JetControl.vhd?ref_type=heads).

9. Add the `.vhd` wrappers in the location `correlator-common/jetmet/seededcone/firmware/`. These are similar to the ones for the btagger [here](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-common/-/tree/btag_nn_token/jetmet/seededcone/firmware?ref_type=heads). So, use those and change things accordingly.

10. Add the `BitonicSort.vhd` file and the change the respective `.dep` file similar to the way done in [these folders](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-common/-/tree/btag_nn_token/l2-deregionizer/RUFL/IO/firmware?ref_type=heads). 

11.   The main changes will be in the `LLPag` folder (`cd ..` from `firmware_hls`). We need the `.tcl` to `C synthesize` the wrapper `.cpp` files. One of the `.tcl` files is for the `NN`, while the other is for the `synchronizer`. For me, `firmware_hls/algo_LLP.cpp` is the wrapper, which also contains the `synchronizer`. Once we have those things, we can run everything with the following `.sh` file and run it using the `bash` command (make sure you use Vitis 2022.2 for this):
```bash
#Use vivado2022.2 for compilation
command="vitis_hls -f"

rm -rf proj_llp/ #made by run_hls_LLP.tcl
rm -rf proj_llp_sync/ #made by run_llptag_sync.tcl
rm -rf firmware/

$command run_llptag_sync.tcl
$command run_hls_LLP.tcl
```

12. Run all other `.tcl` files using `vivado_hls -f <file.tcl>`, but vivado version `2019.2`.

13. In the folder `correlator-common/jetmet/jec`, I had to use vitis 2022.2 and run `vitis_hls -f run_Synth.tcl`. Otherwise, either `jec_main.vhd` would come out with different name after synthesis or the bits did not match in `JetCorrectionWrapped.vhd`.

14. Same thing in the folder `correlator-common/jetmet/htmht`. Use vitis 2019.2 and run `bash synth_all.sh`.

15. Make sure to add the `JetLLPtag` library by adding the line `include -c jetmet/seededcone/LLPtag LLPtag.dep` in the file `correlator-common/jetmet/seededcone/firmware/cfg/jet.dep`.

16. Did a minor change to `JetFormatWrapped.vhd`. I chaged the port `q` to `q_V` to match `seededcone/JetFormat/solution/syn/vhdl/jet_format.vhd/jetFormat.vhd`.

17. Made minor change to `JetCorrectionWrapped.vhd`. `jet_in` and `jet_out` should amount to the bits used in `struct Jet` in the file `correlator-common/CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/jets.h`. The specific objects there are defined in `gt_datatypes.h` and `datatypes.h`. However, the bits we use are taken from `datatypes.h`. Then, the rest out of the 64 bits in `.data` datatype, we set to zero as `qjet.data(63) <= '0';`. This is is done in the btagger, too. Error 1 was related to this.


18. After everything is synthesized without errors, follow the previous section `Building Project and Running a Quick Simulation`.

 


## Possible errors: 

1. Error about not finding the `source.txt` file. Download it from this [link](https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-layer2/-/jobs/41147266/artifacts/file/emu_patterns/l2_in_merged_0.txt.gz) and name it `source.txt`. Then, add it to the respective locations shown by the error. This is the reason why I have the two `cp` commands in the `useful commands...` section.

2. This happened after messing with the `.data` type located in `correlator-common/l2-deregionizer/RUFL/IO/firmware/hdl/PkgIO.vhd`. It needs to have elements `[63:0]`. 
```bash
ERROR: [VRFC 10-666] expression has 64 elements; expected 63 [/home/rmarroqu/EMP/LLPtag-work/src/correlator-layer2/jet_seededcone/firmware/hdl/input.vhd:38]
```
