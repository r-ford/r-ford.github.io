---
title: "Running CESM2 on Burst-DAES"
layout: post
mathjax: true
---

CESM2 was ported onto a DAES server thanks to the work of Li Zhuo, Kevin Tyle, and others. The following is a short step-by-step guide to running the model.

## Log in and environment

Log in:

```
ssh <NetID>@head.its.albany.edu
```

In your home directory, add the following lines to `.bashrc`. This is only done once.

```
source /network/rit/lab/snowclus/modules/intel24.bash
source /network/rit/lab/snowclus/bin/conda_python_only.bash
ulimit -s unlimited
```

Run `. ~/.bashrc` once per session.

## Create and set up new case

The model code and scripts are stored in the directory `/zhoulab_rit/lzhuo/my_cesm_sandbox/`. Create a new case with 

```
/network/rit/lab/zhoulab_rit/lzhuo/my_cesm_sandbox/cime/scripts/create_newcase –-compset <compset> --res <res> --machine burst-daes --case <case-name> --run-unsupported
```

The flag `--run-unsupported` is optional, depending on whether you are running an "unsupported" compset/res combination. The script will tell you if you are trying to run unsupported options.

Next, change the PE layout in `env_mach_pes.xml`. Since there are a few things that need to be changed, I just copy the file from a previous run. 
If you want to run your case on 4 nodes (the maximum suggested), you can just copy this file over from the one in `/roselab_rit/rford/cesm2/` 
or change the `NTASKS` for each component to 128 (see my previous post for syntax). You can preview the PE layout with `./pelayout`.

The case setup script we are about to run creates the `bld` and `run` directories. By default, the script will try to create folders in a directory that you will not have write access to, 
so you will need to change it. For me, this looks like:

```
./xmlchange CIME_OUTPUT_ROOT=/network/rit/lab/roselab_rit/rford/cesm2
```

Finally, you can run `case.setup`.

## Changing more directories

For the same reason as above, we need to change the archive directory. This is the directory where you will access the netCDF output files. For me, this looks like:

```
./xmlchange DOUT_S_ROOT=/network/rit/lab/roselab_rit/rford/cesm2_archive/<case-name>
```

I'm not sure if there is a way to get the case name without typing it out. Replacing `<case-name>` with `$CASE` does not seem to work for me, but there might be another way.

The model uses various input files which are automatically downloaded by the model as needed and stored for future use. This requires write permissions to the default `input_data` folder. 
This time, there are two directories to change, which for me look like:

```
./xmlchange DIN_LOC_ROOT=/network/rit/lab/roselab_rit/rford/cesm2/input_data
./xmlchange DIN_LOC_ROOT_CLMFORC=/network/rit/lab/roselab_rit/rford/cesm2/input_data
```

## Case-specific changes

Now is the time to make other changes relevant to your experiment: run length, namelist changes, forcing files, output frequency, etc.

## Build and submit

Run `case.build`, which will take some time. Then submit your job to the cluster with `case.submit`.

## Benchmarking

| Compset                        | Grid    | Nodes | Sim length | Run time (s)        | Cost* (wall hrs/sim year) |
|--------------------------------|---------|-------|------------|---------------------|-----------------------|
| B1850 (CAM6, POP2)             | f09_g17 | 4     | 3 m        | \\(\lessim\\)16,580 | 18.42                 |
| B1850 (CAM6, POP2)             | f19_g16 | 4     | 1 y        | \\(\lessim\\)21,432 | 5.95                  |
| B1850C4L45BGCBPRP (CAM4, POP2) | f09_g16 | 4     | 1 y        | \\(\lessim\\)23,150 | 6.34                  |
| B1850C4L45BGCBPRP (CAM4, POP2) | f19_g16 | 4     | 1 y        | \\(\lessim\\)12,029 | 3.34                  |

*For a 12-hour time limit per job, the maximum number of years you can run per job is \\(\lfloor 12/\mathrm{Cost}\rfloor\\). If this is zero, you need to run in monthslong chunks.
