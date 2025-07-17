## How To Quickly Inspect a .root File:
(Check out `L1TaggerHistograms.ipynb` on lxplus. I played with this on that notebook)

1. `import ROOT as r`
2. `inFile = r.TFile.Open("NtuplesSet1.root", "READ")`, reads the .root file.
3. `inFile.cd()` has to print `True`. I believe this is to cd into a folder.
4. `inFile.ls()` will print contents of the file
5. `tree = inFile.Get("ntuple0/objects")` accesses the tree `objects`.
6. `tree.Show(0)` will show the contents in the first event.

## From the terminal:
After activating root (via cmsenv, for example):
1. `root -l pfTuple.root`
2. `[0] .ls`. This will show `ntuple0` as the main tree.
3. `[1] ntuple0 -> cd()`, change into this tree.
4.  `[2] .ls`, this shows the branches.
5. `[3] objects -> Print()`, prints the branches.
6. `[4] new TBrowser()`, opens a TBrowser. We need the `xquartz` application on Mac, for example.
