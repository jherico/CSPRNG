#! /usr/bin/env wish BUILD

# This CMake script has some complications to consider:
#
#   -DARCH=32          Architecture bits to use non-MSVC targets
#   -DARCH=64          (GCC/Clang/etc) that take the -m32 and -m64 flags.
#                      (For MSVC targets, select the correct -G option.)
#
#   -DLINKAGE=direct   Compile csprng.c/cpp directly into the exe
#   -DLINKAGE=static   Build libcsprng.a/csprng.lib and link that with exe
#
# The BUILD script is designed to handle these considerations for you.

cmake_minimum_required (VERSION 3.2)

project(csprng
  VERSION 1.0.0
  DESCRIPTION "OS-native PRNG wrapper"
  LANGUAGES C CXX)

set (CMAKE_C_STANDARD 99)
set (CMAKE_CXX_STANDARD 11)

add_library(csprng)

target_include_directories(csprng
PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

target_sources(csprng
  INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/duthomhas/csprng.h>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/duthomhas/csprng.hpp>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/duthomhas/is_iterable.hpp>
    $<INSTALL_INTERFACE:include/duthomhas/csprng.h>
    $<INSTALL_INTERFACE:include/duthomhas/csprng.hpp>
    $<INSTALL_INTERFACE:include/duthomhas/is_iterable.hpp>
  PRIVATE
    ${CMAKE_SOURCE_DIR}/source/csprng.cpp
)

add_executable(c-example)
target_sources(c-example PRIVATE examples/example.c)
target_link_libraries(c-example PRIVATE csprng)

add_executable(cpp-example)
target_sources(cpp-example PRIVATE examples/example.cpp)
target_link_libraries(cpp-example PRIVATE csprng)

###
### Installation and packaging
###
get_target_property(CSPRNG_PUBLIC_HEADERS csprng INTERFACE_SOURCES)
set_target_properties(csprng PROPERTIES PUBLIC_HEADER "${CSPRNG_PUBLIC_HEADERS}")
install(TARGETS csprng EXPORT csprngTargets PUBLIC_HEADER DESTINATION "include/duthomhas")
install(EXPORT csprngTargets
    FILE csprngTargets.cmake
    NAMESPACE csprng::
    DESTINATION lib/cmake/csprng)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "csprngConfigVersion.cmake"
    VERSION ${csprng_VERSION}
    COMPATIBILITY AnyNewerVersion)
install(FILES "csprngConfig.cmake" "${CMAKE_CURRENT_BINARY_DIR}/csprngConfigVersion.cmake"
    DESTINATION lib/cmake/csprng)
