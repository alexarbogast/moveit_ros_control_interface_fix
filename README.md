# Moveit ROS Control Plugin (patch)

This package is a patched version of the MoveIt2 [moveit_ros_control_interface](https://github.com/ros-planning/moveit2/blob/main/moveit_plugins/moveit_ros_control_interface/src/controller_manager_plugin.cpp) to enable multi-robot system configuration. Specifically, the `moveit_ros_control_interface/Ros2ControlManager` and the `moveit_ros_control_interface/Ros2ControlMultiManager`.

Since 55df0bccd5e884649780b4ceeee80891e563b57b the Ros2ControlMultiManager no longer works with ros2_control controller_managers in multiple namespaces.

For example, when using multiple robots with their own controller_managers in different namespaces:
```yaml
rob1:
  controller_manager:
    ros__parameters:
      update_rate: 100  # Hz
      joint_trajectory_controller:
        type: joint_trajectory_controller/JointTrajectoryController
      joint_state_broadcaster:
        type: joint_state_broadcaster/JointStateBroadcaster

  joint_trajectory_controller:
    ros__parameters:
      joints:
        - rob1_joint_1

      command_interfaces:
        - position

rob2:
  controller_manager:
    ros__parameters:
      update_rate: 100  # Hz
      joint_trajectory_controller:
        type: joint_trajectory_controller/JointTrajectoryController
      joint_state_broadcaster:
        type: joint_state_broadcaster/JointStateBroadcaster

  joint_trajectory_controller:
    ros__parameters:
      joints:
        - rob2_joint_1

      command_interfaces:
        - position

```
The Ros2ControlMultiManager should search for controller_managers in the `rob1` and `rob2` namespaces. The global namespace is searched instead because the namespace is ignored.
```cpp
if (!node_->has_parameter("ros_control_namespace")) 
{
  ns_ = node_->declare_parameter<std::string>("ros_control_namespace", "/");
}
else
{
  node_->get_parameter<std::string>("ros_control_namespace", ns_);
  RCLCPP_INFO_STREAM(LOGGER, "Namespace for controller manager was specified, namespace: " << ns_);
}
``` 
Thus, the user is restricted to a single namespace under the `ros_control_namespace` parameter.

## Usage
The patched plugins use the same names with the "Fix" suffix on the plugins.

In `moveit_controllers.yaml`
```yaml
# multiple namespaces
moveit_controller_manager: moveit_ros_control_interface/Ros2ControlMultiManagerFix

# OR

# single namespace
moveit_controller_manager: moveit_ros_control_interface/Ros2ControlManagerFix
ros_control_namespace: <namespace>
```

## Patch
This package fixes the search logic for the moveit_controller_manager. It now searches for `/controller_manager/list_controllers` in any namespace. In the above example, the `/rob1/controller_manager` and `/rob2/controller_manager` namespaces are found and their controllers are loaded in the controller manager.
