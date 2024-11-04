# Instructions

These instructions assume:
* You are working on correlator 2;
* you already created the python environment `jetID` from the [EMP_setup](https://github.com/Brainz22/useful_notes/blob/main/EMP_setup.md) isntructions;
* you are able to `source ipbb-dev-2023a/env.sh`;
* Your CERN account is added to the groups `emp-fwk-users`, `cms-tcds2-users`, and `cms-cactus`.

1. Let's create a new work area and change into it:
```bash
ipbb init llp-tagger
cd llp-tagger
```
2. Authenticate your account and clone the needed repositories into your working area:
```bash
kinit YOURUSERNAME@CERN.CH
ipbb add git https://:@gitlab.cern.ch:8443/cms-cactus/phase2/firmware/correlator-layer2.git -b llptag_nn
cd correlator-layer2/ci
source add_ipbb_source_areas.sh
```

3. Set up `CMSSW`:
```bash
source /cvmfs/cms.cern.ch/cmsset_default.sh

cd src/correlator-common/
./utils/setup_cmssw.sh -run CMSSW_14_0_0_pre3 cms-l1t-offline:phase2-l1t-integration-14_0_0_pre3 phase2-l1t-1400pre3_v9 
# version and tags were found in .gitlab-ci.yml
export CMSSW_VERSION=CMSSW_14_0_0_pre3 #export to environment
```

4. Everytime I ssh into a computer, the variables are restarted. Thus, we need to export all variables again. These variables are:
```bash
source /cvmfs/cms.cern.ch/cmsset_default.sh
export CMSSW_VERSION=CMSSW_14_0_0_pre3
#license @Correlator2
export XILINXD_LICENSE_FILE=2100@xilinx-lic.fnal.gov
export LM_LICENSE_FILE=2100@xilinx-lic.fnal.gov
```

5. Manually add the `LLPtagger` objects to your CMSSW. Inside the files `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/jets.h`, `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/datatypes.h`, and `CMSSW_14_0_0_pre3/src/DataFormats/L1TParticleFlow/interface/gt_datatypes.h`, 1) repeat the lines with `b_tag` and change to `llp_tag` and 2) repeat the lines with `Btag` and change to `LLPtag`. Make sure you add the correct number of bits when required. **Note:** The model added in the `llptag_nn` branch outputs 6 bits. Thus, changes bits to 6 were needed.

6. When you are in `/correlator-common/jetmet/seededcone`, 
```bash
source /data/Xilinx/Vivado/2019.2/settings64.sh
run bash run_all_tcl/sh
```
