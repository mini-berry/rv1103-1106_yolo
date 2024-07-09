# 基于rv1103/1106的yolov8
## 使用方法
编译不再需要执行build-linux.sh
执行以下代码
```sh
mkdir build
cd build
cmake ..
make -j4
make install
```
## 主要修改
CMakeLists指定了编译器路径
设置了3rdparty和main的编译参数
## 还存在的问题
把install拷贝到/oem目录，找不到lib
把install/*拷贝到/oem目录，正常执行