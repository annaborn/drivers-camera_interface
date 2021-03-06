# CMakeLists.txt has to be located in the project folder and cmake has to be
# executed from 'project/build' with 'cmake ../'.

# ${PROJECT_SOURCE_DIR} refers to the folder of the CMakeLists.txt (project)
# ${PROJECT_BINARY_DIR} refers to the folder from which cmake was executed (project/build).


# If the user specifies -DCMAKE_BUILD_TYPE on the command line, take their definition
# and dump it in the cache along with proper documentation, otherwise set CMAKE_BUILD_TYPE
# to Debug prior to calling PROJECT()
SET (CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
INCLUDE(GenerateDoxygenDoc)

if(DEFINED CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "Choose the type of
build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug
Release RelWithDebInfo MinSizeRel.")
else()
   ##### Build types #################################################
   # single-configuration generator like Makefile generator creates following variables per default
   #
   # None (CMAKE_C_FLAGS or CMAKE_CXX_FLAGS used)
   # Debug (CMAKE_C_FLAGS_DEBUG or CMAKE_CXX_FLAGS_DEBUG)
   # Release (CMAKE_C_FLAGS_RELEASE or CMAKE_CXX_FLAGS_RELEASE)
   # RelWithDebInfo (CMAKE_C_FLAGS_RELWITHDEBINFO or CMAKE_CXX_FLAGS_RELWITHDEBINFO
   # MinSizeRel (CMAKE_C_FLAGS_MINSIZEREL or CMAKE_CXX_FLAGS_MINSIZEREL) 
   ####################################################################
   set(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build,
options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release
RelWithDebInfo MinSizeRel.")
endif()

message("Build type set to: " ${CMAKE_BUILD_TYPE})

cmake_minimum_required(VERSION 2.6)

##### Set Project name and version #####################################
project(camera_interface)
set(PROJECT_VERSION 1.0)
set(PROJECT_DESCRIPTION "Dummy project just to show how the infrastructure is working")
##### End Set Project name and version #################################

##### Specification of build directory #################################
# Specifies a common place where CMake should put all the executables.
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
# Specifies a common place where CMake should put all the libraries
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
# Include headers within source
# If you create subdirectories within source include headers with subdirectory, i.e. 
# #include "subdirectory/myheader.h"
include_directories(${PROJECT_SOURCE_DIR}/src)

include(FindPkgConfig)
pkg_check_modules(BASE "base-types" )
include_directories(${BASE_INCLUDE_DIRS})
link_directories(${BASE_LIBDIR})

pkg_check_modules(CV "opencv" )
include_directories(${CV_INCLUDE_DIRS})
link_directories(${CV_LIBRARY_DIR})

##### End specification of build directory ##############################



# Process CMakeLists.txt in the following subdirectory
add_subdirectory(src)

##### COPY Configuration files into build directory
execute_process(COMMAND ${CMAKE_COMMAND} -E copy_directory ${PROJECT_SOURCE_DIR}/configuration ${PROJECT_BINARY_DIR}/configuration)

# Workaround: Cleanup the in file from build directory
execute_process(COMMAND ${CMAKE_COMMAND} -E remove -f ${PROJECT_BINARY_DIR}/configuration/${PROJECT_NAME}.pc.in)

##### INSTALL Configuration ####################################################

# include/
# TODO: recursive copy with directories
install(DIRECTORY ${PROJECT_SOURCE_DIR}/src/ DESTINATION include/${PROJECT_NAME}
	FILES_MATCHING PATTERN "*.h")

# configuration/<projectname>.pc
configure_file(${PROJECT_SOURCE_DIR}/configuration/${PROJECT_NAME}.pc.in
		${PROJECT_BINARY_DIR}/configuration/${PROJECT_NAME}.pc @ONLY)

# Install pkg-config file
install(FILES ${CMAKE_BINARY_DIR}/configuration/${PROJECT_NAME}.pc DESTINATION lib/pkgconfig)

# Dont' forget to add an install for your target, when you create your executable or library
   

##### End INSTALL Configuration ################################################

