# Instructions to Generate the LLP Tagger Bitstream Using the APx Framework

1. Generated a `Gitlab` token to do the following:
  ```bash
  git clone https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-layer2.git
  cd correlator-layer2
  ```

2. Edited `correlator-layer2/ci/add_apx_source_areas.sh` to include to overwrite `export gitlab` as follows:
  ```bash
  export gitlab="russelld:<myAccessToken>@gitlab.cern.ch" #access token before @
  ```
This is needed because I am working locally and not on CI.

3. From inside `correlator-layer2`, run `source ci/add_apx_source_areas.sh` to create the `submodules` folder.

4. Set up CMSSW (similar to EMP instructions):
  ```bash
  source /cvmfs/cms.cern.ch/cmsset_default.sh
  
  cd submodules/correlator-common/
  ./utils/setup_cmssw.sh -run CMSSW_14_0_0_pre3 cms-l1t-offline:phase2-l1t-integration-14_0_0_pre3 phase2-l1t-1400pre3_v9 # found in .gitlab-ci.yml
  export CMSSW_14_0_0_pre3
  cd CMSSW_14_0_0_pre3/src
  source ~/.bashrc # or ~/.bashrc, which sources all CMSSW variables
  cmsenv
  cd -
  ```

5. Manually add the `LLPtagger` objects to your CMSSW. Inside the files `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/jets.h`, `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/datatypes.h`, and `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/gt_datatypes.h`, 1) repeat the lines with `b_tag` and change to `llp_tag` and 2) repeat the lines with `Btag` and change to `LLPtag`. Make sure you add the correct number of bits when required. Note: The model added in the `llptag_nn` branch outputs 6 bits. Thus, changes bits to 6 were needed.

6. Manually build HLS IP cores because of error 3 described in the `debugging` section below. To build from scratch, I need to `export` the license as a variable:
  ```bash
  source /cvmfs/cms.cern.ch/cmsset_default.sh
  export CMSSW_VERSION=CMSSW_14_0_0_pre3
  #License UCSD
  export XILINXD_LICENSE_FILE=2100@cselm2.ucsd.edu
  export LM_LICENSE_FILE=2100@cselm2.ucsd.edu
  #license @corr4 and @Corr2
  export XILINXD_LICENSE_FILE=2100@xilinx-lic.fnal.gov
  export LM_LICENSE_FILE=2100@xilinx-lic.fnal.gov
  ```
  Then, `cd correlator-common/jetmet/seededcone` and run: 
  ```bash
  source /data/software/xilinx/Vivado/2020.1/settings64.sh #UCSD
  nohup bash run_all_tcl.sh > buildIPs.log 2>&1 &
  ```
7. Synthesize other IP cores in `jec` and `htmht` folders, too, not just `seededcone`. Note that they require `vitis_hls -f <file.tcl>` instead of `vivado_hls` to build the IP cores, as specified by the `cores.yaml` in the `jec/` and `htmht/` folders. I used `vitis_hls` version 2023.2 to be precise.

8. `cd correlator-layer2/jet_seededcone/board/apx` and run `make`. I've been using `vivado` version 2023.2 so far.

  


## Debugging

**[ERROR 1]:** `git-lfs` is missing.

**SOLUTION:** Ruckus needs `git-lfs` on your PATH (v2.1.1+ — this is 3.6.0, fine), and this CVMFS copy works natively on your machine. Put it on your PATH for this session:
```bash
export PATH="/cvmfs/cms.cern.ch/el8_amd64_gcc12/external/git-lfs/3.6.0-715980fbc5129d70a5db436ae65004d2/bin:$PATH"
git-lfs version   # sanity check
```

**[ERROR 2]:** `config_webtalk` problem.
  ```
  ## source -quiet ${RUCKUS_DIR}/vivado/messages.tcl
  ## set_property target_language VHDL [current_project]
  ## config_webtalk -user off
  invalid command name "config_webtalk"
      while executing
  "config_webtalk -user off"
      (file "/home/users/russelld/APX_BUILD/correlator-layer2/submodules/ruckus/vivado/project.tcl" line 38)
  ```
**SOLUTION:**  update the ruckus submodule to `v4.9.0`:
  ```bash
  cd /home/users/russelld/APX_BUILD/correlator-layer2/submodules/ruckus
  git fetch --tags
  git checkout v4.9.0
  ```

**[ERROR 3]:** Used `python3 util/hls_cores.py ...` to download already synthesized HLS IP cores (`JetCompute, JetFormat, etc...`). However, the error arises because of mismatches in existing `JetFormatWrapped.vhd` and downloaded `JetFormat` core, which does not have `q_v`.
   ```bash
    ********************************************************
    ********************************************************
    The following error(s) were detected during synthesis:
     formal port/generic <q_v> is not declared in <jet_format> [/home/users/russelld/APX_BUILD/correlator-layer2/submodules/correlator-common/jetmet/seededcone/firmware/hdl/JetFormatWrapped.vhd:36]
     formal port 'token_d' has no actual or default value [/home/users/russelld/APX_BUILD/correlator-layer2/submodules/correlator-common/jetmet/seededcone/firmware/hdl/JetFormatWrapped.vhd:39]
    ERROR: [Synth 8-439] module 'apd1_top' not found
    ERROR: [Common 17-69] Command failed: Synthesis failed - please see the console or run log file for details
    ********************************************************
    ********************************************************
    ********************************************************
   ```
**[ERROR 3.1]:** Tried building HLS cores with `python3 util/hls_cores.py ...`, as well. But for some reason, the program hung for a day until I stopped it.

**SOLUTION:** Building cores as before, e.g. running `vivado_hls -f run_JetCompute.tcl`, etc... in `correlator-common/jetmet/seededcone`.


