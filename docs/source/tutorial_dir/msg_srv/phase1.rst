PHASE 1 — Custom Message ROS 2 (Hands-On dan Penjelasan Teknis)
===============================================================

Dokumen ini menjelaskan **Phase 1 pembuatan Custom Message di ROS 2**
secara **praktis dan teknis**.

Target Akhir Phase 1
---------------------

Setelah Phase 1 selesai, sistem memiliki:

- Satu *interface package* (custom ``.msg``) menggunakan ``ament_cmake``
- Satu *node package* (Python publisher) menggunakan ``ament_python``
- Custom message dengan desain stabil dan eksplisit
- Node dapat dijalankan dengan ``ros2 run``
- Data dapat dipantau menggunakan ``ros2 topic echo``
- Tidak ada warning fatal atau workaround build

Mental Model Utama Phase 1
---------------------------

Phase 1 **bukan sekadar membuat node bisa publish**, melainkan:

    *mendefinisikan tipe data ROS yang dapat dipercaya oleh sistem lain*

Custom message berfungsi sebagai **kontrak data**, bukan sekadar wadah nilai.

Arsitektur Package yang Digunakan
----------------------------------

Struktur arsitektur yang digunakan:

+-------------------+----------------+----------------+
| Jenis Package     | Isi            | Build Type     |
+-------------------+----------------+----------------+
| Interface package | ``.msg``       | ament_cmake    |
+-------------------+----------------+----------------+
| Node package      | Python node    | ament_python   |
+-------------------+----------------+----------------+

Interface dan node **tidak boleh dicampur** dalam satu package.

Urutan Kerja yang Benar
-----------------------

Urutan kerja berikut **tidak boleh dibalik**:

.. code-block:: text

    1. Tentukan DATA apa yang ingin dikirim
    2. Bentuk MSG yang stabil
    3. Buat interface package
    4. Generate msg (build)
    5. Verifikasi msg bisa di-import
    6. Baru buat node publisher
    7. Publish & echo

Kesalahan yang paling umum adalah membuat node terlebih dahulu sebelum message
terverifikasi.

.. note::

    📦 Apa itu interface package?

    Interface Package adalah yang isinya hanya definisi data dan tidak boleh punya node:

    - .msg

    - .srv

    - .action

Struktur yang benar
::

    mcu_interfaces/
    ├── msg/
    │   └── McuStatus.msg
    ├── CMakeLists.txt
    └── package.xml

Step 0 — Membersihkan Workspace
--------------------------------

Masuk ke workspace dan hapus artefak build lama:

.. code-block:: bash

   cd ~/coding/ros2_ws_2
   rm -rf build install log

Pastikan struktur workspace:

.. code-block:: text

   ros2_ws_2/
   └── src/

Step 1 — Membuat Interface Package
-----------------------------------

Buat interface package menggunakan ``ament_cmake``:

.. code-block:: bash

   cd ~/coding/ros2_ws_2/src
   ros2 pkg create mcu_interfaces --build-type ament_cmake

Struktur awal package:

.. code-block:: text

   mcu_interfaces/
   ├── CMakeLists.txt
   └── package.xml

Step 2 — Menambahkan Folder Message
------------------------------------

Tambahkan direktori ``msg``:

.. code-block:: bash

   cd mcu_interfaces
   mkdir msg


Step 3 — Membuat Custom Message
--------------------------------

File ``msg/McuStatus.msg``:

.. code-block:: text

   # MCU system status message
   # Phase 1 – flat, professional, future-safe

   builtin_interfaces/Time stamp
   uint32 seq

   bool connected     # transport alive
   bool valid         # data fresh & trustworthy

   float32 battery_voltage   # volts
   float32 board_temperature # celsius

   uint8 error_code  # 0-OK, 1-OVERVOLT, 2-UNDERVOLT, 3-OVERTEMP

Message ini dirancang **flat, eksplisit, dan mudah dikembangkan**.


Step 4 — Konfigurasi CMakeLists.txt
------------------------------------

Tambahkan blok kode berikut:

.. code-block:: cmake

    find_package(rosidl_default_generators REQUIRED)
    find_package(builtin_interfaces REQUIRED)

    rosidl_generate_interfaces(${PROJECT_NAME}
     "msg/McuStatus.msg"
     DEPENDENCIES builtin_interfaces
   )

   ament_export_dependencies(rosidl_default_runtime)

Sehingga file ``CMakeLists.txt`` akan menjadi seperti ini:

.. code-block:: cmake

    cmake_minimum_required(VERSION 3.8)
    project(mcu_interfaces)

    if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wall -Wextra -Wpedantic)
    endif()

    # find dependencies
    find_package(ament_cmake REQUIRED)
    find_package(rosidl_default_generators REQUIRED)
    find_package(builtin_interfaces REQUIRED)
    # uncomment the following section in order to fill in
    # further dependencies manually.
    # find_package(<dependency> REQUIRED)

    rosidl_generate_interfaces(${PROJECT_NAME}
    "msg/McuStatus.msg"
    DEPENDENCIES builtin_interfaces
    )

    if(BUILD_TESTING)
    find_package(ament_lint_auto REQUIRED)
    # the following line skips the linter which checks for copyrights
    # comment the line when a copyright and license is added to all source files
    set(ament_cmake_copyright_FOUND TRUE)
    # the following line skips cpplint (only works in a git repo)
    # comment the line when this package is in a git repo and when
    # a copyright and license is added to all source files
    set(ament_cmake_cpplint_FOUND TRUE)
    ament_lint_auto_find_test_dependencies()
    endif()

    ament_export_dependencies(rosidl_default_runtime)
    ament_package()

File ini **wajib** agar ROS menghasilkan binding Python dan C++ dari ``.msg``.

.. note::

    **CMakeLists.txt (KENAPA WAJIB)?**

    Ini bukan formalitas.

    CMakeLists.txt memberi tahu ROS:

    *“tolong generate code dari msg ini”*

    Contoh inti yang wajib ada:

        find_package(rosidl_default_generators REQUIRED)

        rosidl_generate_interfaces(${PROJECT_NAME}
        "msg/McuStatus.msg"
        DEPENDENCIES builtin_interfaces
        )

Step 5 — Konfigurasi package.xml
---------------------------------

Tambahkan blok kode berikut:

.. code-block:: xml

    <depend>rosidl_default_generators</depend>
    <depend>rosidl_default_runtime</depend>
    <depend>builtin_interfaces</depend>

    <member_of_group>rosidl_interface_packages</member_of_group>

Sehingga file ``package.xml`` akan menjadi seperti ini:

.. code-block:: xml

    <?xml version="1.0"?>
    <?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
    <package format="3">
    <name>mcu_interfaces</name>
    <version>0.0.0</version>
    <description>TODO: Package description</description>
    <maintainer email="youremail@gmail.com">yourname</maintainer>
    <license>TODO: License declaration</license>

    <buildtool_depend>ament_cmake</buildtool_depend>

    <test_depend>ament_lint_auto</test_depend>
    <test_depend>ament_lint_common</test_depend>

    <depend>rosidl_default_generators</depend>
    <depend>rosidl_default_runtime</depend>
    <depend>builtin_interfaces</depend>

    <member_of_group>rosidl_interface_packages</member_of_group>

    <export>
        <build_type>ament_cmake</build_type>
    </export>
    </package>


Interface package **tidak memiliki** ``setup.py`` dan **tidak menggunakan**
``ament_python``.

Step 6 — Build dan Verifikasi Interface
---------------------------------------

Build workspace:

.. code-block:: bash

   cd ~/coding/ros2_ws_2
   colcon build
   source install/setup.bash

Verifikasi interface:

.. code-block:: bash

   ros2 interface show mcu_interfaces/msg/McuStatus
   python3 -c "from mcu_interfaces.msg import McuStatus; print(McuStatus)"

Hasil:

.. code-block:: bash

    doni@doniubuntu:~/coding/ros2_ws_2$ ros2 interface show mcu_interfaces/msg/McuStatus
    # MCU system status message
    # Phase 1 – flat, professional, future-safe

    builtin_interfaces/Time stamp
            int32 sec
            uint32 nanosec
    uint32 seq

    bool connected     # transport alive
    bool valid         # data fresh & trustworthy

    float32 battery_voltage   # volts
    float32 board_temperature # celsius

    uint8 error_code  # 0=OK, 1=OVERVOLT, 2=UNDERVOLT, 3=OVERTEMP
    doni@doniubuntu:~/coding/ros2_ws_2$ python3 -c "from mcu_interfaces.msg import McuStatus; print(McuStatus)"
    <class 'mcu_interfaces.msg._mcu_status.McuStatus'>
    doni@doniubuntu:~/coding/ros2_ws_2$ 

Jika perintah ini gagal, **jangan lanjut ke pembuatan node**.


Penjelasan Teknis: Build dan Source
-----------------------------------

Saat ``colcon build`` dijalankan, ROS:

1. Membaca file ``.msg``
2. Menghasilkan binding C++ dan Python
3. Menginstal hasilnya ke direktori ``install/``

Tanpa ``source install/setup.bash``, Python tidak mengetahui lokasi message
yang telah di-generate.


Step 7 — Membuat Node Package (Python)
--------------------------------------

Buat node package dengan ``ament_python``:

.. code-block:: bash

   cd ~/coding/ros2_ws_2/src
   ros2 pkg create mcu_status_node --build-type ament_python --dependencies rclpy mcu_interfaces

Struktur package:

.. code-block:: text

   mcu_status_node/
   ├── package.xml
   ├── setup.py
   └── mcu_status_node/
       ├── __init__.py
       └── status_publisher.py

Node package **bergantung pada interface package**, bukan sebaliknya.


Step 8 — Implementasi Publisher Node
------------------------------------

File ``mcu_status_node/status_publisher.py``:

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

            self.seq = 0
            self.timer = self.create_timer(1.0, self.publish_status)

        def publish_status(self):
            msg = McuStatus()
            msg.stamp = self.get_clock().now().to_msg()
            msg.seq = self.seq

            msg.connected = True
            msg.valid = True

            msg.battery_voltage = 12.3
            msg.board_temperature = 35.8
            msg.error_code = 0

            self.publisher_.publish(msg)
            self.seq += 1


    def main():
        rclpy.init()
        node = McuStatusPublisher()
        rclpy.spin(node)
        node.destroy_node()
        rclpy.shutdown()

    if __name__ == '__main__':
        main()


Step 9 — Konfigurasi setup.py
------------------------------

Tambahkan block kode berikut sehingga entry_points di setup.py akan menjadi seperti ini:

.. code-block:: python

    entry_points={
            'console_scripts': [
                'mcu_status_publisher = mcu_status_node.mcu_status_publisher:main'
            ],


Step 10 — Build dan Menjalankan Node
------------------------------------

Build ulang workspace:

.. code-block:: bash

   cd ~/coding/ros2_ws_2
   colcon build
   source install/setup.bash

Jalankan node:

.. code-block:: bash

   ros2 run mcu_status_node mcu_status_publisher

Pantau topic:

.. code-block:: bash

   ros2 topic echo /mcu/status
