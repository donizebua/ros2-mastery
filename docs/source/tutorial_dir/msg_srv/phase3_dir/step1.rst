STEP 1 — DESAIN SERIAL FRAME (INDUSTRIAL GRADE)
==========================================================

Mental Model Wajib Phase 3
--------------------------

Frame serial **bukan ROS message**.

Frame serial adalah *transport-level contract*

Sistem terdiri dari tiga dunia yang **harus dipisahkan secara tegas**:

1. MCU world  
   Byte, register, ADC, interrupt

2. Transport world  
   Frame, boundary, CRC, sequence

3. ROS world  
   Typed message, nested message, semantic data

Frame serial **hidup di dunia transport**, dan **tidak boleh bergantung
pada ROS** dalam bentuk apa pun.


Tujuan Desain Frame Serial
--------------------------

Frame serial yang dirancang **harus memenuhi seluruh kriteria berikut**:

- Membawa data ADC A0 sampai A5 (data nyata, banyak kanal)
- Memiliki penanda awal dan akhir frame
- Memiliki panjang frame yang eksplisit
- Memiliki tipe frame untuk versioning
- Memiliki sequence number
- Dapat dicek integritas datanya
- Mudah diparse baik di MCU maupun di ROS

Jika satu saja poin di atas tidak terpenuhi,
maka desain frame dianggap **gagal secara engineering**.


Keputusan Desain Utama
----------------------------

Binary Frame (Bukan String)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Frame dikirim dalam bentuk **binary**, bukan ASCII string.

Alasan pemilihan:

- Parsing deterministik
- Bandwidth lebih hemat
- Tidak ambigu
- Mudah dilakukan checksum dan CRC

String terlihat mudah di awal,
namun menjadi sumber masalah besar di sistem nyata.

Fixed-length Payload
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Payload menggunakan **fixed-length**.

Keuntungan utama:

- Implementasi MCU lebih sederhana
- Parser ROS lebih stabil
- Tidak membutuhkan alokasi memori dinamis

Data ADC Mentah (uint16)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

MCU mengirim **nilai ADC mentah**, bukan nilai fisik.

Alasan:

- MCU hanya mengirim fakta mentah
- Interpretasi dilakukan di sisi ROS
- Skala dan kalibrasi bisa berubah tanpa reflash MCU

Ini adalah keputusan desain yang **dewasa secara sistem**.

Frame Serial Final (Versi 1)
----------------------------------

Struktur frame serial ditetapkan sebagai berikut:

.. code-block:: text

    ┌────┬────┬────┬────┬───────────────┬────┬────┐
    │STX │LEN │TYPE│SEQ │   PAYLOAD     │CRC │ETX │
    └────┴────┴────┴────┴───────────────┴────┴────┘

Desain ini **final untuk Phase 3** dan tidak akan diubah
selama implementasi.

Definisi Field Frame
--------------------------

STX — Start of Text
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    Ukuran : 1 byte
    Nilai  : 0x02

STX berfungsi sebagai penanda awal frame.
Parser dapat melakukan sinkronisasi ulang
jika terjadi korupsi data di tengah stream.

LEN — Length
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    Ukuran : 1 byte
    Isi    : panjang TYPE + SEQ + PAYLOAD

Field LEN **tidak mencakup** STX, CRC, dan ETX.

Dengan LEN, parser mengetahui
berapa byte yang harus dibaca untuk satu frame penuh.

TYPE — Frame Type
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    Ukuran : 1 byte
    Nilai  : 0x01 - ADC_STATUS_V1

TYPE adalah kunci **versioning** frame.
Field ini akan menjadi sangat penting
pada fase lanjutan.


SEQ — Sequence Number
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    Ukuran : 1 byte
    Rentang: 0–255 (rollover)

SEQ digunakan untuk:

- Deteksi frame drop
- Debug latency
- Sinkronisasi data

PAYLOAD — Data Utama
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Payload versi 1 berisi enam kanal ADC:

.. code-block:: text

    A0  A1  A2  A3  A4  A5

Setiap kanal direpresentasikan sebagai:

.. code-block:: text

    uint16 (2 byte, little endian)

Total payload:

.. code-block:: text

    6 × 2 byte - 12 byte

CRC — Checksum
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    Ukuran : 1 byte

Metode checksum Phase 3:

- XOR dari semua byte mulai TYPE
- Sampai byte terakhir payload

Metode ini sederhana, cepat,
dan cukup untuk kebutuhan Phase 3.

ETX — End of Text
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    Ukuran : 1 byte
    Nilai  : 0x03

ETX menandai akhir frame
dan membantu parser memvalidasi boundary.

Contoh Frame Nyata (HEX View)
-------------------------------

Dengan data berikut:

- SEQ - 0x21
- A0 - 1234
- A1 - 2048
- A2 - 3000
- A3 - 0
- A4 - 512
- A5 - 4095

Frame dalam representasi heksadesimal:

.. code-block:: text

    02
    0E
    01
    21
    D2 04
    00 08
    B8 0B
    00 00
    00 02
    FF 0F
    CRC
    03

Ini adalah contoh **frame nyata**, bukan dummy.

Relasi dengan ROS Message Phase 2
-----------------------------------

Pemetaan konseptual antara ADC dan ROS message:

+-----+----------------------------+
| ADC | ROS Field                  |
+-----+----------------------------+
| A0  | power.battery_voltage      |
+-----+----------------------------+
| A1  | power.battery_current      |
+-----+----------------------------+
| A2  | thermal.board_temperature  |
+-----+----------------------------+
| A3  | system / aux               |
+-----+----------------------------+
| A4  | system / aux               |
+-----+----------------------------+
| A5  | system / aux               |
+-----+----------------------------+

Frame serial **tidak mengetahui ROS**,
dan ROS **tidak mengetahui frame mentah**.

Satu-satunya penghubung adalah **bridge**.

Kenapa Phase 3 Adalah Titik Kritis
------------------------------------

Perbandingan kapabilitas sistem:

+---------------+---------+---------+---------+
| Aspek         | Phase 1 | Phase 2 | Phase 3 |
+---------------+---------+---------+---------+
| Data nyata    | ❌      | ❌      | ✅      |
+---------------+---------+---------+---------+
| Banyak kanal  | ❌      | ❌      | ✅      |
+---------------+---------+---------+---------+
| Robust        | ❌      | ❌      | ✅      |
+---------------+---------+---------+---------+
| Versionable   | ❌      | ⚠️      | ✅      |
+---------------+---------+---------+---------+
| Industrial    | ❌      | ❌      | ✅      |
+---------------+---------+---------+---------+

Phase 3 adalah **pertama kalinya sistem siap
berhadapan dengan hardware nyata**.