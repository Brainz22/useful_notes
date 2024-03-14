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

3. As instructed in the repo suggested in step 2, we have to do `cmsRun`. First, I did `cd /uscms/home/rmarroqu/nobackup/CMS_L1Trigger_Analysis/work/CMSSW_14_0_0_pre3/src/PhysicsTools/L1Nano` (this the folder `\L1Nano` in the repo you cloned above). Then, `cmsRun test/v33_rerunL1wTT_cfg.py`.

4. Run the `cmsDriver` Command as follows:
```bash
cmsDriver.py step1 --conditions 131X_mcRun4_realistic_v9 -n 2 --era Phase2C17I13M9 --eventcontent NANOAOD -s RAW2DIGI,L1,L1TrackTrigger,L1P2GT,USER:PhysicsTools/L1Nano/l1tPh2Nano_cff.l1tPh2NanoTask --datatier GEN-SIM-DIGI-RAW-MINIAOD --fileout file:test.root --customise "SLHCUpgradeSimulations/Configuration/aging.customise_aging_1000,Configuration/DataProcessing/Utils.addMonitoring,L1Trigger/Configuration/customisePhase2.addHcalTriggerPrimitives,L1Trigger/Configuration/customisePhase2FEVTDEBUGHLT.customisePhase2FEVTDEBUGHLT,L1Trigger/Configuration/customisePhase2TTNoMC.customisePhase2TTNoMC,PhysicsTools/L1Nano/l1tPh2Nano_cff.addFullPh2L1Nano" --geometry Extended2026D95 --no_exec --nThreads 8 --filein root://cmsxrootd.fnal.gov///store/mc/Phase2Spring23DIGIRECOMiniAOD/MinBias_TuneCP5_14TeV-pythia8/GEN-SIM-DIGI-RAW-MINIAOD/PU200_L1TFix_Trk1GeV_131X_mcRun4_realistic_v9_ext1-v2/80000/0061cc5d-056b-41f5-ba7b-aada40915e3f.root --mc --inputCommands="keep *, drop l1tPFJets_*_*_* --outputCommands=drop l1tPFJets_*_*_*" --no_exec
```
This command raises the error: 
```bash
----- Begin Fatal Exception 14-Mar-2024 17:28:40 CDT-----------------------
An exception of category 'ProductNotFound' occurred while
   [0] Processing  Event run: 1 lumi: 2610 event: 770889 stream: 1
   [1] Running path 'NANOAODoutput_step'
   [2] Prefetching for module NanoAODOutputModule/'NANOAODoutput'
   [3] Calling method for module SimpleCandidateFlatTableProducer/'hpsTauTable'
Exception Message:
Principal::getByToken: Found zero products matching all criteria
Looking for a container with elements of type: reco::Candidate
Looking for module label: l1HPSPFTauEmuProducer
Looking for productInstanceName: HPSTaus

   Additional Info:
      [a] If you wish to continue processing events after a ProductNotFound exception,
add "TryToContinue = cms.untracked.vstring('ProductNotFound')" to the "options" PSet in the configuration.

----- End Fatal Exception -------------------------------------------------
----- Begin Fatal Exception 14-Mar-2024 17:28:40 CDT-----------------------
An exception of category 'ProductNotFound' occurred while
   [0] Processing  Event run: 1 lumi: 2610 event: 770890 stream: 2
   [1] Running path 'NANOAODoutput_step'
   [2] Prefetching for module NanoAODOutputModule/'NANOAODoutput'
   [3] Calling method for module SimpleCandidateFlatTableProducer/'hpsTauTable'
Exception Message:
Principal::getByToken: Found zero products matching all criteria
Looking for a container with elements of type: reco::Candidate
Looking for module label: l1HPSPFTauEmuProducer
Looking for productInstanceName: HPSTaus

   Additional Info:
      [a] If you wish to continue processing events after a ProductNotFound exception,
add "TryToContinue = cms.untracked.vstring('ProductNotFound')" to the "options" PSet in the configuration.

----- End Fatal Exception -------------------------------------------------
14-Mar-2024 17:28:40 CDT  Closed file root://cmsxrootd.fnal.gov///store/mc/Phase2Spring23DIGIRECOMiniAOD/MinBias_TuneCP5_14TeV-pythia8/GEN-SIM-DIGI-RAW-MINIAOD/PU200_L1TFix_Trk1GeV_131X_mcRun4_realistic_v9_ext1-v2/80000/0061cc5d-056b-41f5-ba7b-aada40915e3f.root
TimeReport> Time report complete in 177.784 seconds
 Time Summary: 
 - Min event:   21.2449
 - Max event:   21.6435
 - Avg event:   21.4442
 - Total loop:  121.588
 - Total init:  56.1953
 - Total job:   177.784
 - Total EventSetup: 34.9198
 - Total non-module: 733.02
 Event Throughput: 0.0164489 ev/s
 CPU Summary: 
 - Total loop:     71.9775
 - Total init:     43.2373
 - Total extra:    0
 - Total children: 0.134905
 - Total job:      115.216
 Processing Summary: 
 - Number of Events:  2
 - Number of Global Begin Lumi Calls:  1
 - Number of Global Begin Run Calls: 1
```
Because the above did not work. However, using the command on the [twiki](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideL1TPhase2Instructions#Recipe_for_phase2_l1t_1400pre3_v2) works: 
```bash
cmsDriver.py step1 --conditions 131X_mcRun4_realistic_v9 -n 2 --era Phase2C17I13M9 --eventcontent FEVTDEBUGHLT  RAW2DIGI,L1,L1TrackTrigger,L1P2GT --datatier GEN-SIM-DIGI-RAW-MINIAOD --fileout file:test.root --customise "SLHCUpgradeSimulations/Configuration/aging.customise_aging_1000,Configuration/DataProcessing/Utils.addMonitoring,L1Trigger/Configuration/customisePhase2.addHcalTriggerPrimitives,L1Trigger/Configuration/customisePhase2FEVTDEBUGHLT.customisePhase2FEVTDEBUGHLT,L1Trigger/Configuration/customisePhase2TTNoMC.customisePhase2TTNoMC" --geometry Extended2026D95 --nThreads 8 --filein "root://cmsxrootd.fnal.gov///store/mc/Phase2Spring23DIGIRECOMiniAOD/MinBias_TuneCP5_14TeV-pythia8/GEN-SIM-DIGI-RAW-MINIAOD/PU200_L1TFix_Trk1GeV_131X_mcRun4_realistic_v9_ext1-v2/80000/0061cc5d-056b-41f5-ba7b-aada40915e3f.root" --mc --inputCommands='keep *, drop l1tPFJets_*_*_*, drop l1tTrackerMuons_l1tTkMuonsGmt_*_*' --outputCommands="drop l1tPFJets_*_*_*, drop l1tTrackerMuons_l1tTkMuonsGmt_*_*"
```
***Note:*** You will have to validate your grid certificate in order to access the input file via `xrootd`.

5. This will produce the file `step1_RAW2DIGI_L1_L1TrackTrigger_L1P2GT_USER.py`, which can be run by running `cmsRun step1_RAW2DIGI_L1_L1TrackTrigger_L1P2GT_USER.py` with `cmsenv`. After running, you will have the `test.root` we need to use to get the `LLPScores`.
You can check that everything looks good on the `test.root` file via `edmDumpEventContent test.root > out.txt` and opening `out.txt`.

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

