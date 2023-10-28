This is an example how to create a correct link to nvcc. Instead of 

ls -n /usr/local/cuda/bin/nvcc

just copy nvcc file from this directory to /usr/bin:

sudo cp ./nvcc /usr/bin/nvcc
    mkdir build
    cd build
    cmake ..
    cmake --build .
