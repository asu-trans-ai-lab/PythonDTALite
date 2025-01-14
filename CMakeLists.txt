cmake_minimum_required (VERSION 3.8)
project ("DTALite")

option(BUILD_EXE "BUILD Executable or not" ON)
configure_file(build_config.h.in build_config.h)

if(BUILD_EXE)
    message("Building executable on ${CMAKE_SYSTEM_NAME}")
else()
    message("Building shared library on ${CMAKE_SYSTEM_NAME}")
endif()

# specify the C++ standard
# C++11 is necessary as teestream.h requires C++11 to be complied
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Set the MSVC runtime library based on your needs
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")
add_definitions(-D_ITERATOR_DEBUG_LEVEL=0)

include_directories(include)

# default name for Windows
set(YAML_CPP_LIB_NAME yaml-cpp)
set(YAML_CPP_LIB_DIR "${CMAKE_SOURCE_DIR}/lib")

if (CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    if(CMAKE_HOST_SYSTEM_PROCESSOR STREQUAL "arm64")
        set(YAML_CPP_LIB_NAME yaml-cpp_mac-arm)
    elseif(CMAKE_HOST_SYSTEM_PROCESSOR STREQUAL "x86_64")
        set(YAML_CPP_LIB_NAME yaml-cpp_mac-x86)
    endif()

    if (CMAKE_CXX_COMPILER_ID STREQUAL "AppleClang")
        set(CMAKE_CXX_FLAGS "-Xpreprocessor -fopenmp -fpic")
    else()
        set(CMAKE_CXX_FLAGS "-fopenmp -fpic")
    endif()
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    set(YAML_CPP_LIB_NAME yaml-cpp_linux)
    set(CMAKE_CXX_FLAGS "-fopenmp -fpic")
endif()

# note CMAKE_BUILD_TYPE WILL BE INGNORED for IDE generators, like Visual Studio and Xcode
# it only works for single-configuration generators, e.g., make and Ninja
# see https://cmake.org/cmake/help/v3.0/variable/CMAKE_BUILD_TYPE.html?highlight=cmake_build_types for details
set(CMAKE_BUILD_TYPE  "Release")
set(LIBRARY_OUTPUT_PATH  ${CMAKE_BINARY_DIR}/bin)

if (BUILD_EXE)
    add_executable(${PROJECT_NAME} src/utils.cpp src/flash_dta.cpp src/main_api.cpp src/simulation.cpp src/DTA_geometry.cpp)
else()
    add_library(${PROJECT_NAME} SHARED src/utils.cpp src/flash_dta.cpp src/main_api.cpp src/simulation.cpp src/DTA_geometry.cpp)
endif()

find_library(YAML_CPP NAMES ${YAML_CPP_LIB_NAME} PATHS ${YAML_CPP_LIB_DIR} NO_SYSTEM_ENVIRONMENT_PATH)
find_package(OpenMP REQUIRED)

target_link_libraries(${PROJECT_NAME} PRIVATE OpenMP::OpenMP_CXX)
target_link_libraries(${PROJECT_NAME} PRIVATE ${YAML_CPP})

target_include_directories(${PROJECT_NAME} PUBLIC ${PROJECT_BINARY_DIR})