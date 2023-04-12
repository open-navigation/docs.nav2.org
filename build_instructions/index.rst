.. _build-instructions:

Build and Install
#################

Install
*******

Nav2 and its dependencies are released as binaries.
You may install it via the following to get the latest stable released version:

.. code:: bash

  source /opt/ros/<distro>/setup.bash
  sudo apt install \
    ros-$ROS_DISTRO-navigation2 \
    ros-$ROS_DISTRO-nav2-bringup \
    ros-$ROS_DISTRO-turtlebot3*

Build
*****

There are a few ways to build Nav2:

* Using Released Distribution Binaries
* Using Rolling Development Source
* Using Docker Container Images

.. tip::
  For a *repeatable*, *reproducible* and *streamlined* development experience, check the Nav2 documentation on using :ref:`dev-containers`!

.. rst-class:: content-collapse

Released Distribution Binaries
==============================

To build Nav2, you'll first need to build or install ROS 2 and related development tools, including: ``colcon``, ``rosdep`` and ``vcstool``.

.. seealso::
  For more information on building or installing ROS 2 distros, see the official documentation:

  * `ROS 2 Installation <https://docs.ros.org/en/rolling/Installation.html>`_
  * `Install development tools and ROS tools <https://docs.ros.org/en/rolling/Installation/Alternatives/Ubuntu-Development-Setup.html#install-development-tools-and-ros-tools>`_

Once your environment is setup, clone the repo, install all dependencies, and build the workspace:

.. attention::
   For ROS 2 distros prior to ``galactic``, the branch naming schema was ``<distro>-devel``.

.. code:: bash

  source /opt/ros/<distro>/setup.bash
  export NAV2_WS=~/nav2_ws
  mkdir -p $NAV2_WS/src && cd $NAV2_WS
  git clone https://github.com/ros-planning/navigation2.git --branch $ROS_DISTRO ./src/navigation2
  rosdep install -y \
    --from-paths ./src \
    --ignore-src
  colcon build \
    --symlink-install

You can then ``source $NAV2_WS/install/setup.bash`` to get ready for demonstrations!

.. hint::
  For more examples on building Nav2 from released distribution binaries, checkout `distro.Dockerfile <https://github.com/ros-planning/navigation2/blob/main/tools/distro.Dockerfile>`_.

.. rst-class:: content-collapse

Using Rolling Development Source
================================

Building Nav2 using rolling development source is similar to building Nav2 from released distribution binaries, where instead you build dependencies from source using the main development branches for all ROS based packages.

.. seealso::
  For more information on building ROS 2 from source, see the official documentation:

  * `ROS 2 Building from source <https://docs.ros.org/en/rolling/Installation.html#building-from-source>`_

Once your environment is setup, clone the repo, import all dependencies, and build the workspace:

.. attention::
   Be sure to check that all dependencies you need are included and uncommented in the ``.repos`` file.

.. code:: bash

  source <ros_ws>/install/setup.bash
  export NAV2_WS=~/nav2_ws
  mkdir -p $NAV2_WS/src && cd $NAV2_WS
  git clone https://github.com/ros-planning/navigation2.git --branch $ROS_DISTRO ./src/navigation2
  vcs import ./src < ./src/navigation2/tools/underlay.repos
  rosdep install -y \
    --from-paths ./src \
    --ignore-src
  colcon build \
    --symlink-install

You can then ``source $NAV2_WS/install/setup.bash`` to get ready for demonstrations!

.. hint::
  For more examples on building Nav2 from rolling development source, checkout `source.Dockerfile <https://github.com/ros-planning/navigation2/blob/main/tools/source.Dockerfile>`_.

.. rst-class:: content-collapse

Using Docker Container Images
=============================

Building Nav2 using Docker container images provides a repeatable and reproducible environment to automate and self document the entire setup process. Instead of manually invoking the development tools as documented above, you can leverage the project's Dockerfiles to build and install Nav2 for various distributions.

.. seealso::
  For more information on installing Docker or leaning about Dockerfiles, see the official documentation:

  * `Docker Engine <https://docs.docker.com/engine/install>`_
  * `Dockerfile reference <https://docs.docker.com/engine/reference/builder>`_

Once your system is setup, you can build the Nav2 Dockerfile from the root of the repo:

.. code:: bash

  export ROS_DISTRO=rolling
  git clone https://github.com/ros-planning/navigation2.git --branch main
  docker build --tag navigation2:$ROS_DISTRO \
    --build-arg FROM_IMAGE=ros:$ROS_DISTRO \
    --build-arg OVERLAY_MIXINS="release ccache lld" \
    --cache-from ghcr.io/ros-planning/navigation2:main \
    ./navigation2

The `docker build <https://docs.docker.com/engine/reference/commandline/build/>`_ command above creates a tagged image using the `Dockerfile` from the context specified using the path to the repo, where build-time variables are set using additional arguments, e.g. passing a set of `colcon mixins <https://github.com/colcon/colcon-mixin-repository>`_ to configure the workspace build. Check the ``ARG`` directives in the `Dockerfile` to discover other build-time variables available. 

.. tip::
  The images cached from are used for Nav2 CI, but can also be used with Nav2 :ref:`dev-containers`!

!!!!

Generate Doxygen
****************

Run ``doxygen`` in the root of the Nav2 repository.
It will generate a ``/doc/*`` directory containing the documentation.
The documentation entrypoint in a browser is index.html.

!!!!

Help
****

:ref:`build-troubleshooting-guide`


.. toctree::
   :hidden:

   build_docs/build_troubleshooting_guide.rst
