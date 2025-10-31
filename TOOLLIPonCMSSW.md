# Method to Integrate and Test a TOOLLIP Version on CMSSW

### Testing TOoLLiP_v2

I am following a combination of the (a) `Jet Tagging CMSSW Recipe` [instructions](https://codimd.web.cern.ch/pB3K4fFiSrmblUHFAMYoxA?view) and (b) `AXOV5 Emulator Instructions` [instructions](https://codimd.web.cern.ch/s/-6VCkWSpE#New-Model-Testing).

1. (a) Start with a CMSSW version 
   ```bash
   export SCRAM_ARCH=el8_amd64_gcc13
   source /cvmfs/cms.cern.ch/cmsset_default.sh
   cmsrel CMSSW_16_0_0_pre1
   cd CMSSW_16_0_0_pre1/src
   cmsenv
   git cms-init # git asked me to create a fork before this
   ```
   **Note:** I had started with CMSSW_15_1_0_pre1, following instructions (a). However, this CMSSW release does not have the folder `DPGAnalaysis/Phase2L1TNanoAOD`, which I needed to produce minBias jets. The next best version that worked was CMSSW_16_0_0_pre1.
2. (b) Clone the respective packages we need from CMSSW
   ```bash
   git cms-addpkg L1Trigger/Phase2L1ParticleFlow
   git cms-addpkg L1Trigger/Configuration
   git cms-addpkg DataFormats
   git cms-addpkg DPGAnalysis

   git remote set-url origin https://github.com/Brainz22/cmssw.git # I switched to my fork
   git remote set-url my-cmssw git@github.com:Brainz22/cmssw.git #to keep both fetch and push origins with ssh keys
   git remote -v #check the push and fetch origins
   git remote remove official-cmssw #if official cmssw origing still appeared
   
   
   #build CMSSW release (8 is the number of threads)
   scram b -j 8
   ```


3. When adding a new model like `TOoLLiP_v2`, we need to make changes to some files in the repo. A commit showing all the changes with [updates](https://github.com/cms-hls4ml/TOoLLiP/commit/6064629a002391a6822791513f8610e2d66747ff) is in the link.

4. (a) Get `hls4ml` emulator extras needed for building the jet tagger emulator
   ```bash
   git clone https://github.com/cms-hls4ml/hls4mlEmulatorExtras.git 
   cd hls4mlEmulatorExtras 
   git checkout -b v1.1.3 tags/v1.1.3
   make install
   cd ..
   ```
5. (a) Clone HLS libraries for building jet tagger emulator
   ```bash
   git clone --quiet https://github.com/Xilinx/HLS_arbitrary_Precision_Types.git hls
   ```
6. (b) Clone `TOOLLIP` emulator
   ```bash
   git clone git@github.com:cms-hls4ml/TOoLLiP.git
   ```

7. (b,a) Run make to create binaries (.so files)
   ```bash
   cd TOoLLiP
   make install
   cd ..
   ```

8. We need to tell CMSSW to use the `TOOLLIP` version we want. We can check the files that were changed during the first [pull-request](https://github.com/cms-sw/cmssw/pull/43638/files) to integrate `TOoLLiP/TOoLLiP_v1` in CMSSW. I only see two places we need to make changes inside `CMSSW_15_1_0_pre1/src/L1Trigger/Phase2L1ParticleFlow/plugins/TOoLLiPProducer.cc`: 1) where `TOoLLiP_v1` appears and 2) where we point it to the correct folder with the `.so` file. Find the function shown below and change to `TOoLLiP_v2` if that's the correct folder (subsequent lines might also need changes)
   ```c++
   loader(hls4mlEmulator::ModelLoader(cfg.getParameter<string>(""TOoLLiPVersion""))) {
   .
   .
   .
   void TOoLLiPProducer::fillDescriptions(edm::ConfigurationDescriptions& descriptions) {
     edm::ParameterSetDescription desc;
     desc.add<edm::InputTag>("jets", edm::InputTag("scPFL1Puppi"));
     desc.add<bool>("useRawPt", true);
     desc.add<std::string>("TOoLLiPVersion", std::string("/home/users/russelld/TOOLLIP_TESTS/cmssw-tests/CMSSW_16_0_0_pre1/src/TOoLLiP/TOoLLiP_v2/TOoLLiP_v2"));
     desc.add<std::string>("NNInput", "input:0");
     desc.add<std::string>("NNOutput", "sequential/dense_2/Sigmoid");
   ```
9. Run `scram b -j 8` compile and check that no errors arise. 

### Integrating a new TOoLLiP version, e.g. ![TOoLLiP_v3](https://github.com/cms-hls4ml/TOoLLiP/tree/main/TOoLLiP_v3)

1. Create a fork of the `TOoLLiP` repo linked on the heading heading. Then, `git clone <this fork>`.
   
2. Copy any of other folder and name it `TOoLLiP_v3`. Inside, change lines with `_v2` (for example) to `_v3` to accomodate for new version.

3. Convert a trained and tested model to HLS using this ![notebook](https://github.com/Brainz22/L1LLPJetTagger/blob/2590070869380e8bf9078abc7789dc979044a344/qkL1JetTagModel_hls_config.ipynb) (need to update my commit in TOoLLiP). Make sure you add the emulation commands in `hls4ml.converters.convert_from_keras_model(...)`.

4. After running the conversion script on step 3, look for the folder `firmware` in the `output_dir` you specified.


   
# Testing TOoLLiP_v3

1. Start with a CMSSW version and add necessary submodules.
   ```bash
   export SCRAM_ARCH=el8_amd64_gcc13
   export TOOLLIP_PATH=$PWD/TOoLLiP/TOoLLiP_v3
   source /cvmfs/cms.cern.ch/cmsset_default.sh
   cmsrel CMSSW_16_0_0_pre1
   cd CMSSW_16_0_0_pre1/src
   cmsenv
   #git cms-init # git asked me to create a fork before this
   git cms-checkout-topic -u Brainz22:from-CMSSW_16_0_0_pre1
   git cms-addpkg L1Trigger/Configuration
   git cms-addpkg DataFormats
   git cms-addpkg DPGAnalysis
   ```

2. Get `hls4ml` emulator extras needed for building the jet tagger emulator
   ```bash
   git clone https://github.com/cms-hls4ml/hls4mlEmulatorExtras.git 
   cd hls4mlEmulatorExtras 
   git checkout -b v1.1.3 tags/v1.1.3
   make install
   cd ..
   ```

3. (a) Clone HLS libraries for building jet tagger emulator
   ```bash
   git clone --quiet https://github.com/Xilinx/HLS_arbitrary_Precision_Types.git hls
   ```
4. (b) Clone `TOOLLIP` emulator, make binaries, and give it a `TOOLLIP_PATH` that points to `.so` file to be used in the `toollip producer`.
   ```bash
   git clone git@github.com:cms-hls4ml/TOoLLiP.git
   cd TOoLLiP
   make install
   export TOOLLIP_PATH=$PWD/TOoLLiP_v3
   cd ..
   ```
5. Complile these changes in `src` with `scram b -j 8`.

### TOoLLiP_v3: Producing minBias Jets

6. In a `.sh` file, save the following `cmsDriver` command for 10 events:
   ```bash
   cmsDriver.py -s L1,L1TrackTrigger,L1P2GT,NANO:@Phase2L1DPGwithGen \
   --conditions auto:phase2_realistic_T33 \
   --geometry ExtendedRun4D110 \
   --era Phase2C17I13M9 \
   --eventcontent NANOAOD \
   --datatier GEN-SIM-DIGI-RAW-MINIAOD \
   --customise SLHCUpgradeSimulations/Configuration/aging.customise_aging_1000,Configuration/DataProcessing/Utils.addMonitoring,L1Trigger/Configuration/customisePhase2TTOn110.customisePhase2TTOn110 \
   --filein root://cmsxrootd.fnal.gov///store/mc/Phase2Spring24DIGIRECOMiniAOD/MinBias_TuneCP5_14TeV-pythia8/GEN-SIM-DIGI-RAW-MINIAOD/PU140_Trk1GeV_140X_mcRun4_realistic_v4-v1/120000/00a8a3a7-388d-488a-a184-ac1725eacce9.root \
   --fileout file:output_Phase2_L1T.root \
   --python_filename rerunL1_cfg.py \
   --inputCommands="keep *, drop l1tPFJets_*_*_*, drop l1tTrackerMuons_l1tTkMuonsGmt*_*_HLT" \
   --mc \
   -n 10 --nThreads 4 --no_exec
   ```
This will produce the config file `rerunL1_cfg.py`.

7_A. Run `cmsRun rerunL1_cfg.py` to produce `output_Phase2_L1T.root`. Check that one of the branches is `L1puppiExtJetSC4_llpTagScore`.

7_B. Run a CRAB job. I am using the crab file [here](https://gist.github.com/Brainz22/69cf0c8602e6f3eabbfcea860f60c7f0).
   ```bash
   mkdir -p CRABjobs
   cd CRABjobs
   cp ../rerunL1_cfg.py . #config needs to be in same dir
   ```
   * Create a `CRAB.py` file with the contents from the file in the link.
   * submit job as: `crab submit -c CRAB.py`
   * Note that it might complain about numCores (in `CRAB.py`) and numberOfThreads (in `rerunL1_cfg.py`). It asked me to match them a couple of times.





   
   
