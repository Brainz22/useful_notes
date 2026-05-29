# Produce JetNTuples for LLP Tagging Purposes

### Requirements

* Access to the `submissions` fork [here](https://github.com/Brainz22/submission/tree/ucsd-uaf-setup).
* Access to `lxplus` and active grid certificate.
* Access to `FastPUPPI` repo [here](https://github.com/Brainz22/FastPUPPI/tree/rm-l1tsc4ngjettagger).

## Produce Ntuples

1. ```bash
   source /cvmfs/cms.cern.ch/cmsset_default.sh
   cmsrel CMSSW_15_1_0_pre4
   cd CMSSW_15_1_0_pre4/src
   cmsenv
   git cms-init 
   git branch 

   git cms-addpkg L1Trigger/Phase2L1ParticleFlow
   git cms-addpkg L1Trigger/Configuration
   git cms-addpkg DataFormats
   git cms-addpkg L1Trigger/TrackTrigger
   git cms-addpkg SimTracker/TrackTriggerAssociation
   #git cms-checkout-topic -u Brainz22:from-CMSSW_15_1_0_pre4
   git clone git@github.com:Brainz22/FastPUPPI.git -b rm-l1tsc4ngjettagger
   scram b -j8 #compile modules

   git clone https://github.com/Brainz22/submission.git -b ucsd-uaf-setup
   ```
   This setup works on UCSD uaf.
   
2. Need to know where you are starting from. If you are starting from `GEN-SIM-DIGI-RAW` file, we need to slim down root files using the cms config `cmssw_config: ../FastPUPPI/NtupleProducer/python/runInputs151X.py`, which lives in a different branch. Thus, 
  ```bash
  cd FastPUPPI
  git checkout 15_1_X_LLPtagging #checks out branch with correct cms config.
  cd ../
  scram b -j8
  cd ../submission
  ```
  * CMS DAS can be misleading. It might show sites, but the files are not there.

  FIX: Through the terminal, check for DISK sites with `dasgoclient --query="site dataset=/HiddenGluGluH_mH-125_Phi-15_ctau-1_cccc_TuneCP5_14TeV-pythia8/Phase2Spring24DIGIRECOMiniAOD-PU200_Trk1GeV_140X_mcRun4_realistic_v6-v1/GEN-SIM-DIGI-RAW-MINIAOD"`, for example. Then, check that at least one of the files is there with: `dasgoclient --query="file dataset=/HiddenGluGluH_mH-125_Phi-15_ctau-1_cccc_TuneCP5_14TeV-pythia8/Phase2Spring24DIGIRECOMiniAOD-PU200_Trk1GeV_140X_mcRun4_realistic_v6-v1/GEN-SIM-DIGI-RAW-MINIAOD site=T1_IT_CNAF_Disk" | head -1`.

We can also perform a quick check to see if file is readable via `cms-xrd-global: xrdfs cms-xrd-global.cern.ch stat /store/mc/Phase2Spring24DIGIRECOMiniAOD/HiddenGluGluH_mH-125_Phi-15_ctau-10_cccc_TuneCP5_14TeV-pythia8/GEN-SIM-DIGI-RAW-MINIAOD/PU200_Trk1GeV_140X_mcRun4_realistic_v6-v1/2810000/0078c306-2f74-4bc0-a181-2aedaa9d81b1.root`. It should say isReadable at end of the output. 

3. Once we have found the correct sites with files on DISK. We can add it to the `.yaml` and submit using `submit.py` as shown on **Step 2** section on the instructions [here](https://github.com/Brainz22/useful_notes/blob/main/HTCondor_signal.md). Basically, 
  ```bash
  python3 submit.py -f submit_INFP_151X_<sample_name>.yaml --create # creates folder with task name and creates sandbox.
  python3 submit.py -f submit_INFP_151X_<sample_name>.yaml --submit # submits HTCondor.
  ```
  In the link attached, there are useful commands and debugging notes.

5. If we produced the slimed down ntuples (fp_inputs), or skipped steps 3 and 4 (fp_inputs files exist), then we can directly use the `jetNtupler` at `../FastPUPPI/NtupleProducer/python/runJetNTuple.py`.
   * Switch to the correct branch
     ```bash
     git checkout rm-l1tsc4ngjettagger
     cd ..
     scram b -j8
     cd submissions/
     ```
  
7. Once in the correct branch, the submissions has example `.yaml` files for this step, e.g. `submit_NTP_151X_QCD_Pt15To3000_Flat_PU200_UCSD.yaml` uses QCD fp_inputs files stored on lxplus `eos/`. Usage is the same:
   ```bash
   python3 submit.py -f submit_NTP_151X_QCD_Pt15To3000_Flat_PU200_UCSD.yaml --create
   python3 submit.py -f submit_NTP_151X_QCD_Pt15To3000_Flat_PU200_UCSD.yaml --submit
   ```
The files produced will be stored in the `output_dir` specified in the `.yaml`.

