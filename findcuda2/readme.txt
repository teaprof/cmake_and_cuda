This is a demo how to use CUDACXX environment variable to
help check_language(CUDA) find CUDA toolkit.

Just run:
    source ./initCUDACXX.sh
    mkdir build
    cd build
    cmake ..
    cmake --build .