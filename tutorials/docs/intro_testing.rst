.. _intro_testing:

Introduction to Testing
***********************
**Overview:**

- `Benefits of Testing`_
- `Hands on Testing`_
- `How to Write Your First Test`_
- `Tips & Tricks for Writing Tests`_

Benefits of Testing
===================

 .. image:: images/Testing/codecov.io_show_covered_code.png
  :width: 80%

Whenever a feature gets added or altered, it is best practice to confirm the intended outcome with tests.
This document features an easy step by step guide how tests should be integrated alongside new PRs.
Adding tests to your new code does not only help you to make sure that corner cases can be reliable tested 
over and over, but also to make sure that your code continues to work, when there are new changes made elsewhere, 
like the ROS2 infrastructure or other dependencies.

Tests can help in two scenarios. First, during programming they can be conducted manually, while they are still being tweaked to work in the intended way.
In order to test them manually and include them into your workflow, a few steps are necessary that are all explained in the next chapter.
Second, the tests added during the active programming phase of this new code can also be integrated into automated testing performed by the CI system setup with the nav2 github repository.
This helps others later in the continuous development process of this open source project.

Also, not only new code should have automated testing through CI. To find nasty bugs and corner cases, it is Nav2's ultimate goal to have a test coverage of nearly 90% !
This will lead to a more improved, stable, and industry-grade code basis, as new PRs will see if they break things in other sub-systems, that they normally would not think of impacting.
It would be great if you can always check into the automated tests results of CI during your PR.

The current code-coverage of the whole nav2 project can be tracked on 
`codecov.io/gh/ros-planning/navigation2 <https://codecov.io/gh/ros-planning/navigation2>`_ . 

During your PR process, CI will publish automated testing results directly into your PR as a comment as soon as you mark your PR as ready for review and/or new commits are added.
These results also give you great insights about what code coverage areas you added or even lost! 
It is a great visualization to help you understand what code is currently being used and what might be just dead code, as it is not tested and therefore cannot really be told if it works or not.

So, if you already work inside or with one of our components, please have a look what kind of tests are already there and where is room for you to improve our testing!
For example, the conditions in the code above did pass the test, but if there might be a more complex mechanism in the red part (the part that is not covered by automated testing) it can come to an unpredictive outcome.
Before heading into writing new tests that could easily test those corner cases, let's start with working with existing tests. 

Hands on Testing
================
0. Get Nav2 Working
-------------------
Before you can test anything, make sure you have the basic examples of the :ref:`getting_started` guides up and running.

For integrating and running your own tests, you have to build nav2 from source. Using nav2 main with ros2 installed as binaries technically works, 
but the automated CI testing, happening on github when you merge, also uses ros2 rolling (= latest/main).
Therefor, it is recommend to test your new code with the nav2 stack based on the main branch and also let the script (getting started guide) install ros2 from source.

1. Run Existing Tests
---------------------
Nav2 specific tests can be found in each individual component under a dedicated test folder. 
One additional package, ``nav2_system_tests``, `exists <https://github.com/ros-planning/navigation2/tree/main/nav2_system_tests>`_ for testing components, 
sub-systems and full system tests, in addition to unit and integration tests in individual packages.
This package also comes with a ``README.md`` pointing out, that unit tests for sub-functions of components should be provided within each individual component.

To run existing tests from a certain package like ``nav2_system_tests``, you can use this procedure:

.. code-block:: bash

  $ cd <your_nav2_workspace>
  $ colcon build --symlink-install # build nav2 workspace including the components that you are interested in
  $ colcon test [--event-handlers console_direct+] --packages-select <pkg-name> # run all the tests of <package name> [with output]

To run all tests of the complete nav2 stack, you can also use ``colcon`` for convenience: 

.. code-block:: bash

  $ colcon test
  
Now you should see a couple of tests run in your command line. After they have finished, 
``colcon`` outputs with the optional flag ``--event-handlers console_direct+`` an report about the tests.
This looks something like this:

.. code-block:: bash

  [.. Output of 30 individual tests ..]
  100% tests passed, 0 tests failed out of 30

  Label Time Summary:
  copyright     =  10.18 sec*proc (1 test)
  cppcheck      =   1.46 sec*proc (1 test)
  cpplint       =   2.72 sec*proc (1 test)
  gtest         =  49.72 sec*proc (24 tests)
  lint_cmake    =   1.18 sec*proc (1 test)
  linter        =  19.24 sec*proc (6 tests)
  uncrustify    =   1.65 sec*proc (1 test)
  xmllint       =   2.04 sec*proc (1 test)

  Total Test time (real) =  69.00 sec


You can see that in this case 30 individual tests did run without any errors.
Besides the 24 `gtest` tests that represent functional tests, there are also 12 tests of another kind.
Those other tests are `linters`.

2. What are Linters?
--------------------

While code might be functional correct and a compiler generously compiles the desired functionality already, 
`linters` ensure that the source code follows special coding, design and organizational guidelines.
This not only helps to comply with legal regulations, but also to sustain a standard way of how code is represented.
Especially while reading and understanding unfamiliar code, it helps that there are certain design guidelines enforced.

But what do they actually do?
Starting with `copyright`, a set of files with certain extensions for python, cpp, and others 
in this open source project must be published under certain licenses and include copyright holders.
But also code style guidelines are checked with `linters` like `cpplint` or `flake8` (python).
Such checks might include tests like: strong checks against how comments have to look like, no tailing white-spaces,
not too many following blank lines, not too many characters per line (99 chars), position of brackets, and so on.

There exists a common stack of linters used by ROS2. This ros2 documentation about `ament cmake <https://index.ros.org/doc/ros2/Tutorials/Ament-CMake-Documentation/#testing-and-linting>`_
not only shows how linters are correctly integrated into your package but also points towards the ``ament_lint_common`` documentation.
Here are all linters explained in there full extent and also listed which are added by default through the common package. 
Also, it is possible to write and add your own linters for more consistency checks.

In more complex code stacks like nav2, linters are included into the test process by ``CMakeList.txt`` 
and ``package.xml``, like pointed out by the ros2 documentation above. 
Therefore, **all** packages of nav2 have to include them by themselves.
A minimal setup for linters looks like this (`source <https://github.com/ament/ament_lint/blob/master/ament_lint_auto/doc/index.rst>`_):

``CMakeLists.txt``:

.. code:: cmake

    # this must happen before the invocation of ament_package()
    if(BUILD_TESTING)
      find_package(ament_lint_auto REQUIRED)
      ament_lint_auto_find_test_dependencies()
    endif()

``package.xml``:

.. code:: xml

    <test_depend>ament_lint_auto</test_depend>

    <!-- this recursively depends on a set of common linters -->
    <test_depend>ament_lint_common</test_depend>

To run some linter - ``ament_{cpplint, cppcheck, uncrustify, flake8, etc}`` you can us ``cpplint`` etc. as keyword for the regex in the next section.

3. Run Individual Tests
-----------------------
In the event that one specific test out of many tests might have failed, 
it makes sense to work on getting this one test to succeed again.
Also while developing individual tests, it might make sense to only run one out of multiple tests.
Referring to the *chapter 4.7* of the `colcon documentation <https://buildmedia.readthedocs.org/media/pdf/colcon/latest/colcon.pdf>`_, one can also run individual tests through ``colcon``, as it uses ``ctest`` and ``pytest`` under the hood. 
This can be done with:

.. code-block:: bash

 $ colcon test --event-handlers console_direct+ --packages-select <pkg-name> --ctest-args -R <regex>

Where ``regex`` represents the name or search-expression for the test(s) you want to run manually. 
For example with the ``nav2_system_tests`` package, the value for a valid regex could be ``bt``.
This would include all tests for behavior tree related tests. Also, respectively the keyword ``planner`` 
or another full name keyword would run the specific test you want to run. 
You can find the name of a certain test by running all tests of the desired package with the option 
``$ colcon test --event-handlers console_direct+ [<...>]`` or look the name up in the corresponding ``CMakeList.txt`` 
of the test sub-directory of the package you are currently working with. 

How to Write Your First Test
============================
After dealing with working and understanding the existing tests on a higher level, 
it is time to focus on adding and writing your own tests.
There are multiple packages your test might fit in. So it is important to plan your next steps.
After you chose the direction you want to head into, you have to decide between an unit-test or a launch-based test.

1. Plan Your Own Test
------------------------
**Before writing a new test**, you have to think about what you want to test:

- Is my feature relevant in combination with other (sub-)components of the nav2 stack? --> Integrate it into launch based ``nav2_system_tests``
- Is my feature only component specific? --> Write dedicated unit tests or launch based test inside of the individual component

**What makes a good test?**

- Code Coverage: Are all my new code lines run at least once with my test? (check manually or with codecov.io automated with each PR on github)
- Corner Cases: Monkey proof input, test the limits (overflow etc)
- Expect things to break: this is good! -> make sure to catch all errors and handle them accordingly
- Combine your components with other test: If feasible create corner scenarios where your code really should improve things
- Quantity over complexity: Better write multiple small tests rather than making them to complicate, so others cannot understand why it fails in the future

**What tools do I have?**

- look and learn from existing code in the nav2 stack, we have plenty of tests!
- play with different launch parameters: Have a look at overwriting them test-dependent in the last next section

2. Write Your First Test
------------------------

Tests need to be declared in individual ``CMakeList.txt`` files in the test sub-folders of components or in ``nav2_system_tests``.
To add a new test additional to the linters mentioned in the chapter before, we have to add them in the ``BUILD_TESTING`` condition.

This can be achieved by two methods to include multiple sub-folders with ``CMakeList.txt`` inside:

.. code:: cmake

    # this must happen before the invocation of ament_package()
    if(BUILD_TESTING)
      find_package(ament_lint_auto REQUIRED)
      ament_lint_auto_find_test_dependencies()
      # add gtest and pytest 
      find_package(ament_cmake_gtest REQUIRED)
      find_package(ament_cmake_pytest REQUIRED)

      # add your subdirectories directly with dedicated CMakeList.txt with pytests or gtests inside 
      add_subdirectory(src/planning)
      add_subdirectory(src/localization)
      add_subdirectory(src/system)
      add_subdirectory(src/system_failure)
      # ...

      # or 
      add_subdirectory(folder)
      # this one CMakeList.txt inside of this folder will then include the rest of the folders with their respective CMakeList.txt files

    endif()


a) Launch-file based Test - pytest
""""""""""""""""""""""""""""""""""
Here is an example for testing with python and pytest, especially useful for testing launch sequences.
Interesting to note are the free set-able environment variables that can later be used to rewrite parameter values for launch scripts.
`Source for the launch-based test <https://github.com/ros-planning/navigation2/blob/main/nav2_system_tests/src/system/CMakeLists.txt>`_

.. code-block:: cmake

  ament_add_test(test_bt_navigator_with_groot_monitoring
    GENERATE_RESULT_FOR_RETURN_CODE_ZERO
    COMMAND "${CMAKE_CURRENT_SOURCE_DIR}/test_system_launch.py"
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
    TIMEOUT 180
    ENV
      TEST_DIR=${CMAKE_CURRENT_SOURCE_DIR}
      TEST_MAP=${PROJECT_SOURCE_DIR}/maps/map_circular.yaml
      TEST_WORLD=${PROJECT_SOURCE_DIR}/worlds/turtlebot3_ros2_demo.world
      GAZEBO_MODEL_PATH=${PROJECT_SOURCE_DIR}/models
      BT_NAVIGATOR_XML=navigate_w_replanning_and_recovery.xml
      ASTAR=False
      GROOT_MONITORING=True
  )

This `cmake` macro ``ament_add_test()`` can handle raw `pytests`. Arguments are line or white-space separated.
The first argument is the name of your new test, which can later than be used as a `regex` search keyword to only run your new test.
The ``GENERATE_RESULT_FOR_RETURN_CODE_ZERO`` is a flag for `pytest` and necessary for this process.
``COMMAND`` describes the `pytest` entry point for your test.
``WORKING_DIRECTORY`` and ``TIMEOUT`` are self explanatory.
Under the group ``ENV`` environment variables can be set. Those can then directly be used in your python script as input via ``os.getenv('KEYWORD')``.

Depending on your test, it might not be necessity to declare all keywords in each test. 
In combination of ``RewrittenYaml()`` from our ``nav2_common`` package, 
we can use this to rewrite default parameters from the main ``params.yaml`` with a few easy steps.
A small example for this can be seen in the last section ``Tips & Tricks for Writing Tests``. 

Now, we add a few parameters and make sure that the parameters for launching our nodes are all setup correctly.
The next step involves dealing with ``pytest`` and testers. This code is from the same file as the ``RewrittenYaml()`` refers to.

.. code-block:: python

    # configure all the parameters and nodes based on our input in the CMakeList.txt
    ld = generate_launch_description()

    # setup our tester with the `tester_node.py` and a few additional input parameters
    # here multiple test can be created to test your tests in more versatile environments
    # even start fuzzing the input values might add robustness to your code
    test1_action = ExecuteProcess(
        cmd=[os.path.join(os.getenv('TEST_DIR'), 'tester_node.py'),
             '-r', '-2.0', '-0.5', '0.0', '2.0'],
        name='tester_node',
        output='screen')

    lts = LaunchTestService()
    lts.add_test_action(ld, test1_action)
    ls = LaunchService(argv=argv)
    ls.include_launch_description(ld)
    return lts.run(ls)

The next and final step would be to implement ``tester_node.py``. The node `here <https://github.com/ros-planning/navigation2/blob/main/nav2_system_tests/src/system/tester_node.py>`_ is quite a good example.
It features argument groups to take various parameters as input that can be seen been used in the code section above.
To name its core features: the test engages with multiple `lifecycle_nodes`, waits for all `action_servers` to be available, sends a goal,
tests if the goal is reached. 

This is a great example to use when one must implement a new pytest with ROS2 integration.

.. note::
  When testing with launch files and testers also written with ``pytest``, it is possible to rerun tests 
  in between iterations of your test without rebuilding your work-space.
  Although, this requires to build your package with ``$ colcon build --symlink-install [colcon build --packages-select <name-of-pkg>]`` .
  
b) Unit Test - gtest
""""""""""""""""""""
This ``CMakeList.txt`` example is for adding tests surrounding behavior tree actions.
`Source for the c++ test with gtest <https://github.com/ros-planning/navigation2/blob/main/nav2_behavior_tree/test/plugins/action/CMakeLists.txt>`_ 

.. code-block:: cmake

  ament_add_gtest(test_action_spin_action test_spin_action.cpp)
  target_link_libraries(test_action_spin_action nav2_spin_action_bt_node)
  ament_target_dependencies(test_action_spin_action ${dependencies})

  ament_add_gtest(test_action_back_up_action test_back_up_action.cpp)
  target_link_libraries(test_action_back_up_action nav2_back_up_action_bt_node)
  ament_target_dependencies(test_action_back_up_action ${dependencies})

Starting with ``ament_add_gtest``, analog to the ``pytest`` `ament`-macro it is necessary to add your test. With ``ament_add_gtest`` 2 arguments are needed.
The first is the name of the test also specified inside a cpp-file stating your test. 
The second argument is the name of the cpp-file which includes the test with the given name.
``target_link_libraries`` takes in the name of the new gtest (as target) and links libraries to it. 
In the nav2 context this step is necessary if those tests run against plugins that where declared as libraries in a "parent" CMakeList.txt of the package.
Also, libraries that are not already included into the ROS2 build system might be added this way (like map_io or other 3rd party libraries).
Finally, with ``ament_target_dependencies`` the standard ROS dependencies are checked to be qualified before building the test.

Another interesting example in combination with ``pytest`` would be the `map_server gtests <https://github.com/ros-planning/navigation2/blob/main/nav2_map_server/test/component/CMakeLists.txt>`_:

.. code-block:: cmake

  ament_add_gtest_executable(test_map_saver_node
    test_map_saver_node.cpp
    ${PROJECT_SOURCE_DIR}/test/test_constants.cpp
  )

  ament_target_dependencies(test_map_saver_node rclcpp nav_msgs)
  target_link_libraries(test_map_saver_node
    ${library_name}
    stdc++fs
  )

  # And now the interesting part

  ament_add_test(test_map_saver_node
    GENERATE_RESULT_FOR_RETURN_CODE_ZERO
    COMMAND "${CMAKE_CURRENT_SOURCE_DIR}/test_map_saver_launch.py"
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
    ENV
      TEST_DIR=${TEST_DIR}
      TEST_LAUNCH_DIR=${TEST_LAUNCH_DIR}
      TEST_EXECUTABLE=$<TARGET_FILE:test_map_saver_node>
  )

This shows the usage of ``ament_add_gtest_executable`` in combination with a ``pytest``, which is quite interesting. 
It always pays off to look into the existing ways of testing in nav2 before writing your own tests.

Finally, now we have a look at the actual c++ code describing the test, `for example this gtest <https://github.com/ros-planning/navigation2/blob/main/nav2_behavior_tree/test/plugins/condition/test_goal_reached.cpp>`_.
Gtest is a really massive tool set, provided by Google. Explaining it from ground up would be overwhelming.
While pointing towards the standard `Readme of googletest <https://github.com/google/googletest/blob/master/googletest/README.md>`_ and also a really simple `cpp based gtest <https://github.com/google/googletest/blob/master/googletest/samples/sample1_unittest.cc>`_, 
in a nutshell using ``gtest`` depends multiple basic steps you must follow.
In the behavior_tree example there are some setup and teardown routines included in the cpp. Those are relevant for the test setting itself but are not subject of any data-inputs that will be tested.
There exists even ``SetUpTestCases`` routines to setup yet another more abstract layer for specific tests, `like in this bt test class <https://github.com/ros-planning/navigation2/blob/main/nav2_behavior_tree/test/test_behavior_tree_fixture.hpp>`_.
Generally, setup routines are not strictly needed by ``gtest``, but highly recommend in the context of complex architectures like ROS.

`There exists some nuances in the TEST macros from gtest <https://stackoverflow.com/questions/58600728/what-is-the-difference-between-test-test-f-and-test-p>`_. 
At the core functionality, the important part is always if certain conditions generate the expected behaviors. 
This can be checked for example by the ``EXPECT_EQ()`` macro like so:

.. code-block:: cpp

  //...
  TEST_F(GoalReachedConditionTestFixture, test_behavior)
  {
    EXPECT_EQ(tree_->tickRoot(), BT::NodeStatus::FAILURE);

    geometry_msgs::msg::Pose pose;
    pose.position.x = 0.0;
    pose.position.y = 0.0;
    transform_handler_->updateRobotPose(pose);
    std::this_thread::sleep_for(500ms);
    EXPECT_EQ(tree_->tickRoot(), BT::NodeStatus::FAILURE);

    //...
  }
  //...

To summarize some concrete implementation details. A unit test is declared in a dedicated cpp-file. 
This file must be included in a CMakeList.txt file, as described above.
One unit test mostly includes multiple checkpoints for multiple corner cases before ultimately failing or succeeding.
Just one failed test is enough for the whole unit test to fail.
Determining if a test passes a checkpoint is achieved by the ``EXPECT_EQ`` macro or similar ones.
For this gtest-macro, the checkpoint only passes if both arguments contain the same value.

This concludes the first jumping point inside the big field of unit tests in ROS2 with ``gtest`` and ``pytest``.
For a deeper insight it is recommend to check the corresponding documentation of the given tools and learn from existing examples.

3. Check Code Coverage locally or with CI
-----------------------------------------

.. image:: images/Testing/github_coverage_diff.png
    

Is there room to improve the test coverage near your code? Ain't you just the right expert about this code section?
Think about adding tests that exceed your own focus and help improve nav2/ros2 reach a higher overall code coverage and ultimately also quality.

The report above is an automated post by codecov.io-bot on github that posts results of CI automatically for every new PR.
Please consider helping increase the code coverage and use the opportunity to learn more about the internals of the navigation2 stack! 

Alternatively, if you want to track your code coverage before uploading your changes as a PR on the nav2 git, 
you can manually check your code coverage report with these steps:

.. code-block:: bash

  1. Build with GCOV enabled - only package granular or complete workspace
  $ cd ~/navigation2_ws/
  $ colcon build --symlink-install [--packages-select nav2_map_server] --cmake-args -DCMAKE_CXX_FLAGS='--coverage' -DCMAKE_C_FLAGS='--coverage'

  2. Run testcases manually - keep in mind that system wide tests can also improve your test-coverage!
  $ colcon test [--packages-select <pkg-name>]

  3. Run LCOV to get html-file
  $ lcov --capture --directory build/[nav2_map_server/CMakeFiles/map_server_core.dir/src/costmap_filter_info] --output-file coverage.info
  $ genhtml coverage.info --output-directory out
  $ firefox out/index.html


In order to get the specific directory of a specific package for LCOV containing the required ``*.gcda`` file:

.. code-block:: bash

  $ find "build/<pkg_name>" -name "*.gcda"

Alternatively just use the full build folder when building the full code stack with the given ``cmake-args``. 
(But this might as well include some non-nav2 specific tests)

After checking the results of the different packages and exploring room for optimization, you might be interested into writing additional tests. 
The section above might just help you with this!
.. 
.. 
.. 

Tips & Tricks for Writing Tests
===============================
This informal section shall provide best practices and things not very obvious to a new test programmer.
Also, consider checking out the tutorial about unit tests and integration tests with colcon provided by the autoware foundation, 
`here for unit tests <https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/how-to-write-tests-and-measure-coverage.html>`_ and `here for integration tests <https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/integration-testing.html>`_.


1. Rewriting Parameter Values from YAML Files in Launch scripts
---------------------------------------------------------------

In most occasions some small new features are added and made available through a few new parameters. As the standard nav2 user should not be overloaded with features it makes sense to disable most of the additional or drop-in features in the default ``params.yaml`` file.
But tests should still be comparable and only alter the test-scope specific parameters. 
Therefor, it makes no sense to copy most of the ``params.yaml`` file into multiple test.yaml files that would be prone to fail future changes.

ROS2 with its launch systems already includes many substitution mechanisms, that seem to handle a few dozen different scenarios, but are a little bit hard to use from a user standpoint with lacking example material.
Luckily, nav2 already includes a nice helper function for replacing values in YAML files.
Here is an example showing a small feature set of the capability of the launch system:

.. code-block:: python
  
  # ...
  import yaml 
  # ...
  # Replace the default parameter values for testing special features without having multiple params_files inside the nav2 stack
  context = LaunchContext()
  param_substitutions = {}

  if (os.getenv('ASTAR') == "True"):
      param_substitutions.update({'use_astar': "True"})
      print ("ASTAR set True")

  if (os.getenv('GROOT_MONITORING') == "True"):
      param_substitutions.update({'enable_groot_monitoring': "False"})
      print ("GROOT_MONITORING set True")

  # Fails -> multi dimensional keys ['planner_server']['ros__parameters']['GridBased']['use_astar'] cannot be combined in such manner
  # param_substitutions = {'planner_server.ros__parameters.GridBased.use_astar': "True"}

  # Fails -> value for 'bt_navigator' gets overwritten with 'ros__parameters' as value and not as next stage dict
  # param_substitutions = {'bt_navigator':{'ros__parameters':{'enable_groot_monitoring' : 'True'}}} 

  # Obviously not the needed behavior but shows that 'HELLOO...' only gets written when perform gets triggered
  # param_substitutions = {'bt_navigator':'HELLOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO'} 

  # Finally works with LaunchContext and perform sub-function
  # param_substitutions = {'enable_groot_monitoring' : 'True'} 

  # This would also work, but then the whole params_file gets recursively searched and replaces "False" with "False" -> time wasted
  # param_substitutions = { 
  #    'use_astar': os.getenv('ASTAR', default = "False"),
  #    'enable_groot_monitoring': os.getenv('GROOT_MONITORING', default = "False")
  #    }

  configured_params = RewrittenYaml(
      source_file=params_file,
      root_key='',
      param_rewrites=param_substitutions,
      convert_types=True)
  
  new_yaml = configured_params.perform(context)

  # Check if value has the desired value now before loading the yaml as launch_argument
  data = yaml.safe_load(open(new_yaml, 'r'))
  # print (data['planner_server']['ros__parameters']['GridBased']['use_astar'])


This can also be investigated in a real scenario in the nav2-CI test. Just have a look at the ``nav2_system_tests`` `test for the whole system <https://github.com/ros-planning/navigation2/tree/main/nav2_system_tests/src/system>`_.