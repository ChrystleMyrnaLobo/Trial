# Caffe SSD MobileNet w/o root access

## 1. CUDA
- Setup cuda and cudnn [cuda]

## 2. Caffe
*Any BVLC/caffe forked repo should be similar. Following is for [ssd]. For [nvcaffe], see section on nvidia digits below* 
- Clone [ssd] as ssd_caffe. This repo is forked from BVLC/caffe. 
- Checkout the correct branch. For [ssd], `git checkout caffe`
- Make build directory `mkdir build && cd build`
- Create new conda env  
```
conda create -n ssd python=2.7 anaconda
conda activate ssd
export ENV_PATH=$HOME/anaconda3/envs/ssd
```
- Download dependancies via conda or build [from source] (see issues below)  
`conda install cmake lmdb openblas leveldb opencv boost protobuf glog gflags hdf5 -y`
- Install any other unmet dependancies
- In file `CMakeLists.txt` (after the last 'set' of CMAKE_CXX_FLAG around line 62) add this line (if not present)   
`set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")`
- After `cmake`, check if the correct paths to libraries are picked   
 ```
 cmake -DBLAS=open -DCUDNN_INCLUDE=$CUDA_HOME/include/ -DCUDNN_LIBRARY=$CUDA_HOME/lib64/libcudnn.so -DCMAKE_PREFIX_PATH=$ENV_PATH -DCMAKE_INSTALL_PREFIX=$ENV_PATH -DCUDA_CUDART_LIBRARY=$CUDA_HOME/lib64/libcudart.so -D CUDA_TOOLKIT_ROOT_DIR=$CUDA_HOME ..
 
 make all -j 8
 make install
 ```
- Export `export PYTHONPATH=$HOME/ssd_caffe/python:$PYTHONPATH`. Refer [conda] to add to conda env vars.
- Reference: [setup caffe without root]

---
## Issues
- I had to build protobuf and boost from source
- gcc-5.x.x requires -std=c++11, but the boost lib are built without such an option. So I had to build my own version of boost.
### Build boost
- Install boost `conda install boost` and check the version resolved by conda
- Download the same version boost lib from https://www.boost.org/. Unzip.
- Build from source ([easy build]) override the conda lib files via --prefix to install in /lib and /include/boost
```
./bootstrap.sh --prefix=~/anaconda3/
./bjam install

# For python 2.0, link if not present
ln /home/chrystle/anaconda3/envs/nvd/lib/libboost_python27.so /home/chrystle/anaconda3/envs/nvd/lib/libboost_python-py27.so
```
- Alternatively, manually [build boost]
- CMake did not use my conda installation ([alternate boost]) from cmake flags. So I manually updated boost path in `build/CMakeCache.txt` and ran without `make clean`.

### Build protobuf
- Install dependancy and make. Office site: [protobuf for digits]
```
git clone https://github.com/google/protobuf.git
conda install autoconf automake libtool
cd protobuf/
./autogen.sh
./configure --prefix=/home/chrystle/anaconda3/envs/nvd
make 
make install
cd python
python setup.py install --cpp_implementation
```

***
## SSD MobileNet
- Clone [mobilenet ssd] 
- Uncomment `engine: CAFFE` under `convolution_param` in every convolution layers in `*.prototxt`
- Run `demo.py` 

## Nvidia digits
### Caffe dependancy
- For [nvcaffe], before `cmake` install `conda install -c conda-forge libjpeg-turbo`
- Build boost and protobuf from source
- In ~/anaconda3/envs/nvcaffe/etc/conda/activate.d/env_vars.sh set `export CAFFE_ROOT=$HOME/MTP2/nvcaffe`
- Check if caffe is installed correctly

### Setup digits
- Download repo [digits github]
- Use conda to install all unmet in `digits/requirements.txt`
```
conda install -c anaconda gevent-websocket -y &&
conda install -c anaconda flask-wtf -y &&
conda install -c conda-forge flask-socketio -y &&
conda install -c anaconda pydot -y &&
conda install -c anaconda scikit-fmm -y &&
conda install -c conda-forge python-magic -y
```

### Run
Keep two terminals open
- From remote, run the webapp `./digits-devserver`
- From local, port forward `ssh -L 5000:10.208.23.220:5000 viz -N`
- From local, open `localhost:5000`

[ssd]: https://github.com/weiliu89/caffe/tree/ssd
[mobilenet ssd]: https://github.com/chuanqi305/MobileNet-SSD
[cuda]: https://jin-zhe.github.io/guides/installing-caffe-with-cuda-on-anaconda/
[setup caffe without root]: https://jin-zhe.github.io/guides/installing-caffe-with-cuda-on-anaconda/
[build boost]: https://github.com/BVLC/caffe/issues/6043#issuecomment-423049323
[nvcaffe]:https://github.com/NVIDIA/caffe
[easy build]:https://www.boost.org/doc/libs/1_46_1/more/getting_started/unix-variants.html#easy-build-and-install
[alternate boost]:https://stackoverflow.com/questions/3016448/how-can-i-get-cmake-to-find-my-alternative-boost-installation
[from source]:https://autchen.github.io/guides/2015/04/03/caffe-install.html
[conda]:https://github.com/ChrystleMyrnaLobo/scribble/blob/master/conda.md
[protobuf for digits]:https://github.com/NVIDIA/DIGITS/blob/master/docs/BuildProtobuf.md
[digits github]: https://github.com/NVIDIA/DIGITS