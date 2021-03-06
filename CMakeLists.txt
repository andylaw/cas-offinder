cmake_minimum_required (VERSION 2.6)
project (CAS-OFFinder)

set (CAS-OFFinder_VERSION_MAJOR 2)
set (CAS-OFFinder_VERSION_MINOR 4)

set(CMAKE_BUILD_TYPE Release)

include (CheckIncludeFiles)
check_include_files (dirent.h HAVE_DIRENT_H)

set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/CMakeModules")
if (APPLE)
	find_library(OPENCL_LIBRARY OpenCL)
else (APPLE)
	find_package (OpenCL REQUIRED)
	include_directories ("${OPENCL_INCLUDE_DIR}")
endif (APPLE)
include_directories ("${PROJECT_SOURCE_DIR}")
if (NOT HAVE_DIRENT_H)
	message ("Could not find dirent.h on your system.")
	message ("For workaround, using a custom dirent.h written for MS system (by Toni Ronkko).")
	message ("More information, visit http://softagalleria.net/dirent.php")
	include_directories ("${PROJECT_SOURCE_DIR}/ext")
endif (NOT HAVE_DIRENT_H)

find_package(OpenMP)
if (OPENMP_FOUND)
	set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
	set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif(OPENMP_FOUND)

configure_file (
	"${PROJECT_SOURCE_DIR}/config.h.in"
	"${PROJECT_SOURCE_DIR}/config.h"
)

execute_process(COMMAND ${CMAKE_COMMAND} "-DPROJECT_SOURCE_DIR=${PROJECT_SOURCE_DIR}" -P "${CMAKE_MODULE_PATH}/CopyKernels.cmake")

set (SOURCES
	main.cpp
	cas-offinder.cpp
	read_fasta.cpp
	read_twobit.cpp
	cas-offinder.cl
)

set (HEADERS
	config.h
	cas-offinder.h
	read_fasta.h
	read_twobit.h
	oclfunctions.h
	oclkernels.h
)

add_executable(cas-offinder ${SOURCES} ${HEADERS})
target_link_libraries(cas-offinder ${OPENCL_LIBRARY})
add_custom_command(TARGET cas-offinder PRE_BUILD COMMAND ${CMAKE_COMMAND} "-DPROJECT_SOURCE_DIR=${PROJECT_SOURCE_DIR}" -P "${CMAKE_MODULE_PATH}/CopyKernels.cmake")
