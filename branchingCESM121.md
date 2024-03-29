---
use_math: true
---

# Branching CESM1.2.1 on SNOW at UAlbany

This is a continuation of [Running CESM1.2.1](runningCESM121.md) for creating branched runs.
See the [CESM1.2 User's Guide](https://www.cesm.ucar.edu/models/cesm1.2/cesm/doc/usersguide/x1894.html)
for another example.

## Case setup

This is the same as creating a non-branch run. For example:

```
/network/rit/lab/roselab_rit/cesm/cesm1_2_1/scripts/create_newcase -mach snow -compset B_1850 -res f19_g16 -case <branch-name>
```

In the case directory, edit the `NTASKS` in `env_mach_pes.xml` as desired.
If you are running a fully-coupled configuration, you may want to give the ocean component more resources. 
To do this, you can, for example, change `ROOTPE_OCN` in `env_mach_pes.xml` from 0 to 64, while leaving all `NTASKS` at 64.
Then when you run `./cesm_setup` and check `<branch-name>.run`, you will see

```
#SBATCH --nodes=4
#SBATCH --ntasks=128
```
And if you scroll down a bit, you will see:

```
#   total number of hw pes = 128
#     cpl hw pe range ~ from 0 to 63
#     cam hw pe range ~ from 0 to 63
#     clm hw pe range ~ from 0 to 63
#     cice hw pe range ~ from 0 to 63
#     pop2 hw pe range ~ from 64 to 127
#     sglc hw pe range ~ from 0 to 63
#     swav hw pe range ~ from 0 to 63
#     rtm hw pe range ~ from 0 to 63
```

This gave the ocean component 64 processing elements (pes) and all of the other components 64 to share, for a total of 128 PEs.

## Runtime options

Follow the parallel section in [Running CESM1.2.1](runningCESM121.md) for all the non-branch changes you will need to make.
After that, run the following:

```
./xmlchange RUN_TYPE=branch
./xmlchange RUN_REFCASE=<ctrl-name>
./xmlchange RUN_REFDATE=<YYYY-MM-DD>
./xmlchange GET_REFCASE=FALSE
```

Where `<ctrl-name>` is the case name of the run you want to branch from and `<YYYY-MM-DD>` is the date of branching (e.g. `0020-07-01`).
You will need the corresponding restart files, which will be in `/data/rose_scr/$CCSMUSER/cesmruns/<ctrl-name>/rest/` or wherever
you ran the control run.

Depending on where you chose to archive your runs, copy over the restart files with something like

```
cp /network/rit/lab/roselab_rit/rford/cesm_archive/<ctrl-name>/rest/<YYYY-MM-DD>-00000/* /data/rose_scr/<NetID>/cesmruns/<branch-name>/run/
```

As with a non-branch run, you need to change the archiving directory to somewhere you have write permissions. For example, I run

```
./xmlchange DOUT_S_ROOT=/network/rit/lab/roselab_rit/rford/cesm_archive/<branch-name>
```

## Build and submit

Same as before:

```
srun -p snow ./<branch-name>.build
./<branch-name>.submit
```
