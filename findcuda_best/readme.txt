This is a final combination of check_language and findCUDAToolkit that
can be used in your projects.

First, it searches CUDA using check_language(CUDA). If CUDA is not found,
it makes a second attempt using FindCUDAToolkit.

just run:
    mkdir build
    cd build
    cmake ..
    cmake --build .
