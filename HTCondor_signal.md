# Submitting Signal MC Jobs: INFP → FP Ntuples

This guide covers how to produce FastPUPPI input ROOT files (INFP step) and subsequently FastPUPPI performance ntuples (FP step) for a signal MC sample using HTCondor at CERN.

---

## Prerequisites

- Access to lxplus with a valid CERN account
- A CMSSW environment set up with the relevant packages compiled
- Input GEN-SIM-DIGI-RAW-MINIAOD files accessible on EOS (`/eos/cms/store/mc/...`)
- The `submission` [repository](https://github.com/CMS-L1T-Jet-Tagging/submission) (located under `src/submission/`)

```bash
cd /path/to/CMSSW_X_Y_Z/src
cmsenv
cd submission/
```

---

## Step 1: Identify Input Files

Check which datasets have ROOT files available:

```bash
find /eos/cms/store/mc/<campaign>/<dataset_name>* -name "*.root" 2>/dev/null \
  | sed 's|/[^/]*\.root$||' | sort -u
```

To count the number of events in a dataset directory:

```bash
python3 -c "
import ROOT, os
ROOT.gROOT.SetBatch(True)
ROOT.gErrorIgnoreLevel = ROOT.kError
d = '/eos/cms/store/mc/.../<dataset_dir>'
files = [f for f in os.listdir(d) if f.endswith('.root')]
total = 0
for f in files:
    tf = ROOT.TFile.Open(os.path.join(d, f))
    if tf and not tf.IsZombie():
        t = tf.Get('Events')
        if t:
            total += t.GetEntries()
    tf.Close()
print(f'Files: {len(files)}, Total events: {total}')
"
```

---

## Step 2: Produce FastPUPPI Input ROOT Files (INFP Step)

This step runs `runInputs151X.py` over the GEN-SIM-DIGI-RAW-MINIAOD files to produce compact FastPUPPI input ntuples.

### 2a. Create the YAML submission config

Create a file `submit_INFP_151X_<sample_name>.yaml`:

```yaml
Common:
  mode: INFP
  name: fpinputs

  tasks:
    - <TaskName>   # e.g. HiddenGluGluH_mH125_Phi15_ctau1_bbbb_PU200

  cmssw_config: ../FastPUPPI/NtupleProducer/python/runInputs151X.py
  version: v151Xv1
  output_dir_base: /eos/user/<initial>/<username>/fpinputs
  ncpu: 1
  output_file_name: inputs151X.root


<TaskName>:
  input_directory: /eos/cms/store/mc/<campaign>/<dataset>/GEN-SIM-DIGI-RAW-MINIAOD/<conditions>/<run_number>/
  crab: False
  splitting_mode: file_based
  splitting_granularity: 1
  job_flavor: longlunch
  max_events: -1
```

**Notes:**
- `splitting_granularity: 1` means one input file per job.
- `job_flavor: longlunch` gives 2 hours per job, sufficient for ~500 events.
- The TOoLLiP HLS4ML model path is set inside `runInputs151X.py` via `$CMSSW_BASE` and does not need to be configured here.

### 2b. Create and submit the jobs

```bash
python3 submit.py -f submit_INFP_151X_<sample_name>.yaml --create
python3 submit.py -f submit_INFP_151X_<sample_name>.yaml --submit
```

### 2c. Monitor jobs

```bash
condor_q $USER
```

Output files will appear in:
```
/eos/user/<initial>/<username>/fpinputs/<TaskName>/INFP/v151Xv1/
```

---

## Step 3: Produce FastPUPPI Performance Ntuples (FP Step)

This step runs `runPerformanceNTuple.py` over the INFP output files to produce the final performance ntuples.

### 3a. Create the YAML submission config

Create a file `submit_FP_151X_<sample_name>.yaml`:

```yaml
Common:
  mode: FP
  name: fp

  tasks:
    - <TaskName>   # same name as used in Step 2

  cmssw_config: ../FastPUPPI/NtupleProducer/python/runPerformanceNTuple.py
  version: v151Xv1
  output_dir_base: /eos/user/<initial>/<username>/fp_ntuples
  ncpu: 1
  output_file_name: perfNano.root


<TaskName>:
  input_directory: /eos/user/<initial>/<username>/fpinputs/<TaskName>/INFP/v151Xv1/
  crab: False
  splitting_mode: file_based
  splitting_granularity: 1
  job_flavor: longlunch
  max_events: -1
```

**Note:** The `input_directory` here points to the output of Step 2.

### 3b. Create and submit the jobs

```bash
python3 submit.py -f submit_FP_151X_<sample_name>.yaml --create
python3 submit.py -f submit_FP_151X_<sample_name>.yaml --submit
```

### 3c. Monitor jobs

```bash
condor_q $USER
```

Output files will appear in:
```
/eos/user/<initial>/<username>/fp_ntuples/<TaskName>/FP/v151Xv1/
```

---

## Troubleshooting

### Jobs complete but no output files

Check the job error log:
```bash
cat submission/fp/v151Xv1/<TaskName>/logs/job.<ClusterID>.0.err
```

Common causes:

| Error | Cause | Fix |
|-------|-------|-----|
| `'Process' object has no attribute 'TFileService'` | Missing `jobCustomization_INFP_cfg.py` template, falling back to DEFAULT | Ensure `templates/jobCustomization_INFP_cfg.py` exists with `process.out.fileName` |
| `failed to load hls4ml emulator model` | `TOOLLIP_PATH` not set on batch nodes | Set `TOoLLiPVersion` explicitly in the CMSSW config using `os.environ['CMSSW_BASE']` |
| `FileOpenError: Unable to open file /eos/user/...` | Wrong XRootD redirector (`eoscms` used for `/eos/user/` path) | Use `root://eosuser.cern.ch/` for `/eos/user/` paths in `submit.py` |

### Checking a specific job config

After `--create`, verify the generated job config looks correct:
```bash
cat fp/v151Xv1/<TaskName>/conf/job_config_0.py
```
