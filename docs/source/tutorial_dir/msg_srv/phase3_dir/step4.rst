STEP 4 — Robustness & Fault Handling (Phase 3 lanjut)
======================================================

Sekarang mindset kita berubah:

- ❌ “asal data keluar”
- ✅ **“sistem tetap hidup saat dunia nyata jelek”**

Tujuan Step ini
-----------------

Node ``mcu_serial_parser`` harus:

1. Tidak crash walau data rusak
2. Tahu MCU masih hidup atau tidak
3. Pulih sendiri saat MCU dicabut / reconnect
4. Memberi sinyal ke ROS kalau data tidak valid

Catatan:
Robustness dilakukan **di level node**, bukan di ``.msg``.
Interface **belum diubah**.

Jenis masalah dunia nyata yang ditangani
----------------------------------------

+-------------------+------------------+
| Masalah           | Dampak           |
+===================+==================+
| MCU reset         | sequence lompat  |
+-------------------+------------------+
| Kabel USB dicabut | read error       |
+-------------------+------------------+
| Frame terpotong   | parsing kacau    |
+-------------------+------------------+
| CRC salah         | data salah       |
+-------------------+------------------+
| MCU mati          | data beku        |
+-------------------+------------------+

Prinsip desain robustness
-------------------------

1. Parser tidak pernah percaya input
2. Semua error harus recoverable
3. Timeout lebih penting dari throughput
4. Node boleh drop data, tapi tidak boleh mati

Perubahan desain (konseptual)
-----------------------------

Penambahan yang dilakukan:

1. Timeout watchdog  
   - Simpan ``last_frame_time``  
   - Jika lebih dari X detik, MCU dianggap disconnected

2. Counter error  
   - CRC fail counter  
   - Frame drop counter

3. Serial auto-reconnect  
   - Exception → close → retry

4. Safe reset state machine  
   - Error → kembali ke ``WAIT_STX``

Tambahan pada __init__
-----------------------

.. code-block:: python

    self.last_frame_time = time.time()
    self.crc_error_count = 0
    self.timeout_sec = 1.0

Update read_serial()
--------------------

.. code-block:: python

    def read_serial(self):

        if not self.ser.is_open:
            self.get_logger().warn('Serial closed, retrying...')
            self.reconnect_serial()
            return
        try:
            while self.ser.in_waiting>0:
                byte = self.ser.read(1)[0]
                self.parse_byte(byte)
        except (serial.SerialException, OSError) as e:
            self.get_logger().warn(f'Serial error: {e}')
            self.reconnect_serial()
        #watchdog
        if time.time()-self.last_frame_time > self.timeout_sec:
            self.get_logger().warn('MCU timeout')

Update process_frame()
-----------------------

.. code-block:: python

    def process_frame(self, data):
        frame_type = data[0]
        seq = data[1]
        payload = data[2:-1]
        recv_crc = data[-1]

        calc_crc = 0
        for b in data[:-1]:
            calc_crc ^= b

        if calc_crc != recv_crc:
            self.crc_error_count += 1
            self.get_logger().warn(f'CRC error ({self.crc_error_count})')
            return

        self.last_frame_time = time.time()

Tambah fungsi reconnect
-----------------------

.. code-block:: python

    def reconnect_serial(self):
        try:
            self.serial.close()
            time.sleep(1)
            self.serial.open()
            self.get_logger().info('Serial reconnected')
        except Exception:
            pass

Apa yang dicapai di Step ini
----------------------------

+------------+----------------+-------------+
| Aspek      | Sebelum        | Sekarang    |
+============+================+=============+
| CRC fail   | crash / silent | aman        |
+------------+----------------+-------------+
| MCU dicabut| node mati      | auto recover|
+------------+----------------+-------------+
| Frame rusak| parser kacau   | reset       |
+------------+----------------+-------------+
| Debug      | susah          | jelas       |
+------------+----------------+-------------+

Insight penting
---------------

Robustness **bukan fitur tambahan**.  
Robustness adalah **syarat minimum sistem nyata**.

Banyak sistem:
- sukses di laptop
- gagal di robot

Step ini memastikan sistem **tetap hidup di dunia nyata**.

Status sistem
-------------

- MCU framing: OK
- Serial parsing: OK
- ROS integration: OK
- Robustness dasar: OK
