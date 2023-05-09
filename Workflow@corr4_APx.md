(Credits: Tony Aportela)

If the goal is to run things on the APx board at Fermilab, this tutorial will show you how to get started. The steps here need to be run on the `Correlator 4` server after accessing through LPC. Access to the correlator was granted by someone at Fermilab (Christian Herwig for me). 

This tutorial is best if you want or plan to ssh authenticate with several services. Apparently, you just can't use github and gitlab at the same time. You need a config file and some other stuff. Wish someone told me.

The operating thing to change for other services is "gitlab" and "github". The emails are just labels and don't actually matter. It's just good practice to use the email associated with those services.

1. Generate SSH key pairs for both services:

-   Open a terminal on your local machine.
-   Generate an SSH key pairs for both services by running the following commands:

<pre>
ssh-keygen -t ed25519 -C "rmarroquinsolares@gmail.com" -f ~/.ssh/github
</pre>

<pre>
ssh-keygen -t ed25519 -C "russelld@cern.ch" -f ~/.ssh/gitlab
</pre>

2. Add the SSH keys to the SSH agent:

* Start the SSH agent by running:
<pre>
eval "$(ssh-agent -s)"
</pre>

* add the ssh keys to the agent:
<pre>
ssh-add ~/.ssh/github
</pre>

<pre>
ssh-add ~/.ssh/gitlab
</pre>

3. Create a `config` file to manage multiple SSH keys:

 * In the terminal, create and edit the `config` file by running:
<pre>
nano ~/.ssh/config
</pre>

  * then add the following content to the file:
<pre>
# GitHub account
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github

# CERN GitLab account
Host gitlab.cern.ch
  HostName gitlab.cern.ch
  User git
  IdentityFile ~/.ssh/gitlab
</pre>

  * Save and close the file by pressing `Ctrl + X`, then `Y`, and then `Enter`.

4. Add the public SSH keys to your GitHub and GitLab accounts:
-   Copy the public SSH key to your clipboard by copying the results of these command.
- For github:
<pre>
cat ~/.ssh/github.pub
</pre>

* Then add the resulting ssh key [here](https://github.com/settings/ssh/new).

* similarly for gitlab:
<pre>
cat ~/.ssh/gitlab.pub
</pre>
* Add ssh key under preferences and under ssh keys.

5. Finally, test the ssh connections
* For github:
<pre>
ssh -T git@github.com
</pre>

* for gitlab:
<pre>
ssh -T git@gitlab.cern.ch
</pre>

6. If no errors at this point, we have to clone Christian's Gitlab branch by doing the following: 

<pre>
git clone -b APxFWSv2 --single-branch --recurse-submodules ssh://git@gitlab.cern.ch:7999/cms-cactus/phase2/firmware/correlator-layer2.git
mkdir correlator-layer2/build
cd correlator-layer2/met/apx
</pre>

7. We need to source `vitis` through a file in Christian's directory. This needs to be done for `Vitis` commands to work. Do the following:
<pre>
source /home/therwig/setup_vitis_20222.sh
</pre>

8. We need to build the framework for the firmware (I think those are the right words). We can do:
<pre>
make sources
# Vivado project will now be available at build/top
make syn
make bit
</pre>

Or, alternatively, we can do it all in one by run (I did not try this):

<pre>
make
</pre>

9. We can simulate with the following:
<pre>
make xsim
</pre>

10. Go to the `correlator-common` directory. For me, it is located here: `/home/rmarroqu/correlator-layer2/submodules/correlator-common`.

11. Set up CMSSW firmware by running these two commands two commands :
<pre>
source /cvmfs/cms.cern.ch/cmsset_default.sh
./utils/setup_cmssw.sh -run CMSSW_12_3_0_pre4 p2l1pfp:L1PF_12_3_X lict-125x-v1.15
</pre>
Once, we have the correct version of CMSSW, run: `export CMSSW_VERSION=CMSSW_12_3_0_pre4`.

12. Go to go to `correlator-common/jetmet/seededcone` (it’s in the submodules directory in `APx`). We should see the files here: https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-common/-/tree/master-125x/jetmet/seededcone. Run:
`source /home/therwig/19p2_setup_vivado.sh`
Then, there are several tcl files, which we need to run with `vivado_hls <the tcl file>` (you don’t really need to run the run_Sim.tcl if you don’t really need to test).
Note: we might need to run the three commands below every time we exit the correlator: 
<pre>
source /cvmfs/cms.cern.ch/cmsset_default.sh
source /home/therwig/19p2_setup_vivado.sh
export CMSSW_VERSION=CMSSW_12_3_0_pre4
</pre>

