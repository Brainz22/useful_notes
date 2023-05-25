1. Access is need to CMS-Cactus by asking Tom Williams (suggested by Christian Herwig). Tom's CERN contact info can be found on the CERN Directory. Then, Christian asked to get me access to other things by contacting someone at Fermilab. So, you need to ask for the different access authorizations in order to clone the git repositories that will be need along the work here.

2. If you haven't, follow the Version 1 tutorial to set up all ssh keys to access (steps 1-5 I believe).

3. On correlator 4, clone Duc's Apx V2 branch following the isntructions found on this link: 
<pre>
https://gitlab.cern.ch/cms-cactus/phase2/firmware/correlator-layer2/-/tree/JetID_APxV2/JetID/apx
</pre>

4. Go to the `correlator-common` directory. For me, it is located here: `/home/rmarroqu/correlator-layer2/submodules/correlator-common`.

5. Set up CMSSW firmware by running these two commands two commands :
<pre>
source /cvmfs/cms.cern.ch/cmsset_default.sh
./utils/setup_cmssw.sh -run CMSSW_12_3_0_pre4 p2l1pfp:L1PF_12_3_X lict-125x-v1.15
</pre>
Once, we have the correct version of CMSSW, run: `export CMSSW_VERSION=CMSSW_12_3_0_pre4`.

6. Go to go to `correlator-common/jetmet/seededcone` (it’s in the submodules directory in `APx`) Run:
`source /home/therwig/19p2_setup_vivado.sh`
Then, there are several tcl files, which we need to run with `vivado_hls <the tcl file>` (you don’t really need to run the run_Sim.tcl if you don’t really need to test).

I ran all of them by employing a bash for-loop embedded in a `.sh`, which was written as follows:

<pre>
command="vivado_hls"

# Iterate over the files using a for loop
for file in /home/rmarroqu/APXv2/correlator-layer2/submodules/correlator-common/jetmet/seededcone/*.tcl; do
    # Run the command on each file
    $command "$file"
done
</pre>

Then, this file can be for run by doing `bash for.sh`, for example.

Note: we might need to run the three commands below every time we exit the correlator: 

<pre>
source /cvmfs/cms.cern.ch/cmsset_default.sh
source /home/therwig/19p2_setup_vivado.sh
export CMSSW_VERSION=CMSSW_12_3_0_pre4
</pre>

7. Once that is done, `cd` to the `apx` directory. I can get there as follows: `cd /home/rmarroqu/APXv2/correlator-layer2/JetID/apx'.

8. Run `make`. It might ask for a `build` directory. If it does, you can follow the instructions the error provides and try `make` again.

9. Test `make xsim`
