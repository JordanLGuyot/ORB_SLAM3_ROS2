cmake_minimum_required(VERSION 3.5)
project(orbslam3)

# You should set the PYTHONPATH to your own python site-packages path
set(ENV{PYTHONPATH} "/opt/ros/humble/lib/python3.10/site-packages")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Make find_package() use <PackageName>_ROOT variables (CMP0074)
if(POLICY CMP0074)
  cmake_policy(SET CMP0074 NEW)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(message_filters REQUIRED)
find_package(Sophus REQUIRED)
find_package(Pangolin REQUIRED)
find_package(ORB_SLAM3 REQUIRED)

include_directories(
  include
  ${ORB_SLAM3_ROOT}/include
  ${ORB_SLAM3_ROOT}/include/CameraModels
  ${ORB_SLAM3_ROOT}/Thirdparty/Sophus
)

# --- sanity: print once so you can see the final value -----------------
message(STATUS "ORB_SLAM3_ROOT = ${ORB_SLAM3_ROOT}")

# --- stop immediately if the header isn't found -----------------------
if(NOT EXISTS "${ORB_SLAM3_ROOT}/include/CameraModels/GeometricCamera.h")
  message(FATAL_ERROR
    "GeometricCamera.h not found at expected path:\n  "
    "${ORB_SLAM3_ROOT}/include/CameraModels/GeometricCamera.h")
endif()

# --- make sure the target gets the paths ------------------------------
include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${ORB_SLAM3_ROOT}/include
  ${ORB_SLAM3_ROOT}/include/CameraModels
)

link_directories(
  include
)

add_executable(mono
  src/monocular/mono.cpp
  src/monocular/monocular-slam-node.cpp
)
ament_target_dependencies(mono rclcpp sensor_msgs cv_bridge ORB_SLAM3 Pangolin OpenCV)

add_executable(rgbd
  src/rgbd/rgbd.cpp
  src/rgbd/rgbd-slam-node.cpp
)
ament_target_dependencies(rgbd rclcpp sensor_msgs cv_bridge message_filters ORB_SLAM3 Pangolin OpenCV)

add_executable(stereo
  src/stereo/stereo.cpp
  src/stereo/stereo-slam-node.cpp
)
ament_target_dependencies(stereo rclcpp sensor_msgs cv_bridge message_filters ORB_SLAM3 Pangolin OpenCV)

add_executable(stereo-inertial
  src/stereo-inertial/stereo-inertial.cpp
  src/stereo-inertial/stereo-inertial-node.cpp
)
ament_target_dependencies(stereo-inertial rclcpp sensor_msgs cv_bridge ORB_SLAM3 Pangolin OpenCV)

install(TARGETS mono rgbd stereo stereo-inertial DESTINATION lib/${PROJECT_NAME})

# Install launch files.
#install(DIRECTORY launch config vocabulary
#  DESTINATION share/${PROJECT_NAME}/)

ament_package()

