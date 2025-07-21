---
modified: 2025/07/22 - 01:04
created: 2025/07/16 - 09:38
---
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
- I'm running the terminal `3183942.pts-1.nashira` to plot the efficiencies

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
