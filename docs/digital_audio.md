# Chapter 13 Working with Digital Audio
# Introduction
เสียงที่เป็นสัญญาณ Analog สามมารถเอาสัญญาณเข้าและออกจากระบบ Digital ได้ผ่าน Audio interface ที่เป็น **DACs** และ **ADCs** โดยจะถูกแปลงเป็นสัญญาณ Digital และสามารถถูกเปลี่ยนแปลงหรือปรับแต่งได้ด้วย DSP (Digital Signal Processing) และแน่นอน การลดขนาดไฟล์ก็เป็นส่วนหนึ่งของ DSP เช่นกัน เช่นการบีบอัดไฟล์เสียง
# MIDI
MIDI หรือ Musical Instrument Digital Interface คือโปรโตคอลที่เครื่องดนตรี Digital คุยกันโดยจะมีข้อมูลสำคัญคือ
- Note On or Note Off
- Pitch of the Note

เราสามารถใช้ MBED เพื่อต่อและส่งข้อมูลด้วย Protocol MIDI ผ่าน USB ได้ด้วย Library `USBMIDI`
```C
/* Program Example 13.1: MIDI messaging with variable scroll speed
*/
#include "mbed.h"
#include "USBMIDI.h"
USBMIDI midi; // initialise MIDI interface
AnalogIn Ain(p19); // create analog input (potentiometer)
int main() {
    while (1) {
        for(int i=48; i<72; i++) { // step through notes
            midi.write(MIDIMessage::NoteOn(i)); // note on
            wait(Ain); // pause
            midi.write(MIDIMessage::NoteOff(i)); // note off
            wait(2*Ain); // pause
        }
    }
}
```

# Digital Audio Processing
- DSP คือการประมวลผลสัญญาณด้วยวิธีแบบ Digital
- ชิพ DSP เร็วเรื่องการ shift-and-add และก็ multiply-and-add มากเพราะเป็นพื้นฐานของการประมวลสัญญาณ
- นอกจากนั้นยังชิพมี Instruction ที่เอาไว้สำหรับ Filter และ Frequency Analysis ได้ด้วยเพื่อให้ทำงานได้เร็วและเขียนโปรแกรมได้ง่าย
- จริงๆใช้ Microcontroller ก็ได้แต่จะช้ากว่าในเรื่องของเวลาการทำงาน

## DSP on MBED
MBED สามารถทำ DSP ได้โดยรับสัญญาณเข้าผ่าน ADC ประมวลผลและส่งสัญญาณออกผ่านทาง DAC (สัญญาณเสียงอ่ะนะ) เรื่องก็คือสญญาณเสียงเป็นสัญญาณ AC ที่มีทั้งบวกและลบแต่ MBED อ่านสัญญาณได้แค่ฝั่งบวกเท่านั้น ก็เลยต้องใส่ Offset ของสัญญาณให้มันนิดนึง

โดยจะใส่วงจรนี้เพื่อเพิ่ม Offset ให้กับสัญญาณด้วยวงจรนี้

<figure markdown="span" align="center">
  ![MCU Architecture](images\offset-signal-circuit.png){ width="500"}
  <figcaption>วงจรใส่ Offset ให้กับสัญญาณ</figcaption>
</figure>


<figure markdown="span" align="center">
  ![MCU Architecture](images\ac-signal.svg){ width="500"}
  <figcaption>สัญญาณเสียงปกติ</figcaption>
</figure>

<figure markdown="span" align="center">
  ![MCU Architecture](images\offset-signal.svg){ width="500"}
  <figcaption>สัญญาณเสียงที่ใส่ Offset แล้ว</figcaption>
</figure>

```c
/* Program Example 13.5 DSP input and output using a 20 kHz ticker object
*/
#include "mbed.h"
//mbed objects
AnalogIn Ain(p15);
AnalogOut Aout(p18);
Ticker s20khz_tick;
//function prototypes
void s20khz_task(void);
//variables and data
float data_in, data_out;
//main program start here
int main() {
    s20khz_tick.attach_us(&s20khz_task,50); // attach task to 50us tick
}
// function 20khz_task
void s20khz_task(void){
    data_in=Ain;
    data_out=data_in;
    Aout=data_out;
}
```

## Signal Reconstuction
DAC ไม่สามารถ Output ให้หน้าตาเหมือนกับ Input ได้เนื่องจากมีความละเอียดที่จำกัด

<figure markdown="span" align="center">
  ![MCU Architecture](images\reconstruction.png){ width="500"}
  <figcaption>สัญญาณที่ออกจาก DAC</figcaption>
</figure>

เส้นบนสุดคือสัญญาณ Input, เส้นกลางคือ Output จาก DAC

<figure markdown="span" align="center">
  ![MCU Architecture](images\reconstruction-filter.png){ width="500"}
  <figcaption>Reconstruction Filter</figcaption>
</figure>

สังเกตว่าเส้นล่างคือสัญญาณที่ออกจาก DAC มันเป็นขั้นบันได้ เพราะงั้นต้องเอาไปใส่ Reconstruction Filter เพื่อให้หน้าตาสัญญาณเหมือน input มากขึ้น

<figure markdown="span" align="center">
  ![MCU Architecture](images\mbed-dsp.png){ width="800"}
  <figcaption>Circuit of DSP with MBED</figcaption>
</figure>

## Digital Filtering Example
เอาง่ายๆ หลักๆถ้ามีสัญญาณที่ผสมกันหลายๆความถี่ ถ้าอยากได้สัญญาณความถี่สูงก็ใช้ High Pass แต่ถ้าอยากได้สัญญาณเฉพาะความถี่ต่ำก็ใช้ Low Pass Filter

<figure markdown="span" align="center">
  ![MCU Architecture](images\low-high-pass.png){ width="800"}
  <figcaption>Low pass and High Pass filter</figcaption>
</figure>

## Digital Filters
ไม่สนเรื่องสูดรละกัน แต่จะพูดถึง IIR และ FIR Filter
- FIR (Finite Impulse Response) ไม่มี Feedback แต่มีข้อดีเรื่อง Phase Shift
- IIR (Infinite Impulse Response) ใช้ Feedback จาก Output กลับมาคำนวนใหม่

<figure markdown="span" align="center">
  ![MCU Architecture](images\fir-irr.png){ width="800"}
  <figcaption>FIR IIR Filter</figcaption>
</figure>

# Working with ave audio files
นามสกุลไฟล์เสียงยอดนิยมคือ `.wav` โดยจะมี Header เพื่อบอกข้อมูลของไฟล์เช่น Bit rate, Sample Frequency ที่ต้องมี Header เพื่อเครื่องอ่านจะได้เล่นถูก
หน้าตามันก็จะประมาณนี้

<figure markdown="span" align="center">
  ![MCU Architecture](images\wav-header.png){ width="800"}
  <figcaption>Wav File Header Content</figcaption>
</figure>

ข้อมูลเป็น **Last Significant Byte First** เช่น Sample Rate ที่มีค่า Hex เป็น `80 3E 00 00` ให้อ่านจากขวาไปซ้าย ก็จะกลายเป็น `00 00 3E 80` ซึ่งแปลงเป็นเลขฐาน 10 ก็จะได้ 16000 นั่นเอง

```c
/* Program Example 13.8 Wave file header reader (wav data stored on SD card)
*/
#include "mbed.h"
#include "SDFileSystem.h"
SDFileSystem sd(p5, p6, p7, p8, "sd");
Serial pc(USBTX,USBRX); // set up terminal link
char c1, c2, c3, c4; // chars for reading data in
int AudioFormat, NumChannels, SampleRate, BitsPerSample ;
int main() {
    pc.printf("\n\rWave file header reader\n\r"); // binary file in read mode
    FILE *fp = fopen("/sd/sinewave.wav", "rb"); // open SD card wav file
    fseek(fp, 20, SEEK_SET); // set pointer to byte 20
    fread(&AudioFormat, 2, 1, fp); // check file is PCM
    if (AudioFormat==0x01) {
        pc.printf("Wav file is PCM data\n\r");
    }
    else {
        pc.printf("Wav file is not PCM data\n\r");
    }
    fread(&NumChannels, 2, 1, fp); // find number of channels
    pc.printf("Number of channels: %d\n\r",NumChannels);
    fread(&SampleRate, 4, 1, fp); // find sample rate
    pc.printf("Sample rate: %d\n\r",SampleRate);
    fread(&BitsPerSample, 2, 1, fp); // find resolution
    pc.printf("Bits per sample: %d\n\r",BitsPerSample);
    fclose(fp);
}
```
ในต้วอย่างนี้จะเช็คไฟล์ใน SD Card ว่าเป็น File PCB มั้ย ถ้าเป็นก็บอก ไม่เป็นก็บอก ทีนี้ก็อ่านและก็เล่นออกมาผ่าน DAC

## Circular Buffer
ก็เขียนข้อมูลใส่ Buffer ไปเรื่อยๆจนจบ Array แล้วกลับมาเริ่มต้นใหม่ เอาไว้ควบคุม Timing ระหว่างการอ่านและส่งข้อมูลของข้อมูลแต่ละ Byte

<figure markdown="span" align="center">
  ![MCU Architecture](images\circular_buffer.png){ width="800"}
  <figcaption>Circular Buffer</figcaption>
</figure>

```c
/* Program Example 13.9: 16-bit mono wave player
*/
#include "mbed.h"
#include "SDFileSystem.h"
#define BUFFERSIZE 4096 // number of data in circular buffer
SDFileSystem sd(p5, p6, p7, p8, "sd");
AnalogOut DACout(p18);
Ticker SampleTicker;
int SampleRate;
float SamplePeriod; // sample period in microseconds
int CircularBuffer[BUFFERSIZE]; // circular buffer array
int ReadPointer=0;
int WritePointer=0;
bool EndOfFileFlag=0;
void DACFunction(void); // function prototype
int main() {
    FILE *fp = fopen("/sd/testa.wav", "rb"); // open wave file
    fseek(fp, 24, SEEK_SET); // move to byte 24
    fread(&SampleRate, 4, 1, fp); // get sample frequency
    SamplePeriod=(float)1/SampleRate; // calculate sample period as float
    SampleTicker.attach(&DACFunction,SamplePeriod); // start output tick
    while (!feof(fp)) { // loop until end of file is encountered
        fread(&CircularBuffer[WritePointer], 2, 1, fp);
        WritePointer=WritePointer+1; // increment Write Pointer
        if (WritePointer>=BUFFERSIZE) { // if end of circular buffer
            WritePointer=0; // go back to start of buffer
        }
    }
    EndOfFileFlag=1;
    fclose(fp);
}

// Program Example 13.9 Continued
// DAC function called at rate SamplePeriod
void DACFunction(void) {
    if ((EndOfFileFlag==0)|(ReadPointer>0)) { // output while data available
        DACout.write_u16(CircularBuffer[ReadPointer]); // output to DAC
        ReadPointer=ReadPointer+1; // increment pointer
        if (ReadPointer>=BUFFERSIZE) {
          ReadPointer=0; // reset pointer if necessary
        }
    }
}
```
อันนี้คือตัวอย่างของ Circular Buffer ที่อ่านจาก SD Card มาใส่ ตัวแปรเป็นแบบ Circular แล้วส่งออกทาง DAC

<figure markdown="span" align="center">
  ![MCU Architecture](images\2complimentdac.png){ width="800"}
  <figcaption>2's Compliment DAC Data</figcaption>
</figure>

## High-fidelity Audio
เอาง่ายๆ ADC กับ DAC ของบอดมันกากครับ เพราะงั้นเลยต้องหาตัวช่วยถ้าอยากให้ Sample เสียงคุณภาพดีขึ้นเช่นตัวนี้ TLV320AIC23B เป็นทั้ง ADC และ DAC (ก็คือ CODEC อ่ะแหละ) ความละเอียดสูงสุด 16, 20, 24 หรือ 32 bit และก็ Sample Rate สูงสุด 96kHz
<figure markdown="span" align="center">
  ![MCU Architecture](images\TLV320AIC23B.png){ width="800"}
  <figcaption>TLV320AIC23B ADC DAC</figcaption>
</figure>

และแน่นอน จะส่งสัญญาณเสียงไปให้มันต้องผ่านโปรโตรคอล I2S (I Square S) มันจะมีสายหลักๆอยู่  3 เส้นก็คือ

|Pin|Name|Function|
|---|--------|--|
|WS |Word Select|เอาไว้เลือกซ้ายขวา 0 = Left|
|SCK |Data Clock|เอาไว้ Sync สัญญาณ|
|SD |Serial Data|สัญญาณ|

ที่เหลือต่อใน Slide