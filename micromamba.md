1. Remove all current conda files following the steps here: https://github.com/conda-forge/miniforge#mambaforge. Make sure sure `.condarc` and `.conda` are deleted, as well. Then, you can check your `.bashrc` or `.bash_profile` to make sure no conda lines are in there.
2. Restart terminal session.
3. Check your machine type using `uname -m` (from chatgpt) and choose the commands from the [documentation](https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html) based on that. My commands for correlator 4 were:
```bash
cd ~/
curl -Ls https://micro.mamba.pm/api/micromamba/linux-64/latest | tar -xvj bin/micromamba
./bin/micromamba shell init -s bash -r ~/micromamba
source ~/.bashrc
micromamba info
micromamba activate #check that it activates the base environment
micromamba deactivate
```
