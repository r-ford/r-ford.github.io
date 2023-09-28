# Cloud locking CESM1.2.1 on SNOW

## Outline

### Control run

- Copy files from the `NCAR/cesm_cloud_locking` repository to somewhere they can later be copied into CESM case directories
- Create the case `<ctrl-name>` for the control run
- Copy source code mods (all of the files in `NCAR/cesm_cloud_locking/cesm2_0_1/`) into `<ctrl-name>/SourceMods/src.cam/`
- Add the following lines to `user_cam_nl`:
  ```
  fincl2 = 'DEI_rad:I','MU_rad:I','LAMBDAC_rad:I','ICIWP_rad:I','ICLWP_rad:I','DES_rad:I','ICSWP_rad:I','CLD_rad:I','CLDFSNOW_rad:I','PS:I'
  nhtfrq = 0,-2
  mfilt = 1,12
  ndens = 2,1
  ```
- Make various changes to `env_run.xml` (run type, run length, etc.), including any SNOW-specific changes indicated in previous tutorials
- Make any other changes (i.e. 2xCO2)
- Build the model and submit the case
