# Tutorial 1 - Biogeochemistry Demo with ORCA2 (NEMO v5.0.2)

## Description

This test case provides a demonstration of PISCES, the biogeochemical (BGC) model included within NEMO V5.0.2, over the full global ocean at 2° horizontal resolution. Provided with this test case are scripts for determining the dominant phytoplankton and zooplankton at each grid point in the global ocean for two reference runs, one based on the PISCES (P4Z) configuration and one based on the PISCES QUOTA (P5Z) configuration. 

This demonstrator helps to educate new users of NEMO interested in running with BGC on the changes necessary to run the different configurations of PISCES. Comparison of the outputs from each run also allows to explore the effect of having a higher complexity model (through the additional phytoplankton included within PISCES QUOTA) and the impact this has on the dominance of each plankton type.

## Background

PISCES is a biogeochemical model that simulates marine biological productivity and describes the biogeochemical cycles of carbon and five nutrients (nitrates, ammonium, phosphate, silicate, and iron). The model represents four living pools: two phytoplankton size classes (nanophytoplankton and diatoms) and two zooplankton size classes (microzooplankton and mesozooplankton), and three non-living compartments: semi-labile dissolved organic matter, and small and big sinking particles. Nutrients are supplied to the ocean from three sources: atmospheric dust, rivers, and sediment, and phytoplankton growth is limited by the availability of each nutrient.

<div align="center">
  <img src="images/PISCES.png" width="600" height="450">
</div>

The PISCES QUOTA model builds upon the standard operational version of PISCES to include a quota-based description of phytoplankton growth with fully variable C:N:P:Si:Fe:Chl ratios. The basic structure of the model is also modified with the addition of a third phytoplankton group, picophytoplankton.

<div align="center">
  <img src="images/PISCES_Quota.png" width="600" height="450">
</div>


## Getting Started
### Dependencies

The following dependencies are required for this demonstrator (in addition to the pre-requisited mentioned in Tutorial 0 - NEMO_basics):
* An install of ImageMagick (for converting images to animations)
* An install of Python version 3.8 or above
* Installs of the following Python packages:
  * numpy, pandas, netCDF4, matplotlib, xarray, sys

### Provided files

To be able to run the demonstrator, the directory `PISCES_FILES` has the `namelist` reference and configuration files needed for running PISCES. The directories `P4Z_FILES` and `P5Z_FILES` also include the following files specific to each configuration:
* `namelist_top_cfg` with the specific parameter values for each configuration
* `file_def_nemo_pisces.xml` with the required variables for each reference run

We also include the following files for post-processing of the outputs in the directory `SCRIPTS`:
* A `mesh_mask_v5.nc` file for use with NEMO V5.0.2
* `models.csv` which lists details for the models we are comparing
* A shell script `compare.sh` to visualise the comparison of the dominant species for each run
* Python scripts `dominant-phyto.py` and `dominant-zoo.py` to calculate the dominant phyto- and zooplankton, respectively, at each grid point
* A python script `summary.py` to generate a global average summary of each plankton type for comparison

In your work directory, create the directory `BGC_DEMO_FORCINGS` and download the files described above either from the Github repositry using git clone or from the corresponding Zenodo directory using wget.

In addition to these files, our demonstrator uses the `ORCA2_ICE_v5.0.0.tar.gz` and `ORCA2_INPUTS_PISCES_v5.0.0.tar.gz` inputfiles, which can be found [here](https://gws-access.jasmin.ac.uk/public/nemo/sette_inputs/). These files should be downloaded with wget:
```
wget https://gws-access.jasmin.ac.uk/public/nemo/sette_inputs/extras/ORCA2_INPUTS_PISCES_v5.0.0.tar.gz ./
wget https://gws-access.jasmin.ac.uk/public/nemo/sette_inputs/r5.0.0/ORCA2_ICE_v5.0.0.tar.gz ./
```
then untar those files with:
```
tar -xzvf ORCA2_ICE_v5.0.0.tar.gz
tar -xzvf ORCA2_INPUTS_PISCES_v5.0.0.tar.gz
```

### Installing

First follow the instructions on dowlooading and compiling NEMO and XIOS from the NEMO_basics tutorial.

This demonstrator is based on the [ORCA2_ICE_PISCES](https://sites.nemo-ocean.io/user-guide/cfgs.html#orca2-ice-pisces) reference configuration. Build your new `BGC_DEMO` configuration by duplicating the reference configuration with the command. (If you used the auto build functionality to automatically build your arch file them MY_COMPUTER would simply be "auto".)

```
./makenemo -m MY_COMPUTER -r ORCA2_ICE_PISCES -n BGC_DEMO -j 8 --add_key key_xios3

```

### Running each configuration of PISCES

You need to go in your `BGC_DEMO` directory:
```cd cfgs/BGC_DEMO```

You should then create two additional directories `EXP_P4Z` and `EXP_P5Z`, where we will run each configuration of PISCES.
```
cp EXP00 EXP_P4Z
cp EXP00 EXP_P5Z
```

Copy the files from `BGC_DEMO_FILES` into each directory:

```
cp PATH_to_BGC_DEMO_FORCINGS/* EXP_P4Z/.
cp PATH_to_BGC_DEMO_FORCINGS/* EXP_P5Z/.
```

Then for `EXP_P4Z` experiment, you need to rename `namelist_top_P4Z` to `namelist_top_cfg`, and for `EXP_P5Z` experiment, you need to rename `namelist_top_P5Z` to `namelist_top_cfg`.

Finally, link all the netcdf forcing files into both experiment directories.
```
ln -sf PATH_to_ORCA2_ICE_v5.0.0/* EXP_P4Z/.
ln -sf PATH_to_ORCA2_INPUTS_PISCES_v5.0.0/* EXP_P4Z/.
ln -sf PATH_to_ORCA2_ICE_v5.0.0/* EXP_P5Z/.
ln -sf PATH_to_ORCA2_INPUTS_PISCES_v5.0.0/* EXP_P5Z/.
```

**Namelist management**

We explain here how to manage the various namelists and activate the different options between the two configurations. 

In NEMO there are 2 kind of namelist, the reference namelist (_ref) that contains default settings and the configuration one (_cfg) that contains user settings. In principle, one must not make changes in namelist_ref, it should remain pristine. Use namelist_cfg to personalise your changes. By doing so, NEMO will overwrite the default choices set in namelist_ref

**Overwrite default run settings**

But default, NEMO looks for a restart file. To start from rest, we need to deactivate the reading of this restart file.

Under `&namrun` in namelist, set the restart option to `.false.`. We also suggest that you mask land points as this makes for more intuitive plots:
```
!-----------------------------------------------------------------------
&namrun        !   parameters of the run
   ln_rstart   =  .false.   !  start from rest (F) or from a restart file (T)
   ln_mskland  = .true.   !  mask land points in NetCDF outputs
/
!-----------------------------------------------------------------------
```
The type of PISCES model to use is defined in the namelist. 

**PISCES model (P4Z)**

To activate PISCES (P4Z) set `ln_p4z` to `.true.` in P4Z `namelist_pisces_cfg` by replacing this empty block as follows:
```
!-----------------------------------------------------------------------
&nampismod     !  Model used 
!-----------------------------------------------------------------------
  ln_p2z      = .false.      !  LOBSTER model used
  ln_p4z      = .true.       !  PISCES model used
  ln_p5z      = .false.      !  PISCES QUOTA model used
/
!-----------------------------------------------------------------------
```
and set the number of tracers to use to *24* for the run in `namelist_top_cfg` by replacing the empty block as follows:
```
!-----------------------------------------------------------------------
&namtrc          !   tracers definition
!-----------------------------------------------------------------------
   jp_bgc        =  24
/
```

Then, we need to add extra outputs in `file_def_nemo-pisces.xml`. In the file description with output at 7d frequency. To do this, you need to replace:
```
   <file_group id="7d"  output_freq="7d"  output_level="10" enabled=".TRUE."/>  <!-- 7d files -->
```
by:
```
   <file_group id="7d"  output_freq="7d"  output_level="10" enabled=".TRUE.">  <!-- 7d files -->
     <file id="file1" name_suffix="_ptrc_T" description="pisces sms variables" >
       <!-- ln_p4z variables -->
       <field field_ref="PHY"                             />
       <field field_ref="PHY2"                            />
       <field field_ref="ZOO"                             />
       <field field_ref="ZOO2"                            />
     </file>
   </file_group>
```
Take care of the `/>` at the end of the first line and `</file_group>` to close the file group.

**PISCES QUOTA model (P5Z)**

Similarly activate in EXP_P5Z by setting `ln_p5z` to `.true.` (and `ln_p4z` to `.false.`) and set the numbers to *40*.

For P5Z, as for P4Z, we need to add extra outputs in `file_def_nemo-pisces.xml`. In the file description with output at 7d frequency. To do this, you need to replace:
```
   <file_group id="7d"  output_freq="7d"  output_level="10" enabled=".TRUE."/>  <!-- 7d files -->
```
by:
```
   <file_group id="7d"  output_freq="7d"  output_level="10" enabled=".TRUE.">  <!-- 7d files -->
     <file id="file1" name_suffix="_ptrc_T" description="pisces sms variables" >
       <!-- ln_p5z variables -->
       <field field_ref="PHY"                             />
       <field field_ref="PHY2"                            />
       <field field_ref="PIC"                             />
       <field field_ref="ZOO"                             />
       <field field_ref="ZOO2"                            />
     </file>
   </file_group>
```

**Run the configuration**

Now you can run each model configuration. To do so, follow the steps described for copying across xios server, editing iodef.xml and building and runing the job script [here](https://github.com/NEMO-consortium/0_NEMO_basics/blob/main/Turorial1.md#3-how-to-run-nemo-502). 

To go faster you can ask for more than 4 cores in your job script, we recommend 32 or 64 depening on your HPC and your ressources available. To do so, modify the ressources asked in the header and in the `mpirun` (or equivalent) command.

- 

The model runs over the full global ocean for one year (1948) with a timestep of `5400s` and outputs at a 7-day time frequency. When the model has finished, it will output a file `ORCA2_7d_19480101_19481231_ptrc_T.nc` with the concentrations for each phyto- and zooplankton.


## Results

We describe here the steps for running the scripts to visualise the difference in dominant species between each configuration of PISCES. The first step is to copy the scripts from the directory `SCRIPTS` into your `$WORK/BGC_DEMO_RUN` directory. We then need to update the `models.csv` file to include the correct directory location for your `$WORK` directory.

We run the comparison with the command `./compare.sh`, which will run a series of python scripts to:
* calculate depth-averaged values for each phyto- and zooplankton
* determine the dominant species at each grid point for each reference run. This will be contained in the following files, saved in the run directory:
  * `ORCA2_7d_19480101_19481231_domphy_T.nc`
  * `ORCA2_7d_19480101_19481231_domzoo_T.nc`
* generate comparison images of the dominant phyto- and zooplankton for each configuration
* combine these images into a single animation for each plankton type (named `DOMPHY.gif` and `DOMZOO.gif`, respectively)

The script will output the current week as it runs to show progression. An example comparing the dominant phytoplankton for a particular week is shown below:

<div align="center">
  <img src="images/Dominant-species-phyto-w25.png" width="600" height="300">
</div>

A script is also included to calculate the global average value for each plankton type and then plot a comparison between the two configurations. This is run with the command `./summary.py`, with an example of the output shown below:

<div align="center">
  <img src="images/seasonal-summary.png" width="630" height="280">
</div>


## Authors

Contributors:
* Philip Townsend
* Renaud Person
* Julien Palmieri  

