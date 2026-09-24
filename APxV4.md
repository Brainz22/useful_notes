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

5. Manually build HLS IP cores from scratch because of error 3 in V3 instructions. I need to export the license as a variable and run `bash run_all_tcl.sh` (I have the alias `vars`; activate it via `source ~/.bashrc` and run `vars`):
  ```bash
  alias vars='{ source /cvmfs/cms.cern.ch/cmsset_default.sh;
  export CMSSW_VERSION=CMSSW_17_0_0_pre2;
  export XILINXD_LICENSE_FILE=2100@xilinx-lic.fnal.gov;
  export LM_LICENSE_FILE=2100@xilinx-lic.fnal.gov;
  export LD_LIBRARY_PATH=/opt/cactus/lib:$LD_LIBRARY_PATH;
  export PATH=/opt/cactus/bin:$PATH;
  export PATH=/opt/cactus/bin/uhal/tools:$PATH;
  }'
  ```
Then run, 
  ```bash
  source /data/Xilinx/Vitis/2023.2/settings64.sh
  bash run_all_tcl.sh
  ```

6. Repeat this in `/jec` and `htmht/` folders, i.e. build the HLS IPs.

7. Run `make`.

## Useful Commands:

* The following commands allow me to highlight with a different color each algorithm on the `payload.vhd`. The commands must be input into the `tcl` CLI on the `Vivado GUI`:
  ```tcl
  highlight_objects -color red [get_cells -hierarchical -filter {NAME =~ *JetLoopInstance*}]
  select_objects [get_cells -hierarchical -filter {NAME =~ *JetLoopInstance*}]

  highlight_objects -color blue [get_cells -hierarchical -filter {NAME =~ *JetComputeInstance*}]
  select_objects [get_cells -hierarchical -filter {NAME =~ *JetComputeInstance*}]

  highlight_objects -color orange [get_cells -hierarchical -filter {NAME =~ *JetCorrectionInstance*}]
  select_objects [get_cells -hierarchical -filter {NAME =~ *JetCorrectionInstance*}]

  highlight_objects -color yellow [get_cells -hierarchical -filter {NAME =~ *JetTaggerInstance*}]
  select_objects [get_cells -hierarchical -filter {NAME =~ *JetTaggerInstance*}]

  highlight_objects -color green [get_cells -hierarchical -filter {NAME =~ *Deregionizer*}]
  select_objects [get_cells -hierarchical -filter {NAME =~ *Deregionizer*}]

  highlight_objects -color magenta [get_cells -hierarchical -filter {NAME =~ *HTMHTAlgo*}]
  select_objects [get_cells -hierarchical -filter {NAME =~ *HTMHTAlgo*}]
  ```
* The following are to highlight SLR regions (no color needed):
  ```tcl
  select_objects [get_pblocks slr0]
  select_objects [get_pblocks slr1]
  select_objects [get_pblocks slr2]
  ```
  Or all three outlined at once:
  ```tcl
  show_objects -name slr_view [get_pblocks {slr0 slr1 slr2}]
  ```

## Debugging: 

[ISSUE 1]: `make` was failing because of the input links in `correlator-layer2/jet_seededcone/board/apx/rtl/input.vhd` were wrongly defined. There should only be 32.

**Solution:** So far, I had `claude` define the ports for me...

[ISSUE 2]: `correlator-layer2/jet_seededcone/board/apx/cfg/PrjSpecPkg.vhd` allocates the 100 links that need to be used. 

**Solution:** Make sure the links defined in `input.vhd` are not off.

