# ATLAS_lab

[[detlab_dortmund]]
[[CNNtrain.py#2024/02/28 Screen]]

### Screen commands

- To create a terminal ---> `screen`
- To see a list of *detached* terminals ---> `screen -ls`
- To access a terminal ---> `screen -xr TERMINAL_NAME`
- To delete a terminal ---> press `Ctrl+a` and then write `:quit` and press `enter`
### Running notes

> Some quick notes to be rearranged in a better and clearer shape

#### Currently running

- I'm running the terminal `2224961.pts-5.nashira` to plot the efficiencies

(The `.exe` files are created by running `make` inside `/nfs/homes/zprime/zprime12/Code`. To clean up the folder from previously compiled files use `make clean`. The compilation options are defined inside the `Makefile` file).

#### Real/MC files folder

The folder that contains the “real” files is as follows, these are very heavy and to process them you need to use `screen`

```
/ceph/e4/users/bgocke/Zprime/Samples/
```

#### `scanEfficiency`

I created the file `scanEfficiency.C` and then, compiling with `make`, generated `scanEfficiency.exe`, then created the folder `/nfs/homes/zprime/zprime12/output_scanEfficiency` in which the script saves the produced efficiency graphs.

I ran the scan from inside the `/nfs/homes/zprime/zprime12` folder with

```
./Code/scanEfficiency.exe /ceph/e4/users/bgocke/Zprime/Samples/*
```

Next I modified the `runSelection.C` file, implementing cuts on variables.
Such a file can be tested on `data_example.root` and `ttbar_example.root` with (from inside `/nfs/homes/zprime/zprime12/Code`) e.g., `data_example.root` and `ttbar_example.root`.

```
./runSelection.exe ../data_example.root
```

#### `plotEfficiencies`

I create the file `plotEfficiencies.C` in the folder `/nfs/homes/zprime/zprime12/Code/output_runSelection` to run in the folder with

```
root -l -q plotEfficiencies.C
```

(This command should be executed only after performing the run the selection over real/MC files)

#### run the selection over real/MC files

I wrote the script `script.sh`

```
#!/bin/bash
for filename in /ceph/e4/users/bgocke/Zprime/Samples/*; do
./runSelection.exe $filename;
done
```

with which to go and run `runSelection.exe` on all the files contained in `/ceph/e4/users/bgocke/Zprime/Samples/`.

This is to be run from inside the `/nfs/homes/zprime/zprime12/Code` folder.

It returns as output: a `.txt` file, containing the values of all efficiencies, for each `.root` file processed and an `eff_summary.txt` file containing the global efficiency of all `.root` files. The latter file is what is used by `plotEfficiencies.C` to create the histogram of the efficiencies.

##### Chosen cuts

```
// Cut-flow counters
  const int nSteps = 8;
  vector<Long64_t> nPass(nSteps, 0);
  vector<string> stepLabels = {
      "Total events",
      "Exactly 1 lepton",
      "Lepton pT > 50 GeV",
      "Lepton |η| < 2.5",
      "Jet multiplicity: 3-5 jets",
      "≥1 jet with pT ≥ 90 GeV",
      "≥1 b-tagged jet (70% WP)",
      "MET > 40 GeV"
  };

  // Pre-calculate constants
  const double minLepPt = 50000;
  const double minJetPt = 90000;
  const double minMET = 40000;

  // Counter for events passing ALL cuts
  Long64_t nPassedAll = 0;

  // Event processing
  for (Long64_t iEntry = 0; iEntry < nEntries; ++iEntry) {
      tree->GetEntry(iEntry);
      if ((iEntry+1) % 10000000 == 0) {
          cout << "Processing event " << iEntry+1 << "/" << nEntries << endl;
      }

      nPass[0]++; // Step 0: Total events

      // Step 1: Exactly one lepton
      if (tree->lep_n != 1) continue;
      nPass[1]++;

      // Create lepton four-vector
      TLorentzVector lep;
      lep.SetPtEtaPhiE(
          tree->lep_pt->at(0),
          tree->lep_eta->at(0),
          tree->lep_phi->at(0),
          tree->lep_E->at(0)
      );

      // Step 2: Lepton pT > 50 GeV
      if (lep.Pt() < minLepPt) continue;
      nPass[2]++;

      // Step 3: Lepton |η| < 2.5
      if (fabs(lep.Eta()) >= 2.5) continue;
      nPass[3]++;

      // Step 4: Jet multiplicity (3-5 jets)
      if (tree->jet_n < 3 || tree->jet_n > 5) continue;
      nPass[4]++;

      // Step 5: ≥1 jet with pT ≥ 90 GeV
      bool highPtJetFound = false;
      for (UInt_t i = 0; i < tree->jet_n; ++i) {
          if (tree->jet_pt->at(i) >= minJetPt) {
              highPtJetFound = true;
              break;
          }
      }
      if (!highPtJetFound) continue;
      nPass[5]++;

      // Step 6: ≥1 b-tagged jet (70% WP)
      bool bTagFound = false;
      for (UInt_t i = 0; i < tree->jet_n; ++i) {
          if (tree->jet_MV2c10->at(i) > 0.83) {
              bTagFound = true;
              break;
          }
      }
      if (!bTagFound) continue;
      nPass[6]++;

      // Step 7: MET > 40 GeV
      if (tree->met_et <= minMET) continue;
      nPass[7]++;
      
      // Event passed ALL cuts
      nPassedAll++;
      newTree->Fill();
  }
```

#### 2025/07/22

The run selection over real/MC files has been completed and the overall efficiency histogram has been plotted with (`plotEfficiencies`), so we can now proceed with `4 Plot several fundamental distributions`.

#### Original bare file structure

The initial file list, bare structure, of `/nfs/homes/zprime/zprime12` was

```
├── Code
│   ├── Makefile
│   ├── Neutrino.h
│   ├── NeutrinoReco.cc
│   ├── analysis.root
│   ├── atlasstyle-00-04-02
│   ├── chiSquare.C
│   ├── chiSquare.exe
│   ├── fileHelper.cxx
│   ├── fileHelper.h
│   ├── fileHelper.o
│   ├── mini.cxx
│   ├── mini.h
│   ├── mini.o
│   ├── output_runSelection
│   ├── physicsHelper.h
│   ├── plotDistribution.C
│   ├── plotDistribution.exe
│   ├── runSelection.C
│   ├── runSelection.exe
│   ├── stackedPlots.C
│   └── stackedPlots.exe
├── data_example.root
└── ttbar_example.root
```

The `.exe` files are created by running `make` inside `/nfs/homes/zprime/zprime12/Code`. To clean up the folder from previously compiled files use `make clean`. The compilation options are defined inside the `Makefile` file.

