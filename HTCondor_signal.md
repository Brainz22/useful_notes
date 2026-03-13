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
---

### 1. Jobs complete but no output files

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
---
### 2. File dataset on CMS DAS at `T1_ES_PIC_Disk`, but jobs failed to access it

[Issue 1]: Global XRootD redirector does not route to PIC. **Symptom:** Jobs failed with:
```[ERROR] No servers are available to read the file``` using `root://cms-xrd-global.cern.ch//` as the redirector.

**Fix:** Use PIC's own XRootD server directly:

```root://xrootd.pic.es//``` for `input_files` in `submit.py` (~ lines 200-230).

[Issue 2]: `[FATAL] Auth failed: No protocols left to try` because Condor worker nodes did not have a VOMS proxy forwarded to them, so they could not authenticate with PIC's XRootD server.

**Fix:** Add proxy forwarding to the Condor submit template (`condorSubmit_DEFAULT.sub`):
`use_x509userproxy = true`, then copy your proxy to AFS (since `/tmp` is node-local and not shared across lxplus nodes) and specify the path explicitly:
  ```bash
  bash
  mkdir -p ~/private
  cp /tmp/x509up_u<uid> ~/private/x509up
  chmod 600 ~/private/x509up
  export X509_USER_PROXY=~/private/x509up
  ```
 Then in `condorSubmit_DEFAULT.sub`, add `x509userproxy = /afs/cern.ch/user/<u>/<username>/private/x509up` to use your grid certificate (after validation).

[ISSUE 3]:  Files not physically present for the bbbb sample

**Symptom**: Jobs ran and authenticated successfully but failed with:
[ERROR] No such file (errno=3011)
Cause: DAS listed the bbbb sample at T1_ES_PIC_Disk, but the files were not actually stored there. Verified by checking directly:

`xrdfs xrootd.pic.es ls /pnfs/pic.es/data/cms/store/mc/Phase2Spring24DIGIRECOMiniAOD/ | grep HiddenGluGlu`
The bbbb sample was absent; only cccc and uuuu variants were present.

**Fix:** Switch to the cccc variant which is physically available:
`/HiddenGluGluH_mH-125_Phi-30_ctau-10_cccc_TuneCP5_14TeV-pythia8/
Phase2Spring24DIGIRECOMiniAOD-PU200_Trk1GeV_140X_mcRun4_realistic_v6-v1/
GEN-SIM-DIGI-RAW-MINIAOD`

[Issue 4]: DAS returns no files for PIC even though files exist

**Symptom:** `getFilesForDataset(..., site='T1_ES_PIC_Disk')` returned 0 files, resulting in only 1 job submitted.

**Cause:** DAS metadata for this dataset at PIC was not registered, but the files were physically present. Also, PIC's XRootD does not serve files via the standard LFN path (`/store/mc/...`) — it requires the full PNFS path:

`/pnfs/pic.es/data/cms/store/mc/...`
**Fix:** Add `input_xrd_directory` support to `submit.py` to list files directly from the `XRootD` server using `xrdfs ls -R`, bypassing DAS entirely. In the `submit_INFP_151X_HiddenGluGluH_Phi30_ctau10_cccc.yaml` (since this is the input FastPUPPI step using the cfg `/FastPUPPI/NtupleProducer/python/runInputs151X.py`):

```bash
input_xrd_directory: root://xrootd.pic.es//pnfs/pic.es/data/cms/store/mc/Phase2Spring24DIGIRECOMiniAOD/HiddenGluGluH_mH-125_Phi-30_ctau-10_cccc_TuneCP5_14TeV-pythia8/GEN-SIM-DIGI-RAW-MINIAOD/PU200_Trk1GeV_140X_mcRun4_realistic_v6-v1
```

The URL is parsed as:
`root://<host>//<path>`

where `<host> = xrootd.pic.es` and `<path> = /pnfs/pic.es/data/cms/store/mc/...`.
This resulted in 46 jobs submitted and 46 output files successfully produced on EOS.


