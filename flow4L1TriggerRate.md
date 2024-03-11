I ran the `cmsDriver` as:
```bash
cmsDriver.py step1 --conditions 131X_mcRun4_realistic_v9 -n 2 --era Phase2C17I13M9 --eventcontent NANOAOD -s RAW2DIGI,L1,L1TrackTrigger,L1P2GT,USER:PhysicsTools/L1Nano/l1tPh2Nano_cff.l1tPh2NanoTask --datatier GEN-SIM-DIGI-RAW-MINIAOD --fileout file:test.root --customise SLHCUpgradeSimulations/Configuration/aging.customise_aging_1000,Configuration/DataProcessing/Utils.addMonitoring,L1Trigger/Configuration/customisePhase2.addHcalTriggerPrimitives,L1Trigger/Configuration/customisePhase2FEVTDEBUGHLT.customisePhase2FEVTDEBUGHLT,L1Trigger/Configuration/customisePhase2TTNoMC.customisePhase2TTNoMC,PhysicsTools/L1Nano/l1tPh2Nano_cff.addFullPh2L1Nano --geometry Extended2026D95 --nThreads 8 --filein root://cmsxrootd.fnal.gov///store/mc/Phase2Spring23DIGIRECOMiniAOD/MinBias_TuneCP5_14TeV-pythia8/GEN-SIM-DIGI-RAW-MINIAOD/PU200_L1TFix_Trk1GeV_131X_mcRun4_realistic_v9_ext1-v2/80000/0061cc5d-056b-41f5-ba7b-aada40915e3f.root --mc --inputCommands=keep
```
The flags below were at the very, and I left them because they raised error:
`*, drop l1tPFJets_*_*_* --outputCommands=drop l1tPFJets_*_*_*ls`.
