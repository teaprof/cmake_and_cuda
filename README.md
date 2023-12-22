# cmake_and_cuda

This project contains four examples of how to enable CUDA in cmake-based projects. 

- findcuda1 example shows the default recommended approach using CUDACXX environment variable
- findcuda2 example shows how to use findCUDAToolkit package to help cmake locate CUDA
- findcuda3 example shows how to make correct link to nvcc compiler and place it in folder from PATH
- findcuda_best is a combination of examples 1 and 2 (first, try the default recommended way, then try to use findCUDAToolkit package).

For details, see [this file](how_to_enable_CUDA_in_cmake.md)

