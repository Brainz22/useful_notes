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

5. Download HLS IPs instead of building them from scratch as I did in the EMP instructions:
  ```bash
  cd /home/users/russelld/APX_BUILD/correlator-layer2
  python3 util/hls_cores.py --get-ci -r $CORRELATOR_COMMON_VERSION -c hls-cores.yaml -p l2-seededcone \
  --cc-path submodules/correlator-common --inject
  ```

6. `cd correlator-layer2/jet_seededcone/board/apx` and run `make`.
   Currently running into issues because I am running `correlater-layer2 [master branch]` and `correlater-common [llptag_nn branch]`, which means I need to define some `LLPtag` variables in `correlator-layer2/jet_seededcone/firmware/hdl/PkgConstants.vhd`, for example. Latest error:
   ```bash
     ********************************************************
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
   I need to resemble the code from my `llptag_nn` branch.

## Debugging

**[Error 1]:** `git-lfs` is missing.

**Solution:** Ruckus needs `git-lfs` on your PATH (v2.1.1+ — this is 3.6.0, fine), and this CVMFS copy works natively on your machine. Put it on your PATH for this session:
```bash
export PATH="/cvmfs/cms.cern.ch/el8_amd64_gcc12/external/git-lfs/3.6.0-715980fbc5129d70a5db436ae65004d2/bin:$PATH"
git-lfs version   # sanity check
```

**[Error 2]:** `config_webtalk` problem.
  ```
  ## source -quiet ${RUCKUS_DIR}/vivado/messages.tcl
  ## set_property target_language VHDL [current_project]
  ## config_webtalk -user off
  invalid command name "config_webtalk"
      while executing
  "config_webtalk -user off"
      (file "/home/users/russelld/APX_BUILD/correlator-layer2/submodules/ruckus/vivado/project.tcl" line 38)
  ```
**Solution:**  update the ruckus submodule to `v4.9.0`:
  ```bash
  cd /home/users/russelld/APX_BUILD/correlator-layer2/submodules/ruckus
  git fetch --tags
  git checkout v4.9.0
  ```



