# Chapter 12 Internet Communication and Controls
# Introducing
## The Internet 
ที่สำคัญของส่วนนี้มีแค่อุปกรณ์ Embed ก็ใช้ Internet ได้นะ เรียกว่า **IOT (Internet of things)**
## The Ethernet
ใช้มาตรฐาน IEEE 802.3 ที่มีความเร็บยสูงสุดได้ถึง 100Gbps (ได้แค่ในจินตนาการแหละ) เป็น Protocol แบบ Serial โดยจะเชื่อมต่อกับ Network ที่แบ่งเป็น 2 แบบคือ

1. LAN ระยะใกล้ๆเช่นในตึก อาจจะไม่มีเน็ตก็ได้
1. WAN ระยะไกลขึ้น แน่นอนมีเน็ต

ใช้สายต่อ 4 เส้น โดยเป็นสัญญาณแบบ Differential เพื่อลดสัญญาณรบกวนประกอบด้วย TX+,TX- และ RX+,RX-

Ethernet ส่งข้อมูลเป็น Frame โดยสามารถส่งข้อมูลได้เร็วและใหญ่ แต่ละ Frame จะประกอบด้วย MAC Address ของต้นทางและปลายทาง

<figure markdown="span" align="center">
  ![MCU Architecture](images\ethernetframe.png){ width="500"}
  <figcaption>Ethernet Frame</figcaption>
</figure>

ผมไม่จำครับ

Ethernet ส่งข้อมูลโดย Encode แบบ Manchester ก็คือ High->Low นับเป็น 0, Low->High นับเป็น 1 แต่ไม่รู้มันดูยังไงแต่มันใช้ Sync Clock ของข้อมูลตอนส่งได้ด้วย

## Ethernet on mbed
```c
/* Program Example 12.7: Ethernet write - sends two data bytes every 200 ms
from an mbed’s Ethernet port. The two byte values are arbitrarily chosen as
0xB9 and 0x46.
*/
#include "mbed.h"
#include "Ethernet.h"
Ethernet eth; // The Ethernet object
char data[]={0xB9,0x46}; // Define the data values
int main() {
while (1) {
    eth.write(data,0x02); // Write the package
    eth.send(); // Send the package
    wait(0.2); // wait 200 ms
}
}
```
```c
/* Program Example 12.8: Ethernet read - allows an mbed to read Ethernet data traffic
and display the captured data to the screen
*/
#include "mbed.h"
Ethernet eth; // Ethernet object
Serial pc(USBTX, USBRX); // tx, rx for host terminal coms
char buf[0xFF]; // create a large buffer to store data
int main() {
pc.printf("Ethernet data read and display\n\r");
while (1) {
    int size = eth.receive(); // get size of incoming data packet
    if (size > 0) { // if packet received
        eth.read(buf, size); // read packet to data buffer
        pc.printf("size = %d data = ",size); // print to screen
        for (int i=0;i<size;i++) { // loop for each data byte
            pc.printf("%02X ",buf[i]); // print data to screen, %X hexadecimal
        }
        pc.printf("\n\r");
        }
    }
}
```
<figure markdown="span" align="center">
  ![MCU Architecture](images\ethernet2mbed.png){ width="500"}
  <figcaption>Ethernet MBED</figcaption>
</figure>

อันบนโค้ดส่งข้อมูล อันล่างโค้ดรับข้อมูล ส่วนรูปแสดงการต่อสาย จบ

## LAN on MBED

<figure markdown="span" align="center">
  ![MCU Architecture](images\lanmbed.png){ width="500"}
  <figcaption>MBED LAN Connection</figcaption>
</figure>

ขี้เกียจละบทนี้

## HTTP on MBED 
## WAN on MBED
# The IOT (Internet of things)