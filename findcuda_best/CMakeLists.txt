cmake_minimum_required(VERSION 3.18)

project(findcuda) #we need to declare project before calling findpackage(CUDAToolkit) to avoid warnings

# Find CUDA block
# first, try check_language as the recommended default way to find CUDA
include(CheckLanguage)
check_language(CUDA)

if(NOT CMAKE_CUDA_COMPILER)
	# try to find CUDA using findCUDAToolkit package, since it has a slightly better algorithm
	# than check_language(CUDA).
	message(STATUS "CMAKE_CUDA_COMPILER = ${CMAKE_CUDA_COMPILER}, running FindCUDAToolkit")
	find_package(CUDAToolkit) #New in version 3.17  	
	if(${CUDAToolkit_FOUND})
		#i f found, we set CMAKE_CUDA_COMPILER variable as a hint to check_language/enable_language 
		set(CMAKE_CUDA_COMPILER ${CUDAToolkit_NVCC_EXECUTABLE})
		# And now we run check_language again
		check_language(CUDA)
	endif()
endif()

if(CMAKE_CUDA_COMPILER)
	set(HAS_GPU ON)
	message(" -- Building with GPU support")
	enable_language(CUDA)
else()  
	set(HAS_GPU OFF)
	message(" -- Building without GPU support")
endif()
# end of find CUDA block


add_executable(example example.cu)

