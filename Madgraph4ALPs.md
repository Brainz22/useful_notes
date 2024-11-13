# Instructions for ALPs Study
0. Made a virtual environment with python3 in it.

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

4. We need put the `.dat` file and the respective model package in `madgraph_models`. So, I made a directory and put these two there, i.e.,
```bash
mkdir MADGRAPH_Test
```
and copy things there. `cp` the model wee need from `/madgraph_models/` and the card from `/madgraph_generation/`. For the `ALP_WB`, do:
 * `cp -r madgraph_models/ALP_WB MADGRAPH_Test/`
    * We might need to convert this model to python3 via opening madgraph with the command: `mg5_aMC`. Once in the madgraph terminal, use `convert model ./ALP_WB`. 
 * `cp madgraph_generation/generate_walp_1W0B_1GeV_2Jets.dat MADGRAPH_Test/generate_walp_1W0B_1GeV_2Jets.dat`.
 * Open `generate_walp_1W0B_1GeV_2Jets.dat` by going into that directory and using `vim generate_walp_1W0B_1GeV_2Jets.dat`, for example. Change the output path to `output llp_gen` 

5. `cd MADGRAPH_Test` and run the `.dat file` with `mg5_aMC <file-name>.dat`.

6. After execution, the above will generate a new folder based on the output command at the end of the `.dat` file. For me, it was `llp_gen`. In this folder, we will need the `proc`(process) card and `run` card, which are also `.dat` files.


## Producing CMS Gridpack

1. I am starting from the directory 
```bash
[russelld@lxplus820 work]$ pwd
/afs/cern.ch/user/r/russelld/ALPs/work
[russelld@lxplus820 work]$ ls
LLP-Reinterpretation
[russelld@lxplus820 work]$
```
Then, run `git clone git@github.com:cms-sw/genproductions.git genproductions`. This will clone the needed repo.

2. Run `cd genproductions/bin/MadGraph5_aMCatNLO/`. At this location, we can try the example cards to run a simulation to see if things work. Run `./gridpack_generation.sh wplustest_4f_LO cards/examples/wplustest_4f_LO 1nd`. This command should run without any immediate errors.

3. The next would be to copy over the `proc` and `run` cards and create a new folder in `/cards/`, following the same format as the above: `cards/examples/wplustest_4f_LO`. Basically,
  * `cd ~/ALPs/work/genproduction/bin/cards`, `mkdir llp_gen`. This directory comes from the definition in the file `generate_walp_1W0B_1GeV_2Jets.dat` we used to produce the `llp_gen` folder in the previous section.
  * change directory (cd) to the cards location: `cd ~/ALPs/work/LLP-Reinterpretation/MADGRAPH_Test/llp_gen/madgraph_pythia/Cards`
  * Copy the cards to where they need to be in folder: `genproduction`: `cp proc_card_mg5.dat run_card.dat ~/ALPs/work/genproductions/bin/MadGraph5_aMCatNLO/cards/llp_gen`
  * change directory (cd) to where you copied the cards:  `cd ~/ALPs/work/genproductions/bin/MadGraph5_aMCatNLO/cards/llp_gen` 
  * rename the cards names by using the move (mv) command: `mv proc_card_mg5.dat llp_gen_proc_card.dat` and `mv run_card.dat llp_gen_run_card.dat`.
  * go back two directories: `cd ../..`. You should see several `.sh` files if you do `ls`.


4. Create a gridpack locally using the `./gridpack_generation.sh llp_gen llp_gen/cards 1nd`. See details of this command on the header of the file `generate_gridpack.sh`. This `.sh` script is found in `~/ALPs/work/genproductions/bin/MadGraph5_aMCatNLO/`. See details of this command on the header of the file `generate_gridpack.sh`. It will generate a file like the following: `llp_gen_el8_amd64_gcc10_CMSSW_12_4_8_tarball.tar.xz`.

5. Go back to the working directory. You need to be in `ALPs/work`. Now, we need to install a CMS Software (cmssw) package. You can do that as follows:
 * Run the following commands: 
```bash
source /cvmfs/cms.cern.ch/cmsset_default.sh
export SCRAM_ARCH=el8_amd64_gcc10
cmsrel CMSSW_12_4_14_patch3
```   

6. Step 4 gave us the gridpack we need. Now, we need to input this into a cms `fragment.py` and run the `cmsDriver.py` command:
   * Change directory to `CMSSW_12_4_14_patch3`. Assuming you are in `ALPs/work`
   * Use the fragment in this [link](https://gist.github.com/Brainz22/8538908efe29ab002eb1863be3db0589) (it already has edits from Sie Xie). Only the change the path in line 5 for the file you generated in the previous step.
   * git-add configuration package. Do `git cms-addpkg Configuration/Generator`
   * Copy the file content from the link. Open a file with your terminal: `vim fragment.py`. Once in the `vim` editor, press `i` to switch to insert mode and paste the contents you copied. Exit `vim` via `esc` to exit insert mode and `:wq` to exit and save the file content with its contents.
   * Change directory to and using `cd Configuration/Generator`.
   * Make a directory named `python`: `mkdir python` and change into it: `cd python`.
   * Bring the `fragment.py` by going back `src`, i.e. go back 3 directories: `cd ../../..` and `mv fragment.py Configuration/Generator/python`.
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



