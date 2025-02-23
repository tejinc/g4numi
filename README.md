How To Build the main_root6_geant4_11 branch
1. Install root 6 and geant4.11 using conda
2. Get and build dk2nu from https://github.com/NuSoftHEP/dk2nu using CMake, i.e. `mkdir build_dir; cd build_dir; cmake ../; make`
3. Once dk2nu is built, 
    a. copy the content of build_dir/lib to $CONDA_PREFIX/lib
    b. copy the content of dk2nu/tree/ to $CONDA_PREFIX/include/dk2nu/tree
4. Inside the g4numi root folder, do `mkdir build_dir; cd build_dir; cmake ../; make`
