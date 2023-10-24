.. _iron_migration:

Iron to Jazzy
##############

Moving from ROS 2 Iron to Jazzy, a number of stability improvements were added that we will not specifically address here.

Introduce a new Multi-Robot Bringup Launch
******************************************

`PR #3572 <https://github.com/ros-planning/navigation2/pull/3572>`_ introduces a new way of bringup tb3 multi-robot that names as ``cloned_tb3_simulation_launch.py`` for simulation. ``cloned_tb3_simulation_launch.py`` enables to bring up multiple robots with same parameter that described in ``nav2_multirobot_param_all.yaml``. And multiple robots are separeted by namespaces which are given as a Launch Arguments.
Existing ``multi_tb3_simulation_launch.py`` which was utilized in previous is replaced with ``unique_tb3_simulation_launch.py``, allowing for multiple unique robot instances utilizing ``nav2_multirobot_params_<N>.yaml`` configuration files.


New option for the Voxel and Obstacle Layers
********************************************
`PR #3612 <https://github.com/ros-planning/navigation2/pull/3612>`_ adds a new MaxWithoutUnknownOverwrite option to combination_method parameter in Voxel and Obstacle Layers. This can be used to make sure that the static map is the dominant source of information, and
easily prevent the robot to go through places that are not present in the static map.

Changes to MPPI Goal Critic
***************************

The MPPI Goal critic's formulation is changed to better keep up with speed on approach to goal instead of preemptively slowing too significantly. It also allows you to better use the weight to adjust the degree at which it slows more naturally. This change involves adjusting the ``threshold_to_consider`` to be the same as your prediction horizon (e.g. samples * dt * max speed) for both the goal critic and path follower critic to have a good hand-off between them without deceleration. 

Changes to MPPI Path Angle Critic
*********************************

MPPI's Path Angle critic now has a ``mode`` setting to adjust behavior depending on robot's desired behavioral traits. Previously, it penalized path orientations that deviated far the the robot's forward orientation to turn the robot towards sharp changes in the path. This is still default (``mode: 0``), but other modes now exist too. 

``mode: 1`` sets the penalization of path's relative directions by either forward orientation or the opposite for reversing to allow for true bidirectional motion when one way or another is not preferable for a symmetric robot. This uses only the path's relative points to the robot to decide which direction to incentivize. 

``mode: 2`` instead uses the path's orientations when a feasible path is given from the Smac Planners or the Smoother server's algorithms. This way, the globally planned orientations are followed rather than the based solely on the path's relative points. This is useful for non-circular robots in highly confined settings where there may be restricted opportunities to change directions so following the global path's orientation are required to end in the orientation you require.


Changes to MPPI Path Handling For Directionality
************************************************

MPPI's Path Align Critic and Path Handler object now have options to utilize the path's orientation information to force the controller to change directions when and only when requested by a feasible planner. When ``enforce_path_inversion`` is ``true``, the path handler will prune the path to the first time the directions change to force the controller to plan to the inversion point and then be set the rest of the path, once in tolerance. The Path Align critic also contains a parameter ``use_path_orientations``  which can be paired with it to incentivize aligning the path containing orientation information to better attempt to achieve path inversions where requested **and** not do them when not requested.

See MPPI's configuration guide for complete information.

Move Error Code Enumerations
****************************

`PR #3693 <https://github.com/ros-planning/navigation2/pull/3693>`_ moves the enumeration codes from the goal to the result section. 

Substitution in parameter file
******************************

Enabled substitution in parameter file. For example, you can write the following

.. code-block:: yaml

    bt_navigator:
      ros__parameters:
        default_nav_to_pose_bt_xml: $(find-pkg-share my_package)/behavior_tree/my_nav_to_pose_bt.xml

For more information about substitutions syntax, see `here <https://docs.ros.org/en/rolling/How-To-Guides/Launch-files-migration-guide.html#substitutions>`_

Allow Behavior Server Plugins to Access The Action Result
*********************************************************

`PR #3704 <https://github.com/ros-planning/navigation2/pull/3704>`_ allows behavior servers plugins to access and modify the action result.

Smac Planner Debug Param Name Change
************************************

``debug_visualizations`` replaces ``viz_expansions`` parameter in Hybrid-A* to reflect the new inclusion of footprint debug information being published as well.


Added GPS Waypoint Follower Server
**********************************

`This PR 2814 <https://github.com/ros-planning/navigation2/pull/2814>`_  adds the ``follow_gps_waypoints`` action server in ``nav2_waypoint_follower``. This server accepts a set of GPS goals instead of cartesian goals and provides all the other functionalities available on ``nav2_waypoint_follower``. A new tutorial demonstrating its functionality was also added on `PR 70 on navigation2_tutorials <https://github.com/ros-planning/navigation2_tutorials/pull/70>`_ and can be found on the General Tutorials directory on this website. 

Smac Planner Hybrid-A* New Features
***********************************

New features ``allow_primitive_interpolation`` which allows for more primitives in the search set, ``use_quadratic_cost_penalty``  to impact the cost penalty order in the traversal and heuristic functions, and ``downsample_obstacle_heuristic`` to optionally not downsample the obstacle heuristic's costmap were added. The default behavior will remain the same. If you would like to use these new features, please check out the Smac Planner Hybrid-A* configuration guide. 

New node in nav2_collision_monitor: Collision Detector
******************************************************

In this `PR #3693 <https://github.com/ros-planning/navigation2/pull/3500>`_ A new node was introduced in the nav2_collision_monitor: Collision Detector. 
It works similarly to the Collision Monitor, but does not affect the robot's velocity. It will only inform that data from the configured sources has been detected within the configured polygons via message to the ``collision_detector_state`` topic that might be used by any external module (e.g. switching LED or sound alarm in case of collision).

Dynamic enabling/disabling of sources/polygons in Collision Monitor/Detector
****************************************************************************

In this `PR #3825 <https://github.com/ros-planning/navigation2/pull/3825>`_ we added the ability to dynamically enable/disable sources and polygons in the Collision Monitor/Detector.

Expose action server's result timeout
*************************************

In this `PR #3787 <https://github.com/ros-planning/navigation2/pull/3787>`_ the timeout for action server's result was exposed in all nodes having action servers. 
This is because in this `PR #1012 <https://github.com/ros2/rcl/pull/1012>`_ in rcl a change was introduced which makes action servers discard a goal handle if the result
is not produced within 10 seconds, when the default was set to 15 minutes before. Since some actions in Nav2 may take more than 10 seconds to complete, the user has now the ability
to set this value through the ``action_server_result_timeout`` parameter, which defaults to 15 minutes in the ``bt_navigators`` and ``waypoint_follower`` and to 10 seconds in all other nodes.

RewrittenYaml could add new parameters to YAMLs
***********************************************

Now ``RewrittenYaml`` widely used in Nav2 launch-scripts, could do not only substitutions of ROS-parameters existing in original YAML, but rather additions of new parameters, that did not exist in the YAML. Certainly, these parameters should be declared for target ROS-nodes, otherwise they won't be processed in run-time. In such functionality, they should be expressed in absolute values, separated by a dot. For example, the rewrite for a ``prune_distance`` parameter of a ``FollowPath`` node will look like ``'controller_server.ros__parameters.FollowPath.prune_distance': '1.0'`` in a ``param_rewrites`` dictionary of ``RewrittenYaml()`` argument.
The change was intoroduced in the scope of `PR #3785 <https://github.com/ros-planning/navigation2/pull/3785>`_ fix.

Simple Commander API Allows Multi-Robot Namespacing
***************************************************

The Simple Navigator API now allows multi-robot namespacing by exposing a ``namespace`` field in the constructor to allow you to specify the Nav2 stacks' namespace for a robot or system. See `this PR for details <https://github.com/ros-planning/navigation2/pull/3803/files>`_.

Change duration type in wait_action node
****************************************

In this `PR #3871 <https://github.com/ros-planning/navigation2/pull/3871>`_ the type of duration variable in wait_action node is changed from int to double, which allows you to use floating values for wait_action.

The costmap activation fails when required transforms are not available
***********************************************************************

In this `PR #3866 <https://github.com/ros-planning/navigation2/pull/3866>`_ the parameter ``initial_transform_timeout`` is added to the costmap. The activation of the costmap now fails,
if the transformation from the robot base frame to the global frame does not become available during this timeout.

Publishing of collision points in collision monitor/detector for debugging purposes
***********************************************************************************

In this `PR #3879 <https://github.com/ros-planning/navigation2/pull/3879>`_ the publishing of the collision points that will be considered by the configured polygons was added