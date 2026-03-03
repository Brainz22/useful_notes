# Submitting FastPUPPI performance ntuples with HTCondor at CERN

### Prerequisites

* CMSSW environment set up and cmsenv sourced.
* The [cerminar/submission](https://github.com/CMS-L1T-Jet-Tagging/submission) repository cloned into your CMSSW src/ directory.
* Input files accessible on EOS.
* An output directory you have write access to (e.g. CERN box: `/eos/user/r/<initial>/<username>/`).

### 1. Create the output directory
`mkdir -p /eos/user/r/russelld/fp_ntuples`.

### 2. Write a YAML config
Model it after `submit_FP_131X.yaml`. Key fields:

```bash
Common:
  mode: FP
  name: fp
  tasks:
    - NuGunAllEta_PU200_151Xv0
  cmssw_config: ../FastPUPPI/NtupleProducer/python/runPerformanceNTuple.py
  version: v151Xv1
  output_dir_base: /eos/user/r/russelld/fp_ntuples
  ncpu: 1
  output_file_name: perfNano.root

NuGunAllEta_PU200_151Xv0:
  input_directory: /eos/cms/store/cmst3/group/l1tr/FastPUPPI/15_1_X/fpinputs_140X/v1/MinBias_TuneCP5_14TeV-pythia8/NuGunAllEta_PU200_151Xv0/250910_165617/0000/
  crab: False
  splitting_mode: file_based
  splitting_granularity: 1   # files per job
  max_njobs: 1 # test a single job. Remove this for all jobs.
  job_flavor: longlunch      # 2 hour wall time
  max_events: -1
```
Use `mode: FP` (not `NTP`) for `runPerformanceNTuple.py` — it uses the correct job customization template that sets `process.outnano.fileName`. Use `input_directory` (not `input_dataset`) since the files are pre-processed FastPUPPI inputs on EOS, not registered in DAS. Use `crab: False` for the same reason.

Output files are saved as `perfNano_<ClusterID>_<ProcID>.root` under:
`<output_dir_base>/<task_name>/FP/<version>/`

### 3. Apply two bug fixes to `submit.py`
These are needed when using a non-`/eos/cms/` output path and no inline_customize:

Fix 1 — guard CRAB-only parameter (~line 269):

```bash
# before:
params['TEMPL_CRABOUTDIR'] = task_conf.output_dir_base.split('/eos/cms')[1]...
# after:
if task_conf.crab:
    params['TEMPL_CRABOUTDIR'] = task_conf.output_dir_base.split('/eos/cms')[1]...
```
Fix 2 — default TEMPL_CUSTOMIZE to empty string (~line 271):

```bash
if hasattr(task_conf, 'inline_customize'):
    params['TEMPL_CUSTOMIZE'] = '\n'.join(task_conf.inline_customize)
else:
    params['TEMPL_CUSTOMIZE'] = ''
```

### 4. Create, submit, and monitor

**Note:** Under `src`, I had to `mkdir data` and move the file `TOoLLiP_v3.so` in there to make it work.

`cd src/submission` and delete the sandbox if you submitted a previous job using the same `.yaml`: `rm fp/v151Xv1/sandbox.tgz`

1. Create job configs and sandbox
`python3 submit.py -f submit_FP_151X_MinBias.yaml --create`

2. Submit to HTCondor
`python3 submit.py -f submit_FP_151X_MinBias.yaml --submit`

3. Monitor
`condor_q $USER`

4. Count output files as they arrive
`ls /eos/user/r/russelld/fp_ntuples/NuGunAllEta_PU200_151Xv0/FP/v151Xv1/ | wc -l`
