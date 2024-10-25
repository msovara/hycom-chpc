# hycom-chpc

Instructions for how to setup and run HYCOM on CHPC
If any of the instructions are unclear, please do not hesitate to leave a comment.
About the ocean model.  

HYCOM is a hybrid coordinate ocean model, meaning that it has density following coordinates in the interior, z-coordinates in the mixed-layer and the option to have sigma-layer along the coast. To read about the fundamentals of HYCOM, I refer you to the paper by Bleck (2002).

The version of HYCOM that we will be running is mostly the same as the original HYCOM, but with a few modifications.

You can find the code that we will be using here. If you scroll down on the page, you will find a number of instructions on how to install the different elements needed to install and run the model.  The instructions here follow these, but with specific instructions related to this machine. Especially take a look at the part about HYCOM-CICE directory structure, understanding this will help you remember where to look for the different routines.

This model has been used in the Ahulhar region earlier, here are a few examples of papers:
https://doi.org/10.1029/2017JC013602
https://doi.org/10.1029/2019JC015365
https://doi.org/10.1080/1755876X.2008.11020093

## Preparing HYCOM for running on CHPC

```
To login to CHPC machine:
ssh -X username@scp.chpc.ac.za
```
## 1. Loading modules:
In order to compile and run the model, I had to load the following modules:
```
module purge
cp /mnt/lustre/groups/ERTH0904/hycom/setupHyCOM .
source setupHyCOM
```

The last command must be done every time after you log in (also on a computational node), if you want to run HYCOM or postprocess results.

If you normally have other modules loaded, it is probably a good idea to clear those before loading the new ones (in case there are conflicts).


2. Cloning the model code
Go to your home directory and the folder you want to have the code in.
use this command to get the code:
cd
git clone https://github.com/nansencenter/NERSC-HYCOM-CICE
cd NERSC-HYCOM-CICE/
to checkout the branch (normally we use the branch called ‘develop’, but some changes needed to run on CHPC are awaiting review):
git checkout issue_hycom_standalone
git branch 
This step can be done only once, but do the command ‘git pull’ once in a while to get the latest updates.
3. Installing python
Many routines for pre- and postprocessing of files for the model are written in python.  You can find the majority of these routines in the folder called bin.

You can find an overview of the python-packages needed here.

Some will come with the python-module loaded under (1).
I had to install the following:
cp /mnt/lustre/groups/ERTH0904/hycom/hycom_env.yml .
conda env create -f hycom_env.yml
conda activate hycomenv

And everybody will have to install the python routines that comes with code:
pip install --user NERSC-HYCOM-CICE/pythonlibs/gridxsec/
pip install --user NERSC-HYCOM-CICE/pythonlibs/abfile/
pip install --user NERSC-HYCOM-CICE/pythonlibs/modelgrid/
pip install --user NERSC-HYCOM-CICE/pythonlibs/modeltools/
If you use python to plot your results, the abfile package will be useful to read the model output.

This step should (in principle) only need to be done only once, but must be redone if you change the python version.
4. Compiling MSCPROGS
MSCPROG contains a number of routines for preparing forcing files and postprocessing data developed at NERSC.  Instructions on how to compile can be found here.

You should find a file called make.chpc.ifort in the Make.Inc directory, link that to make.inc
cd  NERSC-HYCOM-CICE/hycom/MSCPROGS/src/Make.Inc/
ln -sf make.chpc.ifort make.inc
cd ..

Then, while in: NERSC-HYCOM-CICE/hycom/MSCPROGS/src/:
In addition to the modules in setupHYCOM, it is necessary to load python7.3.0 in order to compile
module load chpc/python/3.7.0
make clean
make all
make install
This step should (in principle) only needs to be done once, but must be redone if you update the modules you load, or there has been a upgrade of the chpc.

5.Compiling HYCOM-ALL
HYCOM-ALL contains a number of routines for preparing forcing files and postprocessing data developed by the HYCOM developers.  Instructions on how to compile can be found here.

here you can simply use:
cd 
cd NERSC-HYCOM-CICE/hycom/hycom_ALL/hycom_2.2.72_ALL/
csh ./Make_all.com
You will notice that a few of the routines fail, these are routines we do not use, it also complains about some variables nor being used, but as long as the majority of the routines report “worked” you can ignore these messages.

This step should (in principle) only needs to be done once, but must be redone if you update the modules you load, or there has been an upgrade of the chpc.

6. Setting up the model domain folder
The general instructions for setting up the region are here and to set up the experiment  folder here.  This part will have to be done in your work directory.  We are going to set up a small toy model of the Agulhas region which is about 100x110 model points and is able to run on just a few processors.
make a directory called AGUa1.00
From my work directory copy to your own AGUa1.00:

Faster way:
cd /mnt/lustre/users/USERNAME/
cp -r /mnt/lustre/groups/ERTH0904/hycom/AGUa1.00/ .

/mnt/lustre/groups/ERTH0904/hycom/AGUa1.00/REGION.src
edit this file so that NHCROOT points to the hycom-code in your home directory.

Copy the whole folder /mnt/lustre/groups/ERTH0904/hycom/AGUa1.00/topo/ to your own AGUa1.00-directory.

These files contain the grid information for this model domain.  The [ab] file pairs are in the file format used by HYCOM.  The a-file is a binary file containing the date, the b-file is a text file with a header and an inventory of the fields contained in the corresponding a-file. 
For example if you view regional.grid.b you can see the grid size and a list of the grid varibøes with minimum and maximum value.

Copy the whole folder /mnt/lustre/groups/ERTH0904/hycom/AGUa1.00/expt_01.0 to your own AGUa1.00-directory.
In the expt_01.0-directory make a link to the bin-directory in you hycom-code (NERSC-HYCOM-CICE/bin):

Specifically when running hycom either coupled to sea ice using oasis or standalone as we will do here, make a link called mysource from the folder Oasis in the code in the expt_01.0-directory:

cd /mnt/lustre/users/USERNAME/AGUa1.00/expt_01.0/
ln -sf ~/NERSC-HYCOM-CICE/hycom/RELO/src_2.2.98ZA-07Tsig0-i-sm-sse_relo_mpi/Oasis/ mysource
ln -sf ~/NERSC-HYCOM-CICE/bin .

To compile the model:

cd ~/NERSC-HYCOM-CICE/
git pull
cd /mnt/lustre/users/USERNAME/AGUa1.00/expt_01.0/
bin/compile_model.sh -m chpc ifort

  

Figure 1: model resolution and model grid of the coarse agulhas model

Configuration for running.

The model is run in parallel on a number of tiles.  You can generate these tiles by running:
 ./bin/tile_grid.sh 2 2 01
This will give you a 2x2 tiled grid, so the model can be run on 4 processors.

Forcing data:
We do not have all the input files we need to generate model forcing on the CHPC, yet, so for the time being you can just copy the forcing:
The whole folder:
/mnt/lustre/groups/ERTH0904/hycom/AGUa1.00/relax/
/mnt/lustre/groups/ERTH0904/hycom/AGUa1.00/force/
The force-folder contains forcing for the period 1. January 2020 to 1. January 2021, so your simulations must stay within this period.

To prepare your own atmospheric forcing:
NetCDF-files of 6-hourly ERA5 forcing fields have been placed in export /mnt/lustre/groups/ERTH0904/hycom/ERA5_6h/. For the time being it is only 2018, but it will be expanded.
First make sure to go to the HYCOM-code make sure you have the latest version of the code:
git pull

Edit AGUa1.00/REGION.src to include (replaced) the forcing path:
export ERA5_PATH=/mnt/lustre/groups/ERTH0904/hycom/ERA5_6h/ 

Go to your experiment-folder (cd expt_XX.X):

Set start and end-date and run the script that generates the atmospheric forcing, e.g.
If you are just testing, do this on an interactive node:

qsub -I -P ERTH0904 -q serial -l select=1:ncpus=1:mpiprocs=1
Make sure that python is loaded
START="2018-09-15T12:00:00"
END="2018-09-19T12:00:00"
./bin/atmo_synoptic.sh era5_chpc $START $END

Normally it is just done in the submission script by adding:
  # Generate atmospheric forcing :
  atmo_synoptic.sh era5_chpc $START $END 


Before running:

Before running set NMPI to the corresponding number in EXPT.src: export NMPI=4
For the submission script you can find an example here:
https://github.com/nansencenter/NERSC-HYCOM-CICE/blob/issue_hycom_standalone/TP0a1.00/expt_01.1/pbsscript_chpc.sh
-	add you email if you want to receive emails about the run
-	set start and end time by editing ‘START’ and ‘END’ in the file - for the moment I have only made atmospheric forcing for the year 2020, so the start and end date must be within that range.
-	If you do not have a restart-file or want to initialize the model from climatology, set INIT to “--init”.
-	make sure the folder where you write the system error and output messages to exists.

Now everything should be ready  to run, but things may still go wrong, here are some tips.
1.	set the START and the END and INIT in the terminal, then run the script ./bin/exp_preprocess.sh:
START="2020-09-15T12:00:00"
END="2020-09-19T12:00:00"
INITFLG="--init"
./bin/expt_preprocess.sh $START $END $INITFLG

In this case we start the model from rest and climatology of temperature and salinity.  Later, when you have restart-files available you can set:
INITFLG=""
The model will then start from the restart-file.

This will prepare the model for running and tell you if something is wrong.  If you get an error, search for the keyword “FATAL” in the output.

If the model still does not run, check the error/output files from the run.  Often the relevant information is not at the very end of the output.

If you successfully run the model, it is a good idea to backup the region-configureation and experiment configuration to your home directory:
mkdir ~/AGUBKUP
cd $WORK/AGUa1.00
~/HYCOM-CICE/NERSC-HYCOM-CICE/bin/region_backup.sh ~/AGUBKUP/
cd expt_01.0/
~/HYCOM-CICE/NERSC-HYCOM-CICE/bin/expt_config_backup.sh ~/AGUBKUP/

6.1 Additional experiments
Often you would like to keep the present configuration as a reference run, but make additional simulations with different parameters or forcing, in this case you can make additional experiment folders based on the present configurations, from the experiment-folder: 
./bin/expt_new.sh 01.0 01.1
where the fist number is the old experiment number and the second is the new. 

7. Model output
Once the model has run, there should be output-files in your directory: AGUa1.00/expt_01.0/data/

For example: 
restart-files:
AGUrestart.2020_263_12_0000.[ab] 
HYCOM will always write one at the end of the run, otherwise how often the model writes restart-files can be set in blkdat.input:
99999.0  	'rstrfq' = number of days between model restart output

Snapshots of the model, the output frequency can also be set in blkdat.input by setting “'diagfq' = number of days between model diagnostics”
AGUarchv.2020_263_12.[ab] 

Temporal means of the model fields, the time period can also be set in blkdat.input by setting “ 'meanfq' = number of days between model diagnostics (time averaged)”
AGUarchm.2020_263_12.[ab] 

You can always look at the .b files to get information about the contents of these files.
for example: 
more AGUa1.00/topo/depth_AGUa1.00_01.b

8 Visualization
I assume at this point you have successfully run the model, the output will then be in this folder: AGUa1.00/expt_01.0/data/
I you have not run the model, there are example output files in this folder: 
/mnt/lustre/groups/ERTH0904/hycom/Model_output/ the you can copy to AGUa1.00/expt_01.0/data/
8.1 Converting to netcdf-format
You can easily transfer the model output from [ab] to netcdf using  either.
h2nc - transfers the model on the original grid, fast if you want to look at multiple fields
hyc2proj - transfers the model output to z-levels and a map projection of your choice.
8.1.1 Using h2nc
Provided you successfully installed MSCPROGS the routines are here: 
NERSC-HYCOM-CICE/hycom/MSCPROGS/bin/
In the data-directory:
ln -sf ~/NERSC-HYCOM-CICE/hycom/MSCPROGS/bin/h2nc .
cp ~/NERSC-HYCOM-CICE/hycom/MSCPROGS/Input/extract.archv .
mv extract.archv extract.archm
Edit extract.archv to select the output you want, if you want to process the AGUarchm-files, rename it extract.archm. There is an example extract.archm in /mnt/lustre/groups/ERTH0904/hycom/Model_output/ that take out temperature, salinity, layer thickness and and u- and v- velocities in the upper 10 layers, this can be a good starting point.
cp ../../topo/depth_AGUa1.00_01.a regional.depth.a
cp ../../topo/depth_AGUa1.00_01.b regional.depth.b
cp ../../topo/regional.grid.* .
./h2nc ‘list of files’
For example:
./h2nc AGUarchm.2020_2*_12.b

This output a file name tmp1.nc, you can rename to something more descriptive if you like

8.1.2 Using hyc2proj
link to routine
In the data-directory:
ln -sf ~/NERSC-HYCOM-CICE/hycom/MSCPROGS/bin/hyc2proj .
cp ~/NERSC-HYCOM-CICE/hycom/MSCPROGS/Input/extract.archv .
mv extract.archv extract.archm
Edit extract.archv to select the output you want (ore use the one from running h2nc), if you want to process the AGUarchm-files, rename it extract.archm. There is an example extract.arcmh in /mnt/lustre/groups/ERTH0904/hycom/Model_output/ that take out temperature, salinity, layer thickness and and u- and v- velocities in the upper 10 layers, this can be a good starting point.
Copy the grid information
cp ../../topo/depth_AGUa1.00_01.a regional.depth.a
cp ../../topo/depth_AGUa1.00_01.b regional.depth.b
cp ../../topo/regional.grid.* .
cp ../../topo/grid.info .

Configure the grid projection you like, this is configured in a file called proj.in.  In ~/NERSC-HYCOM-CICE/hycom/MSCPROGS/Input, there are examples of mercator, native, regular. and stereographic grids.  Copy the one you want, for example regular grid, to your data directory.
cp ~/NERSC-HYCOM-CICE/hycom/MSCPROGS/Input/proj.in.regular_grid proj.in
Configure the depth levels you would like to extract, first copy the file
cp ~/NERSC-HYCOM-CICE/hycom/MSCPROGS/Input/depthlevels.in .
Then edit this file with the depth levels you want.  

You can run the hyc2proj with different routines to interpolate to the constant depth levels:
A cubic spline (default)
./hyc2proj ‘list of files’
Linear, this if faster
./hyc2proj --vertint linear ‘list of files’
No interpolation, even faster
./hyc2proj --vertint staircase ‘list of files’

8.2 Exploring using ncview
You can of course plot the netcdf files using your favorite plotting tool, ferret, python or matlab, but a good way to quickly assess the results is to explore it using ncview, for example on the output from h2nc:

module load chpc/earth/ncview/2.1.7-gcc
ncview tmp1.nc

8.3 Plotting using the ab-files directly
The ab-files can als be loaded and plotted directly in python or matlab, for ferret, we do not have such a package unfortunately. The reading(and writing) in python, requires the installation of the abfile-package from step 3.  There are also matlab routines available in the github-repository: ~NERSC-HYCOM-CICE/hycom/MSCPROGS/matlab/toolbox/, in the folder My_Loaddata/ you can find routines to load different kind of file.

To run a jupyter notebook on CHPC, follow the example here:https://wiki.chpc.ac.za/tipsntricks:ipython_notebook

I have put two examples on how the model grid and output can be plotted using python here:
/mnt/lustre/groups/ERTH0904/hycom/Notebooks/
 
8.4 Further analysis
The installed MSCPROGS contains several routines for post-processing.  You can read more about this in chapter 6 of this report.

9. Assignment
Run the model for one year, make monthly averages of temperature and salinity on a regular grid using hyc2proj, compare the monthly fields climatology. 

T2M9. Setting up a grid
9.1 Generating the grid

The grid that we use in this version of HYCOM is a grid generated using a conformal mapping tool.

This grid is defined by two poles that are placed outside where you want the grid.  You can find the parameters defining the grid we are currently using in:
/mnt/lustre/groups/ERTH0904/hycom/AGUa1.00/topo/grid.info:

  58.0 -162.0 	! Position of pole A (lattitude, longitude):
  51.0 -162.0  	! Position of pole B (lattitude, longitude):
 179.5  181.2 100 ! Longitude interval in new grid (west limit, east limit, gridsize):
   5.0   78.9 110 ! Lattitude interval in new grid (south limit, north limit, gridsize):
.true.        	! Generate topography
.true.        	! dump teclatlon.dat
.true.        	! dump micom latlon.dat
.true.         	! mercator grid (true, false)
	0.10 .false. 	! merc fac
.false.       	! Smoothing, Shapiro filter
 8   2        	! Order of Shapiro filter,  number of passes

This means that the two poles in this case are places in the North Pacific.  If you are generating a grid in the same region as you are now, you can probably keep the poles there.

The 3rd and 4th line is the limits of the grid (in the transformed grid) and the resolution in each direction.

When you want to set up a new grid frist make a folder called XXXaY.YY, where XXX is three letter of your choice and Y.YY is the approximate resolution in degrees (for example if you want to have a model in the Mozambique Channel with about 1/10 it could be MCHa0.10).
mkdir AGUa0.50
cd AGUa0.50
mkdir topo
cd topo
cp /mnt/lustre/groups/ERTH0904/hycom/AGUa1.00/topo/grid.info .

All the routines needed to make a grid are places here: and locally in your home directory NERSC-HYCOM-CICE/bin/Grid_Bathy/

I have placed bathymetries from GEBCO 2014 and 2021 here: /mnt/lustre/groups/ERTH0904/hycom/bathymetry/, so before you start you must edit the variable “bathyDir” in NERSC-HYCOM-CICE/bin/Grid_Bathy/hycom_bathy.py
 	bathyDir="/mnt/lustre/groups/ERTH0904/hycom/bathymetry/GEBCO_2014/"

If you now run the following two commands you will generate the same grid as we are currently using (it is a good idea to do this on an interactive computational node):
source ~/setupHyCOM
~/NERSC-HYCOM-CICE/bin/Grid_Bathy/hycom_grid.py confmap
~/NERSC-HYCOM-CICE/bin/Grid_Bathy/hycom_bathy.py regional.grid.a --cutoff 5 --input_bathymetry=GEBCO_2014

Cutoff means that the minimum depth in the new bathymetry will be at least 5 meters.  The options for bathymetry is currently GEBCO_2014 and GEBCO_2021.  The input files are places here: /mnt/lustre/groups/ERTH0904/hycom/bathymetry. These two routines generates the .ab-files needed for the grid and some netcdf-files that you can view using ncview:

hycom_bathymetry.nc  
hycom_grid.nc

Now if we edit grid.info and move the two poles 10 degrees to the west, you can see that the grid has moved eastward. And if the poles are moved southward, the grid itself moves northward. If you want to zoom in, adjust, the limits in line 3 and 4. Likewise, if you want higher resolution increase the number of grid-point in each direction, also in line 3 and 4.

These lines in regional.grid.b indicate the minimum and maximum size of the grid:
scpx:  min,max =   	40872.363    	55097.751
scpy:  min,max =   	40872.395    	55097.751

9.2 Finalizing the grid
Once you are happy with the grid location and the resolution, some final things must be done.
1.	Run the grid through a consistency check - this should make sure you have nor single-gridpoint islands or single gridpoint channels. You can can set a bathymetry threshold here:
usage: hycom_bathy_consistency.py [-h] [--no-remove-isolated-basins]
                              	[--no-remove-islets]
                              	[--no-remove-one-neighbour-cells]
                              	[--bathy-threshold BATHY_THRESHOLD]
                              	[--no-remove-inconsistent-nesting-zone]
                              	[--basin-point [BASIN_POINT [BASIN_POINT ...]]]
                              	infile
For example:
~NERSC-HYCOM-CICE/bin/Grid_Bathy/hycom_bathy_consistency.py depth_bathy_01.b

This will create a new grid called depth_CONSISTENT_00.[ab] and a corresponding bathy_consistency.nc.

2.	Inspecting the coastline visually: Finally it is a good idea to inspect the coastline visually by zooming in small regions along the coast.  Look for areas where you have single-point channels etc. Although the routine above should take care of it, it is a good idea to double-check. Areas/points where you wish to change the bathymetry can be changed by passing the grid indices and new depths information to hycom_bathy_modify.py.
3.	If you are nesting in a coarser grid, smooth the outer part of the grid to match the model you are nesting in.  This can be done with hycom_bathy_merge_grids.py
usage: hycom_bathy_merge_grids.py [-h] [--check-consistency]
                              	[--ncells-linear NCELLS_LINEAR]
                              	[--ncells-exact NCELLS_EXACT]
                              	[--bathy-threshold BATHY_THRESHOLD]
                              	infile_coarse gridfile_coarse infile_fine






Once you are happy with your grid, it important to back it up using ~/HYCOM/NERSC-HYCOM-CICE/bin/region_backup.sh (to avoid that it gets deleted from the work-directory)


—
Agenda 4. October

13-13:30: Catch up/ outstanding questions
13:30-14:00 Making atmospheric forcing
14:00-14:15 break
14:15-15:30 Generating new grids.



```
