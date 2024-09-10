# Embeded System Midterm

## Chapter 1 : Embeded, MCU and ARM

**Embeded system** คือ Product Controlled by Computer for Example

- Vending Machine
- Washing Machine
- Segwa personal transporter

**MPU vs MCU**
MPU ไม่พูดถึงแต่ MCU คือ Computer + Control Functions ประกอบด้วย

- Core
- Memory
- Peripheral

**Embeded uses C or C++**
**ARM (Advance RISC Machine Ltd.)** เจ้านี้ขาย Processor IP แบบ RISC (Reduced Instruction Set Computer) 
มีข้อดีเรื่อง

- 1 binary word instruction
- instruction takes same amout of time
- Piplining

## Chapter 2 : Introducing to MBED
<figure markdown="span">
  ![Image title](images\mbed-lpc1768.jpg){ width="300" align=center}
  <figcaption>ไอบอร์ดเวร</figcaption>
</figure>

**MBED LPC1768** เป็น MCU ที่ต่ออุปกรณ์เสริมมาเรียบร้อยแล้ว (เรียกได้ว่าพร้อมใช้)
**MBED Board Architecture**

- LPC1768 MCU
- Signal Pins
- USB Interface MCU
- 16Mbit USB Disk (เอาง่ายๆ จะอัพโค้ดแบบปกติ หรือว่าจะเอา Compiled ใส่ USB Disk ก็ได้)

<figure markdown="span">
  ![MCU Architecture](images\LPC_Sturcture.png){ width="500"}
  <figcaption>โครงสร้างการทำงานภายใน แต่ไม่ต้องไปจำแม่งหรอกครับ</figcaption>
</figure>

```C
/* Program Example 2.1: Simple LED flashing */
#include "mbed.h"
DigitalOut myled(LED1);
int main() {
    while(1) {
        myled = 1;
        wait(0.2);
        myled = 0;
        wait(0.2);
    }
}
```

## Chapter 3 : Digital IO
ไอบอดเวรนี่มี 26 Pin ที่ใช้เป็น Input และ Output ได้ ก็คือตั้งแต่ Pin 5-30
<!-- 
Built-in
    LED
    Button
 -->
โค้ดมีตามนี้
```C
#include <mbed.h> // ก่อนใช้ก็ Include ก่อน
/** Digital IO */
DigitalOut myLED(LED1);
DigitalIn myButton(btn1);
BusIn busInput(P0,P1,P2); // ใช้หลาย Input Pin พร้อมกัน Up to 16 pins
// ตอนรับค่ารับเป็น int ที่แต่ละ bit represent แต่ละ pin
BusOut busOutput(P0,P1,P2) // ใช้หลาย Output Pin พร้อมกัน  Up to 16 pins
// ตอนสั่งจะสั่งเป็น int ที่แต่ละ bit represent แต่ละ pin
wait(s);
wait_ms(ms);
wait_us(us);
```

<figure markdown="span">
  ![MCU Architecture](images\VoltageLogic.png){ width="500"}
  <figcaption>Voltage as Logic values</figcaption>
</figure>
(GPIO ของชิพจริงไม่มี State Undefined นะครับ)

### **ว่าด้วยเรื่องของ LEDs** สรุปง่ายๆ คือ LED มันรับ**กระแส**ได้จำกัดซึ่งถ้าเราจ่ายแรงดันมากเกินไปอาจะทำให้กระแสไหลเกินจน LED ขาดได้
<figure markdown="span">
  ![MCU Architecture](images\LED.png){ width="500"}
  <figcaption>การจ่ายกระแสให้ LED</figcaption>
</figure>
มันมีอยู่ 2 แบบคือ 

1. Source จะจ่ายออกจากขา IO 
1. Sink จะดึงกระแสเข้ามาในขา IO

### **ว่าด้วยเรื่องการต่อ Switch Input**
<figure markdown="span">
  ![MCU Architecture](images\switchinput.png){ width="500"}
  <figcaption>Input แบบต่างๆ</figcaption>
</figure>

<figure markdown="span">
  ![MCU Architecture](images\opto.png){ width="500"}
  <figcaption>ใช้ Opto เป็น Input ก็ได้</figcaption>
</figure>

### 7-Segment
ตัวเลข ที่เราคุ้นเคย ใน example ใช้ `BusOut` แต่ต้องโน้ตไว้อย่างนึงว่า Ouput Pin แต่ละ Pin มีค่าความต้านทานภายใน 100Ω และ LED Segment มีแรงดันตกคร่อมประมาณ 1.8V เพราะงั้นเวลาคำนวนกระแสต้องคิดค่า R ภายในด้วยเช่นถ้า </br>
I = 5mA </br>
Vled = 1.8V </br>
Vpin = 3.3V </br>
Rinternal = 100Ω</br>
ดังนั้น R จะเท่ากับ ((3.3 - 1.8) / 5m) - 100 = ___Ω

```C
/*Program Example 3.7: Simple demonstration of 7-segment display. Display
digits 0, 1, 2, 3 in turn.
*/
#include "mbed.h"
BusOut display(p5,p6,p7,p8,p9,p10,p11,p12); // segments a,b,c,d,e,f,g,dp
int main() {
    while(1) {
        for(int i=0; i<4; i++) {
            switch (i){
                case 0: display = 0x3F; break; //display 0
                case 1: display = 0x06; break; //display 1
                case 2: display = 0x5B; break;
                case 3: display = 0x4F; break;
            } //end of switch
            wait(0.2);
        } //end of for
    } //end of while
}
```

### Control Larger Load
ใช้ BJT หรือ MOSFET
<figure markdown="span">
  ![MCU Architecture](images\largeLoad.png){ width="500"}
  <figcaption>Large Load Control</figcaption>
</figure>
แต่มีข้อระวังตอนขับโหลดที่เป็นแบบ Inductive ex. Motor เพราะว่าถ้าตัดไฟจาก Load ประเภท Inductive จะทำให้เกิดสิ่งที่เรียกว่า Back EMF ซึ่งจะทำให้เกิดแรงดันจำนวนมหาศาล ซึ่งอาจจะทำลาย Transistor หรือ MOSFET เราได้
<figure markdown="span">
  ![MCU Architecture](images\inductive_load.png){ width="500"}
  <figcaption>Flyback Diode</figcaption>
</figure>

## Chapter 4 : Analog Output

### DAC (Digital to Analog Converter)
Basicly convert **binary input** to **analog output**
<figure markdown="span">
  ![MCU Architecture](images\dac.png){ width="500"}
  <figcaption>DAC Block Diagram</figcaption>
</figure>

Output Voltage of DAC are determined by

$Vo = \dfrac{D * V_{ref}}{2^n}$ </br>
Vo : Output Voltage </br>
D : Digital Input </br>
Vref : Reference Voltage </br>
n : Number of bits </br>