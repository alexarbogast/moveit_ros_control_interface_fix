cmake_minimum_required(VERSION 3.22)
project(moveit_ros_control_interface_fix LANGUAGES CXX)

# This module provides installation directories as per the GNU coding standards.
include(GNUInstallDirs)

find_package(ament_cmake REQUIRED)
find_package(moveit_common REQUIRED)
find_package(rclcpp_action REQUIRED)
find_package(moveit_core REQUIRED)
find_package(moveit_ros_control_interface REQUIRED)
find_package(pluginlib REQUIRED)

include_directories(
  include
)

add_library(moveit_ros_control_interface_fix SHARED
  src/controller_manager_plugin_fix.cpp
)
ament_target_dependencies(moveit_ros_control_interface_fix PUBLIC
  moveit_core
  moveit_ros_control_interface
  pluginlib
)
  
pluginlib_export_plugin_description_file(moveit_core moveit_core_plugins.xml)

# Install our library.
install(
  TARGETS moveit_ros_control_interface_fix
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}      # lib
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}      # lib
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}      # bin

)
  
ament_package()
