cmake_minimum_required(VERSION 3.11...3.14)
project(ratpac VERSION 1.0.1
                DESCRIPTION "Watchman RAT-PAC"
                LANGUAGES C CXX Fortran)

###########################################################
# Require out-of-source builds
file(TO_CMAKE_PATH "${PROJECT_BINARY_DIR}/CMakeLists.txt" LOC_PATH)
if(EXISTS "${LOC_PATH}")
    message(FATAL_ERROR "You cannot build in a source directory (or any directory with a CMakeLists.txt file). Please make a build subdirectory. Feel free to remove CMakeCache.txt and CMakeFiles.")
endif()


###########################################################
# Set default build type
set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

###########################################################
# Set CMAKE standards
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

###########################################################
# Options
set(RATPAC_BINARY_DIR ${PROJECT_BINARY_DIR}/bin CACHE PATH "Where to output executables")
set(RATPAC_LIBRARY_DIR ${PROJECT_BINARY_DIR}/lib CACHE PATH "Where to output library files")
set(RATPAC_INCDIR ${PROJECT_BINARY_DIR}/include CACHE PATH "Where to collect headers")
option(RATPAC_BONSAI "build bonsai tool" ON)
option(RATPAC_ROOT "build bootstrapped root executable" ON)
option(RATPAC_RAT "build rat executable" ON)

###########################################################
# Set up third party libraries

find_package(ROOT CONFIG REQUIRED COMPONENTS Minuit2 PyROOT)
include(${ROOT_USE_FILE})

find_package(Geant4 REQUIRED COMPONENTS qt)
include(${Geant4_USE_FILE})

find_package(Threads REQUIRED)


###########################################################
# Configure header with our version information
configure_file(${CMAKE_SOURCE_DIR}/config/Config.hh.in
        ${CMAKE_SOURCE_DIR}/src/core/include/RAT/Config.hh
        @ONLY NEWLINE_STYLE UNIX)

# Configure local scripts
set(RATROOT ${PROJECT_BINARY_DIR})
set(RATSHARE ${PROJECT_SOURCE_DIR})
configure_file(${CMAKE_SOURCE_DIR}/config/ratpac.sh.in
       ${PROJECT_SOURCE_DIR}/ratpac.sh
       @ONLY NEWLINE_STYLE UNIX)
configure_file(${CMAKE_SOURCE_DIR}/config/ratpac.csh.in
        ${PROJECT_SOURCE_DIR}/ratpac.csh
       @ONLY NEWLINE_STYLE UNIX)

###########################################################
# Compile our libraries and executables

add_subdirectory(src)
add_subdirectory(tools)

###########################################################
# Installation

# Configure global scripts
set(RATROOT ${CMAKE_INSTALL_PREFIX})
set(RATSHARE ${CMAKE_INSTALL_PREFIX}/share/RAT)
configure_file(${CMAKE_SOURCE_DIR}/config/ratpac.sh.in
       ${RATPAC_BINARY_DIR}/ratpac.sh
       @ONLY NEWLINE_STYLE UNIX)
configure_file(${CMAKE_SOURCE_DIR}/config/ratpac.csh.in
       ${RATPAC_BINARY_DIR}/ratpac.csh
       @ONLY NEWLINE_STYLE UNIX)

# Install the data files
install(DIRECTORY data/ DESTINATION ${RATSHARE}/data
        PATTERN "data/*")

# Install the python files
install(DIRECTORY python/ DESTINATION ${RATSHARE}/python
        PATTERN "python/*")

# Install the mac files
install(DIRECTORY mac/ DESTINATION ${RATSHARE}/mac
        PATTERN "mac/*")

# Install the doc files
install(DIRECTORY doc/ DESTINATION ${RATSHARE}/doc
        PATTERN "doc/*")

# Install env files
install(FILES ${RATPAC_BINARY_DIR}/ratpac.sh
             ${RATPAC_BINARY_DIR}/ratpac.csh
       DESTINATION bin)
