#  Install OpenCV 4.5.2 with CUDA 10.2 and CUDNN 8.2.1 in Ubuntu 18.04

Installing OpenCV on the Jetson Nano can be challenging, especially when it comes to enabling CUDA support to accelerate OpenCV. This tutorial will guide you through the process of installing OpenCV with CUDA on the Jetson Nano, which should save you time and simplify the process. Building and compiling OpenCV on the Nano typically takes about 3–4 hours, so following these steps carefully will help you avoid common issues and streamline your installation.



## Setting system

First of all install update and upgrade your system:
    
        $ sudo apt update
        $ sudo apt upgrade
   
    
Then, install required libraries:

* Generic tools:

        $ sudo apt install build-essential cmake pkg-config unzip yasm git checkinstall
	
* Python VirtualEnvironments with VirtualenvWrapper (Not recommand)
```
	Install PIPX:
	-------------
	$ sudo apt install pipx
	$ pipx ensurepath


	Install Virtualenv and Wrapper:
	-------------------------------
	$ pipx install virtualenv
	$ pipx install virtualenvwrapper
	
	Configure ~/.bashrc:
	--------------------
	$ nano ~/.bashrc
	
	... #Append these lines, but changing the user folder name $USER
	#VIRTUAL ENVIRONMENT
	export WORKON_HOME=~/.virtualenvs
	export VIRTUALENVWRAPPER_PYTHON=/home/$USER/.local/share/pipx/venvs/virtualenvwrapper/bin/python3
	source /home/$USER/.local/share/pipx/venvs/virtualenvwrapper/bin/virtualenvwrapper_lazy.sh
	...
	
	$ source ~/.bashrc


	Create virtualenv and activate:
	-------------------------------
	mkvirtualenv opencv
	workon opencv
```
    
* Image I/O libs
    ``` 
    $ sudo apt install libjpeg-dev libpng-dev libtiff-dev
    ``` 
* Video/Audio Libs - FFMPEG, GSTREAMER, x264 and so on.

```
	# Install basic codec libraries
	sudo apt install libavcodec-dev libavformat-dev libswscale-dev

	# Install GStreamer development libraries
	sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev

	# Install additional codec and format libraries
	sudo apt install libxvidcore-dev libx264-dev libmp3lame-dev libopus-dev

	# Install additional audio codec libraries
	sudo apt install libmp3lame-dev libvorbis-dev

	# Install FFmpeg (which includes libavresample functionality)
	sudo apt install ffmpeg

	# Optional: Install VA-API for hardware acceleration
	sudo apt install libva-dev
```
    
* Cameras programming interface libs
```
    # Install video capture libraries and utilities
    sudo apt install libdc1394-25 libdc1394-dev libxine2-dev libv4l-dev v4l-utils

   # Create a symbolic link for video device header
   sudo ln -s /usr/include/libv4l1-videodev.h /usr/include/linux/videodev.h
```

* GTK lib for the graphical user functionalites coming from OpenCV highghui module 
    ```
    $ sudo apt-get install libgtk-3-dev
    ```
    
* Parallelism library C++ for CPU
    ```
    $ sudo apt-get install libtbb-dev
    ```
* Optimization libraries for OpenCV
    ```
    $ sudo apt-get install libatlas-base-dev gfortran
    ```
* Optional libraries:
    ```
    $ sudo apt-get install libprotobuf-dev protobuf-compiler
    $ sudo apt-get install libgoogle-glog-dev libgflags-dev
    $ sudo apt-get install libgphoto2-dev libeigen3-dev libhdf5-dev doxygen
    ```

We will now proceed with the installation (see the Qt flag that is disabled to do not have conflicts with Qt5.0).

    $ cd ~/Downloads && mkdir opencv && cd opencv
    $ wget -O opencv.zip https://github.com/opencv/opencv/archive/refs/tags/4.5.2.zip
    $ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/refs/tags/4.5.2.zip
    $ unzip opencv.zip
    $ unzip opencv_contrib.zip
    $ pip install numpy
    
    $ echo "Procced with the installation"
    $ cd opencv-4.5.2
    $ mkdir build
    $ cd build
    
      cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D WITH_TBB=ON \
    -D ENABLE_FAST_MATH=1 \
    -D CUDA_FAST_MATH=1 \
    -D WITH_CUBLAS=1 \
    -D WITH_CUDA=ON \
    -D BUILD_opencv_cudacodec=OFF \
    -D BUILD_TESTS=OFF \
    -D WITH_CUDNN=ON \
    -D OPENCV_DNN_CUDA=ON \
    -D CUDA_ARCH_BIN=5.3 \
    -D WITH_V4L=ON \
    -D WITH_QT=OFF \
    -D WITH_OPENGL=ON \
    -D WITH_GSTREAMER=ON \
    -D OPENCV_GENERATE_PKGCONFIG=ON \
    -D OPENCV_PC_FILE_NAME=opencv.pc \
    -D OPENCV_ENABLE_NONFREE=ON \
    -D OPENCV_PYTHON3_INSTALL_PATH=~/usr/local/lib/python3.6/dist-packages \
    -D OPENCV_EXTRA_MODULES_PATH=~/Downloads/opencv_contrib-4.5.2/modules \
    -D INSTALL_PYTHON_EXAMPLES=OFF \
    -D INSTALL_C_EXAMPLES=OFF \
    -D PYTHON3_PACKAGES_PATH=/usr/lib/python3/dist-package \
    -D BUILD_NEW_PYTHON_SUPPORT=ON \
    -D BUILD_opencv_python3=ON \
    -D HAVE_opencv_python3=ON \
    -D PYTHON_EXECUTABLE=/usr/lib/python3 \
    -D BUILD_opencv_python3=TRUE \
    -D BUILD_EXAMPLES=OFF ..

	

If you want to build the libraries statically you only have to include the *-D  BUILD_SHARED_LIBS=OFF*

    $ cmake 
         ...
	 -D BUILD_SHARED_LIBS=OFF ..
    
In case you do not want to include include CUDA set *-D WITH_CUDA=OFF*     

    $ cmake 
         ...
	 -D WITH_CUDA=OFF ..
    
If you want also to use CUDNN you must include those flags (to set the correct value of CUDA_ARCH_BIN you must visit https://developer.nvidia.com/cuda-gpus and find the Compute Capability CC of your graphic card). If you have problems with the setting up of CUDDN check the *List of documented problems*:

	-D WITH_CUDNN=ON \
	-D OPENCV_DNN_CUDA=ON \
	-D CUDA_ARCH_BIN=5.3 \

Before the compilation you must check that CUDA has been enabled in the configuration summary printed on the screen. (If you have problems with the CUDA Architecture go to the end of the document).

```
--   NVIDIA CUDA:                   YES (ver 10.2, CUFFT CUBLAS FAST_MATH)
--     NVIDIA GPU arch:             53
--     NVIDIA PTX archs:
-- 
--   cuDNN:                         YES (ver 8.2.1)
```

I've included below the output of my [configuration](#configuration-information).

If it is fine proceed with the compilation (Use nproc to know the number of cpu cores):
    
    $ nproc
    $ make -j3
    $ sudo make install

If you want to have available opencv python bindings in the system environment you should copy the created folder during the installation of OpenCV (* -D OPENCV_PYTHON3_INSTALL_PATH=~/.virtualenvs/opencv/lib/python3.12/site-packages/ *) into the *dist-packages* folder of the target python interpreter:

    $ sudo cp -r ~/.virtualenvs/opencv/lib/python3.12/site-packages/cv2 /usr/local/lib/python3.12/dist-packages
   
    $ echo "Modify config-3.12.py to point to the target directory" 
    $ sudo nano /usr/local/lib/python3.12/dist-packages/cv2/config-3.12.py 
    
    ``` 
	    PYTHON_EXTENSIONS_PATHS = [
	    os.path.join('/usr/local/lib/python3.12/dist-packages/cv2', 'python3.12')
	    ] + PYTHON_EXTENSIONS_PATHS
    ``` 


```
-- 
-- General configuration for OpenCV 4.10.0 =====================================
--   Version control:               unknown
-- 
--   Extra modules:
--     Location (extra):            /home/raul/Downloads/opencv/opencv_contrib-4.5.2/modules
--     Version control (extra):     unknown
-- 
--   Platform:
--     Timestamp:                   2024-09-08T05:42:59Z
--     Host:                        Linux 6.8.0-41-generic x86_64
--     CMake:                       3.28.3
--     CMake generator:             Unix Makefiles
--     CMake build tool:            /usr/bin/gmake
--     Configuration:               RELEASE
-- 
--   CPU/HW features:
--     Baseline:                    SSE SSE2 SSE3
--       requested:                 SSE3
--     Dispatched code generation:  SSE4_1 SSE4_2 FP16 AVX AVX2 AVX512_SKX
--       requested:                 SSE4_1 SSE4_2 AVX FP16 AVX2 AVX512_SKX
--       SSE4_1 (18 files):         + SSSE3 SSE4_1
--       SSE4_2 (2 files):          + SSSE3 SSE4_1 POPCNT SSE4_2
--       FP16 (1 files):            + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 AVX
--       AVX (9 files):             + SSSE3 SSE4_1 POPCNT SSE4_2 AVX
--       AVX2 (38 files):           + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2
--       AVX512_SKX (8 files):      + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2 AVX_512F AVX512_COMMON AVX512_SKX
-- 
--   C/C++:
--     Built as dynamic libs?:      YES
--     C++ standard:                11
--     C++ Compiler:                /usr/bin/c++  (ver 13.2.0)
--     C++ flags (Release):         -fsigned-char -ffast-math -fno-finite-math-only -W -Wall -Wreturn-type -Wnon-virtual-dtor -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -fvisibility-inlines-hidden -O3 -DNDEBUG  -DNDEBUG
--     C++ flags (Debug):           -fsigned-char -ffast-math -fno-finite-math-only -W -Wall -Wreturn-type -Wnon-virtual-dtor -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -fvisibility-inlines-hidden -g  -O0 -DDEBUG -D_DEBUG
--     C Compiler:                  /usr/bin/cc
--     C flags (Release):           -fsigned-char -ffast-math -fno-finite-math-only -W -Wall -Wreturn-type -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -O3 -DNDEBUG  -DNDEBUG
--     C flags (Debug):             -fsigned-char -ffast-math -fno-finite-math-only -W -Wall -Wreturn-type -Waddress -Wsequence-point -Wformat -Wformat-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -g  -O0 -DDEBUG -D_DEBUG
--     Linker flags (Release):      -Wl,--exclude-libs,libippicv.a -Wl,--exclude-libs,libippiw.a   -Wl,--gc-sections -Wl,--as-needed -Wl,--no-undefined  
--     Linker flags (Debug):        -Wl,--exclude-libs,libippicv.a -Wl,--exclude-libs,libippiw.a   -Wl,--gc-sections -Wl,--as-needed -Wl,--no-undefined  
--     ccache:                      YES
--     Precompiled headers:         NO
--     Extra dependencies:          m pthread cudart_static dl rt nppc nppial nppicc nppidei nppif nppig nppim nppist nppisu nppitc npps cublas cudnn cufft -L/usr/local/cuda/lib64 -L/usr/lib/x86_64-linux-gnu
--     3rdparty dependencies:
-- 
--   OpenCV modules:
--     To be built:                 alphamat aruco bgsegm bioinspired calib3d ccalib core cudaarithm cudabgsegm cudafeatures2d cudafilters cudaimgproc cudalegacy cudaobjdetect cudaoptflow cudastereo cudawarping cudev datasets dnn dnn_objdetect dnn_superres dpm face features2d flann freetype fuzzy gapi hdf hfs highgui img_hash imgcodecs imgproc intensity_transform line_descriptor mcc ml objdetect optflow phase_unwrapping photo plot python3 quality rapid reg rgbd saliency sfm shape signal stereo stitching structured_light superres surface_matching text tracking ts video videoio videostab wechat_qrcode xfeatures2d ximgproc xobjdetect xphoto
--     Disabled:                    cudacodec world
--     Disabled by dependency:      -
--     Unavailable:                 cannops cvv java julia matlab ovis python2 viz
--     Applications:                tests perf_tests apps
--     Documentation:               NO
--     Non-free algorithms:         YES
-- 
--   GUI:                           GTK3
--     GTK+:                        YES (ver 3.24.41)
--       GThread :                  YES (ver 2.80.0)
--       GtkGlExt:                  NO
--     OpenGL support:              NO
--     VTK support:                 NO
-- 
--   Media I/O: 
--     ZLib:                        /usr/lib/x86_64-linux-gnu/libz.so (ver 1.3)
--     JPEG:                        /usr/lib/x86_64-linux-gnu/libjpeg.so (ver 80)
--     WEBP:                        /usr/lib/x86_64-linux-gnu/libwebp.so (ver encoder: 0x020f)
--     PNG:                         /usr/lib/x86_64-linux-gnu/libpng.so (ver 1.6.43)
--     TIFF:                        /usr/lib/x86_64-linux-gnu/libtiff.so (ver 42 / 4.5.1)
--     JPEG 2000:                   build (ver 2.5.0)
--     OpenEXR:                     build (ver 2.3.0)
--     HDR:                         YES
--     SUNRASTER:                   YES
--     PXM:                         YES
--     PFM:                         YES
-- 
--   Video I/O:
--     DC1394:                      YES (2.2.6)
--     FFMPEG:                      YES
--       avcodec:                   YES (60.31.102)
--       avformat:                  YES (60.16.100)
--       avutil:                    YES (58.29.100)
--       swscale:                   YES (7.5.100)
--       avresample:                NO
--     GStreamer:                   YES (1.24.2)
--     v4l/v4l2:                    YES (linux/videodev2.h)
-- 
--   Parallel framework:            TBB (ver 2021.11 interface 12110)
-- 
--   Trace:                         YES (with Intel ITT)
-- 
--   Other third-party libraries:
--     Intel IPP:                   2021.11.0 [2021.11.0]
--            at:                   /home/raul/Downloads/opencv/opencv-4.10.0/build/3rdparty/ippicv/ippicv_lnx/icv
--     Intel IPP IW:                sources (2021.11.0)
--               at:                /home/raul/Downloads/opencv/opencv-4.10.0/build/3rdparty/ippicv/ippicv_lnx/iw
--     VA:                          YES
--     Lapack:                      NO
--     Eigen:                       YES (ver 3.4.0)
--     Custom HAL:                  NO
--     Protobuf:                    build (3.19.1)
--     Flatbuffers:                 builtin/3rdparty (23.5.9)
-- 
--   NVIDIA CUDA:                   YES (ver 12.6, CUFFT CUBLAS FAST_MATH)
--     NVIDIA GPU arch:             75
--     NVIDIA PTX archs:
-- 
--   cuDNN:                         YES (ver 8.9.7)
-- 
--   OpenCL:                        YES (INTELVA)
--     Include path:                /home/raul/Downloads/opencv/opencv-4.10.0/3rdparty/include/opencl/1.2
--     Link libraries:              Dynamic load
-- 
--   Python 3:
--     Interpreter:                 /home/raul/.virtualenvs/opencv/bin/python (ver 3.12.3)
--     Libraries:                   /usr/lib/x86_64-linux-gnu/libpython3.12.so (ver 3.12.3)
--     Limited API:                 NO
--     numpy:                       /home/raul/.virtualenvs/opencv/lib/python3.12/site-packages/numpy/_core/include (ver 2.1.1)
--     install path:                /home/raul/.virtualenvs/opencv/lib/python3.12/site-packages//cv2/python-3.12
-- 
--   Python (for build):            /home/raul/.virtualenvs/opencv/bin/python
-- 
--   Java:                          
--     ant:                         NO
--     Java:                        NO
--     JNI:                         NO
--     Java wrappers:               NO
--     Java tests:                  NO
-- 
--   Install to:                    /usr/local
-- -----------------------------------------------------------------
-- 
-- Configuring done (43.4s)
-- Generating done (0.8s)
-- Build files have been written to: /home/raul/Downloads/opencv/opencv-4.5.2/build
```


### List of documented problems

If you have problems with unsupported architectures of your graphic card with the minimum requirements from Opencv, you will get the following error:

```
CUDA backend for DNN module requires CC 5.3 or higher.  Please remove unsupported architectures from CUDA_ARCH_BIN option.
```
It means that the DNN module needs that your graphic card supports the 5.3 Compute Capability (CC) version; in this [link](https://developer.nvidia.com/cuda-gpus) you can fint the CC of your card. Some opencv versions have fixed the minimum version to 3.0 but there is a clear move to filter above 5.3 since the half-precision precision operations are available from 5.3 version. To fix this problem you can modify the *CMakeList.txt* file located in *opencv > modules > dnn > CMakeList.txt* and set the minimum version to the one you have, but bear in mind that the correct functioning of this module will be compromised. However, if you only want GPU for the rest of modules, it could work.

You can also select the target `CUDA_ARCH_BIN` option in the command to generate the makefile for your current target or modify the list of supported architectures:

	$ grep -r 'CUDA_ARCH_BIN' .  //That prompts ./CMakeCache.txt


The restriction is to have a higher version than 5.3, so you can modify the file by removing all the inferior arch to 5.3

```
CUDA_ARCH_BIN:STRING=6.0 6.1 7.0 7.5
```
Now, the makefile was created succesfully. Before the compilation you must check that CUDA has been enabled in the configuration summary printed on the screen.

```
--   NVIDIA CUDA:                   YES (ver 10.2, CUFFT CUBLAS NVCUVID FAST_MATH)
--     NVIDIA GPU arch:             60 61 70 75
--     NVIDIA PTX archs:

```

Some users as TAF2 had problems when configuring CUDNN libraries but it was solved and here is the TAF2's proposal, you can also find it in the comments:

```
sudo apt install libcudnn7-dev  libcudnn7-doc  libcudnn7 nvidia-container-csv-cudnn
```

```
 -D CUDNN_INCLUDE_DIR=/usr/include \
-D CUDNN_LIBRARY=/usr/lib64/libcudnn_static_v7.a \
-D CUDNN_VERSION=7.6.3
```

*If you have any other problem try updating the nvidia drivers.*

### Source
- [pyimagesearch](https://www.pyimagesearch.com/2018/08/15/how-to-install-opencv-4-on-ubuntu/)
- [learnopencv](https://www.learnopencv.com/install-opencv-4-on-ubuntu-18-04/)
- [Tzu-cheng](https://chuangtc.com/ParallelComputing/OpenCV_Nvidia_CUDA_Setup.php)
- [Medium](https://medium.com/@debugvn/installing-opencv-3-3-0-on-ubuntu-16-04-lts-7db376f93961)
- [Previous Gist](https://gist.github.com/raulqf/a3caa97db3f8760af33266a1475d0e5e)
