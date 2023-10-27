cmake_minimum_required(VERSION 3.22)
project(prjname)

find_package(CUDAToolkit) # New in version 3.17

#A boolean specifying whether or not the CUDA Toolkit was found.
message(STATUS "CUDAToolkit_FOUND = " ${CUDAToolkit_FOUND})

#The exact version of the CUDA Toolkit found (as reported by nvcc --version, version.txt, or version.json).
message(STATUS "CUDAToolkit_VERSION = " ${CUDAToolkit_VERSION})

#The major version of the CUDA Toolkit.
message(STATUS "CUDAToolkit_VERSION_MAJOR = " ${CUDAToolkit_VERSION_MAJOR})

#The minor version of the CUDA Toolkit.
message(STATUS "CUDAToolkit_VERSION_MINOR = " ${CUDAToolkit_VERSION_MINOR})

#The patch version of the CUDA Toolkit.
message(STATUS "CUDAToolkit_VERSION_PATCH = " ${CUDAToolkit_VERSION_PATCH})

#The path to the CUDA Toolkit library directory that contains the CUDA executable nvcc.
message(STATUS "CUDAToolkit_BIN_DIR = " ${CUDAToolkit_BIN_DIR})

#List of paths to all the CUDA Toolkit folders containing header files required to compile a project linking against CUDA.
message(STATUS "CUDAToolkit_INCLUDE_DIRS = " ${CUDAToolkit_INCLUDE_DIRS})

#The path to the CUDA Toolkit library directory that contains the CUDA Runtime library cudart.
message(STATUS "CUDAToolkit_LIBRARY_DIR = " ${CUDAToolkit_LIBRARY_DIR})

#New in version 3.18.
#The path to the CUDA Toolkit directory containing the nvvm directory and either version.txt or version.json.
message(STATUS "CUDAToolkit_LIBRARY_ROOT = " ${CUDAToolkit_LIBRARY_ROOT})

#The path to the CUDA Toolkit directory including the target architecture when cross-compiling. When not cross-compiling this will be equivalent to the parent directory of CUDAToolkit_BIN_DIR.
message(STATUS "CUDAToolkit_TARGET_DIR = " ${CUDAToolkit_TARGET_DIR})


#The path to the NVIDIA CUDA compiler nvcc. Note that this path may not be the same as CMAKE_CUDA_COMPILER. nvcc must be found to determine the CUDA Toolkit version as well as determining other features of the Toolkit. This variable is set for the convenience of modules that depend on this one.
message(STATUS "CUDAToolkit_NVCC_EXECUTABLE = " ${CUDAToolkit_NVCC_EXECUTABLE})

set(CMAKE_CUDA_COMPILER ${CUDAToolkit_NVCC_EXECUTABLE})

enable_language(CUDA)
add_executable(example example.cu)
