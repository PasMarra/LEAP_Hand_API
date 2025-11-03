cmake_minimum_required(VERSION 3.14)

project(
  LEAP_HAND_API
  LANGUAGES CXX C
  VERSION 1.0.0)

set(PLATEFORM "linux64" CACHE STRING "Plateform to build dynamixel sdk")
message(STATUS "Please set your plateform with -DPLATEFORM={linux_sbc, linux32, linux64, mac} default: linux64")

find_package(Eigen3 REQUIRED)

## Build dynamixel sdk
add_custom_target(
    run_make_dynamixel_sdk ALL
  COMMAND make
  # Ensure install dirs exist (use CMake -E so parent dirs are created) to avoid Makefile's mkdir without -p
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}/lib
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}/include
    # Remove possible previous SDK symlinks (avoid ln: File exists). Remove common names installed by the SDK.
    COMMAND ${CMAKE_COMMAND} -E remove -f
      ${CMAKE_INSTALL_PREFIX}/lib/libdxl_x64_cpp.so
      ${CMAKE_INSTALL_PREFIX}/lib/libdxl_x64_cpp.so.2
      ${CMAKE_INSTALL_PREFIX}/lib/libdxl_x64_cpp.so.2.0
      ${CMAKE_INSTALL_PREFIX}/lib/libdxl_x64_cpp.so.2.0.0
  # Call make install, pass MKDIR and LD_CONFIG as make variables so they override assignments
  # inside the SDK Makefile (command-line variables take precedence). Use MKDIR=mkdir -p and
  # LD_CONFIG=true to avoid calling ldconfig as root.
  COMMAND make install INSTALL_ROOT=${CMAKE_INSTALL_PREFIX} LD_CONFIG=true MKDIR=mkdir\ -p
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/dynamixel_sdk/build/${PLATEFORM}
  COMMENT "Running dynamixel_sdk Makefile (installs to ${CMAKE_INSTALL_PREFIX} — no sudo)"
)

set(SRC_CLIENT
  src/dynamixel_client.cpp
  src/leap_hand_utils.cpp
  src/leap_controller.cpp
)

## Build dynamixel client library
add_library(dynamixel_client SHARED ${SRC_CLIENT})
add_dependencies(dynamixel_client run_make_dynamixel_sdk)
## Ensure linker and includes look into the CMake install prefix where the SDK Makefile installs
link_directories(${CMAKE_INSTALL_PREFIX}/lib)

# Put the built library under the lib/ subdirectory of the build tree
set_target_properties(dynamixel_client PROPERTIES
  ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_INSTALL_PREFIX}/lib"
  LIBRARY_OUTPUT_DIRECTORY "${CMAKE_INSTALL_PREFIX}/lib"
  RUNTIME_OUTPUT_DIRECTORY "${CMAKE_INSTALL_PREFIX}/lib"
)

target_link_libraries(dynamixel_client PUBLIC
  dxl_x64_cpp
  Eigen3::Eigen
)
target_include_directories(dynamixel_client PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<BUILD_INTERFACE:${CMAKE_INSTALL_PREFIX}/include>
)

# Build test_leap_hand
add_executable(test_leap_hand src/main.cpp)
target_link_libraries(test_leap_hand PRIVATE dynamixel_client)