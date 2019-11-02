# Raspbian(RaspberryPi3) Installation
This is a setup guide for Realsense with RaspberryPi

These steps are for D435 with Raspberry Pi3 and 3+.  

### Add swap
Initial value is 100MB, but we need to build libraries so initial value isn't enough for that.
In this case, need to switch from 100 to `2048` (2GB).  
```
$ sudo vim /etc/dphys-swapfile
CONF_SWAPSIZE=2048

$ sudo /etc/init.d/dphys-swapfile restart swapon -s
```

### Install packages
```
$ sudo apt-get install -y libglu1-mesa libglu1-mesa-dev glusterfs-common libglu1-mesa libglu1-mesa-dev libglui-dev libglui2c2

$ sudo apt-get install -y libglu1-mesa libglu1-mesa-dev mesa-utils mesa-utils-extra xorg-dev libgtk-3-dev libusb-1.0-0-dev
```

### update udev rule
Now we need to get librealsense from the repo(https://github.com/IntelRealSense/librealsense).
```
$ cd ~
$ git clone https://github.com/IntelRealSense/librealsense.git
$ cd librealsense
$ sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/ 
$ sudo udevadm control --reload-rules && udevadm trigger 

```

### update `cmake` version (if your cmake is 3.7.2 or before 3.11.4)
```
$ cd ~
$ wget https://cmake.org/files/v3.11/cmake-3.11.4.tar.gz
$ tar -zxvf cmake-3.11.4.tar.gz;rm cmake-3.11.4.tar.gz
$ cd cmake-3.11.4
$ ./configure --prefix=/home/pi/cmake-3.11.4
$ make -j1
$ sudo make install
$ export PATH=/home/pi/cmake-3.11.4/bin:$PATH
$ source ~/.bashrc
$ cmake --version
cmake version 3.11.4
```

### set path
```
$ nano ~/.bashrc
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

$ source ~/.bashrc

```

### install `protobuf`
```
$ cd ~
$ git clone --depth=1 -b v3.5.1 https://github.com/google/protobuf.git
$ cd protobuf
sudo apt-get install autoconf autogen libtool
$ ./autogen.sh
$ ./configure
$ make -j1
$ sudo make install
$ cd python
$ export LD_LIBRARY_PATH=../src/.libs
Fix compile error:
$ find ./google/protobuf/ -name \*.cc -exec sed -i "s/= PyUnicode_AsUTF8AndSize/= (char*)PyUnicode_AsUTF8AndSize/g" {} \;
$ python3 setup.py build --cpp_implementation 
$ python3 setup.py test --cpp_implementation
$ sudo python3 setup.py install --cpp_implementation
$ export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=cpp
$ export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION_VERSION=3
$ sudo ldconfig
$ protoc --version
```

### install `TBB`
```
$ cd ~
$ wget https://github.com/PINTO0309/TBBonARMv7/raw/master/libtbb-dev_2019U5_armhf.deb
$ sudo dpkg -i ~/libtbb-dev_2019U5_armhf.deb
$ sudo ldconfig
```

### install `OpenCV`
You can build from source code, but it takes so much time. In this case, we will use pre-build version to save time.
```
$sudo apt install libopencv-apps-dev
$ sudo ldconfig
```

### install `RealSense` SDK/librealsense
```
$ cd ~/librealsense
$ mkdir  build  && cd build
$ cmake .. -DBUILD_EXAMPLES=false -DCMAKE_BUILD_TYPE=Release -DFORCE_LIBUVC=true
$ make -j2
$ sudo make install
```

### change `pi` settings (enable OpenGL)
```
$ sudo raspi-config
"7.Advanced Options" - "A7 GL Driver" - "G2 GL (Fake KMS)"
```

Finally, need to reboot pi
```
$ sudo reboot
```


### Try RealSense D435
Connected D435 to the pi and open terminal
```
$ realsense-viewer
```
