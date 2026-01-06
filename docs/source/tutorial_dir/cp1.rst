Checkpoint 1
============

🎯 POSISI KAMU SAAT INI (ringkas)
----------------------------------

Kamu sudah:

* Paham **node, topic, publisher, subscriber**
* Paham **namespace & konvensi ROS**
* Bisa bikin **ROS ↔ MCU serial bridge (full duplex)**
* Paham **thread vs executor**
* Bisa dokumentasi pakai **Sphinx + ReadTheDocs**

Artinya:

👉 **Masalah komunikasi dasar sudah selesai.**

Sekarang waktunya naik level ke **arsitektur robot beneran**.


🚀 NEXT LEARNING PATH (yang PALING MASUK AKAL)
-----------------------------------------------


1️⃣ ROS 2 PARAMETERS & DYNAMIC RECONFIGURATION
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Supaya node kamu tidak “hard-coded”

Yang kamu pelajari:

* Parameter YAML
* Declare & get parameter
* Update parameter saat runtime
* Gunanya untuk tuning MCU / kontrol

📌 Contoh proyek:

PID gain MCU diubah **tanpa reflash** ESP32


2️⃣ ROS 2 INTERFACE (msg / srv / action) 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ini **WAJIB**, karena kamu masih pakai ``std_msgs``.

Yang kamu pelajari:

* Custom ``.msg``
* ``.srv`` (request–response)
* ``.action`` (long-running task)

📌 Contoh:

* ``McuData.msg`` → voltage, rpm, temperature
* ``SetMotor.srv`` → speed, direction
* ``MoveMotor.action`` → target + feedback

Ini bikin sistem kamu **naik kelas banget**.


3️⃣ TIMER, EXECUTOR, & CALLBACK GROUP (LEVEL UP)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Supaya kamu **benar-benar paham concurrency di ROS**

Yang kamu pelajari:

* SingleThreaded vs MultiThreadedExecutor
* CallbackGroup (MutuallyExclusive / Reentrant)
* Kapan perlu thread manual, kapan tidak

📌 Ini jawab pertanyaan kamu kemarin:

“Kenapa thread best practice tapi kadang nggak wajib?”


4️⃣ ROS 2 LAUNCH FILE (wajib sebelum lanjut)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Supaya gak run node satu-satu lagi.

Yang kamu pelajari:

* ``launch.py``
* Group namespace
* Parameter via launch
* Remapping topic

📌 Contoh:

.. code-block:: bash

   ros2 launch robot_bringup mcu.launch.py

Ini **standar industri ROS**.


5️⃣ ROS 2 + CONTROL (kalau mau serius kontrol)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Karena kamu anak Instrumentasi & Kontrol 👀

Yang kamu pelajari:

* Control loop di ROS
* ROS 2 Control concept
* Hardware interface (bridge ke MCU)

📌 Ini jembatan:

MCU - actuator  
ROS - brain


6️⃣ SENSOR FUSION (Kalman Filter, dsb) 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ini cocok banget sama minat kamu sebelumnya.

Mulai dari:

* Low-pass filter
* Moving average
* 1D Kalman Filter
* Sensor fusion IMU + encoder

📌 Jangan lompat langsung EKF, tapi **bertahap**.


🧭 REKOMENDASI URUTAN PALING IDEAL
------------------------------------

Kalau aku jadi kamu:

.. code-block:: text

   1. Custom msg / srv
   2. Parameters + YAML
   3. Launch file
   4. Executor & callback group
   5. Control + filtering


💡 Mau yang PROJEK-BASED?
--------------------------

Kalau kamu mau **belajar sambil bikin sesuatu**, ini 3 opsi:


🔹 Opsi 1 — “Smart MCU Node”
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Custom msg
* Dynamic parameter
* Launch file
* Full documentation


🔹 Opsi 2 — “Mini Robot Stack”
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* MCU bridge
* Control loop
* Sensor filtering
* Namespace rapi


🔹 Opsi 3 — “Industrial Style ROS Node”
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Clean architecture
* Separation of concerns
* Best practice ROS 2