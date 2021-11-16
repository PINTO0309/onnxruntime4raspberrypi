# onnxruntime4raspberrypi
## 1. For Numpy1.19.5
```bash
$ wget -O onnxruntime-1.9.1-cp37-none-linux_armv7l.whl https://github.com/PINTO0309/onnxruntime4raspberrypi/releases/download/v1.9.1/onnxruntime-1.9.1-cp37-none-linux_armv7l.whl_np1195

$ pip3 install onnxruntime-1.9.1-cp37-none-linux_armv7l.whl
```
## 2. For Numpy1.21.4
```bash
$ wget -O onnxruntime-1.9.1-cp37-none-linux_armv7l.whl https://github.com/PINTO0309/onnxruntime4raspberrypi/releases/download/v1.9.1/onnxruntime-1.9.1-cp37-none-linux_armv7l.whl_np1214

$ pip3 install onnxruntime-1.9.1-cp37-none-linux_armv7l.whl
```
## 3. onnxruntime build - Cross-compile
On **`x86_64`** host machine.
```bash
git clone -b v1.9.1 https://github.com/microsoft/onnxruntime
cd onnxruntime
nano cmake/CMakeLists.txt
```
From:
```cmake
# Enable space optimization for gcc/clang
# Cannot use "-ffunction-sections -fdata-sections" if we enable bitcode (iOS)
if (NOT MSVC AND NOT onnxruntime_ENABLE_BITCODE)
  string(APPEND CMAKE_CXX_FLAGS " -ffunction-sections -fdata-sections")
  string(APPEND CMAKE_C_FLAGS " -ffunction-sections -fdata-sections")
endif()
```
To:
```cmake
# Enable space optimization for gcc/clang
# Cannot use "-ffunction-sections -fdata-sections" if we enable bitcode (iOS)
if (NOT MSVC AND NOT onnxruntime_ENABLE_BITCODE)
  string(APPEND CMAKE_CXX_FLAGS " -w -ffunction-sections -fdata-sections -mfpu=neon-vfpv4 -ftree-vectorize -funsafe-math-optimizations -ftree-loop-vectorize -fomit-frame-pointer -latomic")
  string(APPEND CMAKE_C_FLAGS " -w -ffunction-sections -fdata-sections -mfpu=neon-vfpv4 -ftree-vectorize -funsafe-math-optimizations -ftree-loop-vectorize -fomit-frame-pointer -latomic")
endif()
```
```bash
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

docker run --rm -it \
-v ${PWD}:/workdir \
pinto0309/raspios_lite_armhf:2021-03-04_buster \
/bin/bash

cd /workdir
```
```bash
apt update && apt upgrade -y
mv /usr/bin/python /usr/bin/python_
ln -s /usr/bin/python3 /usr/bin/python

apt install -y protobuf-compiler libcurl4-openssl-dev \
libatlas-base-dev git wget make python3-pip

pip3 install cmake numpy==1.19.5
or
pip3 install cmake numpy==1.21.4

./build.sh \
--config MinSizeRel \
--arm \
--enable_pybind \
--build_wheel \
--parallel $(nproc) \
--skip_tests
```
