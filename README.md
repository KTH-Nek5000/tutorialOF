# tutorialOF
Material for OpenFOAM tutorial, to be used at KTH Engineering Mechanics, e.g. for the CFD course SG2212/SG3114.

## Table of contents

1. [Building instructions for OpenFOAM](https://github.com/KTH-Nek5000/tutorialOF/blob/main/README.md#1-building-instructions-for-openfoam-and-paraview)
2. [Your first case](https://github.com/KTH-Nek5000/tutorialOF#2-your-first-case)
3. [Your second case (with post-processing)](https://github.com/KTH-Nek5000/tutorialOF#3-your-second-case-with-post-processing)
4. [A new solver](https://github.com/KTH-Nek5000/tutorialOF/blob/main/README.md#4-a-new-solver)

## 1. Building instructions for OpenFOAM (and Paraview)

Tested using  [VirtualBox](https://www.virtualbox.org/wiki/Downloads) (recommended). OpenFOAM (and Paraview) will take approx 3.3 Gb once compiled, take this into account.

If you have a native Linux, of course this can be directly used without virtual machine. We tested the below instructions with [Ubuntu 20.04](http://www.releases.ubuntu.com/20.04/). Alternatively, one can also use the free non-commercial version of [VMware](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html).

### 1.1 Install prerequisites for OpenFOAM

> sudo apt-get install build-essential flex libfl-dev bison git-core cmake zlib1g-dev libboost-system-dev libboost-thread-dev libopenmpi-dev openmpi-bin gnuplot libreadline-dev libncurses-dev libxt-dev ptscotch

### 1.2 Install prerequisites for Paraview

> sudo apt-get install libqt5x11extras5-dev libxt-dev qt5-default qttools5-dev curl

### 1.3 create folder and download source code

> cd  
> mkdir OpenFOAM  
> cd OpenFOAM  
> git clone https://github.com/OpenFOAM/OpenFOAM-7.git  
> git clone https://github.com/OpenFOAM/ThirdParty-7.git  

These set of commands should create the folder **OpenFOAM** in your home and, inside this folder, the 2 folders **OpenFOAM-7** and **ThirdParty-7**.

### 1.4 Load environmental variables

> source OpenFOAM-7/etc/bashrc  

### 1.5 (parallel) compilation, for OpenFOAM

> cd OpenFOAM-7  
> ./Allwmake -j  

Note: The building process is fast on a parallel machine, but it will be quite slow on your laptop and even slower on the virtual machine. It can take a couple of hours for OpenFOAM (and a little less for Paraview).

### 1.6 (parallel) compilation, for Third Party and Paraview

> cd ../ThirdParty-7  
> ./Allwmake -j  
> ./makeParaView  

### 1.7 (optional) create an alias

Add at the end your bashrc (**~/.bashrc**) :

> alias of7x="source $HOME/OpenFOAM/OpenFOAM-7/etc/bashrc"

## 2. Your first case

The standard tutorial [cavity](https://github.com/OpenFOAM/OpenFOAM-7/tree/master/tutorials/incompressible/icoFoam/cavity/cavity).

### 2.1. Copy one of the tutorials. 

Loading OpenFOAM bashrc defines a set of variables, such as **$FOAM_TUTORIAL** and **$FOAM_RUN**, that help navigating the code. To run your first tutorial, create the **run** folder, and copy in it the tutorial *cavity* for the incompressible "laminar" solver **icoFoam**. You can use the following commands:

> mkdir -p $FOAM_RUN  
> tut  
> cp -r incompressible/icoFoam/cavity/cavity $FOAM_RUN
> run

Note that the commands **tut** and **run** move to **$FOAM_TUTORIAL** and **$FOAM_RUN**, respectively.

### 2.2 Structure of the case

Entering cavity, you will see the following files and folders
>.  
>├── **0**   
>│   ├── p  
>│   └── U  
>├── **constant**  
>│   └── transportProperties  
>└── **system**  
>    ├── blockMeshDict  
>    ├── controlDict  
>    ├── fvSchemes  
>    └── fvSolution  

Every OpenFOAM case contains:
1) at least one folder for the physical fields, corresponding to a certain simulation time. In this case, the **0/** folder, with the (initial) values of velocity (file **U**) and pressure (file **p**).  
2) one folder, named **constant/**, containing parameters pertaining to the physical properties of the system, such as the viscosity (file **transportProperties**). This folder will also contain the mesh.  
3) one folder, named **system/**, providing the settings for numerical discretization and iterative solvers (files **fvSchemes** and **fvSolution**, respectively), the basic information (file **controlDict**), as well as additional "dictionaries" for pre- or post-processing utilities (in this case, the file **blockMeshDict**).  

### 2.3 Create a Mesh

OpenFOAM provides two pre-processing utilities for mesh creation: **blockMesh**, which creates structured grids, and **snappyHexMesh**, which creates unstructured grids. The tutorial cavity is a simple example of how to use the first. OpenFOAM also provides a list of converters from several formats, such as that of Fluent from the ANSYS package.

The "dictionary" **blockMeshDict** contains the instructions for **blockMesh**. In this file, the user must specify:  
1. a list of "vertices", including their coordinates. Note that each vertex will be identified later on with its position in this list.   
2. a list of the "blocks" which constitutes the mesh. In this example, you see only one hexahedral block, which will contain 20 cells in the x and y directions, and one cell in the z direction. Note that you can use different cell spacing. In this case, the mode **simpleGrading** and the option (1,1,1) will give uniform spacing.  
3. a list of "edges", with appropriate definition, if the sides of the blocks have a particular shape. In this case, since the side of the block are plane, no edges are required.
4. a list of "boundaries", which will correspond to the faces of the block where it is needed to impose boundary conditions. Each boundary requires:
       1. a "name", which is arbitrary.  
       2. a "type", which must correspond to one of the types requested by boundary conditions.  
       3. the list of the block(s) "faces" which belong to the boundary, which are defined based on their vertices. Note that the vertices order is not arbitrary, but it must be such that they are in counter-clockwise order looking from outside the domain.  
5. a list of "merging patch pairs", which is empty in this case.

You can find more information on **blockMesh** [here](https://cfd.direct/openfoam/user-guide/v7-blockMesh/).  
For more advanced examples of its capabilities, I suggest having a look at [this tutorial](http://www.wolfdynamics.com/wiki/meshing_OF_blockmesh.pdf) (from *Wolfdynamics*).

Assuming a proper **blockMeshDict** is provided and located in the system folder, use the command:
> blockMesh  

to create the mesh in a dedicated folder (**constant/polyMesh/**).

Note that, after creating a mesh, either with **blockMesh**, **snappyHexMesh** or from a converter, it is good practice to use the command **checkMesh**, which provides information about the mesh quality (in this case, it will be of no concern).

### 2.4 Boundary conditions and Physical properties

Initial and boundary conditions in OpenFOAM are assigned together, using the same files that also contain the values of the physical variables. Such files have a common structure and are located in folders named after the corresponding time step - typically **0/** for the initial condition.

In this tutorial, the pressure, **p**, and the velocity, **U**, are the only variables.
Note that physical vectors (such as **U**) are always three-dimensional in OpenFOAM. Running two-dimensional cases, such as "cavity", does not alter the dimensionality of vectors, but prevent solving the equations in one of the directions, which is identified by the keyword empty in the boundary conditions.
Each of the "variable" files contain:
1. the physical dimension of the variable, in units of the International System (second, metre, kilogram, ampere, kelvin, mole, candela). Note that each scalar or vector field is defined with its dimensions. Nonetheless, it is always possible to use OpenFOAM using "dimensionless" variables giving scaled values (as in this example).
2. the value of the variable in the domain (**internalField**). In this example, the fluid is at rest at t=0.
3. the boundary conditions, defined for each of the "boundaries" created in **blockMesh**.

Physical properties of the fluid, such as viscosity and density, are set in the dictionary transportProperties in the constant folder (in this case, only the kinematic viscosity is needed).


### 2.5 General settings
In every OpenFOAM case, three files in the system folder determine general settings of the simulation:
1. The file **controlDict**. In this file, you can specify: start and end times of the simulation, time interval and output frequency, format and precision. In this example, the simulation will start at t=0 and end at t=0.5, using a time interval of 0.005 and writing a full outpost in ASCII format every 20 time steps. For more information on the specific parameters, I recommend consulting the [user guide](https://cfd.direct/openfoam/user-guide/v6-controldict/).
2. The file **fvSchemes**. In this file, you can specify numerical schemes for the approximation of each derivate and the interpolation. In particular, the choice of the numerical scheme for the time derivate (**ddtSchemes**) determines if the simulation is of a steady-state solution (setting **steadyState**) or time-dependent (setting, for example, **Euler**). Note that it is possible to set a default scheme as well as specific ones for any variable.
3. The file **fvSolution**. In this file, you can specify the iterative solvers used for matrix inversion as well as the number of specific settings of the algorithm used by the application (in this example, the solver **icoFoam**, which employes the [PISO algorithm](https://openfoamwiki.net/index.php/OpenFOAM_guide/The_PISO_algorithm_in_OpenFOAM) to solve the incompressible Navier-Stokes equation). 

### 2.6 Run and (simple) post-processing
To run the simulation, assuming that boundary and initial conditions are correct, as well as physical parameters and numerical settings, means to use the proper application. In this tutorial, you can simply use:
> icoFoam  

The log contains, for each time step:
1. the mean and maximum Courant number.
2. the residual and number of iterations for each of the iterative solvers. In particular, for the two components of the velocity and the pressure. The number of solution for the pressure (two, in this example) are the number of corrections of the PISO algorithm.
3. the estimate of the continuity error.
4. the time elapsed from the beginning of the simulation.

The tutorial cavity with its default settings will create an outpost of five times (0.1, 0.2, 0.3, 0.4 and 0.5), which you can see after running the simulation.

### Additional material:
If you are interested in OpenFOAM, take advantage of the documentation available online. In particular, I recommend the official websites of different distributions of the code, such as [OpenFOAM foundation](https://openfoam.org/resources/) and [ESI-OpenCFD](https://openfoam.com/documentation/). You may also find of great interest tutorials created by consultants such as [Wolf Dynamics](http://www.wolfdynamics.com/tutorials.html).

## 3. Your second case (with post-processing)

This case is based on the tutorial [pitzDaily](https://github.com/OpenFOAM/OpenFOAM-7/tree/master/tutorials/incompressible/simpleFoam/pitzDaily) ([Pitz and Daily, 1983](https://doi.org/10.2514/3.8290)) and the post-processing is based on that of the tutorial [T3A](https://github.com/OpenFOAM/OpenFOAM-7/tree/master/tutorials/incompressible/simpleFoam/T3A). 

### 3.1 Structure of the case

The structure of the test case is:

>.  
>├── **0**  
>│   ├── epsilon  
>│   ├── f  
>│   ├── k  
>│   ├── nut  
>│   ├── nuTilda  
>│   ├── omega  
>│   ├── p  
>│   ├── U  
>│   └── v2  
>├── **constant**  
>│   ├── transportProperties  
>│   └── turbulenceProperties  
>├── run.sh  
>└── **system**  
>    ├── blockMeshDict  
>    ├── controlDict  
>    ├── decomposeParDict  
>    ├── fvSchemes  
>    ├── fvSolution  
>    ├── streamlines  
>    └── wallShearStressGraph  

* In this case, we will use the solver **simpleFoam**, which uses the [SIMPLE algorithm](https://openfoamwiki.net/index.php/The_SIMPLE_algorithm_in_OpenFOAM) to solve the incompressible N.S. eq., also including turbulence models. Note that this algorithm is typically employed to find steady-state solutions: **simpleFoam** is, therefore, the most common solver to perform RANS with OpenFOAM.
* A turbulence model has to be selected in **turbulenceProperties**. Note that appropriate boundary and initial conditions for the variables of the model (in **0/**), as well as numerical schemes and algorithms for the corresponding transport equations (in **fvSchemes** and **fvSolution**, respectively), are required as well.  
* The dictionary **decomposeParDict** is required by the pre-processing utility **decomposePar**, which is employed for partitioning the domain (various algorithms are available).
* The additional files streamlines and wallShearStressGraph are not actual dictionaries in a strict sense, and are included directly 

### 3.2 Run the case

You can run the case using the script **run.sh** in the working directory. **getApplication**, **runApplication**, and **runParallel** are standard utilities for OpenFOAM tutorials. Note that:
* The result of **getApplication** corresponds to the "application" value in **controlDict**.
* OpenFOAM executables requires the **parallel** option to use multiple cores, thus the **runParallel** call in **run.sh** corresponds to:
> mpirun -n $NCORES simpleFoam -parallel > log.simpleFoam
* The last line in the script **run.sh** is an example of how to use a solver in post-processing mode, and for the last time step only. In post-processing mode, the governing equations are not solved, but the functions in **controlDict** are evaluated. Note that this line is not redundant: with the current setup, the post-processing functions are called only for the standard outpost (determined by the **writeInterval** value in **controlDict**), but not for the time step when the simulation stops (determined by the **residualControl** values in **fvSolution**).

### 3.3 Post-processing

Three post-processing functions are employed in this tutorial:
1. **wallShearStress**, which computes the "wallShearStress" field for each saved time step, and does not require to set any parameter. The definition of this field is described, e.g., [here](https://www.openfoam.com/documentation/guides/latest/doc/guide-fos-field-wallShearStress.html);
2. **sample**, which interpolates a given field on a set of points. In this case, the default dictionaries for this function ([sampleDict.cfg](https://github.com/OpenFOAM/OpenFOAM-7/blob/master/etc/caseDicts/postProcessing/graphs/sampleDict.cfg) and [graph.cfg](https://github.com/OpenFOAM/OpenFOAM-7/blob/master/etc/caseDicts/postProcessing/graphs/graph.cfg)) are used to create a new function called **wallShearStressGraph**, which samples the "wallShearStress" field along a line. 
3. **streamlines**, which compute a list of streamlines and outpost the results as a point set in VTK format. In this case, the file **streamlines** contains the set of option required by this function, part of which are the default ones ([here](https://github.com/OpenFOAM/OpenFOAM-7/blob/master/etc/caseDicts/postProcessing/visualization/streamlines.cfg)).

Note that when the result a post-processing function is a field (i.e. it is defined in principle for each grid point), it is written in the time-step folders. On the other hand, if it is defined on a set of points, it is written in a dedicated folder in **postProcessing/**. 

A list of the possible post-processing functions that can be used in this way is provided [here](https://cfd.direct/openfoam/user-guide/v7-post-processing-cli/#x32-2400006.2.1).

## 4. A new solver

https://github.com/AtzoriMarco/pisoKinematicParcelFoam

This solver is created combining [pisoFoam](https://github.com/OpenFOAM/OpenFOAM-7/tree/master/applications/solvers/incompressible/pisoFoam) and [icoUncoupledKinematicParcelFoam](https://github.com/OpenFOAM/OpenFOAM-7/tree/master/applications/solvers/lagrangian/icoUncoupledKinematicParcelFoam).



