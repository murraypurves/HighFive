IF(COMMAND TRIBITS_PACKAGE)
  TRIBITS_PACKAGE(HighFive)
  SET(HighFive_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
  INCLUDE_DIRECTORIES("${CMAKE_CURRENT_SOURCE_DIR}/include/")
  option(BUILD_SHARED_LIBS "Build shared libraries (DLLs)." ${BUILD_SHARED_LIBS})
ELSE()
  project(HighFive)
  cmake_minimum_required(VERSION 3.0)
ENDIF()

enable_testing()

set(HIGHFIVE_VERSION_MAJOR 2)
set(HIGHFIVE_VERSION_MINOR 0)

IF(COMMAND TRIBITS_PACKAGE)
  list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/CMake
    ${CMAKE_CURRENT_SOURCE_DIR}/CMake/portability
    ${CMAKE_CURRENT_SOURCE_DIR}/CMake/packages
    ${CMAKE_CURRENT_SOURCE_DIR}/CMake/config)
ELSE()
  list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/CMake
    ${PROJECT_SOURCE_DIR}/CMake/portability
    ${PROJECT_SOURCE_DIR}/CMake/packages
    ${PROJECT_SOURCE_DIR}/CMake/config)
ENDIF()

include(DefineInstallationPaths)
include(ReleaseDebugAutoFlags)
include(CheckCXX11Portability)

include(BlueGenePortability)

option(HIGHFIVE_UNIT_TESTS "enable unit tests" TRUE)
option(USE_BOOST "enable Boost Support" TRUE)
option(HIGHFIVE_EXAMPLES "Compile examples" TRUE)
option(HIGHFIVE_PARALLEL_HDF5 "Enable Parallel HDF5 support" FALSE)

set(HIGHFIVE_CPP_STD_FLAG "-std=c++11" CACHE STRING "flag to use to enforce c++ standard (default:c++11)")

if(HIGHFIVE_PARALLEL_HDF5)
    set(HDF5_PREFER_PARALLEL TRUE)
endif()

find_package(HDF5 QUIET REQUIRED)

set(Boost_NO_BOOST_CMAKE TRUE)

if(USE_BOOST)
  find_package(Boost QUIET REQUIRED COMPONENTS system unit_test_framework)
  add_definitions(-DH5_USE_BOOST )
  include_directories(SYSTEM ${Boost_INCLUDE_DIR})
endif()

if(HIGHFIVE_PARALLEL_HDF5)
  find_package(MPI)
endif()

if(NOT COMPILER_SUPPORTS_CXX11)
    message(FATAL "HighFive version >=2.0 requires c++ standard >= c++11")
endif()

# Absolute include used to avoid interaction with downstream projects including
# HighFive as a subproject and using the the same filename
IF(COMMAND TRIBITS_PACKAGE)
  include(${CMAKE_CURRENT_SOURCE_DIR}/CMake/PackageConfig.cmake)
ELSE()
  include(${PROJECT_SOURCE_DIR}/CMake/PackageConfig.cmake)
ENDIF()

if(HIGHFIVE_EXAMPLES)
  add_subdirectory(src/examples)
endif()

add_subdirectory(tests/unit)
add_subdirectory(doc)

IF(COMMAND TRIBITS_PACKAGE_POSTPROCESS)
  TRIBITS_PACKAGE_POSTPROCESS()
ENDIF()
