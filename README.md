ECE382Lab3
==========

title = 'Lab 3 - SPI - "I/O"'

# Lab 3 - SPI - "I/O"

##  Mega Prelab
A hard copy of this Mega Prelab is required to be turned in.  Answers should not be handwritten.  The timing diagram may be NEATLY drawn by hand with the assistance of a straightedge on engineering paper.
### Nokia1202  LCD BoosterPack v4-5
Look at the schematic for the Nokia1202 LCD BoosterPack. Complete the following table.  The pin number should be the pin number that signal connects to on the MSP 430.  You may need to examine page 3 of the MSP430 Data Sheet. The type is from the perspective of the MSP430 and is one of the following: input, output, or power .  For example, the RST' line should  be listed as having an output type because it is an output from the  MSP430 (to an input on the Nokia 1202).  For input and output types (do nothing further with power), list their bit settings in the three registers listed (see pages 328 and 329 of the MSP430g2553 User's Guide). <br>

| Name | Pin # | Type | PxDIR| PxREN | PxOUT |
|:-: | :-: | :-: | :-: | :-: | :-: |
|GND |20 | Power | - | - | - |
| RST | 8 | Out | 1 | 0 | 1 |
| P1.4 | 6 | Out | 1 | 0 | 1 |   
| MOSI| 15 | Out | 1 | 0 | 1 |   
| SCLK | 7 | Out | 1 | 0 | 1 |   
| VCC | 1 | Power | - | - | - |  
| S1 | 9 | In | 0 | 1 | 1 | 
| S2 | 10 | In | 0 | 1 | 1 | 
| S3 | 11 | In | 0 | 1 | 1 | 
| S4 | 12 | In | 0 | 1 | 1 | 


### Configure the MSP430
The following is a portion of the code that initalizes the directions of the pins connecting the MSP430 to the Nokia 1202 display. Use the information from the previous question to identify the names of the registers missing in the following code (identified by the letters A - D).  Put the register names in the table below.

```
mov.b	#LCD1202_CS_PIN|LCD1202_BACKLIGHT_PIN|LCD1202_SCLK_PIN|LCD1202_MOSI_PIN, & A
mov.b	#LCD1202_CS_PIN|LCD1202_BACKLIGHT_PIN|LCD1202_SCLK_PIN|LCD1202_MOSI_PIN, & B
mov.b	#LCD1202_RESET_PIN, & C
mov.b	#LCD1202_RESET_PIN, & D
```
| Mystery Label | Register|
|:-: |:-: |
| A| P1DIR |
| B | P1OUT |
| C | P2DIR |
| D | P2OUT |


The following initializes the SPI subsystem of the MSP430.  For each of the bits listed in the table below, identify how the code-snippet configures that pin and what function is realized by that setting.  For example, setting the UCMSB bit of the UCB0CTL0 register forces the SPI subsystem to output the bits starting from the LSB.  Also, list the bit position that each occupies in its associated register.
```
	bis.b	#UCCKPH|UCMSB|UCMST|UCSYNC, &UCB0CTL0
	bis.b	#UCSSEL_2, &UCB0CTL1
	bic.b	#UCSWRST, &UCB0CTL1
```

| ID | Bit | Function as set in the code |
|:-:|:-:|:-:|
| UCCKPH | 7 | Clock Phase Select -Data is captured on the first UCLK edge and changed on the following edge  |
| UCMSB | 5 | MSB first select. Controls the direction of the receive and transmit shift register – select LSB first |
| UCMST | 3 | Master mode select - slave |
| UCSYNCH| 0 | Synchronous mode enable – Synchronous mode |
| UCSSEL_2| 7-6 | USCI clock source select. These bits select the BRCLK source clock in master mode. UCxCLK is always
used in slave mode.
 |
| UCSWRST| 0 | Software reset enable |

### Communicate to the Nokia1202 display
The following code communicates one byte to the Nokia 1202 display using its 9-bit protocol.  Use this code to draw a timing diagram of the expected behavior of LCD1202_CS_PIN, LCD1202_SCLK_PIN, LCD1202_MOSI_PINs from the begining of this subroutine to the end.  Make sure that you clearly show the relationship of the edges in the clk and data waveforms.
```
;-------------------------------------------------------------------------------
;	Name:		writeNokiaByte
;	Inputs:		R12 selects between (1) Data or (0) Command string
;				R13 the data or command byte
;	Outputs:	none
;	Purpose:	Write a command or data byte to the display using 9-bit format
;-------------------------------------------------------------------------------
writeNokiaByte:

	push	R12
	push	R13

	bic.b	#LCD1202_CS_PIN, &P1OUT				; LCD1202_SELECT
	bic.b	#LCD1202_SCLK_PIN | LCD1202_MOSI_PIN, &P1SEL	; Enable I/O function by clearing
	bic.b	#LCD1202_SCLK_PIN | LCD1202_MOSI_PIN, &P1SEL2	; LCD1202_DISABLE_HARDWARE_SPI

	bit.b	#01h, R12
	jeq	cmd

	bis.b	#LCD1202_MOSI_PIN, &P1OUT			; LCD1202_MOSI_LO
	jmp	clock

cmd:
	bic.b	#LCD1202_MOSI_PIN, &P1OUT			; LCD1202_MOSI_HIGH

clock:
	bis.b	#LCD1202_SCLK_PIN, &P1OUT			; LCD1202_CLOCK positive edge
	nop
	bic.b	#LCD1202_SCLK_PIN, &P1OUT			; negative edge

	bis.b	#LCD1202_SCLK_PIN | LCD1202_MOSI_PIN, &P1SEL	; LCD1202_ENABLE_HARDWARE_SPI;
	bis.b	#LCD1202_SCLK_PIN | LCD1202_MOSI_PIN, &P1SEL2	;

	mov.b	R13, UCB0TXBUF

pollSPI:
	bit.b	#UCBUSY, &UCB0STAT
	jz		pollSPI					; while (UCB0STAT & UCBUSY);

	bis.b	#LCD1202_CS_PIN, &P1OUT				; LCD1202_DESELECT

	pop		R13
	pop		R12

	ret
```
### Configure the Nokia1202 display
The following code configures the Nokia 1202 display to display pixels.  The code consists of two main areas.  The first section holds the reset line low and then high for a specific length of time.  You will measure the duration of the reset pulse later in the lab. <br> <br>
The second section sends a sequence of commands to the Nokia 1202 display.  Your task is to use the the information on page 42 (and beyond) of the STE2007 technical document to decode the symbolic constants moved into register R13 on the lines marked with "DECODE HERE".

```
;-------------------------------------------------------------------------------
;	Name:		initNokia		68(rows)x92(columns)
;	Inputs:		none
;	Outputs:	none
;	Purpose:	Reset and initialize the Nokia Display
;-------------------------------------------------------------------------------
initNokia:

	push	R12
	push	R13

	bis.b	#LCD1202_CS_PIN, &P1OUT

	;-------------------------------------------------------------------------------
	; Measure the time that the RESET_PIN is held low by the delayNokiaResetLow loop
	bic.b	#LCD1202_RESET_PIN, &P2OUT
	mov	#0FFFFh, R12
delayNokiaResetLow:
	dec	R12
	jne	delayNokiaResetLow
	bis.b	#LCD1202_RESET_PIN, &P2OUT
	;-------------------------------------------------------------------------------

	mov	#0FFFFh, R12
delayNokiaResetHigh:
	dec	R12
	jne	delayNokiaResetHigh
	bic.b	#LCD1202_CS_PIN, &P1OUT

	; First write seems to come out a bit garbled - not sure cause
	; but it can't hurt to write a reset command twice
	mov	#NOKIA_CMD, R12
	mov	#STE2007_RESET, R13					; DECODE HERE
	call	#writeNokiaByte


	mov	#NOKIA_CMD, R12
	mov	#STE2007_RESET, R13
	call	#writeNokiaByte

	mov	#NOKIA_CMD, R12
	mov	#STE2007_DISPLAYALLPOINTSOFF, R13			; DECODE HERE
	call	#writeNokiaByte

	mov	#NOKIA_CMD, R12
	mov	#STE2007_POWERCONTROL | STE2007_POWERCTRL_ALL_ON, R13	; DECODE HERE
	call	#writeNokiaByte

	mov	#NOKIA_CMD, R12
	mov	#STE2007_DISPLAYNORMAL, R13				; DECODE HERE
	call	#writeNokiaByte

	mov	#NOKIA_CMD, R12
	mov	#STE2007_DISPLAYON, R13					; DECODE HERE
	call	#writeNokiaByte

	pop	R13
	pop	R12

	ret
```
Complete the table below.  To answer this question you will have to use some common sense in decoding the meaning of the symbolic constants.


| Symbolic Constant | Hex | Function |
| :-: | :-: | :-: |
|#STE2007_RESET| E2 | Internal Reset |
|#STE2007_DISPLAYALLPOINTSOFF| A4 | LCD display 0: normal display |
|#STE2007_POWERCONTROL| - | Sets the on–chip power supply circuit operating mode | 
|#STE2007_POWERCTRL_ALL_ON | 2F | Booster : ON Voltage regulator : ON Voltage follower : ON |
|#STE2007_DISPLAYNORMAL | A6 |  LCD display 0: normal |
|#STE2007_DISPLAYON | AF | LCD display 1: ON |

(This marks the end of the Mega Prelab.)
---------------------------------------------------------------
## Logic Analyzer
The answers to the logic analyzer section will be posted to GitHub along with the functionality code.
###Physical communication
Connect the Nokia 1202 Booster Pack to your TI Launch Pad.  Make sure that the buttons on the Booster Pack are pointed away from the USB connector (and on the same side of the board as the MSP430 buttons), just like in the following picture.
![Board connection](connection.jpg)<br>
Download <a href="lab3.asm">lab3.asm</a> and build a project around the file.
Run the program and observe the output on the LCD every time you press the SW3 button.  It should look something like the following image after a few button presses.<br>
![test program](screen.jpg)<br>
When SW3 is detected as being pressed and released (lines 56-62), the MSP430 generates 4 packets of data that are sent to the Nokia 1202 display, causing a vertical bar to be drawn. Complete the following table by finding the 4 calls to writeNokiaByte that generate these packets. In addition, scan the nearby code to determine the parameters being passed into this subroutine. Finally, write a brief description of what is trying to be accomplished by each call to writeNokiaByte.

|Line|R12|R13|Purpose|
|:-:|:-:|:-:|:-:|
| 66 | move #NOKIA_DATA | move #0xE7 | move #1 into r12 - cmd vs data, this is the pattern that is drawn 1110 0111 |
| 276 | #NOKIA_CMD | 1011 0000 | cmd, row number location on page  |
| 288 | #NOKIA_CMD | 0001 0000 | cmd,  |
| 294 | #NOKIA_CMD | mask upper bits to 0000 1111 | cmd, resets r13 (row) |
Configure the logic analyzer to capture the waveform generated when the SW3 button is pressed and released. Decode the data bits of each 9-bit waveform by separating out the MSB, which indicates command or data. Explain how the packet contents correspond to what was drawn on the display.  Be specific with the relationship between the data values and what and where the pixels are drawn

|Line|Command/Data|8-bit packet|
|:-:|:-:|:-:|
| 66 | Data | E7 |
| 276 | Commamd | B4 |
| 288 | Command | 10 |
| 294 | Command | 04 ||
Hint: in order to probe the signals while the LCD is connected to the LaunchPad, you will need to use the LaunchPad header pins with the probe hook grippers. Be careful when attaching and detaching the grippers to the pins, as they may easily bend and then no longer serve you well. Also, don't forget the ground pin!<br>
![Logic analyzer connection](left.jpg)<br>
You will get a waveform similar to that shown below. Note that the command/data bit is significantly far away from the 8 data bits. <br>
![Logic analyzer output](LA_datastream.jpg)<br>
Next, setup the Logic Analyzer to capture the RESET signal on a falling edge. Measure the duration that the RESET line is held low in the initNokia subroutine. Hint, the code to hold the reset line low can be found on lines 93-100. 
How many counts does the firmware loop count down from? 
Using the delay you just measured and the number of counts, calculate the amount of time each iteration of the delay loop consumes.

The reset counter counts down from 0xFFFF (65535) to zero. But, the delay from the measured reset is 6.544 micro seconds. So each loop is 6.544u/65535 or 0.998 nano seconds.

###Writing modes
The native write operation to the Nokia 1202 will overwrite any information that is was on the display with new information.  However, that may not be the best course of action in your application.  The new bits being added to the image may be merged using the AND, OR, XOR operators.  To do this treat a black pixel as a logic 1 and a white pixel as a logic 0.  The pixel values from the same locations are combined using a logical operator and placed at the corresponding location in the destination imaged.
Import the following image into a paint program and show the result of the operation between the two bits maps combined using the logic operator specified.
![xor picture](bitblock.jpg)
## Functionality
Required functionality: Create a block on the LCD that is 8x8 pixels.  The location of the block must be passed into the subroutine via r12 and r13.
A functionality: Move the 8-pixel block one block in the direction of the pressed button (up, down, left, right).


##Logic Analysis
Each package shows a step from the table above. It looks at line 66, 276, 288, and 294.
![Package 1](Logic1.jpg)<br>
Package 1 shows R13 value of E7 and the Data bit.
![Package 2](Logic2.jpg)<br>
Package 2 shows R13 value of B4. It should read B0, but I had run the code four times and the mask function only masks the most significant nibble. Since I was on row for B4 showed up in the logic analyzer.
![Package 3 ](Logic3.jpg)<br>
Package 3 shows 10 just like line 288.
![Package 4 ](Logic4.jpg)<br>
Package 4 has 04. Again, it should read 00, but I am on row 4 so a 4 is displayed.
![Reset ](resetLength.jpg)<br>
The reset counter counts down from 0xFFFF (65535) to zero. But, the delay from the measured reset is 6.544 micro seconds. So each loop is 6.544u/65535 or 0.998 nano seconds.

##Required Code
https://github.com/Austinbolinger/ECE382Lab3/blob/master/requiredFunctionality
In this code I just added 4 lines of code and modified one other line. I change the pattern in R13 from E7 to FF in order to make a solid line instead of a double dashed line. I added a decrementing loop to call set address 8 times for each button pressed. This may not work like it is suppose to but it gets the job done.

##A Functionality
https://github.com/Austinbolinger/ECE382Lab3/blob/master/AFunctionality
This code took a while to understand. I started off with identifying how to get the box to start in the center of the screen. After much tampering, I found out that I just needed to delete an increment given and initialize R10, row, and R11, column, to what I wanted them to be. Next I rearranged the code so that each time I called makeBox It would do the same thing and I would just need to pass in different cursor locations.After that, I worked on getting the other buttons to be recognized. I started with just one button. After many hours, I received help from Dr. Coulston. He helped me fix my problem with pressing and releasing the buton. I had the code for press but not for release. This lead to the box appearing in a random position depeding on howmany microseconds I held down and allowed the program to run through. After he showed me how to test for a release. I was able to copy the code for each button. The last step was to apply the correct column or row change in order to pass the right values into makeBox. A few trails and errors lead to code that works like a charm.


##Documentation
Dr. York helped me understand the mega prelab time wave form by walking me through it. Dr. York found an error in my coding. I was commenting out part of the working set address function. It messed up the whole program just enough to not let it work. Dr. Coulston helped me with button pressed and button released. I was stuck with just identifying the press and not accounting for the release. He walked me through C code for it and then assembly code for button release.

## Grading

|| Item | Grade | Points | Out of | Date | Due |
|:-: | :-: | :-: | :-: | :-: |
| Mega Prelab | **On-Time** -------------------------------------------------------------------- **Late:** 1Day ---- 2Days | | 20 | | EOC L16 |
| Required Logic Analyzer | **On-Time** -------------------------------------------------------------------- **Late:** 1Day ---- 2Days ---- 3Days ---- 4+Days| | 40 | | COB L19 |
| Required Functionality | **On-Time** -------------------------------------------------------------------- **Late:** 1Day ---- 2Days ---- 3Days ---- 4+Days| | 20 | | COB L19 |
| A Functionality | **On-Time** -------------------------------------------------------------------- **Late:** 1Day ---- 2Days ---- 3Days ---- 4+Days| | 10 | | COB L19 |
| Lab Notebook | **On-Time** -------------------------------------------------------------------- **Late:** 1Day ---- 2Days ---- 3Days ---- 4+Days| | 10 | | COB L20 |
| **Total** | | | **100** | | |

