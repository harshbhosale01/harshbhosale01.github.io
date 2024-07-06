---
layout: post
title: Understanding IR protocols and decoding the IR codes for Bluestar AC
date: 2024-07-07 00:00:00
description: 
tags: IR codes, Electronics basics
categories: 
tabs: true
---

# Understanding IR protocols and decoding the IR codes for Bluestar AC

From my childhood, I was always nosy to know how does the infrared remotes work and what thing lies inside them. So, I along with my friend [Manan Shanghvi](https://www.linkedin.com/in/manan-shanghvi-393537282/) thought of giving it a try and understanding them. 

I decided to work with AC remotes because of the fact that IR codes for AC's are composite and are not just commands, but contain all the necessary information about the operating mode of the AC  and there is usually a check sum as well. While the TV remote just sends a HEX code to its receiver. For example: When you press volume up button on a TV remote only the volume up command will be sended to the TV receiver, but if you press temperature increase button on an AC remote its whole setting like heating or cooling, fan power, temperature, jet direction, etc is sended to the AC.

## What is a IR protocol?
According to the definition on digikey site, In communications, language is referred to as a protocol, which is nothing more than an agreement between the sender and the recipient of the data. The two parties agree to follow a **predefined pattern** and transmit the information in a certain way. 

The simplest method for transmitting binary values with an IR LED would be to turn the infrared LED on (to send a logical 1) or to leave it turned off (which could represent a logical 0) for a certain period. The sender could do this until all the data bits have been transmitted. Unfortunately, this won’t work in reality since many other sources emit IR radiation. The receiver would not be able to filter out unwanted signals from other sources.

To overcome this issue, the sender is required to pulse the LED on and off very quickly, instead of just turning it on and off. Typically, a frequency of 38 kHz is used, and this is also referred to as the carrier frequency of the IR signal. Note that other wavelengths and carrier frequencies are also possible — for example, 940 nm and 36 kHz. However, in reality, that doesn’t seem to be a problem. Most IR receivers will still work with slight variations in wavelength and carrier frequency.

If you want to read more about IR protocols, refer this docs [1](https://www.vishay.com/docs/80071/dataform.pdf), [2](https://docs.kernel.org/userspace-api/media/rc/rc-protos.html#:~:text=IR%20is%20encoded%20as%20a,devices%20for%20a%20given%20protocol.).

So now we will look how to setup an IR receiving environment to get the RAW IR values.



## Receiving the IR commands:

Components used: 
1. ESP8266
2. [IR transmitter and receiver (38 kHz)](https://roboticsdna.in/product/ir-infrared-transmitter-module-ir-digital-38khz-infrared-receiver-sensor-module-for-arduino-electronic-building-block/?src=google&kwd=&adgroup=%7Badgroup%7D&device=c&campaign=%7Bcampaign%7D&adgroup=%7Badgroup%7D&keyword=&matchtype=&gad_source=1)
3. An AC remote
    - I'll be using Bluestar AC remote.
4. Jumper wires 


---

### Step 1:
Implement the following circuit. 

NOTE: For other ESP8266 based devkits attach the data pin of IR receiver to the GPIO 5. 

<img src="https://hackmd.io/_uploads/Hk3eUqU8C.jpg" alt="Pin connections" width="500"/>

---

### Step 2:
Open the Arduino IDE. If the IDE is not already present, download it from [here](https://www.arduino.cc/en/software).
Now we will add the IR library for ESP8266.
```
Sketch> Include Library> Manage Library
```
Now search for IRremoteESP8266. And install the following library. 

<img src="https://hackmd.io/_uploads/HJKTjxDIC.png" alt="Library manager" width="400"/>     

Now clone the following repository using,
```
git clone https://github.com/harshbhosale01/Bluestar-AC-IR-codes
cd Bluestar-AC-IR-codes
```
Inside arduino IDE open files using Ctrl+O, and navigate to Bluestar-AC-IR-codes folder. Open IRrecvDumpV2.ino,
Now build and flash, after selecting appropriate board and COM port. Don't open the serial port of Arduino IDE.


---

### Step 3:
In this step we will take the RAW IR codes into a text file.

Open the command prompt as administrator and navigate to Bluestar-AC-IR-codes folder. Install the dependencies using following commands.
```
pip install pyserial
pip install openpyxl
``` 
Now we will run the pyserial.py file where we will have to specify UART baudrate, COM port and a text file path in which we will write the RAW IR codes. 
```
python pyserial_1.py COM14 -b 115200 -f serial_output.txt
```
This python file will read the UART messages from specified COM port and will write the RAW IR signal values to a file named serial_output.txt 
Note: Everytime the pyserial.py records an IR signal the previous record of signal from serial_output.txt is cleared and new record is written.


---

### Step 4:

Now let try to analyse the RAW IR codes.

Run the following command on cmd prompt. The following python file parses the RAW IR codes and makes some conclusion. This code is developed and maintained by [David Conran](https://github.com/crankyoldgit). I've modified this code to write the decoded IR signal directly to a excel file named Blue_star_data.xlsx
```
python auto_analyse_raw_data.py -f serial_output.txt
```
Output will look like this,
```
Found 211 timing entries.
Potential Mark Candidates:
[5000, 558]
Potential Space Candidates:
[5060, 1578, 576]

Guessing encoding type:
Looks like it uses space encoding. Yay!

Guessing key value:
kHdrMark   = 4983
kHdrSpace  = 5060
kBitMark   = 476
kOneSpace  = 1544
kZeroSpace = 542

Decoding protocol based on analysis so far:

kHdrMark+kHdrSpace+11010101111111101101011101010111111110100101111111111010010111111111111101111111011001001111110111011100

Binary string inserted into 'Blue_star_data.xlsx' in sheet 'BinaryData'.

  Bits: 104
  Hex:  0xD5FED757FA5FFA5FFF7F64FDDC (MSB first)
        0x3BBF26FEFFFA5FFA5FEAEB7FAB (LSB first)
  Dec:  16954468142548628673968778575324 (MSB first)
        4733620368601665983662124662699 (LSB first)
  Bin:  11010101,11111110,11010111,01010111,11111010,01011111,11111010,01011111,11111111,01111111,01100100,11111101,11011100 (MSB first)
        0b00111011101111110010011011111110111111111111101001011111111110100101111111101010111010110111111110101011 (LSB first)
kHdrMark+
Total Nr. of suspected bits: 104
```
## Analysing the IR signal:
Now let's interpret the terms in the above output.
The guessing key value part is important here, 

* kHdrMark: Header mark
* kHdrSpace: Header space
* kBitMark: Zero Mark/One Mark
* kOneSpace: One Space
* kZeroSpace: Zero Space

To understand the meaning of this terms lets visualise the RAW IR signal which is saved in serial_output.txt file, I'll be using [IRScrutinizer software](https://github.com/bengtmartensson/IrScrutinizer) for visualisation. You can also use the ir_sig_viz.py file for visualisation just change the raw_data array in the code. 
![raw_ir_sig](https://hackmd.io/_uploads/ryS0xrsI0.jpg)
Open the image in another tab to enlarge. This is the raw signal received by IR receiver. If you see closely at start the signal is high for some period and low for some, the same pattern is seen at the end. On increasing the width we can see that the signal is high for 5000 units and low for another 5000 units. 
![image](https://hackmd.io/_uploads/ryVq0WnUA.png)

If you look at the output received [here](#Step-4:), kHdrMark = 4983 and kHdrSpace  = 5060. 

<img src="https://hackmd.io/_uploads/rknJy13LR.jpg" alt="drawing" width="400"/>

So in each IR command to Bluestar AC we will have to send this header mark and header space for specified time period.

Now, lets try understanding how 0's and 1's embedded in the IR signal. 

For 1:  
Each 1 will be composed of some mark+space, more specifically OneMark+OneSpace just like the pattern we saw in start bits. But here the time period of Mark and Space will be different.  
As give in the output [here](#Step-4:),  
kBitMark   = 476  
kOneSpace  = 1544  
Mark period for 0 and 1 will be same so it is called BitMark. So now if at any point, if the IR signal is HIGH for 500 units and then LOW for 1500 units it will be considered as 1.  
<img src="https://hackmd.io/_uploads/S1LgFkhU0.jpg" alt="1-signal-marked" width="400"/>  

For 0:  
Now for 0 the kBitMark = 476 and kZeroSpace = 542.
So now if at any point, if the IR signal is HIGH for 500 units and then LOW for 500 units it will be considered as 0.  


<img src="https://hackmd.io/_uploads/BJf7vW3IR.jpg" alt="0-signal-marked" width="210"/>     



So now we can visually interpret the signal.
![sig-interpretation](https://hackmd.io/_uploads/S1QnYZhL0.jpg)
Using the same analysis whole signal is decoded by auto_analyse_raw_data.py and then the results are written to Blue_star_data.xlsx  
If we ignore the start and stop bits we have 104 bits i.e. 13 bytes of data that we are concerned about. This 13 bytes will contain all settings/parameters of the Bluestar AC that I'm using.  

## Decoding the IR message fields
Currently, we have 13 bytes of data which corresponds to AC settings. So we want to know which byte is associated with which field of setting. Lets check which byte is associated with temperature. For this we just decreased the temperature by 1 deg and recorded the signal. Refer the image below.
![image](https://hackmd.io/_uploads/SJPjCM28C.png)
If you look closely in the above image only the 'D' and 'L' columns are changing with respect to change in set-point temperature. So which of this two column represents set-point temperature? The 'D' column ranges from 11110001 to 11111111 at the temperature 30°C to 16°C, respectively. Therefore, we can conclude that as we are decreasing temperature to its lowest value the byte repesenting it also reaches its highest value. So if column 'D' is representing temperature, what about column 'L'. Upon researching a little bit I got to know that it might be the checksum. Every firm has its own checksum computing formula, and now we have to reverse-engineer to find this formula, duh!  
For other fields like fan-speed, mode, etc, we did the same process as we did to sind byte associated with set-point temperature.  

I read [this](https://electronics.stackexchange.com/questions/183785/understanding-ir-protocol-checksum-generation) thread, to get some idea on checksum calculation methods for IR signals. But the formula given here didn't matched for our scenario. So I took a pen and began some manual calculations. I began summing all the columns except the 'L' column then performed modulo 255 operation on sum, to my surprise I was seeing a pattern for all cases but the checksum was still not matching with my result. After doing some logical math, we finally broke the code.  

![image](https://tenor.com/en-GB/view/high-five-walter-white-jesse-pinkman-breaking-bad-4k-gif-21661584.gif)   

```
Checksum = ((sum of all bytes)%255)-176
```
This was the formula we got. And following is the rough work for it, I know you won't be able to interpret anything from this rough work. But as a souvenir of our work, I have uploaded it here.  
<img src="https://hackmd.io/_uploads/BkNWGGvDC.jpg" alt="Checksum rough work" width="500"/>     
If you made it till here thank you for bearing with me. For suggestions or doubts you can surely hop onto my [mail](mailto:harshbhosaleatwork@gmail.com) or [linkedin](https://linkedin.com/in/hsbh) chat.

