cmake_minimum_required(VERSION 3.1)
project(ChemPlasKin LANGUAGES CXX C)

set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")

find_package(Threads REQUIRED)

include_directories("../cantera/include")
include_directories("../cantera/src")
include_directories("../cantera/include/cantera/ext")
include_directories("../cantera/ext/eigen")
include_directories("/root/mambaforge3/include") # For fmt and yaml-cpp headers
include_directories("/root/mambaforge3/include/eigen3") # For Eigen

include_directories("../cantera/ext/muparser/include")
include_directories("../cantera/ext/bolos/include")

link_directories("../cantera/build/lib")
link_directories("../cantera/ext/muparser/build")
link_directories("../cantera/ext/bolos/build")
link_directories("/root/mambaforge3/lib") # For libfmt and libyaml-cpp

add_executable(ChemPlasKin src/master.cpp)

if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    find_library(ACCELERATE_FRAMEWORK Accelerate)
    target_link_libraries(ChemPlasKin PRIVATE cantera_shared muparser bolos_lib fmt ${ACCELERATE_FRAMEWORK} Threads::Threads)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    target_link_libraries(ChemPlasKin PRIVATE cantera_shared muparser bolos_lib fmt stdc++fs Threads::Threads)
endif()