1. We have to install `CMSSW_14_0_pre3` by following the recipe on the twiki [here](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideL1TPhase2Instructions#Recipe_for_phase2_l1t_1400pre3_v2). Basically, we have to run: 
```bash
cmsrel CMSSW_14_0_0_pre3
cd CMSSW_14_0_0_pre3/src/
cmsenv
git cms-init
git cms-checkout-topic -u cms-l1t-offline:phase2-l1t-1400pre3_v5
scram b -j 8

#Get missing data files for NN Calo Taus
cd ../../
git clone https://github.com/jonamotta/L1Trigger-L1CaloTrigger.git
cd CMSSW_14_0_0_pre3/src/
git cms-addpkg L1Trigger/L1CaloTrigger
mkdir L1Trigger/L1CaloTrigger/data
cp -r ../../L1Trigger-L1CaloTrigger/Phase2_NNCaloTaus L1Trigger/L1CaloTrigger/data
```

2. We need to go to the repo [here](https://github.com/cms-l1-dpg/Phase2-L1Nano/tree/main). Note that the first set of instructions in the "Setup" section looks like the same as what's on the twiki in step 1 above (I haven't checked it line by line). But I stuck to the [twiki](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideL1TPhase2Instructions#Recipe_for_phase2_l1t_1400pre3_v2) since Emyr suggested that. After step 1, add the repo suggested in this step by running:
```bash
### ADDING NANO
git clone git@github.com:cms-l1-dpg/Phase2-L1Nano.git PhysicsTools/L1Nano
scram b -j 8
```
***Note***: I ran into some authentication issues and had to add a Github SSH key to LPC (I am working from LPC). If you run into these issues, my notes [here](https://github.com/Brainz22/useful_notes/blob/main/Workflow%40corr4_APxV1.md). Step 1,2,4,5 explain how to create and add the Github key. It also shows it for Gitlab, but we only need Github. Or, if you already have a github key and still getting the "fatal: could not read from remote repository" error, you may only need to add your Github key to the `ssh-agent`. Step 1 in the link I provided shows how to do that.

3. Do the following things:

* comment out the line [here](https://github.com/cms-l1-dpg/Phase2-L1Nano/blob/main/python/l1tPh2Nanotables_cff.py#L403) in the local repo you just cloned in step 2. 
* add the following line to `PhysicsTools/L1Nano/python/l1tPh2Nanotables_cff.py`:
`llpTagScore = ExtVar(cms.InputTag("l1tTOoLLiPProducerCorrectedEmulator", "L1PFLLPJets"),float, doc="NN LLP Tag score")`,
you can place it under the existing
`btagScore = ...`


4. As instructed in the repo suggested in step 2, we have to do `cmsRun`. First, I did `cd /uscms/home/rmarroqu/nobackup/CMS_L1Trigger_Analysis/work/CMSSW_14_0_0_pre3/src/PhysicsTools/L1Nano` (this the folder `\L1Nano` in the repo you cloned above). Then, `cmsRun test/v33_rerunL1wTT_cfg.py`.

5. Run the `cmsDriver` Command inside a `.sh` file as usual, `bash cmsDriver.sh`, for example. The contents in that `.sh` file will be:

as Daniel suggested: 
```bash
cmsDriver.py step1 \
--conditions 131X_mcRun4_realistic_v9 \
-n 2 \ 
--era Phase2C17I13M9 \
--eventcontent NANOAOD \
-s RAW2DIGI,L1,L1TrackTrigger,L1P2GT,USER:PhysicsTools/L1Nano/l1tPh2Nano_cff.l1tPh2NanoTask \
--datatier GEN-SIM-DIGI-RAW-MINIAOD \
--fileout file:test.root \
--customise "SLHCUpgradeSimulations/Configuration/aging.customise_aging_1000,Configuration/DataProcessing/Utils.addMonitoring,L1Trigger/Configuration/customisePhase2.addHcalTriggerPrimitives,L1Trigger/Configuration/customisePhase2FEVTDEBUGHLT.customisePhase2FEVTDEBUGHLT,L1Trigger/Configuration/customisePhase2TTNoMC.customisePhase2TTNoMC,PhysicsTools/L1Nano/l1tPh2Nano_cff.addFullPh2L1Nano" \
--geometry Extended2026D95 \
--nThreads 8 \ 
--filein "root://cmsxrootd.fnal.gov///store/mc/Phase2Spring23DIGIRECOMiniAOD/MinBias_TuneCP5_14TeV-pythia8/GEN-SIM-DIGI-RAW-MINIAOD/PU200_L1TFix_Trk1GeV_131X_mcRun4_realistic_v9_ext1-v2/80000/0061cc5d-056b-41f5-ba7b-aada40915e3f.root" \
--mc \
--inputCommands='keep *, drop l1tPFJets_*_*_*' --outputCommands='drop l1tPFJets_*_*_*' \
--no_exec
```
***Note:*** You will have to validate your grid certificate in order to access the input file via `xrootd`.

6. This will produce the file `step1_RAW2DIGI_L1_L1TrackTrigger_L1P2GT_USER.py`, which can be run by running `cmsRun step1_RAW2DIGI_L1_L1TrackTrigger_L1P2GT_USER.py` with `cmsenv`. After running, you will have the `test.root` we need to use to get the `LLPScores`.
You can check that everything looks good on the `test.root` file via `edmDumpEventContent test.root > out.txt` and opening `out.txt`.

### step 6 via CRAB jobs:

 Use the `.py` file [here](https://gist.github.com/Brainz22/69cf0c8602e6f3eabbfcea860f60c7f0) to submit a job to produce the NANOAOD file needed for 1000 events using the `MinBias` dataset, for example. To submit this, we will need to run:

```bash
crab submit -c CRAB_L1Nano_Minbias.py
```
where `CRAB_L1Nano_Minbias.py` has he crab job specifications. 
The command `crab status -d crab_projects/crab_ucsd_MinBias` allows me to check the status of the CRAB job.

We can check that a Tier server does exist by doing for example: `crab checkwrite --site=T3_US_FNALLPC` on LPC. I am running into permission issues, which might be related to my LPC and CERN grid Certificate being different...

### Producing Rates via [Phase2-L1MenuTools](https://github.com/cms-l1-dpg/Phase2-L1MenuTools/tree/main):

1. Follow the setup instructions. Basically, we need a separate environment with python 3.11, `git clone` the repo, and run `pip install -e .` to install it. I am creating my environment as follows:
```bash
mamba create --name <name> python=3.11
```
***Note:*** I was running into issues because `cmsenv` (I think). It fixed things after I restarted the `ssh` connenction and ran the command above to create the environment.

3. Then inside it, go into `configs/v38nano/caching.yaml` and uncomment things that are not needed. Then, add the things for `MinBias`, for example. Right now, I have (spacing might be off): 

```yaml
V38nano:
  MinBias:
    ntuple_path: /uscms/home/rmarroqu/nobackup/cmsL1trigger_Analysis/work/CMSSW_14_0_0_pre3/src/MinBias/hadded/complete_hadd.root
    trees_branches:
      Events:
        # PV
        L1PV: [z0]
        ## EG
        # L1tkPhoton: "all" 
        # L1tkElectron: "all" 
        # L1EGbarrel: "all" 
        # L1EGendcap: "all" 
        ## MUONS
        # L1gmtTkMuon: "all" 
        # L1gmtMuon: "all"  # aka gmtMuon
        # L1gmtDispMuon: "all"
        ## TAUS
        # L1nnPuppiTau: "all" 
        # L1hpsTau: "all" 
        # L1caloTau: "all" 
        # L1nnCaloTau: "all"
        ## MET/Sums
        # L1puppiMET: [pt, phi]
        # L1puppiMLMET: [pt]
        # L1puppiJetSC4sums: [pt, phi]
        # L1puppiHistoJetSums: [pt, phi]
        # # jets
        L1puppiJetSC4: [pt, eta, phi]
        L1puppiJetSC8: [pt, eta, phi]
        L1puppiExtJetSC4: [pt, eta, phi, btagScore, llpTagScore]
        L1puppiJetHisto: [pt, eta, phi]
        L1caloJet: [pt, eta, phi]

```
3. We need to cache our objects. Run `cache_objects configs/V38nano/caching.yaml`. I was getting an error about a missing directory `cache/V38nano`. So, I just created manually via mkdir and it worked. The cached files needed for the rates will be here.

4. Make some additions to some files... Emyr sent me instructions and I will put them here. I have already added

5. To make rate plots, run `rate_plots configs/V38nano/rate_plots/jets.yaml`. Output of this command tells you where files will be saved.  


I am caching all the files overnight so that I can run `rate_plots` in the morning... remember, I will need to activate RatesV38.



-----------------------------------------------------
7. The branch with `LLPscore` function can installed as `git cms-checkout-topic -u ddiaz006:TOoLLip-integration`. This has the LLP tagger integration in cmssw.

8. The sample code where we can get the scores from the LLP tagger is attached [here](https://gist.github.com/ddiaz006/58c547c2dfc0828c4487ed7523bc14d7). Currently, jupyter notebook fails at the loop on line 23 of this code. Then if I run the whole code as in a `.py` instead, I get error:
```bash
 *** Break *** segmentation violation



===========================================================
There was a crash.
This is the entire stack trace of all threads:
===========================================================
#0  0x00007f6e687e6659 in waitpid () from /lib64/libc.so.6
#1  0x00007f6e68763f62 in do_system () from /lib64/libc.so.6
#2  0x00007f6e68764311 in system () from /lib64/libc.so.6
#3  0x00007f6e61c6480d in TUnixSystem::StackTrace() () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/lcg/root/6.30.03-723f04ba093d0553281d42c7b0f6eee1/lib/libCore.so
#4  0x00007f6e61dbc5a3 in (anonymous namespace)::TExceptionHandlerImp::HandleException(int) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/lcg/root/6.30.03-723f04ba093d0553281d42c7b0f6eee1/lib/libcppyy_backend3_9.so
#5  0x00007f6e61c64041 in TUnixSystem::DispatchSignals(ESignals) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/lcg/root/6.30.03-723f04ba093d0553281d42c7b0f6eee1/lib/libCore.so
#6  <signal handler called>
#7  0x00007f6e59354093 in fwlite::BranchMapReader::getFileVersion(TFile*) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libFWCoreFWLite.so
#8  0x00007f6e5935427f in fwlite::BranchMapReader::BranchMapReader(TFile*) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libFWCoreFWLite.so
#9  0x00007f6e3cba1e11 in fwlite::Event::Event(TFile*, bool, std::function<void (TBranch const&)>) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#10 0x00007f6e3cba257c in fwlite::ChainEvent::switchToFile(long long) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#11 0x00007f6e3cba28ef in fwlite::ChainEvent::ChainEvent(std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > const&) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#12 0x00007f6e3caba04d in ?? ()
#13 0x00007fff7c82bf80 in ?? ()
#14 0x00007fff7c82c270 in ?? ()
#15 0x00000000128b7bd0 in ?? ()
#16 0x00007fff7c82c008 in ?? ()
#17 0x0000000000000000 in ?? ()
===========================================================


The lines below might hint at the cause of the crash. If you see question
marks as part of the stack trace, try to recompile with debugging information
enabled and export CLING_DEBUG=1 environment variable before running.
You may get help by asking at the ROOT forum https://root.cern/forum
preferably using the command (.forum bug) in the ROOT prompt.
Only if you are really convinced it is a bug in ROOT then please submit a
report at https://root.cern/bugs or (preferably) using the command (.gh bug) in
the ROOT prompt. Please post the ENTIRE stack trace
from above as an attachment in addition to anything else
that might help us fixing this issue.
===========================================================
#7  0x00007f6e59354093 in fwlite::BranchMapReader::getFileVersion(TFile*) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libFWCoreFWLite.so
#8  0x00007f6e5935427f in fwlite::BranchMapReader::BranchMapReader(TFile*) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libFWCoreFWLite.so
#9  0x00007f6e3cba1e11 in fwlite::Event::Event(TFile*, bool, std::function<void (TBranch const&)>) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#10 0x00007f6e3cba257c in fwlite::ChainEvent::switchToFile(long long) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#11 0x00007f6e3cba28ef in fwlite::ChainEvent::ChainEvent(std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > const&) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#12 0x00007f6e3caba04d in ?? ()
#13 0x00007fff7c82bf80 in ?? ()
#14 0x00007fff7c82c270 in ?? ()
#15 0x00000000128b7bd0 in ?? ()
#16 0x00007fff7c82c008 in ?? ()
#17 0x0000000000000000 in ?? ()
===========================================================


 *** Break *** segmentation violation



===========================================================
There was a crash.
This is the entire stack trace of all threads:
===========================================================
#0  0x00007f6e687e6659 in waitpid () from /lib64/libc.so.6
#1  0x00007f6e68763f62 in do_system () from /lib64/libc.so.6
#2  0x00007f6e68764311 in system () from /lib64/libc.so.6
#3  0x00007f6e61c6480d in TUnixSystem::StackTrace() () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/lcg/root/6.30.03-723f04ba093d0553281d42c7b0f6eee1/lib/libCore.so
#4  0x00007f6e61dbc423 in (anonymous namespace)::TExceptionHandlerImp::HandleException(int) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/lcg/root/6.30.03-723f04ba093d0553281d42c7b0f6eee1/lib/libcppyy_backend3_9.so
#5  0x00007f6e61c64041 in TUnixSystem::DispatchSignals(ESignals) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/lcg/root/6.30.03-723f04ba093d0553281d42c7b0f6eee1/lib/libCore.so
#6  <signal handler called>
#7  0x00007f6e59354093 in fwlite::BranchMapReader::getFileVersion(TFile*) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libFWCoreFWLite.so
#8  0x00007f6e5935427f in fwlite::BranchMapReader::BranchMapReader(TFile*) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libFWCoreFWLite.so
#9  0x00007f6e3cba1e11 in fwlite::Event::Event(TFile*, bool, std::function<void (TBranch const&)>) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#10 0x00007f6e3cba257c in fwlite::ChainEvent::switchToFile(long long) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#11 0x00007f6e3cba28ef in fwlite::ChainEvent::ChainEvent(std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > const&) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#12 0x00007f6e3caba04d in ?? ()
#13 0x00007fff7c82bf80 in ?? ()
#14 0x00007fff7c82c270 in ?? ()
#15 0x00000000128b7bd0 in ?? ()
#16 0x00007fff7c82c008 in ?? ()
#17 0x0000000000000000 in ?? ()
===========================================================


The lines below might hint at the cause of the crash. If you see question
marks as part of the stack trace, try to recompile with debugging information
enabled and export CLING_DEBUG=1 environment variable before running.
You may get help by asking at the ROOT forum https://root.cern/forum
preferably using the command (.forum bug) in the ROOT prompt.
Only if you are really convinced it is a bug in ROOT then please submit a
report at https://root.cern/bugs or (preferably) using the command (.gh bug) in
the ROOT prompt. Please post the ENTIRE stack trace
from above as an attachment in addition to anything else
that might help us fixing this issue.
===========================================================
#7  0x00007f6e59354093 in fwlite::BranchMapReader::getFileVersion(TFile*) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libFWCoreFWLite.so
#8  0x00007f6e5935427f in fwlite::BranchMapReader::BranchMapReader(TFile*) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libFWCoreFWLite.so
#9  0x00007f6e3cba1e11 in fwlite::Event::Event(TFile*, bool, std::function<void (TBranch const&)>) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#10 0x00007f6e3cba257c in fwlite::ChainEvent::switchToFile(long long) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#11 0x00007f6e3cba28ef in fwlite::ChainEvent::ChainEvent(std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > > const&) () from /cvmfs/cms.cern.ch/slc7_amd64_gcc12/cms/cmssw/CMSSW_14_0_0_pre3/lib/slc7_amd64_gcc12/libDataFormatsFWLite.so
#12 0x00007f6e3caba04d in ?? ()
#13 0x00007fff7c82bf80 in ?? ()
#14 0x00007fff7c82c270 in ?? ()
#15 0x00000000128b7bd0 in ?? ()
#16 0x00007fff7c82c008 in ?? ()
#17 0x0000000000000000 in ?? ()
===========================================================
```

