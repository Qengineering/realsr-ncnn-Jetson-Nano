# realsr-ncnn-Jetson-Nano
![output image]( https://qengineering.eu/images/SRflat.png )
## Real Super Resolution with the ncnn framework. <br/>
[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)<br/><br/>
Paper :https://openaccess.thecvf.com/content_CVPRW_2020/papers/w31/Ji_Real-World_Super-Resolution_via_Kernel_Estimation_and_Noise_Injection_CVPRW_2020_paper.pdf<br/><br/>
Made for a Jetson Nano see [Q-engineering deep learning examples](https://qengineering.eu/deep-learning-examples-on-raspberry-32-64-os.html)<br/><br/>
The solution of the _Tencent YouTu Lab_ is the winner of **CVPR NTIRE 2020 Challenge on Real-World Super-Resolution** in both tracks.<br/>
https://arxiv.org/abs/2005.01996

------------

## Dependencies.
We could fork the original code, but to keep the code up to the minute, we'll use a link to the original [repo](https://github.com/nihui/realsr-ncnn-vulkan).<br/>
No need to have the ncnn framework on your Jetson Nano on forehand.

------------

## Installing the app.
### CMake
The CMake version on the Jetpack 4.5.1. is 3.10.2. That is too old. <br/>
Let's update the latest version by building it from scratch
First, remove the old version.<br/>
The old version is located at `/usr/bin`, which will be visit before the new version at `/usr/local/bin` is scanned.<br/>
This way, you still keep using the old version.
```
sudo apt-get remove --purge cmake
```
Download the new version and a single dependency.
```
sudo apt-get install libssl-dev
wget https://cmake.org/files/v3.20/cmake-3.20.0.tar.gz
tar -xzvf cmake-3.20.0.tar.gz
```
compile the package. This takes a while
```
cd cmake-3.20.0
./bootstrap
make -j4
sudo make install
```
And test the new version
```
cmake --version
```
![output image]( https://qengineering.eu/images/Cmake_3_20_Rdy_Nano.png)<br/>
### glslang
With CMake up to date, the next package to install is glslang.
Start with downloading the code.
```
cd ~
git clone https://github.com/KhronosGroup/glslang.git
```
Install other tools
```
cd glslang
git clone https://github.com/google/googletest.git External/googletest
./update_glslang_sources.py
```
![output image]( https://qengineering.eu/images/glslang_external_nano.png)<br/>
Now the configuration can be set
```
mkdir build
cd build
```
Compile and install
```
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="$(pwd)/install" ..
make -j4 install
```
![output image]( https://qengineering.eu/images/glslang_rdy_nano.png)<br/><br/>
Most important is the folder where the binaries are stored.<br/> This directory must be given in the `CMakeLists.txt` later on.<br/><br/>
![output image]( https://qengineering.eu/images/glslang_folder_nano.png)<br/>
### realsr-ncnn-vulkan
Now we can install the realsr ncnn framework.
```
git clone https://github.com/nihui/realsr-ncnn-vulkan.git
cd realsr-ncnn-vulkan
git submodule update --init --recursive
```
Before building the framework, we need to set the path to the glslang folder in de make file.<br/>
![output image]( https://qengineering.eu/images/CMakeLists_nano.png)<br/>
Open `~/realsr-ncnn-vulkan/src/CMakeLists.txt` and alter line 19.<br/> 
If you have the glslang installed in the same folder as we did, you can change the line to
```
find_program(GLSLANGVALIDATOR_EXECUTABLE NAMES glslangValidator PATHS /home/<username>/glslang/build/install/bin)
```
Our username was _jetson_ in the screen dump above. Yours will undoubtedly be different.<br/>
![output image]( https://qengineering.eu/images/CmakeLists_Edit_nano2.png)<br/><br/>
Last step is the building of the realsr framework.
```
mkdir build
cd build
cmake ../src
```
If you end up during `cmake` with errors like the one below, you still have the old verion 3.10.2 CMake running.<br/>
![output image]( https://qengineering.eu/images/realsr_error_nano.png)<br/>
Compile.
```
cmake --build . -j 4
```
Once done, you can test your software. 
```
./realsr-ncnn-vulkan -i ../images/0.jpg -o ../images/out.png -s 4 -x -m ../models/models-DF2K

```
![output image]( https://qengineering.eu/images/realsr_nano_run.png)<br/>
Be patient, it will take quite a while to process. The image is cut into small tiles.<br/>
These are processed one by one. When done, they are glued together with the exception of a 10-pixel border to avoid border artefacts.<br/>
The `models-DF2K_JPEG` is intended for use with jpeg images. Jpeg images can have specific compression artifacts that the regular `models-DF2K` cannot handle.<br/>
Additional information can be found on this [blog](https://linuxreviews.org/RealSR).<br/><br/>
## Enjoy!
![output image]( https://qengineering.eu/images/SRgarden.png)<br/><br/>
![output image]( https://qengineering.eu/images/SRcat.png)<br/><br/>
Many thanks to [nihui](https://github.com/nihui/) again!<br/>
