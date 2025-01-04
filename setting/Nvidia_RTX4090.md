
## For RTX 4090, Cuda12.1, pytorch 2.5.1, python=3.11

### 1. Create conda env
```shell
conda create -n genesis python==3.11
conda activate genesis
```

### 2. Setting up development environment..."

Check CUDA version
```shell
CUDA_VERSION=$(nvcc --version | grep "release" | awk '{print $5}' | cut -d',' -f1)
if [ "$CUDA_VERSION" != "12.1" ]; then
  echo "Warning: Detected CUDA version is $CUDA_VERSION. This script expects CUDA 12.1."
else
  echo "Detected CUDA version: $CUDA_VERSION"
fi
```

Check Python verion on Conda
```shell
PYTHON_VERSION=$(python --version 2>&1 | awk '{print $2}' | cut -d'.' -f1,2)
if [ "$PYTHON_VERSION" != "3.11" ]; then
  echo "Warning: Detected Python version is $PYTHON_VERSION. This script expects Python 3.11."
else
  echo "Detected Python version: $PYTHON_VERSION"
fi
```

Setting env variables
```shell
PYTHON_VERSION=$(python --version 2>&1 | awk '{print $2}')
PYTHON_VERSION=$(echo "${PYTHON_VERSION}" | cut -d'.' -f1,2)
PYTHON_MAJOR_MINOR=$(echo ${PYTHON_VERSION} | tr -d '.')

echo "Python version: $PYTHON_VERSION"
echo "Python major-minor format: $PYTHON_MAJOR_MINOR"
```

Installing system dependencies...

If you have sudo access. Preferred.
It seems that compilation only works on Ubuntu 20.04+, As vulkan 1.2+ is needed and 18.04 only supports 1.1, but we haven’t fully checked this…

Upgrade g++ and gcc to version 11
```shell
sudo apt install build-essential manpages-dev software-properties-common
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update && sudo apt install gcc-11 g++-11
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 110
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 110
```
verify
```shell
g++ --version
gcc --version
```

Vulkan, zlib, RandR headers, libsnappy, etc..
```shell
sudo apt-get update && sudo apt-get install -y --no-install-recommends \
    build-essential \
    manpages-dev \
    libvulkan-dev \
    zlib1g-dev \
    xorg-dev libglu1-mesa-dev \
    libsnappy-dev \
    software-properties-common \
    git \
    curl \
    wget \
    libegl1 \
    libegl-dev \
    libxrender1 \
    libglib2.0-0 \
    ffmpeg \
    libgtk2.0-dev \
    pkg-config \
    libgles2 \
    libglvnd0 \
    libglx0
```
pybind
```shell
pip install "pybind11[global]"
```

Installing Rust...
```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
```

Installing CMake...
```shell
wget https://github.com/Kitware/CMake/releases/download/v3.31.0-rc2/cmake-3.31.0-rc2-linux-x86_64.sh
chmod +x cmake-3.31.0-rc2-linux-x86_64.sh
sudo ./cmake-3.31.0-rc2-linux-x86_64.sh --skip-license --prefix=/usr/local
rm cmake-3.31.0-rc2-linux-x86_64.sh
```

### 3. Clone repository
```shell
git clone git@github.com:hyunkoome/Genesis.git
cd Genesis/
git submodule update --init --recursive
```

### 4. install packages

Install pytorch
```shell
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

Install patchelf
```shell
sudo apt-get update
sudo apt-get install -y patchelf
patchelf --version
```


Building LuisaRender...
```shell
cd genesis/ext/LuisaRender && \
git submodule update --init --recursive && \
mkdir -p build && \
cmake -S . -B build \
    -D CMAKE_BUILD_TYPE=Release \
    -D PYTHON_VERSIONS=$PYTHON_VERSION \
    -D LUISA_COMPUTE_DOWNLOAD_NVCOMP=ON \
    -D LUISA_COMPUTE_DOWNLOAD_OIDN=ON \
    -D LUISA_COMPUTE_ENABLE_GUI=OFF \
    -D LUISA_COMPUTE_ENABLE_CUDA=ON \
    -Dpybind11_DIR=$(python3 -c "import pybind11; print(pybind11.get_cmake_dir())") && \
cmake --build build -j $(nproc)
```

Installing Genesis...
```shell
pip install genesis-world
pip install --no-cache-dir open3d
cd Genesis/
pip install .
pip install --no-cache-dir PyOpenGL==3.1.5
```

Installing OMPL...
```shell
wget https://github.com/ompl/ompl/releases/download/prerelease/ompl-1.6.0-cp${PYTHON_MAJOR_MINOR}-cp${PYTHON_MAJOR_MINOR}-manylinux_2_28_x86_64.whl
pip install ompl-1.6.0-cp${PYTHON_MAJOR_MINOR}-cp${PYTHON_MAJOR_MINOR}-manylinux_2_28_x86_64.whl
rm ompl-1.6.0-cp${PYTHON_MAJOR_MINOR}-cp${PYTHON_MAJOR_MINOR}-manylinux_2_28_x86_64.whl
```

Setting up Surface Reconstruction...
```shell
cargo install splashsurf
export LD_LIBRARY_PATH=Genesis/genesis/ext/ParticleMesher/ParticleMesherPy:$LD_LIBRARY_PATH
```
in my case
```shell
export LD_LIBRARY_PATH=/home/hyunkoo/DATA/HDD8TB/Add_Objects_DrivingScense/Genesis/genesis/ext/ParticleMesher/ParticleMesherPy:$LD_LIBRARY_PATH
```

# Ray Tracing Renderer 설정
# ============================
Setting up Ray Tracing Renderer...
```shell
cp -r ~/workspace/Genesis/genesis/ext/LuisaRender/build/bin /usr/local/lib/
```

in my case
```shell
sudo cp -r genesis/ext/LuisaRender/build/bin/ /usr/local/lib/
```


