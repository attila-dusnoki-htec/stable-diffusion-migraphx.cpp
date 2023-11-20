cmake_minimum_required(VERSION 3.12)
project("stable-diffusion")

set(CMAKE_CXX_STANDARD 17)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if (NOT XCODE AND NOT MSVC AND NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    set(SD_STANDALONE ON)
else()
    set(SD_STANDALONE OFF)
endif()

set(ROCM_PATH /opt/rocm)
set(HIP_PATH "${ROCM_PATH}/hip")
list(APPEND CMAKE_PREFIX_PATH ${ROCM_PATH} ${HIP_PATH})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O3 -W -Wall -pthread -D__HIP_PLATFORM_HCC__=1")
find_package(migraphx REQUIRED)

#
# Option list
#

# general
#option(SD_BUILD_TESTS                "sd: build tests"    ${SD_STANDALONE})
option(SD_BUILD_EXAMPLES             "sd: build examples" ${SD_STANDALONE})
option(BUILD_SHARED_LIBS             "sd: build shared libs" OFF)
#option(SD_BUILD_SERVER               "sd: build server example"                           ON)


# deps
add_subdirectory(ggml)

set(SD_LIB stable-diffusion)

add_library(${SD_LIB} stable-diffusion.h stable-diffusion.cpp)
target_link_libraries(${SD_LIB} PUBLIC ggml migraphx::c hip::device)
target_include_directories(${SD_LIB} PUBLIC . ${ROCM_PATH})
target_compile_features(${SD_LIB} PUBLIC cxx_std_11)


if (SD_BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

