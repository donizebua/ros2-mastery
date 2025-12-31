CheatSheet
===========

Sub Judul
---------

Sub-sub Judul
^^^^^^^^^^^^^

Teks & Format Dasar
-------------------

Tebal
^^^^^
**tebal**

Miring
^^^^^^
*miring*

Kode Inline
^^^^^^^^^^^
``kode inline``

List
----

Bulleted list
^^^^^^^^^^^^^
- Node
- Topic
- Service

Numbered list
^^^^^^^^^^^^^
1. Install ROS2
2. Build workspace
3. Run node

Nested list
^^^^^^^^^^^
- Sensor
  - Encoder
  - IMU

Code block
----------

Python
^^^^^^
.. code-block:: python

   def main():
       print("Hello ROS2")

C++ ROS2
^^^^^^^^^^
.. code-block:: cpp

   rclcpp::init(argc, argv);

Arduino
^^^^^^^
.. code-block:: cpp

   void setup() {
     Serial.begin(115200);
   }

Terminal / CLI
^^^^^^^^^^^^^^
.. code-block:: bash

   colcon build
   source install/setup.bash

Link dan Referensi
------------------

Link biasa
^^^^^^^^^^
`ROS2 Official <https://docs.ros.org>`_

Referensi internal (antar halaman)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. :doc:`installation`

Gambar (diagram robot sering pakai ini)
---------------------------------------

.. image:: images/ipar_sewot.png
   :width: 400px
   :align: center

.. struktur:

.. source/
.. └── images/

Warning / Note / Tip (penting banget)
-------------------------------------

note
^^^^
.. note::

   Ini catatan penting.

warning
^^^^^^^
.. warning::

   Jangan jalankan node sebagai root.

tip
^^^
.. tip::

   Gunakan virtual environment.

Dokumentasi Python (AUTODOC)
----------------------------

.. .. automodule:: my_package.my_node
..    :members:
..    :undoc-members:

.. ini level lanjut, tapi powerful

ROS2-SPECIFIC (best practice)
-----------------------------

Nama package/node
^^^^^^^^^^^^^^^^^
``my_robot_node``

Topic/Service
^^^^^^^^^^^^^
- Topic: ``/cmd_vel``
- Message: ``geometry_msgs/Twist``

Arduino / Embedded
------------------

Pinout table
^^^^^^^^^^^^
+---------+------------+
| Pin     | Function   |
+=========+============+
| GPIO 17 | Servo PWM  |
+---------+------------+

Flow Program
^^^^^^^^^^^^

1. Setup Serial
2. Baca sensor
3. Hitung PID
4. Output PWM

Tabel (sering dipakai di hardware)
----------------------------------

+-----------+---------+
| Parameter | Value   |
+===========+=========+
| Baudrate  | 115200  |
+-----------+---------+
| PWM Freq  | 50 Hz   |
+-----------+---------+

Struktur dokumentasi IDEAL (untuk kamu)
---------------------------------------

 |  docs/source/
 |  ├── index.rst
 |  ├── python/
 |  │   ├── overview.rst
 |  │   └── examples.rst
 |  ├── ros2/
 |  │   ├── nodes.rst
 |  │   └── topics.rst
 |  ├── arduino/
 |  │   ├── setup.rst
 |  │   └── serial.rst
 |  └── images/