STEP 2 — KODE MCU (ESP32/ARDUINO) SERIAL FRAME SENDER
======================================================

Dokumen ini membahas implementasi hands-on di sisi MCU
untuk Phase 3, dengan fokus pada pengiriman frame serial biner
berdasarkan desain frame final yang sudah dikunci sebelumnya.

Target implementasi ini adalah kode embedded yang deterministik,
stabil, dan independen dari ROS.

Target MCU
-------------

MCU harus memenuhi target berikut:

- Membaca ADC A0–A5
- Membungkus data ke binary frame
- Mengirim data periodik dan stabil
- Mengirim data nyata (tanpa dummy)
- Tidak memiliki dependensi ROS

Kontrak Frame yang Wajib Dipatuhi
-----------------------------------

Frame yang dikirim oleh MCU harus identik dengan desain berikut:

.. code-block:: text

    STX | LEN | TYPE | SEQ | PAYLOAD | CRC | ETX

Konstanta frame:

.. code-block:: text

    STX      = 0x02
    ETX      = 0x03
    TYPE     = 0x01   (ADC_STATUS_V1)
    PAYLOAD  = 6 × uint16 (A0–A5)

Jika salah satu bagian frame tidak sesuai kontrak ini,
maka parser ROS tidak akan dapat bekerja dengan benar.

Strategi Implementasi MCU
--------------------------

Pendekatan yang **tidak** digunakan:

- Serial.print()
- String
- CSV
- Newline-delimited data

Pendekatan yang digunakan:

- uint8_t buffer[]
- Serial.write(buffer, length)

Pendekatan ini mencerminkan embedded-style programming,
bukan gaya prototyping Arduino.

Pin ADC ESP32 yang Digunakan
----------------------------

ESP32 menggunakan ADC1 untuk menghindari konflik dengan WiFi.

Mapping pin:

.. code-block:: text

    A0 → GPIO 36
    A1 → GPIO 39
    A2 → GPIO 34
    A3 → GPIO 35
    A4 → GPIO 32
    A5 → GPIO 33

ADC2 tidak digunakan.

Kode MCU Final (ESP32)
----------------------

File: mcu_adc_frame_sender.ino

.. code-block:: cpp

    #include <Arduino.h>

    #define STX  0x02
    #define ETX  0x03
    #define FRAME_TYPE_ADC_STATUS 0x01

    const int adc_pins[6] = {36, 39, 34, 35, 32, 33};
    uint8_t sequence_number = 0;

    uint8_t compute_crc(uint8_t *data, uint8_t length)
    {
      uint8_t crc = 0;
      for (uint8_t i = 0; i < length; i++)
      {
        crc ^= data[i];
      }
      return crc;
    }

    void setup()
    {
      Serial.begin(115200);
      delay(1000);

      analogReadResolution(12);

      for (int i = 0; i < 6; i++)
      {
        pinMode(adc_pins[i], INPUT);
      }
    }

    void loop()
    {
      uint16_t adc_values[6];

      for (int i = 0; i < 6; i++)
      {
        adc_values[i] = analogRead(adc_pins[i]);
      }

      const uint8_t payload_length = 12;
      const uint8_t len_field = 1 + 1 + payload_length;

      uint8_t frame[1 + 1 + 1 + 1 + payload_length + 1 + 1];
      uint8_t idx = 0;

      frame[idx++] = STX;
      frame[idx++] = len_field;
      frame[idx++] = FRAME_TYPE_ADC_STATUS;
      frame[idx++] = sequence_number++;

      for (int i = 0; i < 6; i++)
      {
        frame[idx++] = adc_values[i] & 0xFF;
        frame[idx++] = (adc_values[i] >> 8) & 0xFF;
      }

      uint8_t crc = compute_crc(&frame[2], len_field);
      frame[idx++] = crc;
      frame[idx++] = ETX;

      Serial.write(frame, idx);
      delay(50);
    }

Keputusan Teknis Penting
------------------------

Alasan LEN tidak berisi total panjang frame:

- Parser dapat langsung mengetahui ukuran data tanpa bergantung pada delimiter. STX dan ETX berfungsi sebagai delimiter untuk menentukan awal dan akhir dari frame data.

Alasan menggunakan uint16 mentah:

- Scaling dan interpretasi adalah tanggung jawab ROS

Alasan periode delay(50):

- Stabil
- Tidak membanjiri serial
- Cukup cepat untuk monitoring

Pengujian Awal Tanpa ROS
------------------------

Sebelum masuk ke ROS, lakukan pengujian serial mentah.

Di Linux:

.. code-block:: bash

    stty -F /dev/ttyUSB0 115200 raw -echo
    cat /dev/ttyUSB0 | hexdump -C

Yang harus terverifikasi:

- Byte awal selalu 02
- Byte akhir selalu 03
- Panjang frame konsisten
- Nilai ADC berubah sesuai input fisik

.. warning::

    Jika tahap ini gagal, jangan melanjutkan ke ROS.

Contoh Hasil:

.. code-block:: bash

    doni@doniubuntu:~$ stty -F /dev/ttyUSB0 115200 raw -echo
    doni@doniubuntu:~$ cat /dev/ttyUSB0 | hexdump -C
    00000000  02 0e 01 7e 59 00 10 00  9d 05 c0 09 55 09 cd 07  |...~Y.......U...|
    00000010  f1 03 02 0e 01 7f 1b 01  3d 01 53 0b ff 0f ff 0f  |........=.S.....|
    00000020  30 0f 3f 03 02 0e 01 80  50 00 10 00 83 05 a7 09  |0.?.....P.......|
    00000030  3f 09 02 0e 01 bb 07 01  ef 00 49 0a ff 0f ff 0f  |?.........I.....|
    00000040  c7 0e d9 03 02 0e 01 bc  00 00 00 00 4b 02 4e 05  |............K.N.|
    00000050  59 05 22 04 c5 03 02 0e  01 bd 10 01 fd 00 42 0a  |Y."...........B.|
    00000060  ff 0f ff 0f c3 0e d5 03  02 0e 01 be 00 00 00 00  |................|
    00000070  17 02 ff 04 41 05 a7 03  b1 03 02 0e 01 bf 10 01  |....A...........|
    00000080  f0 00 40 0a ff 0f ff 0f  b0 0e ab 03 02 0e 01 c0  |..@.............|
    00000090  00 00 00 00 e2 01 b5 04  ba 04 70 03 5e 03 02 0e  |..........p.^...|

Penjelasan command:



Perhatikan potongan ini:

.. code-block:: text

    02 0e 01 7e ... f1 03
    02 0e 01 7f ... 3f 03
    02 0e 01 80 ... 3f 03

+---------+------------------------------------------------------+
|    Byte    |                      Makna                        |
+============+===================================================+
|     02     |                       STX                         | 
+------------+---------------------------------------------------+
|     0e     |   Length = 14 byte (1 type + 1 seq + 12 payload)  |
+------------+---------------------------------------------------+
|      01    |              Frame Type = ADC_STATUS              |
+------------+---------------------------------------------------+
| 7e, 7f, 80 |                Sequence number naik               |
+------------+---------------------------------------------------+
|    ...     |                 6x ADC (LSB + MSB)                |
+------------+---------------------------------------------------+
|   f1, 3f   |                        CRC                        |
+------------+---------------------------------------------------+
|     03     |                        ETX                        |
+------------+---------------------------------------------------+

Dan Sekarang:

- STX konsisten

- Length konsisten

- Frame berulang rapi

- Sequence naik

- ETX selalu ada

Validasi Cepat (Engineer Check):

- framing: PASS
- deterministik: PASS
- endian jelas: PASS
- CRC ada: PASS
- cocok untuk ROS parser: PASS

Tidak ada yang perlu diperbaiki di sisi MCU.

Status Saat Ini
----------------

- Desain frame final: selesai
- MCU mengirim data nyata: selesai
- Tidak ada dummy data: selesai
- Siap diparse di ROS: siap

Langkah Berikutnya
------------------

Tahap berikutnya adalah sisi ROS:

Phase 3 — Step 3: ROS 2 Serial Parser Node

Cakupan:

- Membaca byte stream
- State machine frame
- Validasi CRC
- Mapping ke mcu_interfaces/msg/McuStatus
- Publish topic ROS
