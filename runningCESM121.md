# Running CESM1.2.1 on SNOW at UAlbany
Updated June 2022

Based on [Brian's tutorial for ATM 623](http://www.atmos.albany.edu/facstaff/brose/classes/ATM623_Spring2017/2017/03/16/CESM-tutorial.html), as well as [Yuan-Jen's edits](https://hackmd.io/@yuanjenlin/CESM_on_SNOW).
## Environment setup

Log into the head node:
```bash
ssh <NetID>@head.arcc.albany.edu
```

Add the following line to the `.cshrc` file in your home directory:

```
setenv CCSMROOT /network/rit/home/br546577/roselab_rit/cesm/cesm1_2_1
```

The environment variable `$CCSMROOT` now points to CESM source code.

Create a directory that will hold the case directories and `cd` into it.

## Case setup

Running the following line (with a descriptive name replacing `<casename>`), will
create the case directory for a CESM run with the `E_1850_CAM5_CN` compset 
(slab ocean, pre-industrial conditions, CAM5 physics, carbon-nitrogen 
biogeochemistry) and `f19_g16` resolution (2º finite volume for the atmosphere,
1º displaced-pole grid for the ocean and sea ice).

```bash
/network/rit/lab/roselab_rit/cesm/cesm1_2_1/scripts/create_newcase -mach snow -compset E_1850_CAM5_CN -res f19_g16 -case <casename>
```

For other configurations, see the [list of supported compsets](https://www.cesm.ucar.edu/models/cesm1.2/cesm/doc/modelnl/compsets.html).

Now `cd` into `<casename>`. To change the number of nodes that the job will run on, 
edit all `NTASKS` in `env_mach_pes.xml` such that 
$\mathrm{NTASKS}=32*\mathrm{NODES}$. The default is 64 (2 nodes). Then run 

```bash
./cesm_setup
```

More details on case setup can be found in [chapter 2 of the CESM1.2 User's Guide](https://www.cesm.ucar.edu/models/cesm1.2/cesm/doc/usersguide/c513.html).

## Runtime options

The settings that control how the model runs are found in the file `env_run.xml`.
You can either edit the file directly or use `./xmlchange`. Use `./xmlquery` to see 
the current setting.

One common thing to change is the simulation length; note that output is monthly by
default. For example:

```bash
./xmlchange STOP_OPTION=nyears
./xmlchange STOP_N=2
./xmlchange RESUBMIT=9
```

This runs the model for $\mathrm{STOP_N}(1+\mathrm{RESUBMIT})=20$ years in 
`STOP_N`-year chunks. If the cluster is busy, this allows others to run their jobs
between our resubmissions. SNOW has a 12-hour time limit, so make sure each job does not
go over that. See the benchmarking at the end of this document. 
If you use another compset, you may want to do some benchmarking yourself.

If running the SOM (as in compset E), put the forcing file in `/data/rose_scr/cesm_inputdata/ocn/docn7/SOM`,
and back in the case directory run

```bash
./xmlchange DOCN_SOM_FILENAME=<forcingfilename>.nc
```

For the OHU TCR project piControl run, I am using 

```bash
./xmlchange DOCN_SOM_FILENAME=pop_frc.b.e11.B1850C5CN.f19_g16.130429.nc
```

Optionally change CO$_2$ and other stuff in `user_nl_cam` or the other `user_nl`
files. For example, add the following line to `user_nl_cam` for a 2xCO2 
(from pre-industrial) experiment:

```
co2vmr = 569.4e-6
```

### Additional required changes

In `<casename>.run`, change the value of `--mem` from `MaxMemPerNode` to some value
(the max seems to be 128G). Uncomment and edit the email notification lines to be
notified when the job ends.

For now, you need to add the following line to `env_mach_specific` for the Intel
compiler to work:

```bash
source /network/rit/lab/snowclus/modules/latest.csh
```

## Building the model

### Users outside the Rose group

First you need to decide where to keep the executable files. 
If you look in `env_build.xml`, you will find the following line:

```
<entry id="EXEROOT"   value="/data/rose_scr/$CCSMUSER/cesmruns/$CASE/bld"  />
```

This variable `EXEROOT` determines where the model will be built. 
By default it will look for a directory within `/data/rose_scr/`. 
This will only work if you have write access to `/data/rose_scr/`, 
which is probably only true if you are part of the Rose research group. 
If not, you will need to set this to somewhere within your own group's scratch
space.

For example, if you are part of the Zhou group, you could do this:

```bash
./xmlchange EXEROOT=/data/zhou_scr/$CCSMUSER/cesmruns/$CASE/bld
```

At the same time you should also set the run directory `RUNDIR` (where the output
files will be written). Usually this would be another subdirectory alongside the
build:

```bash
./xmlchange RUNDIR=/data/zhou_scr/$CCSMUSER/cesmruns/$CASE/run
```

### All users

Change the archive directory (in `env_run.xml`) to somewhere you know you have write
permissions:

```bash
./xmlchange DOUT_S_ROOT=<directory>
```

By default, the directory is

```
/network/rit/lab/roselab_rit/cesm_archive/<casename>
```

but, for example, I did not have write permissions there and had to change mine to
somewhere I did:

```
/network/rit/lab/roselab_rit/rford/cesm_archive/<casename>
```

Then build the model with the following line:

```bash
srun -p snow `pwd`/<casename>.build
```

This will take about 20 minutes, at least for the compset used here. 
If the build time is really quick (a few seconds), then the run 
configuration is probably messed up, and the job cannot be submitted. 

## Submit the job

Finally, submit the job with

```bash
./<casename>.submit
```

You can check the queue, see node info, or cancel the job with 

```bash
squeue
sinfo
scancel <JOBID>
```

## Benchmarking
1-year runs with the `E_1850_CAM5_CN` compset and `f19_g16` resolution:
- `NTASKS` = 64 (2 nodes) $\rightarrow$ 5h:13m
- `NTASKS` = 128 (4 nodes) $\rightarrow$ 3h:24m
- `NTASKS` = 256 (8 nodes) $\rightarrow$ 2h:10m
