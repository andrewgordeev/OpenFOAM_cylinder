# OpenFOAM_cylinder

This repository contains four OpenFOAM case files, which can be readily modified to model any of the configurations presented in my Physics 814 report. A working installation of OpenFOAM (such as from https://openfoam.org/download/) is required to run these. See https://www.openfoam.com/documentation/user-guide for general OpenFOAM instructions.

**Running a case**
---------------

To run any given case, simply run the `Allrun` script. This will:

1. Initialize flow quantities at the start time with whatever is given in the `initial` folder.
2. Build the mesh defined by `system/blockMeshDict`.
3. Divide the mesh into regions designated for each processor if using parallel processing (this step is irrelevant if not). Each region will correspond to a folder in the case directory called `processor0`, `processor1`, etc...
4. Run `pimpleFOAM` with the current settings. **To use parallel processing, comment out** `runApplication $application` **and uncomment** `runParallel $application`.
5. If using a dynamic mesh together with parallel processing, there is an additional step here to reconstruct the mesh after it has been updated within each region.
6. Combine results from each region to generate folders containing data for each timestep in the main case directory. If not using parallel processing, those folders will be produced in the main directory in the first place.
7. Calculate the vorticity at each timestep. 
8. Create a dummy file called `cylinder.OpenFOAM` for ParaView.

**Analyzing data**
----------------
ParaView should have been automatically installed when installing OpenFOAM. To visualize the results of a simulation, once `Allrun` has finished, enter `paraview cylinder.OpenFOAM` in the command line. 

Here is a simple example using ParaView:

1. In the bottom left _Properties_ panel of ParaView, select any desired _Mesh Parts_ and _Volume Fields_ to visualize (usually I select all) and click the green _Apply_ button. 
2. Under the _Display (GeometryRepresentation)_ section of the _Properties_ panel, select a desired _Representation_ of the mesh (surface, wireframe, etc.) and _Coloring_ (u_x, p magnitude, etc.). This is a simple way to see the value of any calculated field across the mesh. 
3. The 3D model of the mesh can be freely rotated or zoomed in on. In one of the top bars are green arrows that allow you to change timesteps, automatically updating the visualization.
 
ParaView has a large range of visualization options. See https://www.openfoam.com/documentation/user-guide/7-post-processing/7.1-parafoam or https://www.openfoam.com/documentation/tutorial-guide/2-incompressible-flow/2.1-lid-driven-cavity-flow for more detailed examples of using ParaView with OpenFOAM.

**Running more cases**
----------------
To run again in the same case directory (perhaps with different settings), the results from a previous run must be cleared first. This can be done easily by running the `Allclean` script. **Be sure to save any desired data outside of the directory if you do not want them to be wiped.**

An easy way to try out different settings is to copy a directory for each run, so that there is no need to wipe the data from any of them.

Here is a brief summary of the folders and files in each case directory, emphasizing the parts that are modified to produce the results of this study:

**`constant`**
-----------

`dynamicMeshDict` is only necessary when using a dynamically updated mesh. This dictionary contains all parameters governing mesh refinement.

`transportProperties` simply defines the kinematic viscosity, nu. Change this to simulate flow at different Reynolds numbers.

`turbulenceProperties` defines whether the simulation is laminar or turbulent: `simulationType` should be `laminar` for laminar simulations and `RAS` for turbulent ones. The `RAS` dictionary defines the turbulence model, which is set here to `kOmegaSST` in the turbulent case.

**`initial`**
-----------

This folder contains the initial values of all flow variables. Each must have defined physical dimensions and defined behavior at all boundaries: the edges of the mesh as well as the cylinder boundary.

`U` is the flow velocity, by default set to 5 m/s in the x direction in all cases.

`p` is the pressure, by default set to be initially zero everywhere.

`k`, `omega`, and `nut` are the fields used by the `kOmegaSST` model. They are only relevant when turbulence is enabled, and may need to be played with to get good results.

**`system`**
-----------

`blockMeshDict` defines the mesh used by the simulation (the initial mesh if using dynamic meshing). See https://www.openfoam.com/documentation/user-guide/4-mesh-generation-and-conversion/4.1-mesh-description for details.

The four static mesh files I used are also present in this folder so that any one of them can easily be copied over `blockMeshDict` to use it as the new mesh dictionary:

`blockMeshDictCoarse` - coarse cylindrical mesh.

`blockMeshDictFine` - fine cylindrical mesh.

`blockMeshDictCoarseCart` - coarse Cartesian mesh.

`blockMeshDictFineCart` - fine Cartesian mesh.

`controlDict` defines the parameters controlling the run, such as the runtime, timestep, write instructions, and write precision. Of particular note: `adjustTimeStep` allows the timestep to be reduced to keep the Courant number below the value set by `maxCo`. See https://www.openfoam.com/documentation/user-guide/6-solving/6.1-time-and-data-inputoutput-control for details.

`decomposeParDict` defines the parameters of the mesh division if using parallel processing. The most important parameter here is `numberOfSubdomains`, which corresponds to how many processors you want to use. **By default, parallel processing is done with 8 processors; be sure to change this parameter if you want to use a different number.**

`fvSchemes` defines all numerical schemes used in the simulation. See https://www.openfoam.com/documentation/user-guide/6-solving/6.2-numerical-schemes for details.

`fvSolutions` defines the equation solvers used for each fluid quantity, tolerances, and the main algorithm (in this case PIMPLE). Within the PIMPLE subdictionary, you can define the number of each type of corrector you wish to use. See https://www.openfoam.com/documentation/user-guide/6-solving/6.3-solution-and-algorithm-control for details.
