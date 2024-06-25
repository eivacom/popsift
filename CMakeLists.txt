# CMake below 3.4 does not work with CUDA separable compilation at all
cmake_minimum_required(VERSION 3.12)

project(PopSift VERSION 1.0.0 LANGUAGES CXX)

# Set build path as a folder named as the platform (linux, windows, darwin...) plus the processor type
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# make ALL symbols visible on Windows
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS TRUE)

set(PopSift_BUILD_EXAMPLES OFF)
set(PopSift_BUILD_DOCS OFF)
set(PopSift_USE_NVTX_PROFILING OFF)
set(PopSift_ERRCHK_AFTER_KERNEL OFF)
set(PopSift_USE_POSITION_INDEPENDENT_CODE ON)
set(PopSift_USE_GRID_FILTER ON)
set(PopSift_USE_NORMF OFF)
set(PopSift_NVCC_WARNINGS OFF)
set(PopSift_USE_TEST_CMD OFF)
set(PopSift_NO_DEPRECATED_CUDA_SM_WARNINGS ON)

if(MSVC)
  add_compile_definitions(/bigobj)
  string(APPEND CMAKE_C_FLAGS " /FS")
  string(APPEND CMAKE_CXX_FLAGS " /FS")
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake")

# ==============================================================================
# GNUInstallDirs CMake module
# - Define GNU standard installation directories
# - Provides install directory variables as defined by the GNU Coding Standards.
# ==============================================================================
include(GNUInstallDirs)

# Require threads because of std::thread.
find_package(Threads REQUIRED)

###################
#  CUDA
###################
find_package(CUDAToolkit REQUIRED)
enable_language(CUDA)

set(CUDA_SEPARABLE_COMPILATION ON)
set(CMAKE_CUDA_ARCHITECTURES "52;61;72;75;86;89")

set(CMAKE_CUDA_SEPARABLE_COMPILATION ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(PopSift_CXX_STANDARD 14) # Thrust/CUB requires C++14 starting with CUDA SDK 11

if(PopSift_USE_NORMF AND CUDAToolkit_VERSION VERSION_GREATER_EQUAL "7.5")
  set(PopSift_HAVE_NORMF 1)
else()
  set(PopSift_HAVE_NORMF 0)
endif()

if(CUDAToolkit_VERSION VERSION_GREATER_EQUAL "9.0")
  set(HAVE_SHFL_DOWN_SYNC 1)
else()
  set(HAVE_SHFL_DOWN_SYNC 0)
endif()

if(NOT PopSift_USE_GRID_FILTER)
  message(STATUS "Disabling grid filter compilation")
  set(DISABLE_GRID_FILTER   1)
else()
  set(DISABLE_GRID_FILTER   0)
endif()

set(PopSift_USE_NVTX 0)
add_subdirectory(src)