# Instructions to Generate Jet Tagger Bitstreams Using the APx Framework

Generating NG tagger bitstream from Fermilab correlator 4.

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

4. Set up CMSSW inside `submodules/correlator-common`:
   ```bash
    source /cvmfs/cms.cern.ch/cmsset_default.sh
    
    cd submodules/correlator-common/
    ./utils/setup_cmssw.sh -run CMSSW_17_0_0_pre2 p2l1pfp:L1PF_17_0_X l1ct-170x-v1.6 # command found in README.md and [ tag ] in .gitlab-ci.yml
    export CMSSW_17_0_0_pre2
    cd CMSSW_17_0_0_pre2/src
    source ~/.bashrc # or ~/.bashrc, which sources all CMSSW variables
    cmsenv
    cd -
    ```

5. Manually build HLS IP cores from scratch because of error 3 in V3 instructions.I need to export the license as a variable and run `bash run_all_tcl.sh`...

