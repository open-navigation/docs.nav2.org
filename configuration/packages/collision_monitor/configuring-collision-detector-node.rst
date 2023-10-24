.. _configuring_collision_detector_node:

Collision Detector Node
#######################

The Collision Detector is a node similar to the Collision Monitor, so it is recommended to read the :ref:`collision_monitor_tutorial` tutorial first.

In some cases, the user may want to be informed about the detected obstacles without affecting the robot's velocity and instead take a different action within an external node. For example, the user may want to blink LEDs or sound an alarm when the robot is close to an obstacle.
Another use case could be to detect data points in particular regions (e.g extremely close to the sensor) and warn of malfunctioning sensors. For this purpose, the Collision Detector node was introduced.
It works similarly to the Collision Monitor, but does not affect the robot's velocity. It will only inform that data from the configured sources has been detected within the configured polygons via message to the ``collision_detector_state`` topic.

See the package's ``README`` for more information.

Features
********

Similarly to the Collision Monitor, the Collision Detector uses robot's relative polygons to define "zones".
However, unlike the Collision Monitor that uses different behavior models, the Collision Detector does not use any of them and therefore the `action_type` should always be set to `none`. If set to anything else, it will throw an error

The zones around the robot and the data sources are the same as for the Collision Monitor, with the exception of the footprint polygon, which is not supported by the Collision Detector.

Parameters
**********

:frequency:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  double         10.0
  ============== =============================

  Description:
    Frequency of the main loop that checks for detections.

:base_frame_id:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  string         "base_footprint"
  ============== =============================

  Description:
    Robot base frame.

:odom_frame_id:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  string         "odom"
  ============== =============================

  Description:
    Which frame to use for odometry.

:transform_tolerance:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  double         0.1
  ============== =============================

  Description
    Time with which to post-date the transform that is published, to indicate that this transform is valid into the future.

:source_timeout:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  double         2.0
  ============== =============================

  Description:
    Maximum time interval in which source data is considered as valid.

:base_shift_correction:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  bool           True
  ============== =============================

  Description:
    Whether to correct source data towards to base frame movement, considering the difference between current time and latest source time. If enabled, produces more accurate sources positioning in the robot base frame, at the cost of slower performance. This will cause average delays for ``~1/(2*odom_rate)`` per each ``cmd_vel`` calculation cycle. However, disabling this option for better performance is not recommended for the fast moving robots, where during the typical rate of data sources, robot could move unacceptably far. Thus reasonable odometry rates are recommended (~100 hz).

:polygons:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  vector<string> N/A
  ============== =============================

  Description:
    List of zones to check for data points. Causes an error, if not specialized.


:observation_sources:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  vector<string> N/A
  ============== =============================

  Description:
    List of data sources (laser scanners, pointclouds, etc...). Causes an error, if not specialized.

Polygons parameters
===================

``<polygon name>`` is the corresponding polygon name ID selected for this type.

:``<polygon_name>``.type:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  string         N/A
  ============== =============================

  Description:
    Type of polygon shape. Available values are ``polygon``, ``circle``. Causes an error, if not specialized.

:``<polygon_name>``.points:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  vector<double> N/A
  ============== =============================

  Description:
    Polygon vertexes, listed in ``{p1.x, p1.y, p2.x, p2.y, p3.x, p3.y, ...}`` format (e.g. ``{0.5, 0.25, 0.5, -0.25, 0.0, -0.25, 0.0, 0.25}`` for the square in the front). Used for ``polygon`` type. Minimum 3 points for a triangle polygon. If not specified, the collision detector will use dynamic polygon subscription to ``polygon_sub_topic``

:``<polygon_name>``.polygon_sub_topic:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  string         N/A
  ============== =============================

  Description:
    Topic to listen the polygon points from. Causes an error, if not specified **and** points are also not specified. If both ``points`` and ``polygon_sub_topic`` are specified, the static ``points`` takes priority.

:``<polygon_name>``.radius:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  double         N/A
  ============== =============================

  Description:
    Circle radius. Used for ``circle`` type. Causes an error, if not specialized.

:``<polygon_name>``.action_type:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  string         N/A
  ============== =============================

  Description:
    Only ``none`` action type is supported (more options available for collision monitor)

:``<polygon_name>``.min_points:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  int            4
  ============== =============================

  Description:
    Minimum number of data readings within a zone to trigger the action. Former ``max_points`` parameter for Humble, that meant the maximum number of data readings within a zone to not trigger the action). ``min_points`` is equal to ``max_points + 1`` value.

:``<polygon_name>``.visualize:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  bool           False
  ============== =============================

  Description:
    Whether to publish the polygon in a separate topic.

:``<polygon_name>``.polygon_pub_topic:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  string         <polygon_name>
  ============== =============================

  Description:
    Topic name to publish a polygon to. Used only if ``visualize`` is true.

:``<source name>``.enabled:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  bool           True
  ============== =============================

  Description:
    Whether to use this source for collision detection. (Can be dynamically set)

Observation sources parameters
==============================

``<source name>`` is the corresponding data source name ID selected for this type.

:``<source name>``.type:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  string         "scan"
  ============== =============================

  Description:
    Type of polygon shape. Could be ``scan``, ``pointcloud``, ``range`` or ``polygon``.

:``<source name>``.topic:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  string         "scan"
  ============== =============================

  Description:
    Topic to listen the source data from.

:``<source name>``.min_height:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  double         0.05
  ============== =============================

  Description:
    Minimum height the PointCloud projection to 2D space started from. Applicable for ``pointcloud`` type.

:``<source name>``.max_height:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  double         0.5
  ============== =============================

  Description:
    Maximum height the PointCloud projection to 2D space ended with. Applicable for ``pointcloud`` type.

:``<source name>``.obstacles_angle:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  double         PI / 180 (1 degree)
  ============== =============================

  Description:
    Angle increment (in radians) between nearby obstacle points at the range arc. Two outermost points from the field of view are not taken into account (they will always exist regardless of this value). Applicable for ``range`` type.

:``<source name>``.sampling_distance:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  double         0.1
  ============== =============================

  Description:
    Internally the polygon is sampled for collision detection. sampling_distance is the distance between sampled points of the polygon. Applicable for ``polygon`` type.

:``<source name>``.enabled:

  ============== =============================
  Type           Default
  -------------- -----------------------------
  bool           True
  ============== =============================

  Description:
    Whether to use this source for collision detection. (Can be dynamically set)

Example
*******

Here is an example of configuration YAML for the Collision Detector.

.. code-block:: yaml

    collision_detector:
      ros__parameters:
        base_frame_id: "base_footprint"
        odom_frame_id: "odom"
        transform_tolerance: 0.5
        source_timeout: 5.0
        base_shift_correction: True
        polygons: ["PolygonFront"]
        PolygonFront:
          type: "polygon"
          points: [0.3, 0.3, 0.3, -0.3, 0.0, -0.3, 0.0, 0.3]
          action_type: "none"
          min_points: 4
          visualize: True
          polygon_pub_topic: "polygon_front"
        observation_sources: ["scan"]
        scan:
          type: "scan"
          topic: "scan"
          enabled: True
        pointcloud:
          type: "pointcloud"
          topic: "/intel_realsense_r200_depth/points"
          min_height: 0.1
          max_height: 0.5
          enabled: True

