# How does cmake find CUDA

**REM**: Below we describe the algorithm used by cmake to find CUDA. If you need the express solution, please, jump to the [solution](#solution).

When cmake encounters `enable_language(CUDA)` instruction it starts the chain of scripts to find CUDA components. The most important components are the following:


* executable files: nvcc compiler itself, nvlink linker and fatbinary linker;
* folders where CUDA headers and libraries are placed.

cmake follows some rules while searching CUDA. These rules are:

1. First, check variables that can help to locate CUDA. These variables can be passed through command line, cmake cache or environment variables.
2. Next, it tries to locate nvcc using general purpose utility `find_program`. If `find_program` succeeded, cmake stops searching CUDA and jumps to the next stage that is locating CUDA components based on the known location of the nvcc compiler.
3. Finally, cmake tests standard pathes where CUDA can be installed (on linux this is `/usr/local/cuda-X.Y`, this is a path where CUDA is installed if installer have been downloaded from the official NVIDIA site).

When nvcc compiler have been already located the location of other components is determined with `nvcc -v fakefile` command, where `fakefile` is the name of a non-existent file.
The output of such command is like this:

```console
tea@comp:~/projects/findcuda/build$ /usr/local/cuda/bin/nvcc -v fakefile
#$ _NVVM_BRANCH_=nvvm
#$ _SPACE_= 
#$ _CUDART_=cudart
#$ _HERE_=/usr/local/cuda/bin
#$ _THERE_=/usr/local/cuda/bin
#$ _TARGET_SIZE_=
#$ _TARGET_DIR_=
#$ _TARGET_DIR_=targets/x86_64-linux
#$ TOP=/usr/local/cuda/bin/..
#$ NVVMIR_LIBRARY_DIR=/usr/local/cuda/bin/../nvvm/libdevice
#$ LD_LIBRARY_PATH=/usr/local/cuda/bin/../lib:
#$ PATH=/usr/local/cuda/bin/../nvvm/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/snap/bin
#$ INCLUDES="-I/usr/local/cuda/bin/../targets/x86_64-linux/include"  
#$ LIBRARIES=  "-L/usr/local/cuda/bin/../targets/x86_64-linux/lib/stubs" "-L/usr/local/cuda/bin/../targets/x86_64-linux/lib"
#$ CUDAFE_FLAGS=
#$ PTXAS_FLAGS=
nvcc fatal   : Don't know what to do with 'fakefile'
```
**Rem.**: `-v` key means 'verbose'

**Rem.** The only reason to substitute`fakefile` is to initiate compilation process, which starts from printing the location of basic CUDA components.

cmake parses the output of such command and finds the string containing `TOP=<CUDA_ROOT>`, this directory is treated as the directory where CUDA components have been installed.


# Why doesn't a symlink work

When we create symlink named `nvcc` in `/usr/bin` pointing to the nvcc executable, the utility `find_program` finds this link and cmake stops searching CUDA. Then cmake tries to locate
other components of CUDA by running `cmake -v fakefile` which produce something like the this

```
tea@comp:~/projects/findcuda$ /usr/bin/nvcc -v fakefile
#$ _NVVM_BRANCH_=nvvm
#$ _SPACE_= 
#$ _CUDART_=cudart
#$ _HERE_=/usr/bin
#$ _THERE_=/usr/bin
#$ _TARGET_SIZE_=
#$ _TARGET_DIR_=
#$ _TARGET_SIZE_=64
nvcc fatal   : Don't know what to do with 'fakefile'
```

One can see that if compiler is called via symlink it can't determine the location of CUDA components (compare with previous output example). If we try to compile some existent file instead of `fakefile`, the compiler will fail because it doesn't know where to search CUDA include files and libraries (the correct pathes should be passed as command-line parameters using keys -I and -L). But the next problem is that nvcc doesn't know how to find nvlink. Next, anyone can create a system-wide symlink to nvlink. We didn't try this because we have a better solution.

# Solution

## Approach 1. Use findCUDAToolkit package

Use findCUDAToolkit package which is a successor of the deprecated findCUDA package. The findCUDA algorithms have been improved over the years and they migrated to the new findCUDAToolkit package. So, the new findCUDAToolkit package should have very strong capabilities to find CUDA toolkit. 

In contrast, the standard cmake `enable_language(CUDA)` command will not find CUDA if it is installed in `/usr/local/cuda-X.Y` directory. We don't know why, but we can say that CUDA searching scripts have been changed significantly between cmake 3.27 and 3.28 rc2, so we can conclude that these algorithms are not stable yet.

To use findCUDAToolkit package write the following code:

```CMakeLists.txt
cmake_minimum_required(VERSION 3.22)
project(prjname)

find_package(CUDAToolkit) # New in version 3.17
message(STATUS "CUDAToolkit_NVCC_EXECUTABLE = " ${CUDAToolkit_NVCC_EXECUTABLE})
set(CMAKE_CUDA_COMPILER ${CUDAToolkit_NVCC_EXECUTABLE})

enable_language(CUDA)
add_executable(aaa aaa.cu)
```

Note, that you should remove symlink `/usr/bin/nvcc` to your nvcc installation, if it exsists. But you should not remove `/usr/bin/nvcc` if it is a regular file created by nvcc installation procedure (in other words, you should not remove this file if it is not copy of nvcc installed somewhere else, see above).



## Approach 2. Set CUDACXX environment variable

Set the following variable

```console
export CUDACXX=/usr/local/cuda/bin/nvcc
```

Your `CMakeLists.txt` should be like this:

```CMakeLists.txt
cmake_minimum_required(VERSION 3.22) # VERSION should be not too small, for example, 3.14 or higher
project(prjname CUDA)
add_executable(aaa aaa.cu)
```

In this case, cmake will find nvcc using CUDACXX environment variable and calculate the location of other CUDA components using `nvcc -v fakefile`.

## Approach 3. Replace symlink to nvcc with simple script

Replace symlink `/usr/bin/nvcc` with the following script

```bash
#!/bin/sh
#place here your path to nvcc installation
exec /usr/local/cuda/bin/nvcc "$@"
```

This hack calls nvcc using the correct path and this allows nvcc to find other components when `nvcc -v fakefile` is executed. The `CMakeLists.txt` file for this case is the same as in the previous approach.

Do not forget to grant the execution flags: `chmod ugo+x /usr/bin/nvcc`

**Rem.**: This way is used when ```nvidia-cuda-toolkit``` installed via `apt install`.<br/>

# Remarks

## Remark 1

The `find_package(CUDAToolkit)` command also looks for pthreads, we don't know why.

## Remark 2

The findCUDAToolkit script can produce the warning if one or more CUDA components were not found. For example, the following warning can be printed
```console
CMake Warning at /snap/cmake/1336/share/cmake-3.27/Modules/FindCUDAToolkit.cmake:1072 (message):
  Could not find librt library, needed by CUDA::cudart_static
```
This means that static linkink with CUDA libraries will produce an error. To fix this problem, place `project(name)` before `find_package(CUDAToolkit)` in your `CMakeLists.txt` file.


