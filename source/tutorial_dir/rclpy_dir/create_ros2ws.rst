Creating a ROS 2 Workspace
==========================

This guide explains how to create and verify a basic ROS 2 workspace using colcon. A workspace is required to build, run, and manage your own ROS 2 packages.

Create the Workspace Directory
------------------------------

First, create a workspace directory and a src folder inside it.

.. code-block:: bash

    mkdir -p ~/ros2_ws/src
    cd ~/ros2_ws

.. note::

    Workspace naming convention

        - ros2_ws → workspace root directory

        - src/ → contains all ROS 2 packages

        - One workspace can contain multiple packages

Build the Empty Workspace
-------------------------

Even if the workspace is still empty, it is good practice to build it once.
This initializes the workspace structure.

.. code-block:: bash

    colcon build


After a successful build, the following directories will be created automatically:

- build/ → temporary build files

- install/ → compiled packages and environment setup

- log/ → build logs and diagnostics

Source the Workspace
--------------------

To use packages inside the workspace, you must source the workspace environment.

.. code-block:: bash

    source install/setup.bash

.. important::

    - This command must be executed every time you open a new terminal

    - Sourcing affects only the current terminal session

Create a ROS 2 Package
----------------------

Now create your first package inside the src directory.

.. code-block:: bash

    cd src
    ros2 pkg create my_first_node --build-type ament_python --dependencies rclpy

This command will:

- Create a new Python-based ROS 2 package

- Add rclpy as a dependency

- Generate basic package files automatically

Build the Workspace Again
-------------------------

After adding a package, rebuild the workspace so ROS 2 can recognize it.

.. code-block:: bash

    cd ~/ros2_ws
    colcon build

Then source the workspace again:

.. code-block:: bash

    source install/setup.bash

Verify the Workspace
--------------------

To confirm that ROS 2 detects your package, run:

.. code-block:: bash

    ros2 pkg list

Your package (my_first_node) should appear in the list.

Typical Workspace Structure
---------------------------

A minimal ROS 2 workspace structure looks like this:
::

    ros2_ws/
    ├── src/
    │ └── my_first_node/
    │ ├── my_first_node/
    │ │ └── init.py
    │ ├── package.xml
    │ ├── setup.py
    │ └── setup.cfg
    ├── build/
    ├── install/
    └── log/

Common Mistakes
---------------

+-----------------------------------------------------+------------------------------------------------------+
| ❌ Incorrect                                        | ✅ Correct                                           |
+=====================================================+======================================================+
| Running colcon build inside src                     | Always run it from the workspace root                |
+-----------------------------------------------------+------------------------------------------------------+
| Forgetting to source install/setup.bash             | Source it in every new terminal                      |
+-----------------------------------------------------+------------------------------------------------------+
| Expecting the build to affect all terminals         | Build is global, environment is per-terminal         |
+-----------------------------------------------------+------------------------------------------------------+