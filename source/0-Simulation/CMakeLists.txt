#----------------------------------------------------------------------------
# Setup the project
cmake_minimum_required(VERSION 2.6 FATAL_ERROR)
project(starXPproj)

#----------------------------------------------------------------------------
# Find Geant4 package, activating all available Vis drivers by default
# You can set WITH_GEANT4_VIS to OFF via the command line or ccmake/cmake-gui
# to build a batch mode only executable
#
option(WITH_GEANT4_VIS "Build example with Geant4 Vis drivers" ON)
if(WITH_GEANT4_VIS)
  find_package(Geant4 REQUIRED gdml ui_all vis_all)
else()
  find_package(Geant4 REQUIRED gdml)
endif()

#----------------------------------------------------------------------------
##note that for MacOS, no RPATH is allowed for install
##a warning will be given when set INSTALL_RPATH

# for mac
#set(CMAKE_MACOSX_RPATH 1)

# the RPATH to be used when building  
#set(CMAKE_BUILD_RPATH 
#   "./"
#   "/Users/chad/Work/src/programs/geant/geant4.10.03.p03-install/lib/"   
#   "/anaconda3/lib")
#   "/Users/liuqian/Applications/Qt5.6.0/5.6/clang_64/lib")
   
#----------------------------------------------------------------------------
# Setup Geant4 include directories and compile definitions
#
include(${Geant4_USE_FILE})

#----------------------------------------------------------------------------
## Setup ROOT include directories and compile definitions
##
#list(APPEND CMAKE_PREFIX_PATH $ENV{ROOTSYS})
#find_package(ROOT REQUIRED)
#
## check ROOT version
#if(ROOT_FOUND)
#  STRING(REGEX MATCH "6.*" VERSION6MATCH ${ROOT_VERSION})
#  if(NOT VERSION6MATCH)
#    message(FATAL_ERROR "--> ERROR: ROOT 6 is required but not found")
#  endif()
#endif()

include(${ROOT_USE_FILE})

#----------------------------------------------------------------------------
# Find QT (required package)   
#
find_package(Qt5Widgets)

#----------------------------------------------------------------------------
# Locate sources and headers for this project
#
include_directories(${Geant4_INCLUDE_DIR})
include_directories(${ROOTSYS}/include)

MESSAGE( ${PROJECT_SOURCE_DIR} )
MESSAGE( ${CMAKE_SOURCE_DIR} )
include_directories(${PROJECT_SOURCE_DIR}/G4Actions/include)
include_directories(${PROJECT_SOURCE_DIR}/PhysicsList/include)
include_directories(${PROJECT_SOURCE_DIR}/AnalysisManager/include)
include_directories(${PROJECT_SOURCE_DIR}/DetectorConstruction/include)
include_directories(${PROJECT_SOURCE_DIR}/CosmicRayGenerator/include)
include_directories(${PROJECT_SOURCE_DIR}/ParticleSource/include)

# Add common libirary
include_directories(${CMAKE_SOURCE_DIR}/source/CommonLib/SimEvent)

file(GLOB_RECURSE sources "${PROJECT_SOURCE_DIR}/*.cc")
file(GLOB_RECURSE headers "${PROJECT_SOURCE_DIR}/*.hh")

#----------------------------------------------------------------------------
# Generate dictionaries, add ROOT libraries properties
#REFLEX_GENERATE_DICTIONARY(SimEvent ${PROJECT_SOURCE_DIR}/AnalysisManager/include/SimEvent.h SELECTION ${PROJECT_SOURCE_DIR}/AnalysisManager/xml/selection.xml)
#add_library(SimEventDict SHARED SimEvent.cpp)
##add_library(SimEventDict SHARED ${sources} starXP.cc SimEvent.cpp)
#set(libsuffix .so)
#set(ROOT_LIBRARY_PROPERTIES ${ROOT_LIBRARY_PROPERTIES} SUFFIX ${libsuffix})
#set_target_properties(SimEventDict PROPERTIES ${ROOT_LIBRARY_PROPERTIES})
#target_link_libraries(SimEventDict ${ROOT_LIBRARIES})
#target_link_libraries(SimEventDict ${Geant4_LIBRARIES} ${ROOT_LIBRARIES})

#----------------------------------------------------------------------------
# Add the executable, and link it to the Geant4 libraries
#
add_executable(starXP starXP.cc ${sources} ${headers})
target_link_libraries(starXP SimEventDict ${Geant4_LIBRARIES} ${ROOT_LIBRARIES})

#----------------------------------------------------------------------------
# Copy all scripts to the build directory, i.e. the directory in which we
# build My. This is so that we can run the executable directly because it
# relies on these scripts being in the current working directory.

file(GLOB_RECURSE gdml_list RELATIVE ${PROJECT_SOURCE_DIR} "./DetectorConstruction/gdml/*ml")
foreach(gdml ${gdml_list})
  MESSAGE( ${gdml} )
	configure_file(
		${PROJECT_SOURCE_DIR}/${gdml}
		${PROJECT_BINARY_DIR}/gdml/${gdml}
		COPYONLY
		)
endforeach() 

set(SCRIPTS_AND_CARD
SimuCard.card
RootScan.C
README.md
vis.mac
AddCommand.mac
)

foreach(script ${SCRIPTS_AND_CARD})
  configure_file(
    ${PROJECT_SOURCE_DIR}/${script}
    ${PROJECT_BINARY_DIR}/${script}
    COPYONLY
  )
endforeach()

#----------------------------------------------------------------------------
# Add program to the project targets
# (this avoids the need of typing the program name after make)
#
add_custom_target(starXPproj DEPENDS starXP)

#----------------------------------------------------------------------------
# Install the executable to 'bin' directory under CMAKE_INSTALL_PREFIX
#
#set(CMAKE_INSTALL_PREFIX ../StarXP-install) 
#install(TARGETS starXP DESTINATION bin)