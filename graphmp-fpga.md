# Notes on `graphmp-fpga`

How to run QAT script and make plots:

```bash
cd /home/russell/graphmp-fpga
source graph-mp/bin/activate

# 1. QAT training (~1.5 min on one GPU)
python scripts/train.py --warm-start checkpoints/model.npz --qat 8 8 --steps 2000 --device cuda

# 2. training curves → checkpoints/retrained.png
python scripts/plot_training.py checkpoints/retrained.log.json

# 3. LER vs p: fp32 vs W8A8 without QAT vs W8A8 with QAT → plots/ler_curve.png
python scripts/plot_ler.py --shots 5000
```
Then, for synthesis, I only need the new `.h` files and moved to the H200 server. Then, I ran the following (not in any container):
```bash
export XILINXD_LICENSE_FILE=/tools/Xilinx/Vivado_license.lic
source /tools/Xilinx/2026.1/settings64.sh
nohup vitis-run --mode hls --tcl run_hls_zcu216.tcl > run_hls.log 2>&1 &
```
