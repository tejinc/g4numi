How To Build the main_root6_geant4_11 branch
1. Install root 6 and geant4.11 using conda
2. Inside the g4numi root folder, do `mkdir build_dir; cd build_dir; cmake ../; make`

In principle we can build g4numi with a standalone version of dk2nu:
1. Get and build dk2nu from `https://github.com/NuSoftHEP/dk2nu` using CMake, i.e. `mkdir build_dir; cd build_dir; cmake ../ -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX ; make install`
2. Then modify CMakeLists.txt, and build g4numi with `mkdir build_dir; cd build_dir; DK2NU=$CONDA_PREFIX cmake ../; make`
