# Instructions for ALPs Study
### Useful notes
* `nEvents` sets the number of events to generate on our `.root` file.
* `process.maxEvents` is `<= nEvents`. Not so sure yet what it does, but set to `maxEvents=-1` to process all the events we generate.

0. Made a virtual environment with python3 in it. **Note:** Some of the paths will change depending on whether you are working on UAF or lxplus, for example.

1. We need the rights to clone the repo: https://github.com/LLP-LHC/LLP-Reinterpretation/tree/main.
2. Next is: 
```bash
git clone git@github.com:LLP-LHC/LLP-Reinterpretation.git
cd LLP-Reinterpretation
./install.sh
```
(**Note**: the link to `wget` madgraph has chanded. Change this in your `install.sh` to https://launchpad.net/mg5amcnlo/lts/2.9.x/+download/MG5_aMC_v2.9.3.tar.gz. You can also see all other madgraph versions here: https://launchpad.net/mg5amcnlo/+download.)
As the instructions in that repo suggest, this will install `madgraph`.

3. Add executable command to your path, e.g.: `export PATH=$PATH:/afs/cern.ch/user/r/russelld/ALPs/work/LLP-Reinterpretation/MG5_aMC_v2_9_3/bin`.

## Producing CMS Gridpack

1. I am starting from the directory (this will be different in UAF)
```bash
[russelld@lxplus820 work]$ pwd
/afs/cern.ch/user/r/russelld/ALPs/work
[russelld@lxplus820 work]$ ls
LLP-Reinterpretation
[russelld@lxplus820 work]$
```
Then, run `git clone git@github.com:cms-sw/genproductions.git genproductions`. This will clone the needed repo.


2. We need to generate `madgraph` events. From `LLP-interpretation`, we need use the `.dat` file and the respective model package in `LLP-Reinterpretation/madgraph_models`.
* `cd genproductions/bin/MadGraph5_aMCatNLO/`, `mkdir ALP-WORK`, `cd ALP-WORK`.
* `cp -r ~/ALPs/work/LLP-Reinterpretation/madgraph_models/ALP_WB  ~/ALPs/work/LLP-Reinterpretation/madgraph_generation/generate_walp_1W0B_1GeV_2Jets.dat .`. This copies the folder `ALP_WB` and file `generate_walp_1W0B_1GeV_2Jets.dat` to our current directory.
* We might need to convert the `ALP_WB` model to python3 via opening madgraph with the command: `mg5_aMC`. Once in the madgraph terminal, use `convert model ./ALP_WB`. Quit typing `quit`.
* Open `generate_walp_1W0B_1GeV_2Jets.dat` by going into that directory and using `vim generate_walp_1W0B_1GeV_2Jets.dat`, for example. Change the output path to `output llp_gen` .
* Run the `.dat file` with `mg5_aMC generate_walp_1W0B_1GeV_2Jets.dat`. This will create the folder `llp_gen`.
* Change to `cd llp_gen/Cards/` and change the names of process and run cards as follows:
   * `mv <name of process card_m5.dat> llp_gen_proc_card.dat`. Changes the name to `llp_gen_proc_card.dat`.
   * `mv run_card.dat llp_gen_run_card.dat`. Changes the name to `lp_gen_run_card.dat`.

3. Go back to the location `genproductions/bin/MadGraph5_aMCatNLO/`. Then, Create a gridpack locally using the `./gridpack_generation.sh llp_gen llp_gen/Cards 1nd`. See details of this command on the header of the file `gridpack_generation.sh`. It will generate a file like the following: `llp_gen_el8_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz` (UAF will have the prefix `llp_gen_slc7_amd64_gcc10`).

4. Go back to the working directory. You need to be in `ALPs/work`. Now, we need to install a CMS Software (cmssw) package. You can do that as follows:
 * Run the following commands: 
```bash
source /cvmfs/cms.cern.ch/cmsset_default.sh
export SCRAM_ARCH=el8_amd64_gcc10
#In UAF use: export SCRAM_ARCH=llp_gen_slc7_amd64_gcc10
cmsrel CMSSW_12_4_14_patch3
```   

5. Step 3 gave us the gridpack we need. Now, we need to input this into a cms `fragment.py` and run the `cmsDriver.py` command:
* Change directory to `CMSSW_12_4_14_patch3/src`. Assuming you are in `ALPs/work`
* Use the fragment in this  (it already has edits from Sie Xie). Only the change the path in line 5 for the file you generated in the previous step.
* Run `cmsenv`
* make the following directories and `cd` into them: 
   * `mkdir Configuration`, `cd Configuration`
   * `mkdir GenProduction`, `cd GenProduction`
   * `mkdir python`, `cd python` 
* Need fragment with Sie Xie edits. Copy the file content from this [link](https://gist.github.com/Brainz22/8538908efe29ab002eb1863be3db0589). Then,
   * Open a file with your terminal: `vim fragment.py`. Once in the `vim` editor, press `i` to switch to insert mode and paste the contents you copied.
   * Change the path (in line 5) and add the full path to your gridpack (file ending in `.tar.xz`).
   * Exit `vim` via `esc` to exit insert mode and `:wq` to exit and save the file content with its contents.
* Go back to `CMSSW_12_4_14_patch3/src` with `cd ../../..`
* Run `scram b -j8`. This is needed to compile any change we make to our `CMSSW_12_4_14_patch3` package.
* run cmsDriver as follows:
```bash
cmsDriver.py Configuration/Generator/python/fragment.py \
--mc \
--eventcontent NANOAODGEN \
--customise Configuration/DataProcessing/Utils.addMonitoring \
--datatier NANOAOD --conditions 124X_mcRun3_2022_realistic_v12 \
--beamspot Realistic25ns13p6TeVEarly2022Collision \
--step LHE,GEN,NANOGEN \
--geometry DB:Extended \
--era Run3
```



