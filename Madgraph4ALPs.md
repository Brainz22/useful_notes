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
and copy things there.

5. `cd MADGRAPH_Test` and run the `.dat file` with `mg5_aMC <file-name>.dat`.

6. After execution, the above will generate a new folder based on the output command at the end of the `.dat` file. In this folder, we will need the `proc`(process) card and `run` card, which are also `.dat` files.


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

3. The next would be to copy over the `proc` and `run` cards and create a new folder in `/cards/`, following the same format as the above `cards/examples/wplustest_4f_LO`.  


