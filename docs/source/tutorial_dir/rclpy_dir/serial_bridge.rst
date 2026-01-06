MCU Serial Bridge Node
======================

This section explains the ``mcu_serial_bridge`` node, which acts as a **bidirectional
bridge** between a microcontroller (MCU) and ROS 2 using serial communication.

The node:

- Receives commands from ROS 2 and sends them to the MCU
- Receives sensor/data frames from the MCU and publishes them to ROS 2
- Uses a dedicated thread for serial RX to avoid blocking ROS callbacks

Full Program
------------

.. code-block:: python

    import rclpy
    from rclpy.node import Node
    from std_msgs.msg import Float32
    import serial
    import threading


    class MCUSerialBridge(Node):
        def __init__(self):
            super().__init__('mcu_serial_bridge', namespace='mcu')

            # ===== PARAMETERS =====
            self.declare_parameter('port', '/dev/ttyACM0')
            self.declare_parameter('baudrate', 9600)
            self.declare_parameter('timeout', 0.05)

            port = self.get_parameter('port').value
            baudrate = self.get_parameter('baudrate').value
            timeout = self.get_parameter('timeout').value

            # ===== SERIAL =====
            self.ser = serial.Serial(
                port=port,
                baudrate=baudrate,
                timeout=timeout
            )

            self.get_logger().info(
                f"Serial opened | port={port}, baudrate={baudrate}, timeout={timeout}"
            )

            # ===== ROS INTERFACE =====
            self.publisher_ = self.create_publisher(
                Float32,
                'data',      # relative → /mcu/data
                10
            )

            self.subscription_ = self.create_subscription(
                Float32,
                'cmd',       # relative → /mcu/cmd
                self.cmd_callback,
                10
            )

            # ===== SERIAL RX THREAD =====
            self.running = True
            self.rx_thread = threading.Thread(
                target=self.serial_rx_loop,
                daemon=True
            )
            self.rx_thread.start()

        # ---------- ROS → MCU ----------
        def cmd_callback(self, msg: Float32):
            value = msg.data
            frame = f"CMD:{value}\n"
            self.ser.write(frame.encode())
            self.get_logger().info(f"ROS → MCU | {frame.strip()}")

        # ---------- MCU → ROS ----------
        def serial_rx_loop(self):
            while self.running and rclpy.ok():
                try:
                    line = self.ser.readline().decode().strip()
                    if not line:
                        continue

                    if line.startswith("DATA:"):
                        value = float(line[5:])
                        msg = Float32()
                        msg.data = value
                        self.publisher_.publish(msg)
                        self.get_logger().info(f"MCU → ROS | DATA:{value}")

                    else:
                        # optional: ignore or log unknown frames
                        self.get_logger().warn(f"Unknown frame: {line}")

                except ValueError:
                    self.get_logger().warn("Malformed DATA frame")
                except Exception as e:
                    self.get_logger().error(str(e))

        def destroy_node(self):
            self.running = False
            if self.ser.is_open:
                self.ser.close()
            super().destroy_node()


    def main():
        rclpy.init()
        node = MCUSerialBridge()
        try:
            rclpy.spin(node)
        except KeyboardInterrupt:
            pass
        node.destroy_node()
        rclpy.shutdown()


    if __name__ == '__main__':
        main()



Import Libraries
----------------

.. code-block:: python

   import rclpy
   from rclpy.node import Node
   from std_msgs.msg import Float32
   import serial
   import threading

- ``rclpy``  
  ROS 2 Python client library.
- ``Node``  
  Base class for ROS 2 nodes.
- ``Float32``  
  Standard ROS message type for floating-point data.
- ``serial``  
  Used for serial communication with the MCU.
- ``threading``  
  Used to handle serial reception asynchronously.


MCUSerialBridge Class
---------------------

Define the ROS 2 node using a class-based (ROS-style) approach.

.. code-block:: python

   class MCUSerialBridge(Node):

This node encapsulates all logic related to:

- Serial communication
- ROS publisher and subscriber
- Thread management


Node Initialization
-------------------

Initialize the node with a name and namespace.

.. code-block:: python

   super().__init__('mcu_serial_bridge', namespace='mcu')

- Node name  : ``mcu_serial_bridge``
- Namespace : ``mcu``

Full node name:

::

   /mcu/mcu_serial_bridge


Parameters
----------

Declare parameters for serial configuration.

.. code-block:: python

   self.declare_parameter('port', '/dev/ttyACM0')
   self.declare_parameter('baudrate', 9600)
   self.declare_parameter('timeout', 0.05)

These parameters allow runtime configuration using ROS 2 CLI.


Retrieve Parameter Values
-------------------------

Read parameter values from the parameter server.

.. code-block:: python

   port = self.get_parameter('port').value
   baudrate = self.get_parameter('baudrate').value
   timeout = self.get_parameter('timeout').value

These values are used to initialize the serial interface.


Serial Initialization
---------------------

Open the serial connection to the MCU.

.. code-block:: python

   self.ser = serial.Serial(
       port=port,
       baudrate=baudrate,
       timeout=timeout
   )

- Serial initialization happens once
- ``timeout`` ensures non-blocking reads


Serial Status Logging
---------------------

Log serial configuration for debugging purposes.

.. code-block:: python

   self.get_logger().info(
       f"Serial opened | port={port}, baudrate={baudrate}, timeout={timeout}"
   )

This confirms that the serial device has been opened successfully.


ROS Interfaces
--------------

This node exposes a ROS 2 **publisher** and **subscriber**.


Publisher (MCU → ROS)
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   self.publisher_ = self.create_publisher(
       Float32,
       'data',
       10
   )

- Topic name (relative): ``data``
- Fully qualified topic:

::

   /mcu/data

This topic publishes data received from the MCU.


Subscriber (ROS → MCU)
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   self.subscription_ = self.create_subscription(
       Float32,
       'cmd',
       self.cmd_callback,
       10
   )

- Topic name (relative): ``cmd``
- Fully qualified topic:

::

   /mcu/cmd

This topic receives command values from ROS 2 and forwards them to the MCU.


Serial RX Thread
----------------

A dedicated thread is used to handle serial reception.

.. code-block:: python

   self.running = True
   self.rx_thread = threading.Thread(
       target=self.serial_rx_loop,
       daemon=True
   )
   self.rx_thread.start()

Why a thread is used:

- ``ser.readline()`` can block
- ROS callbacks must remain responsive
- Prevents blocking the ROS executor


ROS → MCU (Command Callback)
----------------------------

Handle incoming ROS commands and forward them to the MCU.

.. code-block:: python

   def cmd_callback(self, msg: Float32):


Frame Construction
^^^^^^^^^^^^^^^^^^

.. code-block:: python

   value = msg.data
   frame = f"CMD:{value}\n"

The command frame format:

::

   CMD:<value>\n

This format allows the MCU to distinguish command frames from data frames.


Send Frame to MCU
^^^^^^^^^^^^^^^^^

.. code-block:: python

   self.ser.write(frame.encode())
   self.get_logger().info(f"ROS → MCU | {frame.strip()}")

- ``encode()`` converts string to bytes
- Log indicates data flow direction


MCU → ROS (Serial RX Loop)
--------------------------

Continuously read data from the serial port in a separate thread.

.. code-block:: python

   def serial_rx_loop(self):


RX Loop Condition
^^^^^^^^^^^^^^^^^

.. code-block:: python

   while self.running and rclpy.ok():

The loop stops when:

- Node is shutting down
- ROS 2 context is no longer valid


Read and Parse Serial Data
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   line = self.ser.readline().decode().strip()

- Reads one line from serial
- Converts bytes to string
- Removes whitespace


DATA Frame Handling
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   if line.startswith("DATA:"):
       value = float(line[5:])

Expected MCU frame format:

::

   DATA:<value>


Publish to ROS Topic
^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   msg = Float32()
   msg.data = value
   self.publisher_.publish(msg)

The received value is published to:

::

   /mcu/data


Unknown Frame Handling
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   else:
       self.get_logger().warn(f"Unknown frame: {line}")

This allows graceful handling of unexpected serial messages.


Error Handling
--------------

Handle malformed frames and runtime errors.

.. code-block:: python

   except ValueError:
       self.get_logger().warn("Malformed DATA frame")
   except Exception as e:
       self.get_logger().error(str(e))

This prevents the RX thread from crashing.


Node Destruction
----------------

Override ``destroy_node()`` to ensure clean shutdown.

.. code-block:: python

   def destroy_node(self):
       self.running = False
       if self.ser.is_open:
           self.ser.close()
       super().destroy_node()

- Stops the RX thread
- Closes the serial port
- Calls base class cleanup


Main Function
-------------

Entry point for the ROS 2 node.

.. code-block:: python

   def main():
       rclpy.init()
       node = MCUSerialBridge()


Spin and Shutdown
-----------------

Run the node until interrupted, then clean up.

.. code-block:: python

   try:
       rclpy.spin(node)
   except KeyboardInterrupt:
       pass

   node.destroy_node()
   rclpy.shutdown()


Program Entry Point
-------------------

Optional Python entry point.

.. code-block:: python

   if __name__ == '__main__':
       main()

This is required for Python execution and used by ``ros2 run``.

Setup Node
----------

Place this in setup.py > entry points > console scripts

.. code-block:: python

    'mcu_serial_bridge = ros2_communication.mcu_serial_bridge:main',


Run Node
--------

.. code-block:: bash

    ros2 run ros2 communication mcu_serial_bridge --ros-args -p port:='/dev/ttyACM0' -p baudrate:=9600 -p timeout:=0.05

How to see MCU port:

1. Connect MCU into computer
2. Open bash, type ``>$ ls /dev/ttyACM*`` (for Arduino) and ``>$ ls /dev/ttyUSB*`` (for ESP32)


Summary
-------

- Acts as a **bidirectional bridge** between ROS 2 and an MCU
- Uses ROS topics for command and data exchange
- Serial RX is handled in a dedicated thread
- Clean separation between ROS logic and serial protocol
- Suitable for:
  
  - Sensor data acquisition
  - Actuator command forwarding
  - Embedded–ROS integration
