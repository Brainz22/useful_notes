# Instructions to Generate Jet Tagger Bitstreams Using the APx Framework

Generating NG tagger bitstream.

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

4. Set up CMSSW:
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

