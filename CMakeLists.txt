cmake_minimum_required(VERSION 3.10)
project(yolo-test)
# set(CMAKE_CXX_STANDARD 11)
# 关闭检测器
set(ENABLE_ASAN FALSE)
# 芯片设置编译器，1103和1106都是1106
set(TARGET_SOC rv1106)
# 设置芯片架构
set(CMAKE_SYSTEM_PROCESSOR armhf)
# 设置编译方式(debug\release)
set(CMAKE_BUILD_TYPE Release)
# 设置系统
set(CMAKE_SYSTEM_NAME Linux)
# 设置编译器目录
set(CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_SOURCE_DIR}/install")
# 设置exe文件的lib库目录
set(CMAKE_INSTALL_RPATH "lib")

# 检测器配置
if (ENABLE_ASAN)
	message(STATUS "BUILD WITH ADDRESS SANITIZER")
	set (CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -fno-omit-frame-pointer -fsanitize=address")
	set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-omit-frame-pointer -fsanitize=address")
	set (CMAKE_LINKER_FLAGS_DEBUG "${CMAKE_LINKER_FLAGS_DEBUG} -fno-omit-frame-pointer -fsanitize=address")
endif ()

# 根据平台选择npu1\npu2
set(rknpu_yolov8_file src/rknpu2/yolov8.cc)

if (TARGET_SOC STREQUAL "rv1106" OR TARGET_SOC STREQUAL "rv1103")
    add_definitions(-DRV1106_1103)
    set(rknpu_yolov8_file src/rknpu2/yolov8_rv1106_1103.cc)
    #dma
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include/3rdparty/allocator/dma)
endif()

if(TARGET_SOC STREQUAL "rk1808" OR TARGET_SOC STREQUAL "rv1109" OR TARGET_SOC STREQUAL "rv1126")
    add_definitions(-DRKNPU1)
    set(rknpu_yolov8_file src/rknpu1/yolov8.cc)
endif()

# 设置链接库目录，不设置找不到动态库
set(CMAKE_INSTALL_RPATH "$ORIGIN/../lib")

file(GLOB SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cc)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/include/3rdparty 3rdparty.out)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/include/utils utils.out)

add_executable(${PROJECT_NAME}
    src/main.cc
    src/postprocess.cc
    ${rknpu_yolov8_file}
)

target_link_libraries(${PROJECT_NAME}
    imageutils
    fileutils
    imagedrawing
    ${LIBRKNNRT}
    dl
)

if (CMAKE_SYSTEM_NAME STREQUAL "Android")
    target_link_libraries(${PROJECT_NAME}
    log
)
endif()

message(STATUS "CMAKE_SYSTEM_NAME: ${CMAKE_SYSTEM_NAME}")
if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(THREADS_PREFER_PTHREAD_FLAG ON)
    find_package(Threads REQUIRED)
    target_link_libraries(${PROJECT_NAME} Threads::Threads)
endif()

target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${LIBRKNNRT_INCLUDES}
)

file(GLOB RKNN_FILES "${CMAKE_CURRENT_SOURCE_DIR}/*.rknn")

install(TARGETS ${PROJECT_NAME} DESTINATION .)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/model/bus.jpg DESTINATION model)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/model/coco_80_labels_list.txt DESTINATION model)
install(FILES ${RKNN_FILES} DESTINATION model)

# opencvlib
# set(OpenCV_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include/opencv-mobile/lib/cmake/opencv4")
# find_package(OpenCV REQUIRED)
# target_link_libraries(opencv-mobile-test ${OpenCV_LIBS})
# include_directories(${OpenCV_INCLUDE_DIRS})

# 设置编译器目录
set(CMAKE_C_COMPILER "${CMAKE_CURRENT_SOURCE_DIR}/sdk/tools/linux/toolchain/arm-rockchip830-linux-uclibcgnueabihf/bin/arm-rockchip830-linux-uclibcgnueabihf-gcc")
set(CMAKE_CXX_COMPILER "${CMAKE_CURRENT_SOURCE_DIR}/sdk/tools/linux/toolchain/arm-rockchip830-linux-uclibcgnueabihf/bin/arm-rockchip830-linux-uclibcgnueabihf-g++")


