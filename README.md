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
The Jetson Nano is getting old. More than ten years now. Many software packages evolve and require more up-to-date environments. For example, in the case of glslang, a more recent version of Cmake and GNU compiler. Both will first need to be updated before the realsr app can be installed.
### CMake
The current CMake is the 3.10.2 version. That is too old. <br/>
Let's update the latest version by building it from scratch
First, remove the old version.<br/>
The old version is located at `/usr/bin`, which is visited before the new version at `/usr/local/bin` is scanned.<br/>
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
Compile the package. It takes a while
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
### GNU
On the Jetpack 4.5.1, the GNU compiler is version 7.5.0. That is also too old. <br/>
Let's install version 10.1. It will take a while, more than 3 hours!<br/>
Start with the dependencies.
```
$ sudo apt-get install build-essential wget m4 flex bison
```
Download GNU 10.1
```
$ cd
$ wget https://ftpmirror.gnu.org/gcc/gcc-10.1.0/gcc-10.1.0.tar.xz
$ tar xf gcc-10.1.0.tar.xz
$ cd gcc-10.1.0
$ sudo contrib/download_prerequisites
```
Create a build folder and run Cmake and Make. 
```
$ cd
$ mkdir gcc10build
$ cd gcc10build
$ ../gcc-10.1.0/configure -v \
  --build=aarch64-linux-gnu \
  --host=aarch64-linux-gnu \
  --target=aarch64-linux-gnu \
  --prefix=/usr/local/gcc-10.1.0 \
  --enable-checking=release \
  --enable-languages=c,c++ \
  --disable-multilib \
  --program-suffix=-10.1
$ make -j4
$ sudo make install-strip
```
It is time to activate the GNU-10.1 compiler. It can be a _temporary_ or _permanent_ activation.<br> 
It seems logical to set the compiler to the new GNU-10.1 version. However, previously compiled code with the GNU-7.5.0 version may conflict with the newly installed version.
Care must be taken!<br><br>
In our case, with the realsr app, we using temporary activation.<br>
#### Temporarily activation.
Set a path to the GNU-10.1 location in ~/.bashrc
```
$ cd
$ nano .bashrc

#at the end insert these two lines
export PATH=/usr/local/gcc-10.1.0/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/gcc-10.1.0/lib64:$LD_LIBRARY_PATH
#close with <Ctrl>+<X>, <Y>, <Enter>
$ source ~/.bashrc
```
Activate the GNU-10.1 compiler in the SAME terminal window where you are going to compile glslang.
```
$ export CC=gcc-10.1
$ export CXX=g++-10.1
```
At this point, CMake will select the GNU-10.1 version (only in this terminal). 
All other terminals are still using the original GNU-7.5.0 version.
#### Permanently activation.
To set the GNU-10.1 compiler permantly, you modify a few symbolic links.
```
$ sudo rm cpp
$ sudo ln -s /usr/local/gcc-10.1.0/bin/cpp-10.1 /usr/bin/cpp
$ sudo rm g++
$ sudo ln -s /usr/local/gcc-10.1.0/bin/aarch64-linux-gnu-g++-10.1 /usr/bin/g++
$ sudo rm gcc
$ sudo ln -s /usr/local/gcc-10.1.0/bin/aarch64-linux-gnu-gcc-10.1 /usr/bin/gcc
$ sudo rm gcc-ar
$ sudo ln -s /usr/local/gcc-10.1.0/bin/aarch64-linux-gnu-gcc-ar-10.1 /usr/bin/gcc-ar
$ sudo rm gcc-nm
$ sudo ln -s /usr/local/gcc-10.1.0/bin/aarch64-linux-gnu-gcc-nm-10.1 /usr/bin/gcc-nm
$ sudo rm gcc-ranlib
$ sudo ln -s /usr/local/gcc-10.1.0/bin/aarch64-linux-gnu-gcc-ranlib-10.1 /usr/bin/gcc-ranlib

$ sudo rm aarch64-linux-gnu-cpp
$ sudo ln -s /usr/local/gcc-10.1.0/bin/cpp-10.1 /usr/bin/aarch64-linux-gnu-cpp
$ sudo rm aarch64-linux-gnu-g++
$ sudo ln -s /usr/local/gcc-10.1.0/bin/aarch64-linux-gnu-g++-10.1 /usr/bin/aarch64-linux-gnu-g++
$ sudo rm aarch64-linux-gnu-gcc
$ sudo ln -s /usr/local/gcc-10.1.0/bin/aarch64-linux-gnu-gcc-10.1 /usr/bin/aarch64-linux-gnu-gcc
$ sudo rm aarch64-linux-gnu-gcc-ar
$ sudo ln -s /usr/local/gcc-10.1.0/bin/aarch64-linux-gnu-gcc-ar-10.1 /usr/bin/aarch64-linux-gnu-gcc-ar
$ sudo rm aarch64-linux-gnu-gcc-nm
$ sudo ln -s /usr/local/gcc-10.1.0/bin/aarch64-linux-gnu-gcc-nm-10.1 /usr/bin/aarch64-linux-gnu-gcc-nm
$ sudo rm aarch64-linux-gnu-gcc-ranlib
$ sudo ln -s /usr/local/gcc-10.1.0/bin/aarch64-linux-gnu-gcc-ranlib-10.1 /usr/bin/aarch64-linux-gnu-gcc-ranlib
```
Now, the GNU-10.1 will be your compiler system-wide.<br>
![Screenshot from 2023-11-14 12-29-26](https://github.com/Qengineering/realsr-ncnn-Jetson-Nano/assets/44409029/73394793-137b-410d-8035-6d4f7da3b7f0)<br>
Note that you can always replace the symbolic links to their original values, thereby restoring the GNU-7.5.1 again.<br>
And, once again, you may run into problems mixing the two compilers!

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
Now we can install the realsr ncnn framework.<br/>
Although your Nano has Vulkan, ncnn still needs the header files, hence the installation.
```
cd ~
sudo apt-get install libvulkan-dev
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
