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

