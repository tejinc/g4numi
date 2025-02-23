cmake_minimum_required(VERSION 3.10 FATAL_ERROR)
project(g4numi)

# Options
option(WITH_MODERN_G4 "Build against modern GEANT4 with 1-arg exception" ON)
option(WITH_DK2NU     "Build against Dk2Nu" OFF)
option(WITH_G4NU      "Build against G4NuPhysicsLists" OFF)
option(WITH_GEANT4_UIVIS "Build with GEANT4 UI and Vis drivers" ON)

# Set C++ standard and compiler flags
# set(CMAKE_CXX_STANDARD 17)
# set(CMAKE_CXX_STANDARD_REQUIRED ON)
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w")

#----------------------------------------------------------------------------
# Find GEANT4
#
if(WITH_GEANT4_UIVIS)
  find_package(Geant4 REQUIRED ui_all vis_all)
else()
  find_package(Geant4 REQUIRED)
endif()

# Include GEANT4 settings
include(${Geant4_USE_FILE})

#----------------------------------------------------------------------------
# Find CLHEP
#
find_package(CLHEP REQUIRED)

#----------------------------------------------------------------------------
# Find ROOT
#
find_package(ROOT REQUIRED 
  COMPONENTS 
  Core 
  RIO 
  Hist
  Net
  Graf
  Graf3d
  Gpad
  Tree
  Rint
  Postscript
  Matrix
  Physics
  MathCore
  Thread
  EG
)

# Include ROOT settings
include(${ROOT_USE_FILE})

#----------------------------------------------------------------------------
# Include directories
#
include_directories(${PROJECT_SOURCE_DIR}/include)
include_directories(${Geant4_INCLUDE_DIR}/..)  # For #include "Geant4/file.h"
include_directories(${Geant4_INCLUDE_DIRS})    # Standard GEANT4 paths
include_directories($ENV{ROOTSYS}/include)     # ROOT include directory

#----------------------------------------------------------------------------
# Link directories
#
link_directories($ENV{ROOTSYS}/lib)

#----------------------------------------------------------------------------
# Optional dependencies: DK2NU and G4NU
#
if(WITH_DK2NU)
  set(DK2NU $ENV{DK2NU})
  include_directories(${DK2NU}/include/dk2nu/tree)
  link_directories(${DK2NU}/lib)
  set(DK2NU_LIBRARIES dk2nuTree)
endif()

if(WITH_G4NU)
  set(G4NU $ENV{G4NU})
  include_directories(${G4NU}/include)
  link_directories(${G4NU}/lib)
  set(G4NU_LIBRARIES g4nupl)
endif()

#----------------------------------------------------------------------------
# Locate sources and headers
#
file(GLOB sources ${PROJECT_SOURCE_DIR}/src/*.cc)
if(WITH_DK2NU)
  file(GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hh) 
else()
  file(GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hh ${PROJECT_SOURCE_DIR}/include/*.h) 
endif()
file(GLOB cintheaders ${PROJECT_SOURCE_DIR}/include/*_t.hh ${PROJECT_SOURCE_DIR}/include/*.h)

# Exclude unwanted files
file(GLOB rmsources1 ${PROJECT_SOURCE_DIR}/src/NA49*)
file(GLOB rmsources2 ${PROJECT_SOURCE_DIR}/src/dk*)
file(GLOB rmsources3 ${PROJECT_SOURCE_DIR}/src/g4numiCint*)
file(GLOB rmheaders1 ${PROJECT_SOURCE_DIR}/include/NA49*)
file(GLOB rmheaders2 ${PROJECT_SOURCE_DIR}/include/dk*)
file(GLOB rmheaders3 ${PROJECT_SOURCE_DIR}/include/g4numiCint*)

if(WITH_DK2NU)
  list(REMOVE_ITEM sources ${rmsources1} ${rmsources2} ${rmsources3})
  list(REMOVE_ITEM headers ${rmheaders1} ${rmheaders2} ${rmheaders3})
else()
  list(REMOVE_ITEM sources ${rmsources1} ${rmsources3})
  list(REMOVE_ITEM headers ${rmheaders1} ${rmheaders3})
endif()

#----------------------------------------------------------------------------
# Generate ROOT dictionary
#
if(EXISTS ${PROJECT_SOURCE_DIR}/LinkdefModern.h)
  set(g4numi_LINKDEF ${PROJECT_SOURCE_DIR}/LinkdefModern.h)
else()
  set(g4numi_LINKDEF ${PROJECT_SOURCE_DIR}/Linkdef.h)
endif()

set(g4numi_DICTIONARY ${CMAKE_CURRENT_BINARY_DIR}/g4numiDict)
ROOT_GENERATE_DICTIONARY(${g4numi_DICTIONARY} ${cintheaders} LINKDEF ${g4numi_LINKDEF})
set(sources ${sources} ${g4numi_DICTIONARY})

#----------------------------------------------------------------------------
# Add executable and link libraries
#
add_executable(g4numi g4numi.cc ${sources} ${headers})
target_link_libraries(g4numi ${Geant4_LIBRARIES} ${ROOT_LIBRARIES} ${DK2NU_LIBRARIES} ${G4NU_LIBRARIES})

# Add compile definitions based on options
if(WITH_DK2NU)
  target_compile_definitions(g4numi PRIVATE WITH_DK2NU)
endif()

if(WITH_G4NU)
  target_compile_definitions(g4numi PRIVATE WITH_G4NU)
endif()

if(WITH_MODERN_G4)
  target_compile_definitions(g4numi PRIVATE MODERN_G4)
endif()

#----------------------------------------------------------------------------
# Copy scripts to build directory
#
set(EXAMPLEG4NUMI_SCRIPTS
# me000z200i3.mac
)

foreach(_script ${EXAMPLEG4NUMI_SCRIPTS})
  configure_file(
    ${PROJECT_SOURCE_DIR}/${_script}
    ${PROJECT_BINARY_DIR}/${_script}
    COPYONLY
  )
endforeach()

#----------------------------------------------------------------------------
# Custom target for internal use
#
add_custom_target(G4NUMI DEPENDS g4numi)

#----------------------------------------------------------------------------
# Install the executable
#
install(TARGETS g4numi DESTINATION bin)
