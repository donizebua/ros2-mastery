Serial Communication Node (ROS 2 + pySerial) Using Class-Based Approach
=======================================================================

This section explains a ROS 2 serial communication node implemented using
a **class-based (ROS-style)** approach.
The node communicates with a serial device, processes incoming data,
and sends the result back through the same serial port.

Compared to the procedural version, this implementation follows
recommended ROS 2 design patterns.

Full Program
------------

.. code-block:: python

    #Versi ROS-nish

    import rclpy
    from rclpy.node import Node
    import serial

    class SerialNode(Node):
        def __init__(self):
            super().__init__('serial_node')
            
            self.declare_parameter('port', '/dev/ttyACM0')
            self.declare_parameter('baudrate', 9600)
            self.declare_parameter('period', 0.1)

            Port = self.get_parameter('port').value
            Baudrate = self.get_parameter('baudrate').value
            Period = self.get_parameter('period').value

            self.ser = serial.Serial(
                port=Port,
                baudrate=Baudrate,
                timeout=0.01
            )

            self.get_logger().info(f"Serial Properties = Port: {self.ser.port} | Baudrate: {self.ser.baudrate} | Timeout: {self.ser.timeout}")

            self.timer = self.create_timer(Period, self.timer_callback)

        def timer_callback(self):
            if self.ser.in_waiting > 0:
                line = self.ser.readline().decode().strip()
                if line:
                    try:
                        received = float(line)
                        processed = received/10
                        self.ser.write(f"{processed}\n".encode())
                        self.get_logger().info(f"RX: {received}   TX: {processed}")
                    except ValueError:
                        self.get_logger().info("invalid data!")
    def main():
        rclpy.init()
        node = SerialNode()
        try:
            rclpy.spin(node)
        except KeyboardInterrupt:
            pass
        node.ser.close()
        node.destroy_node()
        rclpy.shutdown()

    # OPTIONAL (tidak wajib untuk ros2 run)
    if __name__ == '__main__':
        main()


Import Libraries
----------------

.. code-block:: python

   import rclpy
   from rclpy.node import Node
   import serial

- ``rclpy``  
  ROS 2 Python client library.
- ``Node``  
  Base class for creating ROS 2 nodes.
- ``serial``  
  Library for serial (UART / USB) communication.



SerialNode Class
----------------

Define a ROS 2 node using a class-based structure.

.. code-block:: python

   class SerialNode(Node):

This approach is commonly used in ROS 2 because it:

- Encapsulates node logic
- Makes the code scalable
- Allows easy extension (publishers, subscribers, services)



Node Initialization
-------------------

The constructor initializes the node and all required resources.

.. code-block:: python

   def __init__(self):
       super().__init__('serial_node')

- Node name: ``serial_node``
- Namespace: default (root)

Full node name:

::

   /serial_node



Declare Parameers
------------------

Declare configurable parameters for serial communication.

.. code-block:: python

   self.declare_parameter('port', '/dev/ttyACM0')
   self.declare_parameter('baudrate', 9600)
   self.declare_parameter('period', 0.1)

Parameters allow runtime configuration using ROS 2 CLI.



Retrieve Parameter Values
-------------------------

Read parameter values and store them in local variables.

.. code-block:: python

   Port = self.get_parameter('port').value
   Baudrate = self.get_parameter('baudrate').value
   Period = self.get_parameter('period').value

These values will be used to configure the serial interface and timer.



Serial Port Initialization
--------------------------

Initialize the serial connection using parameter values.

.. code-block:: python

   self.ser = serial.Serial(
       port=Port,
       baudrate=Baudrate,
       timeout=0.01
   )

- ``timeout`` is set to a small value to avoid blocking behavior
- Serial initialization happens once, similar to ``setup()`` in Arduino



Serial Status Logging
---------------------

Log serial configuration information for debugging purposes.

.. code-block:: python

   self.get_logger().info(
       f"Serial Properties = Port: {self.ser.port} | Baudrate: {self.ser.baudrate} | Timeout: {self.ser.timeout}"
   )

This confirms that the serial device is opened correctly.



Create Timer
------------

Create a ROS 2 timer to periodically execute logic.

.. code-block:: python

   self.timer = self.create_timer(Period, self.timer_callback)

- Timer period is configurable via ROS parameters
- Replaces the ``loop()`` function in Arduino



Timer Callback
--------------

This callback is executed periodically by the ROS 2 timer.

.. code-block:: python

   def timer_callback(self):

This function handles:

- Reading data from serial
- Processing received values
- Sending processed data back



Check Serial Buffer
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   if self.ser.in_waiting > 0:

Ensures data is available before reading from the serial buffer.



Read and Parse Data
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   line = self.ser.readline().decode().strip()

- ``readline()`` reads until newline character
- ``decode()`` converts bytes to string
- ``strip()`` removes whitespace characters



Process and Send Data
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   received = float(line)
   processed = received / 10
   self.ser.write(f"{processed}\n".encode())

- Received data is converted to ``float``
- Processed using simple scaling
- Sent back as encoded bytes



Logging RX and TX
^^^^^^^^^^^^^^^^^

.. code-block:: python

   self.get_logger().info(f"RX: {received}   TX: {processed}")

This log shows the input-output relationship clearly.



Handle Invalid Data
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   except ValueError:
       self.get_logger().info("invalid data!")

Prevents node crash when non-numeric data is received.



Main Function
-------------

The entry point of the ROS 2 node.

.. code-block:: python

   def main():
       rclpy.init()
       node = SerialNode()

Initialize ROS 2 and create an instance of ``SerialNode``.



Spin the Node
-------------

Keep the node alive and process timer callbacks.

.. code-block:: python

   try:
       rclpy.spin(node)
   except KeyboardInterrupt:
       pass

The node runs until interrupted using ``Ctrl+C``.



Cleanup and Shutdown
--------------------

Release resources and shutdown ROS 2 cleanly.

.. code-block:: python

   node.ser.close()
   node.destroy_node()
   rclpy.shutdown()

- Close serial port
- Destroy the node
- Shutdown ROS 2



Program Entry Point
-------------------

Optional Python entry point.

.. code-block:: python

   if __name__ == '__main__':
       main()

This is required for Python execution and used by ``ros2 run``.



Summary
-------

- Class-based **ROS-style** implementation
- Uses ROS 2 parameters for serial configuration
- Timer replaces Arduino ``loop()``
- Clean separation between initialization and runtime logic
- Suitable for scalable ROS 2 applications
