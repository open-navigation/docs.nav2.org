.. _foxy_migration:

Foxy to Galactic
################

Moving from ROS 2 Foxy to Galactic, a number of stability improvements were added that we will not specifically address here.

NavigateToPose BT-node Interface Changes
****************************************

The NavigateToPose input port has been changed to PoseStamped instead of Point and Quaternion.

See :ref:`bt_navigate_to_pose_action` for more information.

BackUp BT-node Interface Changes
********************************

The ``backup_dist`` and ``backup_speed`` input ports should both be positive values indicating the distance to go backward respectively the speed with which the robot drives backward.

BackUp Recovery Interface Changes
*********************************

``speed`` in a backup recovery goal should be positive indicating the speed with which to drive backward.
``target.x`` in a backup recovery goal should be positive indicating the distance to drive backward.
In both cases negative values are silently inverted.

New Plugins
***********

``nav2_waypoint_follower`` has an action server that takes in a list of waypoints to follow and follow them in order. In some cases we might want robot to 
perform some tasks/behaviours at arrivals of these waypoints. In order to perform such tasks, a generic plugin interface `WaypointTaskExecutor` has been added to ``nav2_core``.
Users can inherit from this interface to implement their own plugin to perform more specific tasks at waypoint arrivals for their needs. 

Several example implementations are included in ``nav2_waypoint_follower``. ``WaitAtWaypoint`` and ``PhotoAtWaypoint`` plusings are included in 
``nav2_waypoint_follower`` as run-time loadable plugins. ``WaitAtWaypoint`` simply lets robot to pause for a specified amount of time in milliseconds, at waypoint arrivals.
 While ``PhotoAtWaypoint`` takes photos at waypoint arrivals and saves the taken photos to specified directory, the format for taken photos also can be configured through parameters.
 All major image formats such as ``png``, ``jpeg``, ``jpg`` etc. are supported, the default format is ``png``.

Loading a plugin of this type is done through ``nav2_bringup/params/nav2_param.yaml``, by specifying plugin's name, type and it's used parameters. 

For instance; 
.. code-block:: yaml

    waypoint_follower:
      ros__parameters:
        loop_rate: 20
        stop_on_failure: false
        waypoint_task_executor_plugin: "wait_at_waypoint"
          wait_at_waypoint:
            plugin: "nav2_waypoint_follower::WaitAtWaypoint"
            enabled: True
            waypoint_pause_duration: 0

Original GitHub tickets:

- `WaypointTaskExecutor <https://github.com/ros-planning/navigation2/pull/1993>`_
- `WaitAtWaypoint <https://github.com/ros-planning/navigation2/pull/1993>`_
- `PhotoAtWaypoint <https://github.com/ros-planning/navigation2/pull/2041>`_


Costmap Filters
***************

A new concept interacting with spatial-dependent objects called "Costmap Filters" appeared in Galactic (more information about this concept could be found at :ref:`concepts` page). Costmap filters are acting as a costmap plugins. In order to make a filtered costmap and change robot's behavior in annotated areas, filter plugin reads the data came from filter mask. Then this data is being linearly transformed into feature map in a filter space. It could be passability of an area, maximum speed limit in m/s, robot desired direction in degrees or anything else. Transformed feature map along with the map/costmap, sensors data and current robot position is used in plugin's algorithms to make required updates in the resulting costmap and robot's behavor.

Architecturally, costmap filters consists from ``CostmapFilter`` class which is a basic class incorporating much common of its inherited filter plugins:

- ``KeepoutFilter``: keep-out/safety zones filter plugin.
- ``SpeedFilter``: slow/speed-restricted areas filter.
- Preferred lanes in industries. This plugin is covered by ``KeepoutFilter`` (see discussion in `corresponding PR <https://github.com/ros-planning/navigation2/issues/1522>`_ for more details).

Each costmap filter subscribes to filter info topic (publishing by `Costmap Filter Info Publisher Server <https://github.com/ros-planning/navigation2/tree/main/nav2_map_server/src/costmap_filter_info>`_) having all necessary information for loaded costmap filter and filter mask topic.

High-level design of this concept could be found `here <https://github.com/ros-planning/navigation2/tree/main/doc/design/CostmapFilters_design.pdf>`_. The functionality of costmap filters is being disscussed in `the ticket #1263 <https://github.com/ros-planning/navigation2/issues/1263>`_ and carried out by `PR #1882 <https://github.com/ros-planning/navigation2/pull/1882>`_. The following tutorial: :ref:`navigation2_with_keepout_filter` will help to easily get involved with ``KeepoutFilter`` functionality.

SmacPlanner
***********

A new package, ``SmacPlanner`` was added containing 4 or 8 connected 2D A*, and Dubin and Reed-shepp model hybrid-A* with smoothing, multi-resolution query, and more.

The ``smac_planner`` package contains an optimized templated A* search algorithm used to create multiple A*-based planners for multiple types of robot platforms. We support differential-drive and omni-directional drive robots using the ``SmacPlanner2D`` planner which implements a cost-aware A* planner. We support cars, car-like, and ackermann vehicles using the ``SmacPlanner`` plugin which implements a Hybrid-A* planner. This plugin is also useful for curvature constrained planning, like when planning robot at high speeds to make sure they don't flip over or otherwise skid out of control.

The `SmacPlanner` fully-implements the Hybrid-A* planner as proposed in `Practical Search Techniques in Path Planning for Autonomous Driving <https://ai.stanford.edu/~ddolgov/papers/dolgov_gpp_stair08.pdf>`_, including hybrid searching, CG smoothing, analytic expansions and hueristic functions.
