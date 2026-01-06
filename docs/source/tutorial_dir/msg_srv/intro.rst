Introduction
============

🛠️ Technique
-------------
Materi ini dibagi menjadi **tiga tahap utama**:

1. **Custom ``.msg``** — untuk data streaming
2. **Custom ``.srv``** — untuk command / request–response
3. **Implementasi ke MCU Serial Bridge** — agar dapat digunakan secara nyata


Saat ini, tipe message yang digunakan adalah:

.. code-block:: bash

   std_msgs/msg/Float32

.. note::

    Keterbatasan pendekatan ini:

    * Hanya memuat satu nilai
    * Tidak scalable
    * Tidak bersifat self-descriptive

    Contoh data nyata dari MCU biasanya meliputi:

    * Tegangan
    * Arus
    * RPM
    * Status sistem

➡️ **Custom message memungkinkan seluruh data tersebut dikemas dalam satu struktur data yang terdefinisi dengan jelas.**

Desain Message
^^^^^^^^^^^^^^

Contoh desain message yang profesional:

📄 ``McuData.msg``

.. code-block:: text

   float32 voltage
   float32 current
   int32   rpm
   bool    fault

.. note::

    Prinsip desain message yang baik:

    * Nama field **deskriptif**
    * Tipe data **jelas dan tepat**
    * Struktur **tidak berlebihan**, namun cukup representatif


Struktur File
^^^^^^^^^^^^^

Struktur package ``ros2_communication``:

.. code-block:: text

   ros2_communication/
   ├── msg/
   │   └── McuData.msg
   ├── ros2_communication/
   │   └── mcu_serial_bridge.py
   ├── package.xml
   └── setup.py

.. note::

    📌 Folder ``msg/`` **bukan bagian dari Python**, melainkan **ROS Interface Definition**.


Membuat File ``.msg``
^^^^^^^^^^^^^^^^^^^^^   

A. Membuat folder ``msg``

.. code-block:: bash

   mkdir msg

B. Membuat file message

.. code-block:: bash

   nano msg/McuData.msg

Isi file:

.. code-block:: text

   float32 voltage
   float32 current
   int32 rpm


Mendaftarkan Message Ke Package
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A. ``package.xml``

Tambahkan **dua dependency berikut**:

.. code-block:: xml

   <build_depend>rosidl_default_generators</build_depend>
   <exec_depend>rosidl_default_runtime</exec_depend>

Pastikan juga terdapat baris:

.. code-block:: xml

   <member_of_group>rosidl_interface_packages</member_of_group>


B. ``setup.py``

Tambahkan konfigurasi ``data_files``:

.. code-block:: python

   data_files-[
       ('share/ament_index/resource_index/packages', ['resource/' + package_name]),
       ('share/' + package_name, ['package.xml']),
       ('share/' + package_name + '/msg', ['msg/McuData.msg']),
    ],

.. note::

    📌 Bagian ini **sering menjadi sumber error** apabila terlewat.


Build & Verifikasi
^^^^^^^^^^^^^^^^^^

Lakukan build:

.. code-block:: bash

   colcon build
   source install/setup.bash

Verifikasi interface:

.. code-block:: bash

   ros2 interface show ros2_communication/msg/McuData

Jika struktur message tampil dengan benar, maka proses berhasil ✅


Contoh Penggunaaan di Node (Preview)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Contoh penggunaan (preview):

.. code-block:: python

   from ros2_communication.msg import McuData

   msg - McuData()
   msg.voltage - 12.3
   msg.current - 1.2
   msg.rpm - 1500

   publisher.publish(msg)

.. note:: 

    📌 Tidak ada lagi field ``data`` karena setiap nilai sudah memiliki nama field sendiri.


Tugas
^^^^^

Sebelum melanjutkan ke custom ``.srv``, lakukan langkah berikut:

1. Membuat ``McuData.msg``
2. Build package hingga berhasil
3. Verifikasi dengan ``ros2 interface show``
4. **Belum digunakan di node terlebih dahulu**

🧠 Insight
-----------

Mengapa ROS Memerlukan Custom Message & Service?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Masalah utama jika hanya menggunakan ``std_msgs`` adalah keterbatasan representasi data.

``std_msgs`` hanya menyediakan **tipe data primitif**, seperti:

.. code-block:: text

   Float32
   Int32
   Bool
   String

.. note:: 

    Keterbatasan pendekatan ini:

    - ❌ Tidak memiliki makna konteks  
    - ❌ Tidak mampu merepresentasikan struktur sistem  
    - ❌ Sulit diskalakan untuk sistem kompleks (robot, MCU, sensor)

Contoh penggunaan yang tidak informatif:

.. code-block:: text

   topic: /mcu/data
   Float32: 25.0

Nilai ``25`` tidak memiliki makna eksplisit:

* Apakah suhu?
* Tegangan?
* Kecepatan motor?

Solusi yang ditawarkan ROS adalah **custom message sebagai kontrak data eksplisit**.

Contoh:

.. code-block:: text

   McuData.msg
   float32 voltage
   float32 temperature
   bool    error

.. note::

    Dengan pendekatan ini:

    * Data bersifat **self-describing**
    * Node lain memahami **struktur dan makna data**
    * Aman untuk pemeliharaan jangka panjang


Konsep Fundamental: Message Bukan Variable
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ini merupakan konsep filosofis penting dalam ROS.

Message **bukan** variable Python atau C++.

Message adalah:

> **Interface Definition Language (IDL)**

Implikasinya:

* Bukan Python
* Bukan C++
* Melainkan **kontrak lintas bahasa**

Contoh definisi:

.. code-block:: text

   float32 voltage

Definisi ini dapat digunakan oleh:

* Node Python
* Node C++
* Micro-ROS (C)
* ROS bag
* DDS network

.. note::

    📌 Inilah alasan mengapa syntax ``.msg`` bersifat kaku dan terbatas.


Struktur ``.msg`` — Alasan Kesederhanaan
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Contoh struktur:

.. code-block:: text

   float32 voltage
   float32 current

Pertanyaan umum:

**Mengapa tidak menggunakan syntax Python?**

Karena:

* ROS harus melakukan code generation ke **banyak bahasa**
* Parser harus **deterministik**
* Harus kompatibel dengan middleware DDS

Konsekuensinya:

* Tidak ada assignment ``-``
* Tidak ada fungsi
* Tidak ada logika

📌 File ``.msg`` adalah **skema data murni**.

Mengapa ROS Harus Mengenerate Message?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Saat proses build:

.. code-block:: bash

   colcon build

ROS melakukan pipeline berikut:

.. code-block:: text

   .msg / .srv
       ↓
   rosidl
       ↓
   Generate:
   - Python class
   - C++ struct
   - DDS type support

Inilah alasan dependency berikut **wajib ada**:

.. code-block:: xml

   <build_depend>rosidl_default_generators</build_depend>
   <exec_depend>rosidl_default_runtime</exec_depend>

.. note::


    Tanpa dependency ini:

    - ❌ Kode Python tidak dihasilkan  
    - ❌ ``import ros2_communication.msg`` akan gagal  

    📌 Custom message **bukan interpreted**, melainkan **compiled interface**.


Peran ``data_files`` di ``setup.py``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Contoh konfigurasi:

.. code-block:: python

   ('share/' + package_name + '/msg', ['msg/McuData.msg']),

Fungsi utamanya adalah **mendaftarkan interface ke sistem ROS**.

ROS **tidak melakukan discovery otomatis** terhadap file ``.msg``.

.. note:: 

    Jika langkah ini dilewati:

    - ❌ ``ros2 interface show`` gagal  
    - ❌ Package lain tidak mengetahui keberadaan message  

    📌 Ini merupakan tahap **registrasi interface**.


Message vs Service — Perspektif Arsitektur
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

🔵 Message (Topic)

Digunakan untuk **streaming data asynchronous dan one-way**.

Contoh:

.. code-block:: text

   MCU → ROS: sensor data

Karakteristik:

* Tidak ada response
* Banyak publisher
* Banyak subscriber
* Cocok untuk real-time

Digunakan untuk:

* Sensor
* Status
* Telemetry


🟠 Service

Digunakan untuk **request–response (RPC style)**.

Contoh:

.. code-block:: text

   ROS → MCU: set motor speed
   MCU → ROS: OK / ERROR

Karakteristik:

* Sinkron
* Secara konsep bersifat blocking
* 1 request → 1 response

Digunakan untuk:

* Command
* Konfigurasi
* Trigger

Struktur ``.srv`` dan Makna Pemisah
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Contoh:

.. code-block:: text

   float32 target_speed
   ---
   bool success
   string message

Pemisah ``---`` digunakan untuk:

➡️ Memisahkan **request** dan **response**

Tanpa pemisah ini, ROS tidak dapat menentukan arah komunikasi.


Alasan Service Tidak Cocok untuk Streaming
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        
Menggunakan service untuk data streaming menyebabkan:

- ❌ Blocking  
- ❌ Tidak scalable  
- ❌ DDS overhead  
- ❌ Tidak real-time safe  

Karena itu:

> **Topic digunakan untuk data, service untuk perintah**

Ini merupakan **best practice global ROS**.


Mengapa Field Harus Bernama?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Contoh yang benar:

.. code-block:: text

   float32 voltage

Bukan:

.. code-block:: text

   float32

Alasannya:

* Self-documenting
* Mendukung backward compatibility
* Mudah dikembangkan

ROS mengizinkan penambahan field baru:

.. code-block:: text

   float32 voltage
   float32 current

Node lama tetap dapat berjalan tanpa modifikasi.

Hubungan dengan MCU Serial Bridge
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pada sisi ROS:

.. code-block:: python

   msg - McuData()
   msg.voltage - 12.3
   msg.current - 0.8

Pada sisi serial:

.. code-block:: text

   DATA:12.3,0.8

Custom message berperan sebagai:

* Abstraksi
* Boundary sistem
* Kontrak data

MCU tidak perlu memahami ROS, dan ROS tidak bergantung pada detail serial.


Relevansi Profesional dan Industri
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pendekatan ini memungkinkan:

* Implementasi diganti tanpa mengubah interface
* MCU diganti tanpa memodifikasi node lain
* Simulator ditambahkan tanpa perubahan arsitektur

Ini merupakan prinsip **clean architecture** dalam ROS.


Ringkasan Mental Model
^^^^^^^^^^^^^^^^^^^^^^
Custom message dan service adalah **bahasa kontrak sistem ROS**, bukan sekadar fitur tambahan.

.. list-table::
   :header-rows: 1

   * - Elemen
     - Peran
   * - ``.msg``
     - Struktur data
   * - ``.srv``
     - Protokol request–response
   * - ``rosidl``
     - Generator lintas bahasa
   * - ``package.xml``
     - Deklarasi interface
   * - ``setup.py``
     - Registrasi interface

.. note::

    Langkah lanjutan yang direkomendasikan:

    * Versioning message (backward compatibility)
    * Nested message
    * Message untuk real-time control
    * Mapping message ke frame serial (industrial style)

🧩 Roadmap
-------------

Advanced Custom Message and Service Design — ROS 2

Sebelum masuk teknis, ada 3 prinsip inti:

Message adalah kontrak data, bukan variabel
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- .msg bukan sekadar struct

- .msg adalah API publik antar node

Begitu dipakai banyak node → sulit diubah. Jadi desain msg harus dipikirkan ke depan

ROS msg ≠ MCU packet, tapi harus bisa dipetakan
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- ROS message → semantic

- Serial frame → deterministic & minimal

- Kita desain msg agar mudah diserialisasi

Skalabilitas > kenyamanan sesaat
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Lebih baik 1 msg agak panjang tapi stabil, daripada sering ganti msg & break sistem

PHASE 1 — Message Design Philosophy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Atomic vs Composite message

- Flat vs Nested

- Naming convention profesional

- Timestamp, frame_id, validity

PHASE 2 — Nested Message & Reusability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Reusable sub-message

- Import msg di dalam msg

Contoh sistem robot nyata

PHASE 3 — Versioning & Backward Compatibility
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Menambah field tanpa merusak sistem

- Deprecated field

- Strategy v1 → v2

PHASE 4 — msg vs srv vs action (Decision Matrix)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Kapan HARUS pakai msg

- Kapan SALAH pakai srv

- Action untuk kontrol hardware

PHASE 5 — ROS msg ↔ Serial Frame Mapping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
- Industrial-style protocol

- Fixed-length vs TLV

- Checksum, sequence, ack

- Contoh frame MCU nyata
