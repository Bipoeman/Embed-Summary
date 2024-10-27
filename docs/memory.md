# Chapter 10 Memory and Data Management 
# Memory
## Memory Function Types
<p>Memory ใน microcontroller มีอยู่ 2 หน้าที่หลักๆก็คือใช้เก็บโปรแกรม ก็คือ Program Memory ไว้เก็บโปรแกรม และ Data Memory ไว้เก็บข้อมูลตอนทำงาน</p>
<p>Electronic Memory จะมุ่งเก็บข้อมูลแค่ 0 กับ 1 (ซึ่งมมันก็ Stable แค่ 2 state เราเลยเรียกมันว่า <b>Bistable</b>) เหมือนเหรียญมีหัวกับก้อยอ่ะ กี่ bit ก็คือเหมือนกับมีกี่เหรียญ</p>
<p>แต่เราไม่เก็บข้อมูลในเหรีญเนอะ เพราะงั้นถ้าเก็บในรูปแบบ Electronic ก็ใช้ <b>Flip-Flop</b> แทน</p>

<figure markdown="span" align="center">
  ![MCU Architecture](images\flipflop.png){ width="500"}
  <figcaption>1 bit Flip Flop ที่ไม่ใช่รองเท้า</figcaption>
</figure>

## Electronic Memory Types
แบ่งเป็น 2 ประเภท

- **Volatile** Memory เป็น Memory แบบที่ข้อมูลหายเวลาไฟตัดซึ่งจะถูกใช้เป็น Data Memory หรือ RAM แบ่งได้อีก
    - DRAM
    - SRAM
- **Non-Volatile** Memory อันนี้ข้อมูลไม่หายเวลาไฟตัด เอาไว้เก็บโปรแกรมหรือเป็น Program Memory ซึ่งสมุยก่อนเรารู้จักกันในชื่อ ROM (Read only memory) แบ่งได้อีก
    - ROM
    - PROM
    - EPROM
    - EEPROM
    - FLASH
# Pointer
## Introduction
<p>เข้าใจง่ายๆ ในภาษา C Pointer ใช้เข้าถึง Address ของตัวแปรครับ (เข้าถึงที่ว่านี่คือทั้งรู้ Address และอ่านค่าของข้อมูลได้ด้วยนะ) </p>
และแน่นอน Array ก็เป็น Pointer

ถ้าเขียนแบบนี้
```c
int data[10] = {1,2,3,4,5,6,7,8,9,10};
int main(void){
    printf("%d",data);
    return 0;
}
```
ถ้า `printf("%d",data);` ออกมาจะไม่ได้ค่าทั้ง Array เหมือนใน JS หรือ Python ที่เราคุ้นเคย แต่จะได้ Address ของข้อมูลตัวแรกออกมา

## Defining Pointer
นอกจาก Array เมื่อกี้แล้วยังสามารถ Define Pointer ได้แบบนี้ โดยใส่ `*` ไว้หน้าตัวชื่อแปร
```c
int *ptr; // define a pointer which points to data of type int
```
และแน่นอน เมื่อบอกว่า Pointer ใช้สำรับเก็บ Address ของข้อมูลก็สามารถ Assign Address ของข้อมูลนั้นได้แบบนี้
สังเกตว่าเอา address ของตัวแปรได้จากการใส่ `&` ไว้หน้าตัวแปร
```c
int datavariable=7; // define a variable called datavariable with value 7
int *ptr; // define a pointer which points to data of type int
ptr = &datavariable;// assign the pointer to the address of datavariable
```
หรือจะเป็น Array แบบเมื่อกี้ (กรณีนี้ใช้ Address ของ Element Array index ที่ 0)
```c
int array[]={3,4,6,2,8,9,1,4,6}; // define an array of arbitrary values
int *ptr // define a pointer
ptr = &array[0]; // assign pointer to the address of the first element of the array
```
## Pointers with array and functions
```c
/* Program Example 10.5: Pointers example for an array average function
*/
#include "mbed.h"
Serial pc(USBTX, USBRX); // setup serial comms
char data[]={5,7,5,8,9,1,7,8,2,5,1,4,6,2,1}; // define some input data
char *dataptr; // define a pointer for the input data
float average; // floating point average variable
float CalculateAverage(char *ptr, char size); // function prototype
int main() {
    dataptr=&data[0]; // point to address of the first array element
    average = CalculateAverage(dataptr, sizeof(data)); // call function
    pc.printf("\n\rdata = ");
    for (char i=0; i<sizeof(data); i++) { // loop for each data value
        pc.printf("%d ",data[i]); // display all the data values
    }
    pc.printf("\n\raverage = %.3f",average); // display average value
}
// CalculateAverage function definition and code
float CalculateAverage(char *ptr, char size) {
    int sum=0; // variable for calculating the sum of the data
    float mean; // variable for floating point mean value
    for (char i=0; i<size; i++) {
        sum=sum + *(ptr+i); // add all data elements together
    }
    mean=(float)sum/size; // divide by size and cast to floating point
    return mean;
}
```
เอาง่ายๆไอโค้ดนี้สร้าง function `CalculateAverage` โดยรับ Address ของข้อมูลตัวแรกของ array พร้อมกับ size ของ Array ที่ต้องการคำนวน (ใช่ครับ ใส่ Array เข้าฟังชันต้องใส่ขนาดด้วย) และในตัวอย่างนี้ยังได้เห็นการเอาข้อมูลออกมาจาก Address นั้น หรือการ Dereferencing โดยการใส่ `*` ไว้หน้าตัวแปร pointer
# Files
```c
FILE *fopen(const char *filename, const char*mode); // เปิดไปล์
int fclose(FILE *stream);   // ปิดไปล์
int fgetc(FILE *stream);    // อ่าน Character 1 ตัวจากไฟล์
char *fgets(char *str, int n, FILE *stream);    // อ่าน String จากไฟล์
int fputc(int character, FILE *stream); // เขียน Character 1 ตัวลงไฟล์
int fputs(const char *str, FILE *stream);    // เขียน String ลงไฟล์
int fprintf(FILE *stream, const char *format,...);   // เขียน Formatted String ลงไฟล์
int fseek(FILE *stream, long int offset, int origin);   // เลื่อนตำแหน่งที่จะอ่านไฟล์
```
## Using data files on the mbed
ถ้าจะใช้ File ใน MBED สามารถใช้ `LocalFileSystem` ได้ ใช้งี้
```c
LocalFileSystem local("local"); //Create file system named "local"
```
ทีนี้ก็ใช้ฟังชันของไฟล์ข้างบนได้ตามปกติ 
```c
FILE* pFile = fopen("/local/datafile.txt","w"); // note the “w” refers to a file with write access
```
พอใช้เสร็จก็ปิด
```c
fclose(pFile);
```
ตัวอย่าง
```c
/* Program Example 10.1: read and write char data bytes
*/
#include "mbed.h"
Serial pc(USBTX,USBRX); // setup terminal link
LocalFileSystem local("local"); // define local file system
int write_var;
int read_var; // create data variables
int main ()
{
    FILE* File1 = fopen("/local/datafile.txt","w"); // open file
    write_var=0x23; // example data
    fputc(write_var, File1); // put char (data value) into file
    fclose(File1); // close file
    FILE* File2 = fopen ("/local/datafile.txt","r"); // open file for reading
    read_var = fgetc(File2); // read first data value
    fclose(File2); // close file
    pc.printf("input value = %i \n",read_var); // display read data value
}
```
อันนี้เปิดไฟล์และก็เขียน character 1 ตัวลงไป
```c
/* Program Example 10.2: Read and write text string data
*/
#include "mbed.h"
Serial pc(USBTX,USBRX); // setup terminal link
LocalFileSystem local("local"); // define local file system
char write_string[64]; // character array up to 64 characters
char read_string[64]; // create character arrays (strings)

int main ()
{
    FILE* File1 = fopen("/local/textfile.txt","w"); // open file access
    fputs("lots and lots of words and letters", File1);// put text into file
    fclose(File1); // close file
    FILE* File2 = fopen ("/local/textfile.txt","r"); // open file for reading
    fgets(read_string,256,File2); // read text into variable
    fclose(File2); // close file
    pc.printf("text data: %s \n",read_string); // display read data string
}
```
อันนี้เปิดไฟล์และก็เขียน string กี่ตัวก็ไม่รู้ลงไป

## Using external memory with the mbed
MBED ต่อ Storage ภายนอกได้นะ ต่อได้ 2 แบบคือ SD Card หรือ USB Flashdrive
ถ้าจะใช้ SD Card ต่อแบบนี้

<figure markdown="span" align="center">
  ![MCU Architecture](images\sdcard.png){ width="500"}
  <figcaption>SD Card connection table</figcaption>
</figure>

```c
/* Program Example 10.4: writing data to an SD card */
#include "mbed.h"
#include "SDFileSystem.h"
SDFileSystem sd(p5, p6, p7, p8, "sd"); // MOSI, MISO, SCLK, CS
Serial pc(USBTX, USBRX);
int main() {
    FILE *File = fopen("/sd/sdfile.txt", "w"); // open file
    if(File == NULL) { // check for file pointer
        pc.printf("Could not open file for write\n"); // error if no pointer
    }
    else{
        pc.printf("SD card file successfully opened\n"); // if pointer ok
    }
    fprintf(File, "Here's some sample text on the SD card"); // write data
    fclose(File); // close file
}
```

ไม่ก็ใช้ External USB Flash Drive

MBED บอดนี้สามารถทำตัวเป็น USB Host ได้ (เพราะ USB Builtin ที่ขา 31,32) โดย Library `USBHostMSD`

```c

/* Program Example 10.5: writing data to an USB flash storage device
*/
#include "mbed.h"
#include "USBHostMSD.h"
int main() {
    USBHostMSD usb("usb"); // define USBHostMSD object
    while(!usb.connect()) { // try to connect a USB storage before continuing
        wait(0.5);
        printf("Connecting to USB MSD\n");
    }
    FILE *File = fopen("/usb/usbfile.txt", "w"); // open file
    if(File == NULL) { // check for file pointer
        intf("Could not open file for write\n"); // error if no pointer
    }
    else{
    p   tf("USB card file successfully opened\n"); // if pointer ok
    }
    fprintf(File, "Here's some sample text on the USB card"); // write data
    fclose(File); // close file
}

```

