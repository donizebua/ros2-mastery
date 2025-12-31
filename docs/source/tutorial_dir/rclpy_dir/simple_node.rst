Create Simple Node and Make Simple Node Program
===============================================

Create Node: ``simple_node.py``
-------------------------------

.. code-block:: bash

    cd ~/ros2_ws/src/my_first_node
    touch simple_node.py

Write ``simple_node.py`` Program
--------------------------------

Copy and paste the code below:

.. code-block:: python

    import rclpy

    def main():
        # 1. INIT
        rclpy.init()

        # 2. CREATE NODE
        node = rclpy.create_node(
            node_name='simple_node',
            namespace='demo'
        )

        node.get_logger().info('Node berhasil dibuat')

        # parameter
        node.declare_parameter('message', 'hello world')
        node.declare_parameter('period', 0.1)

        message = node.get_parameter('message').value
        period = node.get_parameter('period').value

        node.get_logger().info(f'Parameter message: {message}')
        node.get_logger().info(f'Parameter period: {period}')

        def timer_callback():
            node.get_logger().info(f'[Timer] {message}')

        node.create_timer(
            period,
            timer_callback
        )

        # 3. SPIN
        try:
            rclpy.spin(node)
        except KeyboardInterrupt:
            pass

        # 4. CLEANUP
        node.destroy_node()
        rclpy.shutdown()

    #OPTIONAL (tidak wajib untuk ros2 run)

    if __name__ == '__main__':
        main()

Program Explanation
-------------------

Import Python ``rclpy`` library.

.. code-block:: python

    import rclpy

Function where all the program is located. When the node is executed, this function will run.

.. code-block:: python

    def main():

Initialize ``rclpy`` library.

.. code-block:: python

    rclpy.init()

Create a node with namespace ``demo`` and name ``simple_node``, then store it in the node variable.

.. code-block:: python

    node = rclpy.create_node(
        node_name='simple_node',
        namespace='demo'
    )

The full node name becomes:

.. code-block:: bash

    /demo/simple_node

| Declare parameter ``message`` with default string value ``"hello world"``
| and ``period`` with default float value ``0.1``.
| These declarations are executed once as default parameters.
| Parameters can be overridden using ROS 2 CLI arguments.

Example:

.. code-block:: bash

    ros2 run my_first_node simple_node --ros-args -p message:="belajar rclpy" -p period:=0.5

ROS CLI arguments will be captured by ``get_parameter()`` and stored in
``message`` and ``period`` variables.
If no CLI argument is provided, the default values are used.

.. code-block:: python

    node.declare_parameter('message', 'hello world')
    node.declare_parameter('period', 0.1)

    message = node.get_parameter('message').value
    period = node.get_parameter('period').value

Similar to ``Serial.print()`` in Arduino, ``get_logger().info()`` prints messages
using ROS 2 logging format.

.. code-block:: python

    node.get_logger().info('Node berhasil dibuat')
    node.get_logger().info(f'Parameter message: {message}')
    node.get_logger().info(f'Parameter period: {period}')

Used to execute code repeatedly at a fixed interval.
This is similar to ``void loop()`` in Arduino.

.. code-block:: python

    def timer_callback():
        node.get_logger().info(f'[Timer] {message}')

    node.create_timer(
    period,
    timer_callback
    )

Try-except logic to keep the node running.
If a ``KeyboardInterrupt`` occurs ``(Ctrl+C)``, the node will stop.

.. code-block:: python

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass

Destroy the node and shutdown ``rclpy`` for cleanup.

.. code-block:: python

    node.destroy_node()
    rclpy.shutdown()

Optional entry point. This is required for Python execution, but ROS 2 uses it indirectly via
``ros2 run``.

.. code-block:: python

    if name == 'main':
    main()

Setup Node in setup.py
----------------------

| Add the following entry in ``console_scripts``.
| Format:
| ``<executable> = <package_name>.<file_name>:main``

.. code-block:: python

    'console_scripts': [
    'simple_node = my_first_node.simple_node:main',
    ],

Rebuild Program
---------------

.. code-block:: bash

    cd ~/ros2_ws
    colcon build
    source install/setup.bash

Workspace Structure
-------------------

::

    ros2_ws/
    ├── src/
    │ └── my_first_node
    | ├── init.py
    | └── simple_node.py
    ├── build/
    ├── install/
    └── log/

Run Program
-----------

Run with default values.

.. code-block:: bash

    ros2 run my_first_node simple_node

Output:

.. code-block:: bash

    doni@doniubuntu:~/coding/ros2_ws_2$ ros2 run my_first_node simple_node
    [INFO] [1767065573.576960718] [demo.simple_node]: Node berhasil dibuat
    [INFO] [1767065573.578662439] [demo.simple_node]: Parameter message: hello world
    [INFO] [1767065573.579480443] [demo.simple_node]: Parameter period: 0.1
    [INFO] [1767065573.681305483] [demo.simple_node]: [Timer] hello world
    [INFO] [1767065573.781224259] [demo.simple_node]: [Timer] hello world
    [INFO] [1767065573.881213978] [demo.simple_node]: [Timer] hello world

Run with custom parameters.

.. code-block:: bash

    Output: 

.. code-block:: bash

    doni@doniubuntu:~/coding/ros2_ws_2$ ros2 run my_first_node simple_node --ros-args -p message:='Belajar rclpy' -p period:=1.0
    [INFO] [1767065780.429482648] [demo.simple_node]: Node berhasil dibuat
    [INFO] [1767065780.431184258] [demo.simple_node]: Parameter message: Belajar rclpy
    [INFO] [1767065780.431985751] [demo.simple_node]: Parameter period: 1.0
    [INFO] [1767065781.433689235] [demo.simple_node]: [Timer] Belajar rclpy
    [INFO] [1767065782.434202266] [demo.simple_node]: [Timer] Belajar rclpy
    [INFO] [1767065783.434240297] [demo.simple_node]: [Timer] Belajar rclpy


Check node in another terminal:

.. code-block:: bash

    ros2 node list

Check parameters:

.. code-block:: bash

    ros2 param list /demo/simple_node
    ros2 param get /demo/simple_node message
