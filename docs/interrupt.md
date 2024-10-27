# Chapter 9 Interrupts Timers and Tasks
>ปํญหาไม่ได้อยู่ที่ปัญหา <br/>
>แต่อยู่ที่ทัศนคติของ**มึง**ในการเข้าถึงปัญหา
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- <cite>รุกกี้</cite>

## Timer and Interrupts
#### MCU ต้องทำพวกนี้ได้
- **วัดเวลา**
- สร้าง **time-based event** (อาจจะเกิดครั้งเดียว หรือหลายครั้ง)
- **ตอบสนอง**ด้วยความรวดเร็ว

## Tasks : Event-Triggered and Time-Triggered
#### Event-Triggered
เกิดเมื่อมี event ภายนอกมากระตุ้น
#### Time-Triggered
เกิดแบบ periodic ซึ่งถูกกำหนดโดย MCU

## Polling
เอาง่ายๆ ก็คือต้องคอยไปอ่านสถานะของอุปกรณ์อยู่เรื่อยๆ

<figure markdown="span" align="center">
  ![MCU Architecture](images\polling.png){ width="400"}
  <figcaption>Polling Diagram</figcaption>
</figure>

#### ทำไมเป็นปัญหา
###### 1. ระหว่าง Poll ก็ทำอะไรไม่ได้
###### 2. ความสำคัญของแต่ละอันเท่ากัน

## Interrupt
เอาง่ายๆคือ interrupt สามารถ**หยุด CPU** ได้เมื่อเกิด event

<figure markdown="span" align="center">
  ![Interrupt Flowchart](images\interrupt_diagram.png){ width="400"}
  <figcaption>Interrupt Flowchart</figcaption>
</figure>

## Simple Interrupt on the mbed
บอร์ด mbed ใช้ pin ได้ตั้งแต่ 5 - 30 เป็น Interrupt ยกเว้น 19,20 

|Function|Usage|
|---|---|
|`InterruptIn(pin)`|สร้าง Object ของ Interrupt|
|`rise(ISR)`|ให้ Interrupt trigger ที่ขอบขาขี้น พร้อม Attach Interrupt Service Routine (ISR) (ก็คือฟังชั่นที่จะให้มันทำตอนเกิด event นั่นแหละ)|
|`fall(ISR)`|เหมือน `rise` แต่เป็นสัญญาณขาลง|
|`mode`|คือใช้ทำไรไม่เข้าใจ|

#### ตัวอย่างโค้ด
```c
#include "mbed.h"
InterruptIn button(p5); //สร้าง object + กำหนด pin
DigitalOut led(LED1);
DigitalOut flash(LED4);
void ISR1() { // ฟังชันที่ทำตอนเกิด interrupt
    led = !led;
}
int main() {
    button.rise(&ISR1); // attach the address of the ISR
    //function to the interrupt rising edge
    while(1) { //continuous loop, ready to be interrupted
        flash = !flash;
        wait(0.25);
    }
}
```

## Deeper into interrupt (ข้อดี Interrupt)
- **Prioritized** บาง event สำคัญกว่าอันอื่นๆได้
- **Masked** เปิดปิดได้ เผื่อไปรบกวนอันที่สำคัญกว่า
- **Nested** ทำๆอยู่โดนแทรกจากอันที่สพคัญกว่าได้
- **Location can be selected** เลือกได้ว่าจะเอาไปไว้ใน memory ส่วนไหน

Delay between event occur and response is called **latancy**
ถ้า Interrupt รอ processor ตอบ เขาเรียกว่า **Pending**

<figure markdown="span" align="center">
  ![Interrupt Flowchart](images\interrupt_response.png){ height="400"}
  <figcaption>Interrupt Response in more detail</figcaption>
</figure>

## Testing interrupt latency
เอาง่ายๆโปรแกรมนี้จะวัดว่า Latency เยอะมั้ยโดยวัดว่า กว่า LED จะเป็น 1 ห่างจาก input squarewave เยอะมั้ย ในขณะที่กระพริบ LED ใน int main() 
```c
#include "mbed.h"
InterruptIn squarewave(p5); //Connect input square wave here
DigitalOut led(p6);
DigitalOut flash(LED4);
void pulse() { //ISR sets external led high for fixed duration
    led = 1;
    wait(0.01);
    led = 0;
}
int main() {
    squarewave.rise(&pulse); // attach the address of the pulse function to
    // the rising edge
    while(1) { // interrupt will occur within this endless loop
        flash = !flash;
        wait(0.25);
    }
}
```

## Interrupt from analog Inputs
ใช้ Comparator ครับสัญญาณขา $V_+$ เกิน $V_-$ Output ก็เป็น High ครับจบ😜

$V_- = V_{sup} * \frac{R_2}{R_1+R_2}$
<figure markdown="span" align="center">
  ![Interrupt Flowchart](images\analog_interrupt.png){ height="300"}
  <figcaption>Interrupt with analog</figcaption>
</figure>

## The Digital Counter
ใช้ Flip-Flop เป็น Counter โดยนับได้เยอะแค่ไหนก็ขึ้นกับจำนวนบิตเช่น 8 bit ก็จะนับได้ตั้งแต่ 0-255 ($0-(2^8-1)$) และก็ถ้าต่อสัญญาณ Input เข้าไปที่ Clock มันก็จะนับ Clock ออกมาเป็น binary

สามารถอ่านได้, Preload ได้, จะ Reset เป็น 0 ก็ได้

## Counting and Timing
ถ้า Clock Source ของ Counter มีความถี่คงที่ Counter นั้นก็จะกลายเป็น Timer
Ex. ถ้าจะให้เข้าใจง่ายๆ ถ้ามี Clock ความถี่ 1MHz (อันบน) ทุก Clock Cycle ที่ผ่านไปจะทำให้ค่าใน Counter เพิ่มไป 1 ซึ่งไอค่า Counter เนี้ยสามารถใช้เป็นตัวนับเวลาได้ โดย Counter แต่ละเลขก็จะมีระยะเวลา 1uS (1/1MHz) เช่น ถ้า Counter มีค่าเท่ากับ 2000 ก็แปลว่ามีการนับมาแล้ว 2000uS

<figure markdown="span" align="center">
  ![Interrupt Flowchart](images\counter.png){ height="400"}
  <figcaption>ดูทรงละรูปไม่ค่อยเกี่ยวเท่าไหร่</figcaption>
</figure>

และพอ Counter มันนับไปจนสุดค่าของมัน (aka Overflow) ก็จะทำให้เกิด Interrupt ขึ้นแล้วก็จะ Reset ค่ากลับเป็น 0

คุณสมบัตินี้ใช้เอามากำหนด event ที่ต้องเกิดแบบ synchronize กันได้

<figure markdown="span" align="center">
  ![Interrupt Flowchart](images\timer_interrupt.png){ height="300"}
  <figcaption>Timer Interrupt Event</figcaption>
</figure>

### MBED Timer
###### MBED มี Timers
- Timer General Purpose 4 ตัว
- Repetitive Interrupt Timer
- System Tick Timer

`Timer` ใช้กับ Simple Timing application <br/>
`Timeout` ใช้เรียกตอนหมดเวลาที่กำหนดไว้ (แบบ non-blocking) <br/>
`Ticker` ใช้เรียกฟังชั่นทุกครั้งที่ถึงเวลาที่กำหนด

###### Timer ทำได้ดังนี้
|Function|Usage|
|---|---|
|`start`|Start Timer|
|`stop`|Stop Timer|
|`reset`|Reset timer to 0|
|`read`|get passed time in seconds|
|`read_ms`|get passed time in milliseconds|
|`read_us`|get passed time in microseconds|

###### Simple timer application
โค้ดนี้ก็ Start Timer -> ปริ้น Hello World และ Stop Timer <br/>
เพื่อวัดเวลาในการส่งข้อมูล(มั้ง)
```c
#include "mbed.h"
Timer t; // define Timer with name “t”
Serial pc(USBTX, USBRX);
int main() {
    t.start(); //start the timer
    pc.printf("Hello World!\n");
    t.stop(); //stop the timer
    //print to pc
    pc.printf("The time taken was %f seconds\n", t.read());
}
```

###### mbed `Timeout`
ฟังชันจะถูกเรียกใช้ตามเวลาที่กำหนด

|Function|Usage|
|---|---|
|`attach(ISR,seconds)`|Attach function และจำนวนวินาที ที่จะเรียกตอนเกิด timeout|
|`attach(ISR,seconds)`|Attach member?? function และจำนวนวินาที ที่จะเรียกตอนเกิด timeout|
|`attach_us(ISR,seconds)`|Attach function และจำนวน microseconds ที่จะเรียกตอนเกิด timeout|
|`attach_us(ISR,seconds)`|Attach member?? function และจำนวน microseconds ที่จะเรียกตอนเกิด timeout|
|`detach(ISR)`|Detach the function|

```c
#include "mbed.h"
Timeout Response; //create a Timeout, and name it "Response"
DigitalIn button (p5);
DigitalOut led1(LED1); //blinks in time with main while(1) loop
DigitalOut led2(LED2); //set high fixed period after button press
DigitalOut led3(LED3); //goes high when button is pressed
void blink() { //this function is called at the end of the Timeout
    led2 = 1;
    wait(0.5);
    led2=0;
}
int main() {
    while(1) {
        if(button==1){
            Response.attach(&blink,2.0); //attach blink function to Response
            //Timeout, to occur after 2 seconds
            led3=1; //shows button has been pressed
        }
        else {
            led3=0;
        }
        led1=!led1;
        wait(0.2);
    }
}
```

###### mbed `Ticker`
ฟังชันจะถูกเรียกใช้ตามเวลาที่กำหนด

|Function|Usage|
|---|---|
|`attach(ISR,seconds)`|Attach function และจำนวนวินาที ที่จะให้ ticker เมื่อถึงเวลา|
|`attach(ISR,seconds)`|Attach member?? function และจำนวนวินาที ที่จะให้ ticker เมื่อถึงเวลา|
|`attach_us(ISR,seconds)`|Attach function และจำนวน microseconds ที่จะให้ ticker เมื่อถึงเวลา|
|`attach_us(ISR,seconds)`|Attach member?? function และจำนวน microseconds ที่จะให้ ticker เมื่อถึงเวลา|
|`detach(ISR)`|Detach the function|

```c
#include "mbed.h"
void led_switch(void); // Prototype function
Ticker time_up; //define a Ticker, with name “time_up”
DigitalOut myled(LED1);

void led_switch(){ //the function that Ticker will call
    myled=!myled;
}
int main(){
    time_up.attach(&led_switch, 0.2); //initialises the ticker
    while(1){ //sit in a loop doing nothing, waiting for

        //Ticker interrupt

    }
}
```

###### Real Time Clock
RTC (Real time clock) เอาง่ายๆ คือนาฬิกาที่กินไฟน้อยมาก <br/>
ใช้ Clock ความถี่ 32kHz

|Function|Usage|
|---|---|
|`Time`|Get Current Time|
|`set_time`|Set Current Time|
|`mktime`|converts converts the time structure into a calender time value|
|`localtime`|Converts a timestamp to a tm structure|
|`ctime`|Converts a timestamp to a human-readable string|
|`strftime`|Converts a tm structure to a custom format human-readable string|

```c
#include "mbed.h"
InterruptIn button(p18); // Interrupt on digital pushbutton input p18
DigitalOut led1(LED1); // digital out to LED1
Timer debounce; // define debounce timer
void toggle(void); // function prototype

int main() {
    debounce.start();
    button.rise(&toggle); // attach the address of the toggle
} // function to the rising edge
void toggle() {
    if (debounce.read_ms()>10) // only allow toggle if debounce timer
    led1=!led1; // has passed 10 ms
    debounce.reset(); // restart timer when the toggle is performed
}
```

## Realtime Operating System (RTOS)
มี OS ช่วยในการจัดสรรทรัพยากร แบ่งทรัพยากรกัน รันโค้ดพร้อมกัน เรียกว่าเป็น Task หรือ Threads โดย RTOS ทำ 2 อย่างหลักๆ

- **Decides** which Program run for how long
- **Provide** Commu and Sync between tasks
- **Control** overall resource

### RTOS Scheduling
RTOS ยังมี Scheduler เอาไว้กำหนดว่า Task ไหนรัน และรันนานแค่ไหน

Ex. Round Robin Scheduler sync activity to clock tick แต่ Round robin จะ Switch ทุก Clock Tick ไม่ว่า Task แต่ละอันจะเป็นไง ก็เลยไม่มี Task Prioritization (แต่ scheduler อันอื่นอาจจะมีนะ)