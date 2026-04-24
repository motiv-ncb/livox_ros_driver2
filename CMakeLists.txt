# Copyright(c) 2020 livoxtech limited.

cmake_minimum_required(VERSION 3.14)
project(livox_ros_driver2)

# Default to C99
@@ -24,17 +24,21 @@ include(cmake/version.cmake)
project(${PROJECT_NAME} VERSION ${LIVOX_ROS_DRIVER2_VERSION} LANGUAGES CXX)
message(STATUS "${PROJECT_NAME} version: ${LIVOX_ROS_DRIVER2_VERSION}")






# find dependencies
# uncomment the following section in order to fill in
# further dependencies manually.
# find_package(<dependency> REQUIRED)
find_package(ament_cmake_auto REQUIRED)
ament_auto_find_build_dependencies()
find_package(PCL REQUIRED)
find_package(std_msgs REQUIRED)
find_package(builtin_interfaces REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(spdlog REQUIRED)
find_package(fmt REQUIRED)



# check apr
find_package(PkgConfig)
pkg_check_modules(APR apr-1)
@@ -52,83 +56,6 @@ rosidl_generate_interfaces(${LIVOX_INTERFACES}
  LIBRARY_NAME ${PROJECT_NAME}
)

# Prefer the ROS2 exported package when available.
find_package(livox_sdk2 QUIET)
set(LIVOX_LIDAR_SDK_LIBRARY "")
set(LIVOX_LIDAR_SDK_INCLUDE_DIR "")

if(TARGET livox_sdk2::livox_lidar_sdk_shared)
  set(LIVOX_LIDAR_SDK_LIBRARY livox_sdk2::livox_lidar_sdk_shared)
  if(DEFINED livox_sdk2_INCLUDE_DIRS)
    set(LIVOX_LIDAR_SDK_INCLUDE_DIR ${livox_sdk2_INCLUDE_DIRS})
  endif()
else()
  # Fallback for non-ament Livox-SDK2 installs (manual cmake/make install).
  unset(LIVOX_LIDAR_SDK_INCLUDE_DIR CACHE)
  unset(LIVOX_LIDAR_SDK_LIBRARY CACHE)

  find_path(LIVOX_LIDAR_SDK_INCLUDE_DIR
    NAMES livox_lidar_api.h
    PATH_SUFFIXES include
    PATHS
      ${CMAKE_CURRENT_SOURCE_DIR}/../Livox-SDK2
      /usr/local
      /usr
  )

  find_library(LIVOX_LIDAR_SDK_LIBRARY
    NAMES livox_lidar_sdk_shared
    PATH_SUFFIXES lib lib64 build/sdk_core sdk_core
    PATHS
      ${CMAKE_CURRENT_SOURCE_DIR}/../Livox-SDK2
      /usr/local
      /usr
  )

  # Explicit workspace fallback: sibling Livox-SDK2 source tree.
  get_filename_component(_WS_SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/.." ABSOLUTE)
  if(NOT LIVOX_LIDAR_SDK_INCLUDE_DIR)
    if(EXISTS "${_WS_SRC_DIR}/Livox-SDK2/include/livox_lidar_api.h")
      set(LIVOX_LIDAR_SDK_INCLUDE_DIR "${_WS_SRC_DIR}/Livox-SDK2/include")
    endif()
  endif()

  if(NOT LIVOX_LIDAR_SDK_LIBRARY)
    if(EXISTS "${_WS_SRC_DIR}/Livox-SDK2/build/sdk_core/liblivox_lidar_sdk_shared.so")
      set(LIVOX_LIDAR_SDK_LIBRARY "${_WS_SRC_DIR}/Livox-SDK2/build/sdk_core/liblivox_lidar_sdk_shared.so")
    elseif(EXISTS "${_WS_SRC_DIR}/Livox-SDK2/build/sdk_core/liblivox_lidar_sdk_static.a")
      set(LIVOX_LIDAR_SDK_LIBRARY "${_WS_SRC_DIR}/Livox-SDK2/build/sdk_core/liblivox_lidar_sdk_static.a")
    endif()
  endif()

  # Last fallback: build sibling Livox-SDK2/sdk_core directly in this build.
  if(NOT LIVOX_LIDAR_SDK_LIBRARY)
    set(_LIVOX_SDK2_LOCAL_DIR "${_WS_SRC_DIR}/Livox-SDK2")
    if(EXISTS "${_LIVOX_SDK2_LOCAL_DIR}/sdk_core/CMakeLists.txt")
      add_subdirectory(
        "${_LIVOX_SDK2_LOCAL_DIR}/sdk_core"
        "${CMAKE_CURRENT_BINARY_DIR}/livox_sdk2_sdk_core"
      )
      if(TARGET livox_lidar_sdk_shared)
        set(LIVOX_LIDAR_SDK_LIBRARY livox_lidar_sdk_shared)
      elseif(TARGET livox_lidar_sdk_static)
        set(LIVOX_LIDAR_SDK_LIBRARY livox_lidar_sdk_static)
      endif()

      if(EXISTS "${_LIVOX_SDK2_LOCAL_DIR}/include/livox_lidar_api.h")
        set(LIVOX_LIDAR_SDK_INCLUDE_DIR "${_LIVOX_SDK2_LOCAL_DIR}/include")
      endif()
    endif()
  endif()
endif()

if(NOT LIVOX_LIDAR_SDK_LIBRARY)
  message(FATAL_ERROR
    "Could not find Livox SDK2 library. Install the ROS2 fork that exports livox_sdk2 "
    "or install Livox-SDK2 to /usr/local (liblivox_lidar_sdk_shared + headers)."
  )
endif()

## PCL library
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})
@@ -164,20 +91,22 @@ target_link_libraries(${PROJECT_NAME} "${cpp_typesupport_target}")
target_include_directories(${PROJECT_NAME} PUBLIC
  ${PCL_INCLUDE_DIRS}
  ${APR_INCLUDE_DIRS}
  ${LIVOX_LIDAR_SDK_INCLUDE_DIR}
  ${LIVOX_INTERFACES_INCLUDE_DIRECTORIES}   # for custom msgs
  3rdparty
  src
)

# link libraries
target_link_libraries(${PROJECT_NAME}
  ${LIVOX_LIDAR_SDK_LIBRARY}
  ${LIVOX_INTERFACE_TARGET}   # for custom msgs
  ${PPT_LIBRARY}
  ${Boost_LIBRARY}
  ${PCL_LIBRARIES}
  ${APR_LIBRARIES}
  spdlog::spdlog
  fmt::fmt

)

rclcpp_components_register_node(${PROJECT_NAME}
@@ -187,12 +116,6 @@ rclcpp_components_register_node(${PROJECT_NAME}

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # uncomment the line when a copyright and license is not present in all source files
  #set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # uncomment the line when this package is not in a git repo
  #set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()
