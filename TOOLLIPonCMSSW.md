# Method to Integrate a TOOLLIP Version on CMSSW

I am following a combination of the (a) `Jet Tagging CMSSW Recipe` [instructions](https://codimd.web.cern.ch/pB3K4fFiSrmblUHFAMYoxA?view) and (b) `AXOV5 Emulator Instructions` [instructions](https://codimd.web.cern.ch/s/-6VCkWSpE#New-Model-Testing).

1. (a) Start with a CMSSW version 
   ```bash
   export SCRAM_ARCH=el8_amd64_gcc12
   source /cvmfs/cms.cern.ch/cmsset_default.sh
   cmsrel CMSSW_15_1_0_pre1
   cd CMSSW_15_1_0_pre1/src
   cmsenv
   git cms-init # git asked me to create a fork before this
   ```
2. (b) Clone the respective packages we need from CMSSW
   ```bash
   git cms-addpkg L1Trigger/Phase2L1ParticleFlow
   git cms-addpkg L1Trigger/Configuration
   git cms-addpkg DataFormats

   git remote set-url origin https://github.com/Brainz22/cmssw.git # I switched to my fork
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

8. We need to tell CMSSW to use the `TOOLLIP` version we want. We can check the files that were changed during the first [pull-request](https://github.com/cms-sw/cmssw/pull/43638/files) to integrate `TOoLLiP/TOoLLiP_v1` in CMSSW. I only see two places we need to make changes inside `CMSSW_15_1_0_pre1/src/L1Trigger/Phase2L1ParticleFlow/plugins/TOoLLiPProducer.cc`: 1) where `TOoLLiP_v1` appears and 2) where we point it to the correct `.so` file (when a version has been pull requested, this 2nd. part is not needed, according to Melissa). Find the function shown below and change to `TOoLLiP_v2` if that's the correct folder (subsequent lines might also need changes)
   ```c++
   loader(hls4mlEmulator::ModelLoader(cfg.getParameter<string>("/home/users/russelld/TOOLLIP_TESTS/work/CMSSW_15_1_0_pre1/src/TOoLLiP/TOoLLiP_v2/TOoLLiP_v2.so"))) {
   .
   .
   .
   void TOoLLiPProducer::fillDescriptions(edm::ConfigurationDescriptions& descriptions) {
     edm::ParameterSetDescription desc;
     desc.add<edm::InputTag>("jets", edm::InputTag("scPFL1Puppi"));
     desc.add<bool>("useRawPt", true);
     desc.add<std::string>("TOoLLiPVersion", std::string("TOoLLiP_v2"));
     desc.add<std::string>("NNInput", "input:0");
     desc.add<std::string>("NNOutput", "sequential/dense_2/Sigmoid");
   ```
   






   
   
