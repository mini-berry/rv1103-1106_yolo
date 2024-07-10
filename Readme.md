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
## 特别说明
所有的yolo demo都可使用一样的程序
只需要修改模型
对于rv1103\rv1106由于内存不足，只能使用yolov5
提供了yolov8供测试
```sh
./yolo_test model/yolov5nu.rknn model/bus.jpg
```
# 模型导出
yolo export format=onnx opset=12 model=yolov5nu.pt
opset=12是必选的,否则内存不足
使用yolov5nu.pt导出后仍然报错
E RKNN: rknn_pack error, Unsupported input type 0 pack to output type 2!
解决不了
