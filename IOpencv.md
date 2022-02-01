OPENCV
#########################
https://www.pyimagesearch.com/2017/09/04/raspbian-stretch-install-opencv-3-python-on-your-raspberry-pi/

###install

####raspberry
wget https://github.com/opencv/opencv/archive/3.4.3.tar.gz
wget https://github.com/opencv/opencv_contrib/archive/3.4.3.tar.gz


1. preapre
```
sudo apt install build-essential cmake pkg-config libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev libgtk2.0-dev libgtk-3-dev libatlas-base-dev gfortran python2.7-dev python3-dev
sudo apt install numpy

#numpy 如果不装则无法安装python 模块 cv2.so miss

```
2. make
```
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE     -D CMAKE_INSTALL_PREFIX=/usr/local     ..
make -j4
sudo make install && sudo ldconfig
```

3. 
