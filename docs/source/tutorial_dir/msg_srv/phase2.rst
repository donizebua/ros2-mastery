PHASE 2 — MCU Status Publisher dengan Nested Message
====================================================

Dokumen ini menjelaskan **implementasi Phase 2 node publisher ROS 2**
yang bertugas mengirim **status MCU secara periodik** menggunakan
**custom message bertingkat (nested message)**.

Phase ini berfokus pada **desain struktur data dan kebersihan node**,
bukan pada komunikasi hardware.

Target Akhir Phase 2
--------------------

Setelah Phase 2 selesai, sistem memiliki:

- Node publisher Python yang ROS-idiomatic
- Publisher berbasis timer (non-blocking)
- Custom message bertingkat (nested message) digunakan secara langsung
- Timestamp berasal dari ROS Clock
- Struktur data siap diperluas ke komunikasi serial (Phase 3)
- Output dapat dipantau dengan ``ros2 topic echo``


Konteks Struktur Package
------------------------

Dokumentasi ini mengasumsikan dua package terpisah: interface dan node.

Interface Package
^^^^^^^^^^^^^^^^^

Package interface hanya berisi definisi message.

.. code-block:: text

    mcu_interfaces/msg/
    ├── McuStatus.msg
    ├── PowerStatus.msg
    ├── ThermalStatus.msg
    └── SystemStatus.msg

``McuStatus.msg``

.. code-block:: text

    # MCU system status message
    # Phase 1 – flat, professional, future-safe

    builtin_interfaces/Time stamp
    uint32 seq

    PowerStatus power
    ThermalStatus thermal
    SystemStatus system

``PowerStatus.msg``

.. code-block:: text

    float32 battery_voltage
    float32 battery_current
    bool power_ok

``ThermalStatus.msg``

.. code-block:: text

    float32 board_temperature
    bool overheat

``SystemStatus.msg``

.. code-block:: text

    bool connected
    bool valid
    uint16 error_code

Node Package
^^^^^^^^^^^^

Package node berisi implementasi publisher.

.. code-block:: text

    mcu_status_node/
    ├── mcu_status_node/
    │   ├── __init__.py
    │   └── mcu_status_publisher.py
    ├── setup.py
    └── package.xml

Interface package dan node package **tidak boleh digabung**.

Update CMakeLists.txt
---------------------

Pastikan semua file ``.msg`` didaftarkan

.. code-block:: text

    rosidl_generate_interfaces(${PROJECT_NAME}
        "msg/McuStatus.msg"
        "msg/PowerStatus.msg"
        "msg/ThermalStatus.msg"
        "msg/SystemStatus.msg"
        DEPENDENCIES builtin_interfaces
    )


Implementasi Final Node Publisher
---------------------------------

Berikut adalah **kode final Phase 2** untuk file
``mcu_status_publisher.py``.

.. code-block:: python

    import rclpy
    from rclpy.node import Node

    from mcu_interfaces.msg import McuStatus


    class McuStatusPublisher(Node):

        def __init__(self):
            super().__init__('mcu_status_publisher')

            self.publisher_ = self.create_publisher(
                McuStatus,
                'mcu/status',
                10
            )

            self.timer_period = 0.5  # seconds
            self.timer = self.create_timer(
                self.timer_period,
                self.publish_status
            )

            self.seq = 0

            self.get_logger().info('MCU Status Publisher (Phase 2) started')

        def publish_status(self):
            msg = McuStatus()

            # -------------------------
            # Header / meta
            # -------------------------
            msg.stamp = self.get_clock().now().to_msg()
            msg.seq = self.seq
            self.seq += 1

            # -------------------------
            # Power domain
            # -------------------------
            msg.power.battery_voltage = 12.3
            msg.power.battery_current = 1.2
            msg.power.power_ok = True

            # -------------------------
            # Thermal domain
            # -------------------------
            msg.thermal.board_temperature = 35.8
            msg.thermal.overheat = False

            # -------------------------
            # System domain
            # -------------------------
            msg.system.connected = True
            msg.system.valid = True
            msg.system.error_code = 0

            self.publisher_.publish(msg)

            self.get_logger().debug(
                f'Published MCU status seq={msg.seq}'
            )


    def main(args=None):
        rclpy.init(args=args)

        node = McuStatusPublisher()

        try:
            rclpy.spin(node)
        except KeyboardInterrupt:
            pass
        finally:
            node.destroy_node()
            rclpy.shutdown()


    if __name__ == '__main__':
        main()

Validasi Teknis Desain Phase 2
------------------------------

Nested Message Digunakan Langsung
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Akses field dilakukan langsung melalui struktur message.

.. code-block:: python

    msg.power.battery_voltage
    msg.thermal.board_temperature
    msg.system.connected

Tidak ada penggunaan dictionary, string, atau workaround lain.

Pemisahan Domain Data
^^^^^^^^^^^^^^^^^^^^^

Data dibagi ke dalam domain yang jelas:

- Power untuk kelistrikan
- Thermal untuk suhu
- System untuk status dan kesehatan node

Pemisahan ini mempermudah parsing, logging, dan pemetaan ke frame serial.

Timer-Based Publisher
^^^^^^^^^^^^^^^^^^^^^

Publisher menggunakan timer ROS.

.. code-block:: python

    self.create_timer(0.5, self.publish_status)

Pendekatan ini aman untuk executor dan tidak memblokir node lain.

Timestamp dari ROS Clock
^^^^^^^^^^^^^^^^^^^^^^^^

Timestamp diambil dari clock ROS, bukan dari sistem OS.

.. code-block:: python

    msg.stamp = self.get_clock().now().to_msg()

Hal ini penting untuk sinkronisasi antar node.

Sequence Counter Eksplisit
^^^^^^^^^^^^^^^^^^^^^^^^^^

Counter sequence digunakan untuk keperluan debugging dan tracing data.

.. code-block:: python

    self.seq += 1

Cara Menjalankan dan Pengujian
------------------------------

Jalankan node dengan perintah berikut.

.. code-block:: bash

    ros2 run mcu_status_node mcu_status_publisher

Pada terminal lain, pantau data.

.. code-block:: bash

    ros2 topic echo /mcu/status

Output akan tampil dalam format YAML bertingkat.

Batasan Phase 2
---------------

Phase ini **secara sengaja tidak mencakup**:

- Komunikasi serial
- Service atau command interface
- Versioning logic
- Error handling berbasis hardware

Semua hal tersebut akan ditangani pada Phase berikutnya.

Phase berikutnya akan memetakan nested message ini ke
komunikasi serial yang deterministik dan terstruktur.
