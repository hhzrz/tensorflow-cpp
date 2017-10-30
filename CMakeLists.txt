cmake_minimum_required(VERSION 3.6)
project(hello_tensorflow)

set(CMAKE_CXX_STANDARD 11)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules/")

link_directories(/usr/lib /usr/local/lib)
# Eigen
find_package(Eigen REQUIRED)
include_directories(${Eigen_INCLUDE_DIRS})

# Protobuf
FIND_PACKAGE(Protobuf 3.4.0 REQUIRED)
MESSAGE(STATUS "Using protobuf ${Protobuf_VERSION}")

# tensorflow
find_package(TensorFlow REQUIRED)
include_directories(${TensorFlow_INCLUDE_DIRS})

add_executable(hello_tensorflow main.cc)
target_link_libraries(hello_tensorflow tensorflow_cc tensorflow_framework ${Protobuf_LIBRARIES})
