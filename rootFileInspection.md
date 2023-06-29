## How To Quickly Inspect a .root File:
(Check out `L1TaggerHistograms.ipynb` on lxplus. I played with this on that notebook)

1. `import ROOT as r`
2. `inFile = r.TFile.Open("NtuplesSet1.root", "READ")`, reads the .root file.
3. `inFile.cd()` has to print `True`. I believe this is to cd into a folder.
4. `inFile.ls()` will print contents of the file
5. `tree = inFile.Get("ntuple0/objects")` accesses the tree `objects`.
6. `tree.Show(0)` will show the contents in the first event.
