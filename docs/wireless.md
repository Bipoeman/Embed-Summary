# Chapter 11 Wireless Communication - Bluetooth and ZigBee
# Wireless Preliminary 
มันมีสูตร $v=f\lambda$ ที่ใช้คำนวนได้ ละก็การใช้ความถี่มีคนคุมอยู่คือ Internation Telecommunication Union โดยจะมีความถี่ในหมวดหมู่ Industrial, Scientific and Medical (ISM) ที่คนทั่วไปสามารถใช้ได้

สัญญาณวิทยุจะมีประโยชน์ก็เมื่อเราสามารถส่งข้อมูลผ่านมันได้แหละครับ เราเรียกขั้นตอนการยัด้อมูลใส้สัญญาณวิทยุนี้ว่า modulation

ซึ่งไอการ Modulation ที่ว่าเนี่ยมันทำให้ความถี่มีการเปลี่ยนแปลงได้ เพราะงั้นมันจะมี Bandwidth ก็คือเป็น**ช่วง**ของความถี่ที่จะรับโดยเครื่องรับ เช่นถ้าเครื่องรับตั้งไว้ให้รับที่ 103MHz (ขึ้นอยู่กับวิธีการ Modulation)ก็จะรับสัญญาณที่ความถี่สูงและต่ำกว่าความถี่ที่ตั้งไว้นิดหน่อย มากน้อยแค่ไหนขึ้นอยู่กับว่า Bandwidth เท่าไหร่
# Wireless Network
- **Personal** Area Network (PAN) อุปกรณ์ที่อยู่ใกล้คน
- **Local** Area Network (LAN) อุปกรณ์ที่อยู่แค่ในอาคาร
- **Wide** Area Network (WAN) อุปกรณ์ที่อยู่ทั่วอ่ะ
# Protocols
International Organi**z**ation for Standardi**z**ation (ISO) ได้กำหนด Protocol for Protocols ไว้เป็น **Open Systems Interconnect** หรือ OSI model

<figure markdown="span" align="center">
  ![MCU Architecture](images\osi.svg){ width="500"}
  <figcaption>OSI Model</figcaption>
</figure>

### Protocol - IEEE Woring groups

IEEE คนกำหนดมาตรฐานให้กับ protocol ต่างๆเกี่ยวกับ network โดยจะขึ้นด้วย `802.`

|IEEE Woring Group|Description|
|---|---|
|802.3|Ethernet|
|802.11|Wireless LAN, including Wi-Fi|
|802.15|Wireless PAN|
|802.15.1|Bluetooth|
|802.15.3|High-rate wireless PAN|
|802.15.4|Low-rate wireless PAN, e.g. Zigbee|

# Bluetooth 
เป็น Digital Radio Protocol ใช้ใน Network แบบ PAN เพื่อใช้สำหรับเชื่อมต่อระหว่างอุปกรณ์กับอุปกรณ์
## Bluetooth Characterisics
|Class|Range|Power|
|-----|-----|-----|
|1|100 meters|100mW|
|2|10 meters|2.5mW|
|3|1 meters|1mW|

อุปกรณ์ BT แต่ละตัวมี Unique MAC Address (หรอวะ!!) โดยจะมี Phase การทำงานแบบนี้
1. Discovery ก็คือ Broadcast ข้อมูลตัวเอง ทั้งชื่อและ MAC Address
1. Pairing ก็คือทั้ง Slave และ Master แลกเปลี่ยนข้อมูลว่าการเชื่อมต่อจะเอาไง
1. Connecting ก็คือ Master เป็นคนเปิดจาก Link ที่ได้ Pair กันไว้แล้ว

## RN-41 Bluetooth Module with MBED
ใช้โมดูล RN-41 ซึ่งเป็น Class 1 ด้วยและก็ต่อกับ Bluetooth ของ โน้ตบุ๊ตได้เลย ส่วนต่อกับ MBED ผ่าน UART Serial เพราะงั้นวิธีใช้ไม่ต่างอะไรกับ Serial เลย
<figure markdown="span" align="center">
  ![MCU Architecture](images\rn41.png){ width="500"}
  <figcaption>Bluetooth Module Connection</figcaption>
</figure>

# ZigBee
ใช้ Standard **IEEE 802.15.4** ตัวนี้คล้าย Bluetooth ก็คือมีการ Pair กันก่อนเหมือนกันแต่ว่ากินไฟน้อยกว่า ส่งข้อมูลได้น้อยกว่า(มากๆ) และกินไฟน้อย(น้อยชิบหาย) เป้าหมายเขาคือถูกว่า (หราา แพงสัสเลย) ง่ายกว่าและมี Overhead ของ Software น้อยกว่า

## Zigbee Device Types
1. End Device เป็นอุปกรณ์ปลายทาง จะคุยกับแค่ Parent (ที่เป็น Router หรือ Coordinator) ที่ต่ออยู่เท่านั้น
2. Router ใช้ route traffic ใช้ส่งข้อมูลระหว่าง End Device กับ Coordinator ได้
3. Coordinator เป็นคนคุมเครือข่าย กำหนด Characteristic ของ Network ด้วย **ไม่มีไม่ได้**

## XBee Wireless Modules
ไอบอร์ดพวกนี้เป็นของ **Digi International**
<figure markdown="span" align="center">
  ![MCU Architecture](images\xbee.png){ width="500"}
  <figcaption>XBee Module</figcaption>
</figure>

เนื่องจากคอมไม่มี Zigbee ก็ต้องใช้ตัวแปลงอันขวาช่วย เพื่อต่อ Module Zigbee เข้าคอมและจะใช้โปรแกรม XCTU เพื่อ Config XBee Module
<figure markdown="span" align="center">
  ![MCU Architecture](images\xctu.png){ width="500"}
  <figcaption>XBee Module</figcaption>
</figure>

## Implementing a Zigbee Link mbed to mbed

<figure markdown="span" align="center">
  ![MCU Architecture](images\xbeembed.png){ width="500"}
  <figcaption>XBee Module with MBED</figcaption>
</figure>

ต่อเหมือน Bluetooth ใช้หมือน Bluetooth เลย (ถ้าใช้กับคอม)

ที่เหลือขออนุญาต ช่างแม่ม