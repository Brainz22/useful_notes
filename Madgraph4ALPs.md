# Instructions for ALPs Study
0. Made a virtual environment with python3 in it.

1. We need the rights to clone the repo: https://github.com/LLP-LHC/LLP-Reinterpretation/tree/main.
2. Next is: 
```bash
git clone git@github.com:LLP-LHC/LLP-Reinterpretation.git
cd LLP-Reinterpretation
./install.sh
```
As the instructions in that repo suggest. This will install `madgraph`.

3. Add executable command to your path, e.g.: `export PATH=$PATH:/afs/cern.ch/user/r/russelld/ALPs/work/LLP-Reinterpretation/MG5_aMC_v2_9_3/bin`.

4. We need put the `.dat` file and the respective model package in `madgraph_models`. So, I made a directory and put these two there, i.e.,
```bash
mkdir MADGRAPH_Test
```
and copy things there. `cp` the model wee need from `/madgraph_models/` and the card from `/madgraph_generation/`. For the `ALP_WB`, do:
 * `cp -r madgraph_models/ALP_WB MADGRAPH_Test/`
    * We might need to convert this model to python3 via opening madgraph with the command: `mg5_aMC`. Once in the madgraph terminal, use `convert model ./ALP_WB`. 
 * `cp madgraph_generation/generate_walp_1W0B_1GeV_2Jets.dat MADGRAPH_Test/generate_walp_1W0B_1GeV_2Jets.dat`

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
  * `cd ~/ALPs/work/genproduction/bin/cards`, `mkdir ALP_WB`.
  * `cd ~/ALPs/work/LLP-Reinterpretation/MADGRAPH_Test/llp_gen/madgraph_pythia/Cards`
  * `cp proc_card_mg5.dat run_card.dat ~/ALPs/work/genproductions/bin/MadGraph5_aMCatNLO/cards/ALP_WB`
  * `cd ~/ALPs/work/genproductions/bin/MadGraph5_aMCatNLO/cards/ALP_WB`
  * `mv proc_card_mg5.dat ALP_WB_proc_card.dat` and `mv run_card.dat ALP_WB_run_card.dat`.

4. Create a gridpack locally using the `./generate_gridpack.sh ALP_WB cards/ALP_WB 1nd`. See details of this command on the header of the file `generate_gridpack.sh`. This `.sh` script is found in `~/ALPs/work/genproductions/bin/MadGraph5_aMCatNLO/`. See details of this command on the header of the file `generate_gridpack.sh`.


