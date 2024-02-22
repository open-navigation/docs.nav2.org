.. _navigation2_with_vector_objects:


Navigating with Vector Objects
******************************

- `Overview`_
- `Requirements`_
- `Configuring Vector Object Server`_
- `Preparing Nav2 stack`_
- `Demo Execution`_
- `Working with Vector Objects`_

.. image:: images/Vector_Object_server/vector_objects_demo.gif

Overview
========

This tutorial shows how to navigate with vector objects added to raster costmaps.
It could be made for different purposes, e.g. for hiding some areas or sticking-out robot parts, for adding virtual obstacles on maps, or like Costmap Filters do - adding keep-out or maximum speed restricted areas, but on vector basis.
In current tutorial added vector objects will be treated as obstacles in costmaps.
For that, we need to prepare Navigation2 stack with enabled Keepout Filter in it, operating along with Vector Object server which publishing an OccupancyGrid map with rasterized vector objects as an input mask for Keepout Filter.
Other use-cases are utilizing the similar principles and could be trivially adjusted after this tutorial completion.

.. note::

  Using with Keepout Filter is the choice for adding virtual obstacles or removing some areas from costmaps. However, Vector Object server is not restricted to be operated with Keepout Filter only. It also could be used with different Costmap Filters for another use-cases. For example, for having speed restriction area defined by vector shape, one could choose Speed Filter; or for polygon-defined room where camera to be switched-off, Vector Object server could be operated with Binary Filter.

Requirements
============

It is assumed ROS2 and Nav2 dependent packages are installed or built locally.
Please make sure that Nav2 project is also built locally as it was made in :ref:`build-instructions`.
For the best understanding how Keepout Filter works (which is the part of current configuration), it is also recommended to pass through the :ref:`navigation2_with_keepout_filter` tutorial.


Configuring Vector Object Server
================================

Vector Object server has its own ``vector_object_server.launch.py`` launch-file and preset parameters in the ``vector_object_server_params.yaml`` file for demonstration, though its trivial to add this to Nav2's main launch file if being used in practice.

In this tutorial, we are focusing on the application how to utilize the simple setup allowing to add virtual obstacles on costmaps.
For demonstration of Vector Object server capabilities, let's specify two obstacle shapes: triangle polygon and circle filled with "occupied" value, in order to don't allow the robot to go through them. The YAML-part for polygon and circle will look as follows:

.. code-block:: yaml

        shapes: ["Poly", "Circle"]
        Poly:
          type: "polygon"
          frame_id: "map"
          closed: True
          value: 100
          points: [0.3, 0.5, -0.4, 1.2, -0.4, -0.2]
        Circle:
          type: "circle"
          frame_id: "map"
          fill: True
          value: 100
          center: [1.5, 0.5]
          radius: 0.7

Where the triangle polygon is specified by ``{0.3, 0.5}, {-0.4, 1.2}, {-0.4, -0.2}`` 3-point shape and the circle has ``{1.5, 0.5}`` coordinate of its center with ``0.7`` meter radius in the ``map`` frame.
``closed`` true-value for the polygon and ``fill`` for the circle mean that both shapes to be filled the with specified ``value``.
This value is equal to ``100`` which means "occupied" in OccupancyGrid format.

.. note::

  The frame for vector objects were specified the same as map's global frame. It was chosen for the simplicity to have static objects on map. However, it is possible to specify the shape in any frame different from global map's one. For this case, Vector Object server will use dynamic output map processing & publishing, suitable for moving objects.

.. note::

  Each shape is being addressed by UUID, which could be specified manually in a string format. In the demonstration, it was skipped to specify UUID of the shapes in the parameters, so Vector Object server will automatically generate a new one for each shape. The list of UUID could be obtained later by calling ``GetShapes.srv`` service.

Costmap Filters are required ``CostmapFilterInfo.msg`` message to be published along with filter mask (rasterized map with vector shapes).
Costmap Filter Info message is being published by Costmap Filter Info server, which is also launching by ``vector_object_server.launch.py`` script.

Overall ``vector_object_server_params.yaml`` YAML-file for the demonstration to be look as follows:

.. code-block:: yaml

    vector_object_server:
      ros__parameters:
        map_topic: "vo_map"
        global_frame_id: "map"
        resolution: 0.05
        default_value: -1
        overlay_type: 0
        update_frequency: 1.0
        transform_tolerance: 0.1
        shapes: ["Poly", "Circle"]
        Poly:
          type: "polygon"
          frame_id: "map"
          closed: True
          value: 100
          points: [0.3, 0.5, -0.4, 1.2, -0.4, -0.2]
        Circle:
          type: "circle"
          frame_id: "map"
          fill: True
          value: 100
          center: [1.5, 0.5]
          radius: 0.7
    costmap_filter_info_server:
      ros__parameters:
        type: 0
        filter_info_topic: "costmap_filter_info"
        mask_topic: "vo_map"
        base: 0.0
        multiplier: 1.0

More detailed information about each Vector Object server parameter and its operating principle could be found on :ref:`configuring_vector_object_server` configuration guide page. Costmap Filter Info server parameters description could be found at :ref:`configuring_map_server` page.

After Vector Objects and Costmap Filters Info servers were configured, launch them by command from below.
Robot should bypass vector obstacles. For the demonstration purposes it is enough to avoid path planning through them.
Thus, vector objects should be added to global costmap, so we need to choose ``global_costmap`` namespace for correct work:

.. code-block:: bash

  ros2 launch nav2_map_server vector_object_server.launch.py namespace:=global_costmap

Preparing Nav2 stack
====================

Vector Object server puts shapes to OccupacyGrid map and publishes it in a topic, which is used as an input mask for enabled in Nav2 Keepout Filter.
Enabling of Keeput Filter in Nav2 stack principles are similar as written in :ref:`navigation2_with_keepout_filter` tutorial.
Since vector objects are being enabled in global costmaps, Keepout Filter called as "vector_object_layer", should be added to the global costmap section of the ``nav2_params.yaml`` standard Nav2 configuration as follows:

.. code-block:: yaml

    global_costmap:
      ros__parameters:
        plugins: ["static_layer", "obstacle_layer", "inflation_layer"]
        filters: ["vector_object_layer"]
        ...
        vector_object_layer:
          plugin: "nav2_costmap_2d::KeepoutFilter"
          enabled: True
          filter_info_topic: "costmap_filter_info"

Demo Execution
==============

After Vector Object server was launched and Vector Object layer was enabled for the global costmap, run Nav2 stack as written in :ref:`getting_started`:

.. code-block:: bash

  ros2 launch nav2_bringup tb3_simulation_launch.py headless:=False

We are using composable nodes technology, so in the console where Vector Object server run the following message should appear:

.. code-block:: text

  [leha@leha-PC nav2_ws]$ ros2 launch nav2_map_server vector_object_server.launch.py namespace:=global_costmap
  [INFO] [launch]: All log files can be found below /home/leha/.ros/log/2023-11-24-13-18-42-257011-leha-PC-18207
  [INFO] [launch]: Default logging verbosity is set to INFO
  [INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/global_costmap/lifecycle_manager_vo_server' in container 'nav2_container'
  [INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/global_costmap/vector_object_server' in container 'nav2_container'
  [INFO] [launch_ros.actions.load_composable_nodes]: Loaded node '/global_costmap/costmap_filter_info_server' in container 'nav2_container'

The last lines mean that all three nodes: Vector Object server, Costmap Filter Info server and Lifecycle Manager handling them, were successfully loaded into running Nav2 container ``nav2_container``.

Set the initial pose for the robot, and check that vector objects were appeared on global costmap:

  .. image:: images/Vector_Object_server/vector_objects_on_costmap.png
    :width: 860px

As well as for the Keepout Filter, robot will consider vector objects as obstacles on costmaps and will avoid them:

  .. image:: images/Vector_Object_server/vector_objects_avoidance.png
    :width: 860px

Working with Vector Objects
===========================

During the operation, vector objects can be changed, added or removed.
Let's change triangle shape to the bar.

As was mentioned above, each shape is handled by its own UUID, which is being generated by Vector Object server if it is not specified explicitly in parameters.
To obtain shapes UUID, run the ``GetShapes.srv`` service request from the console:

.. code-block:: bash

  ros2 service call /global_costmap/vector_object_server/get_shapes nav2_msgs/srv/GetShapes

The output is expected to be the as follows:

.. code-block:: text

  requester: making request: nav2_msgs.srv.GetShapes_Request()

  response:
  nav2_msgs.srv.GetShapes_Response(circles=[nav2_msgs.msg.CircleObject(header=std_msgs.msg.Header(stamp=builtin_interfaces.msg.Time(sec=0, nanosec=0), frame_id='map'), uuid=unique_identifier_msgs.msg.UUID(uuid=array([73, 141, 241, 249, 116, 24, 69, 81, 178, 153, 159, 19, 245, 152, 28, 29], dtype=uint8)), center=geometry_msgs.msg.Point32(x=1.5, y=0.5, z=0.0), radius=0.699999988079071, fill=True, value=100)], polygons=[nav2_msgs.msg.PolygonObject(header=std_msgs.msg.Header(stamp=builtin_interfaces.msg.Time(sec=0, nanosec=0), frame_id='map'), uuid=unique_identifier_msgs.msg.UUID(uuid=array([153, 128, 30, 121, 241, 60, 76, 15, 140, 187, 58, 60, 164, 241, 97, 39], dtype=uint8)), points=[geometry_msgs.msg.Point32(x=0.30000001192092896, y=0.5, z=0.0), geometry_msgs.msg.Point32(x=-0.4000000059604645, y=1.2000000476837158, z=0.0), geometry_msgs.msg.Point32(x=-0.4000000059604645, y=-0.20000000298023224, z=0.0)], closed=True, value=100)])

In our case, UUID for triangle polygon will be ``[153, 128, 30, 121, 241, 60, 76, 15, 140, 187, 58, 60, 164, 241, 97, 39]``.

Calling ``AddShapes.srv`` service will add new shape if no UUID was specified, or given UUID was not found.
If UUID is already existing, the shape will be updated.
The triangle to be changed to the bar polygon with 4 points.
Call the following service request with obtained UUID to do this:

.. code-block:: bash

  ros2 service call /global_costmap/vector_object_server/add_shapes nav2_msgs/srv/AddShapes "polygons: [{points: [{x: 0.0, y: 1.0}, {x: -0.5, y: 1.0}, {x: -0.5, y: 0.0}, {x: 0.0, y: 0.0}], closed: true, value: 100, uuid: {uuid: [153, 128, 30, 121, 241, 60, 76, 15, 140, 187, 58, 60, 164, 241, 97, 39]}}]"

The polygon shape in Vector Object server will be changed, ``vo_map`` will be updated and resulting costmap will look as follows - triangle obstacle was updated to bar:

  .. image:: images/Vector_Object_server/vector_objects_updated.png

Finally, remove all shapes from map.
To remove any existing shape, there is ``RemoveShapes.srv`` service supported. It has array of UUID-s to specify which shapes to remove or just ``all_objects`` option for the case if we want to remove all shapes at once. Let's do the latter:

.. code-block:: bash

  ros2 service call /global_costmap/vector_object_server/remove_shapes nav2_msgs/srv/RemoveShapes "all_objects: true"

As a result, all vector shapes were disappeared from global costmap:

  .. image:: images/Vector_Object_server/vector_objects_removed.png
