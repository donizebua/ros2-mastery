STEP 3 — ROS 2 Serial Parser Node
=========================================================

Dokumen ini membahas inti engineering pada Phase 3, yaitu
node ROS 2 yang bertugas mengubah byte stream serial dari MCU
menjadi data semantik yang dapat dipahami sistem ROS.

Fokus utama adalah determinisme, robustness, dan disiplin arsitektur.

Tujuan Phase 3
-----------------

Pada Phase 1 dan Phase 2, ROS hanya mempublikasikan data yang sudah siap
tanpa memperhatikan asal data tersebut.

Pada Phase 3, ROS bertanggung jawab penuh untuk:

- Membaca byte stream mentah dari hardware
- Memastikan integritas data
- Mengubah data fisik menjadi message ROS yang bermakna

Serial parser adalah batas (boundary) antara dunia fisik dan dunia logika.
Jika boundary ini buruk, sistem akan sulit di-debug dan tidak scalable.

Kontrak Frame Serial
--------------------

Frame dari MCU sudah dikunci dan tidak boleh ditebak oleh sisi ROS.

Format frame:

.. code-block:: text

    STX  LEN  TYPE  SEQ  PAYLOAD (12B)  CRC  ETX
    0x02 0x0E 0x01  0x7E  ADC0..ADC5    XOR 0x03

Node parser harus sepenuhnya patuh pada kontrak ini.

Desain Node
-----------

Nama node: ``mcu_serial_parser``

Tanggung jawab node:

1. Membaca serial byte-per-byte
2. Menggunakan state machine framing
3. Validasi STX dan ETX
4. Validasi panjang frame
5. Validasi CRC
6. Decode payload
7. Publish mcu_interfaces/msg/McuStatus

Node ini tidak memiliki pengetahuan tentang sensor, UI, atau logika robot.
Node ini murni bertindak sebagai translator.

State Machine Parser
--------------------

Parser wajib menggunakan state machine, bukan pendekatan berbasis readline.

State minimal yang digunakan:

.. code-block:: text

    WAIT_STX
    READ_LEN
    READ_BODY
    READ_ETX

Pendekatan ini diperlukan karena data serial dapat terpotong,
datang parsial, atau terkontaminasi noise.

Struktur Package
----------------

Parser berada dalam package yang sudah ada:

.. code-block:: text

    ros2_communication

Dependency package:

.. code-block:: xml

    <depend>rclpy</depend>
    <depend>mcu_interfaces</depend>

Kode Final — ROS 2 Serial Parser Node
---------------------------------------

File: ``mcu_serial_parser.py``

.. code-block:: python

    import rclpy
    from rclpy.node import Node
    import serial
    import struct
    import time

    from mcu_interfaces.msg import McuStatus

    STX = 0x02
    ETX = 0x03
    FRAME_TYPE_ADC_STATUS = 0x01


    class McuSerialParser(Node):
        def __init__(self):
            super().__init__('mcu_serial_parser')

            self.publisher = self.create_publisher(
                McuStatus,
                'mcu/status',
                10
            )

            self.serial = serial.Serial(
                port='/dev/ttyUSB0',
                baudrate=115200,
                timeout=0.01
            )

            # Parser state
            self.state = 'WAIT_STX'
            self.buffer = bytearray()
            self.expected_length = 0

            self.timer = self.create_timer(0.001, self.read_serial)

            self.get_logger().info('MCU Serial Parser started')

        def read_serial(self):
            while self.serial.in_waiting:
                byte = self.serial.read(1)[0]
                self.parse_byte(byte)

        def parse_byte(self, byte):
            if self.state == 'WAIT_STX':
                if byte == STX:
                    self.buffer.clear()
                    self.state = 'READ_LEN'

            elif self.state == 'READ_LEN':
                self.expected_length = byte
                self.buffer.clear()
                self.state = 'READ_BODY'

            elif self.state == 'READ_BODY':
                self.buffer.append(byte)
                if len(self.buffer) == self.expected_length + 1:  # + CRC
                    self.state = 'READ_ETX'

            elif self.state == 'READ_ETX':
                if byte == ETX:
                    self.process_frame(self.buffer)
                self.state = 'WAIT_STX'

        def process_frame(self, data):
            frame_type = data[0]
            seq = data[1]
            payload = data[2:-1]
            recv_crc = data[-1]

            calc_crc = 0
            for b in data[:-1]:
                calc_crc ^= b

            if calc_crc != recv_crc:
                self.get_logger().warn('CRC mismatch')
                return

            if frame_type != FRAME_TYPE_ADC_STATUS:
                return

            adc_values = struct.unpack('<6H', payload)

            msg = McuStatus()
            msg.stamp = self.get_clock().now().to_msg()
            msg.seq = seq
            msg.connected = True
            msg.valid = True

            msg.adc0 = adc_values[0]
            msg.adc1 = adc_values[1]
            msg.adc2 = adc_values[2]
            msg.adc3 = adc_values[3]
            msg.adc4 = adc_values[4]
            msg.adc5 = adc_values[5]

            self.publisher.publish(msg)


    def main():
        rclpy.init()
        node = McuSerialParser()
        rclpy.spin(node)
        node.destroy_node()
        rclpy.shutdown()

Cara Menjalankan Node
-----------------------

Setup dan jalankan node:

.. code-block:: bash

    doni@doniubuntu:~/coding/ros2_ws_2$ setup
    doni@doniubuntu:~/coding/ros2_ws_2$ ros2 run ros2_communication mcu_serial_parser
    [INFO] [1767616358.416641976] [mcu_serial_parser]: MCU Serial Parser started


Liat list topic:

.. code-block:: bash

    doni@doniubuntu:~/coding/ros2_ws_2$ setup
    doni@doniubuntu:~/coding/ros2_ws_2$ ros2 topic list
    /mcu/status
    /parameter_events
    /rosout

Echo pada topic /mcu/status:

.. code-block:: bash

    doni@doniubuntu:~/coding/ros2_ws_2$ ros2 topic echo /mcu/status
    stamp:
    sec: 1767616543
    nanosec: 566839174
    seq: 234
    connected: true
    valid: true
    crc_error_count: 0
    last_frame_age_sec: 7.200241088867188e-05
    adc0: 0
    adc1: 0
    adc2: 15
    adc3: 0
    adc4: 543
    adc5: 144
    ---
    stamp:
    sec: 1767616543
    nanosec: 616889166
    seq: 235
    connected: true
    valid: true
    crc_error_count: 0
    last_frame_age_sec: 0.00011205673217773438
    adc0: 0
    adc1: 0
    adc2: 14
    adc3: 0
    adc4: 542
    adc5: 144

Implementasi ini memenuhi prinsip engineering berikut:

- Non-blocking executor
- Parsing berbasis state machine
- Deterministik di level byte
- Validasi CRC sebelum publish
- Kontrak frame yang eksplisit
- Mudah dikembangkan ke frame lain

Kesalahan yang Harus Dihindari
--------------------------------

Pendekatan berikut tidak boleh digunakan:

- readline() untuk data biner
- Parsing tanpa state
- Publish tanpa validasi CRC
- Menggabungkan parsing dengan logic robot
- Menganggap data serial selalu utuh

Pendekatan tersebut bersifat hobbyist dan tidak cocok
untuk sistem robotik nyata.
