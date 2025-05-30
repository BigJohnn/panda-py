cmake_minimum_required(VERSION 3.15...3.25)

set(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_PACKAGE_NAME "panda-py")
set(CPACK_DEBIAN_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "HIROL Maintainers <kjustdoitno1@gmail.com>")
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "Franka Emika Panda Robot Python Interface")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libfranka, python3")
set(CPACK_TOPLEVEL_DIRECTORY "${CMAKE_BINARY_DIR}/cpack_temp")

project(
  "panda_py"
  VERSION 0.8.1  # Set your project version here
  LANGUAGES CXX
)

# Include CPack module and configure DEB settings
include(CPack)

set (CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE "Release")
set(THREADS_PREFER_PTHREAD_FLAG ON)

option(VACUUM_GRIPPER OFF)
if (VACUUM_GRIPPER)
  add_definitions(-DVACUUM_GRIPPER=${VACUUM_GRIPPER})
endif()

find_package(Eigen3 REQUIRED)
find_package(Threads REQUIRED)
find_package(Franka REQUIRED)
find_package(
  Python
  COMPONENTS Interpreter Development.Module
  REQUIRED)
find_package(pybind11 CONFIG REQUIRED)

## _core module
pybind11_add_module(_core
  src/_core.cpp
  src/panda.cpp
  src/controllers/joint_limits/virtual_wall.cpp
  src/controllers/integrated_velocity.cpp
  src/controllers/joint_position.cpp
  src/controllers/cartesian_impedance.cpp
  src/controllers/applied_torque.cpp
  src/controllers/applied_force.cpp
  src/controllers/force.cpp
  src/controllers/joint_trajectory.cpp
  src/controllers/cartesian_trajectory.cpp
  src/motion/generators.cpp
  src/motion/time_optimal/trajectory.cpp
  src/motion/time_optimal/path.cpp
)

target_link_libraries(_core PUBLIC
  Threads::Threads
  ${Franka_LIBRARIES}
)
include_directories(/usr/include/x86_64-linux-gnu) # for ubuntu 24.04
target_include_directories(_core SYSTEM PUBLIC
  include
  ${EIGEN3_INCLUDE_DIRS}
  ${Franka_INCLUDE_DIRS}
)

#target_compile_definitions(_core
#  PRIVATE VERSION_INFO=${PROJECT_VERSION})

install(TARGETS _core LIBRARY DESTINATION panda_py)

## libfranka module
pybind11_add_module(libfranka
  src/libfranka.cpp)

target_link_libraries(libfranka PUBLIC
  ${Franka_LIBRARIES}
)

target_include_directories(libfranka SYSTEM PUBLIC
  ${Franka_INCLUDE_DIRS}
)

#target_compile_definitions(libfranka
#  PRIVATE VERSION_INFO=${PROJECT_VERSION})

install(TARGETS _core LIBRARY DESTINATION panda_py)
install(TARGETS libfranka LIBRARY DESTINATION panda_py)