Serial Communication Node (ROS 2 + pySerial) Using Procedural Approach
======================================================================

This section explains a simple ROS 2 node that communicates with a microcontroller
using serial communication (``pySerial``).
The node reads data from serial, processes it, and sends the result back.

The programming style used here is **procedural**, similar to Arduino's
``setup()`` and ``loop()`` concept.

Full Program
------------

.. code-block:: python

    # Versi Python-nish

    import rclpy
    import serial
    import time


    def main():
        rclpy.init()

        # === CREATE NODE (procedural style) ===
        node = rclpy.create_node('serial_node', namespace='demo')

        node.get_logger().info('Node berhasil dibuat!')

        # === SETUP (sekali, seperti setup() Arduino) ===
        ser = serial.Serial(
            port='/dev/ttyACM0',   # sesuaikan
            baudrate=9600,
            timeout=0.01
        )

        time.sleep(2)

        node.get_logger().info(f"Serial opened: port = {ser.port}, baudrate = {ser.baudrate}, timeout = {ser.timeout}")

        def process_data(value):
            return value/5.0

        # === CALLBACK (mirip isi loop Arduino) ===
        def timer_callback():
            if ser.in_waiting > 0:
                line = ser.readline().decode().strip()
                # .readline() = read until \n, hasilnya BYTES bukan string (b'123\n')
                # .decode()   = ubah BYTES menjadi string (b'123\n' -> '123\n')
                # .strip()    = menghapus whitespace di awal dan akhir ('123\n' -> '123')
                
                if line:
                    try:
                        received = float(line)
                        processed = process_data(received)
                        assert processed is not None
                        ser.write(f'{processed}\n'.encode())
                        # .encode() = mengubah string menjadi bytes ('123\n' -> b'123\n')
                        node.get_logger().info(f"RX: {received} -> TX: {processed}")
                    except ValueError:
                        node.get_logger().info('invalid data')

        # === TIMER (pengganti loop()) ===
        node.create_timer(0.1, timer_callback)

        # === SPIN (event loop) ===
        try:
            rclpy.spin(node)
        except KeyboardInterrupt:
            pass

        # === CLEANUP ===
        ser.close()
        node.destroy_node()
        rclpy.shutdown()


    # OPTIONAL (tidak wajib untuk ros2 run)
    if __name__ == '__main__':
        main()

Import Libraries
----------------

.. code-block:: python

   import rclpy
   import serial
   import time

- ``rclpy``  
  ROS 2 Python client library.
- ``serial``  
  Used for serial communication (USB / UART).
- ``time``  
  Used to provide delay during serial initialization.


Main Function
-------------

All program logic is placed inside the ``main()`` function.

.. code-block:: python

   def main():

This function will be executed when the node is run using ``ros2 run``.


Initialize ROS 2
----------------

Initialize the ROS 2 Python client library.

.. code-block:: python

   rclpy.init()

This must be called before creating any ROS 2 node.


Create Node
-----------

Create a ROS 2 node using procedural style.

.. code-block:: python

   node = rclpy.create_node('serial_node', namespace='demo')

- Node name  : ``serial_node``
- Namespace : ``demo``

Full node name:

::

   /demo/serial_node


Logging Node Status
-------------------

Print a message to indicate the node has been created.

.. code-block:: python

   node.get_logger().info('Node berhasil dibuat!')

ROS 2 logging provides timestamp, node name, and log level.


Serial Port Setup
-----------------

Initialize the serial communication.
This section is executed **once**, similar to ``setup()`` in Arduino.

.. code-block:: python

   ser = serial.Serial(
       port='/dev/ttyACM0',   # sesuaikan
       baudrate=9600,
       timeout=0.01
   )

- ``port``     : serial device path
- ``baudrate`` : communication speed
- ``timeout``  : non-blocking read timeout (seconds)


Serial Initialization Delay
---------------------------

Provide a delay to allow the microcontroller to reset after opening the port.

.. code-block:: python

   time.sleep(2)

This is commonly required when using Arduino-based boards.


Serial Status Logging
---------------------

Print serial configuration to the ROS log.

.. code-block:: python

   node.get_logger().info(
       f"Serial opened: port = {ser.port}, baudrate = {ser.baudrate}, timeout = {ser.timeout}"
   )

This helps debugging connection issues.


Data Processing Function
------------------------

Define a helper function to process received data.

.. code-block:: python

   def process_data(value):
       return value / 5.0

This function separates **data processing logic** from communication logic,
making the code easier to read and maintain.


Timer Callback Function
-----------------------

This function is executed periodically by a ROS 2 timer.
It is equivalent to the ``loop()`` function in Arduino.

.. code-block:: python

   def timer_callback():


Check Serial Buffer
^^^^^^^^^^^^^^^^^^^

Check whether data is available in the serial buffer.

.. code-block:: python

   if ser.in_waiting > 0:

``in_waiting`` indicates the number of bytes available to read.


Read and Decode Serial Data
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   line = ser.readline().decode().strip()

Explanation:

- ``readline()``  
  Reads data until newline character (``\n``)  
  Output type: ``bytes`` (e.g. ``b'123\n'``)

- ``decode()``  
  Converts bytes to string (``b'123\n'`` → ``'123\n'``)

- ``strip()``  
  Removes whitespace characters (``'123\n'`` → ``'123'``)


Convert and Process Data
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   received = float(line)
   processed = process_data(received)
   assert processed is not None

- Convert received string into ``float``
- Process the value
- ``assert`` ensures the processed value is valid


Send Data Back to Serial
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   ser.write(f'{processed}\n'.encode())

- ``encode()`` converts string to bytes
- Newline character (``\n``) ensures the receiver can parse the data properly


Logging RX and TX Data
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   node.get_logger().info(f"RX: {received} -> TX: {processed}")

This log shows the received value and the processed value sent back.


Handle Invalid Data
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   except ValueError:
       node.get_logger().info('invalid data')

This prevents the node from crashing if non-numeric data is received.


Create Timer
------------

Create a ROS 2 timer as a replacement for Arduino ``loop()``.

.. code-block:: python

   node.create_timer(0.1, timer_callback)

- Timer period: ``0.1`` seconds (10 Hz)
- Calls ``timer_callback()`` periodically


Spin the Node
-------------

Keep the node alive and process callbacks.

.. code-block:: python

   try:
       rclpy.spin(node)
   except KeyboardInterrupt:
       pass

The node runs until interrupted with ``Ctrl+C``.


Cleanup and Shutdown
--------------------

Release resources and shutdown ROS 2.

.. code-block:: python

   ser.close()
   node.destroy_node()
   rclpy.shutdown()

- Close serial port safely
- Destroy the node
- Shutdown ROS 2 client library


Program Entry Point
-------------------

Optional entry point for Python execution.

.. code-block:: python

   if __name__ == '__main__':
       main()

This is required for Python execution and used indirectly by ``ros2 run``.


Summary
-------

- This node acts as a **bridge** between ROS 2 and a serial device
- Uses **procedural programming style**
- Timer replaces Arduino ``loop()``
- Serial setup behaves like Arduino ``setup()``
- Suitable for:
  
  - Sensor data acquisition
  - Microcontroller communication
  - Simple control loops
