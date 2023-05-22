# fpga-doc-2

## ABSTRACT
- This abstract describes the implementation of an LM70 temperature sensor using Serial Peripheral Interface (SPI) protocol. The LM70 is a digital temperature sensor that uses an SPI bus for communication with other devices. The sensor can measure temperatures ranging from -55°C to 150°C with a resolution of ±0.25°C.
This circuit is designed to provide the necessary power and signal conditioning using SPI interface to communicate with the sensor, where data transfer is bidirectional and is displayed in the 7-segment display. This circuit was then implemented in verilog and is synthesized using Xylinx software.

## LM70

![image](https://user-images.githubusercontent.com/84337196/233047224-8a4a9ef9-8d72-49d9-bb33-4fd5a947516b.png)


***Introduction***:

The LM70 is a precision temperature sensor that can accurately measure temperatures in the range of -55°C to +150°C. It is manufactured by Texas Instruments and is commonly used in applications such as automotive, industrial, and medical equipment. The LM70 temperature sensor is a digital sensor that communicates with the host microcontroller using a two-wire interface.

***Features***:

The LM70 temperature sensor offers several features that make it an attractive option for temperature sensing applications. Some of these features include:

1. **High Accuracy**: The LM70 offers an accuracy of ±0.5°C over the entire temperature range of -55°C to +150°C.
2. **Low Power Consumption**: The LM70 operates at a low power consumption of just 1.5mW.
3. **Digital Interface**: The LM70 communicates with the host microcontroller using a two-wire interface, which simplifies the integration process.
4. **Small Size**: The LM70 is available in a small package that makes it ideal for space-constrained applications.
5. **Wide Temperature Range**: The LM70 can measure temperatures in the range of -55°C to +150°C, which makes it suitable for a wide range of applications.

The LM70 component is widely used across the automotive, industrial, and medical industries for various applications.


[Reference Datasheet of LM70 by Texas Instruments](https://www.dropbox.com/s/ot6h1511lpuxlmx/datasheet-LM70-TI-tempSensor.pdf)


## SPI Protocol
The **Serial Peripheral Interface (SPI)** protocol is a synchronous serial communication interface used to exchange data between microcontrollers, sensors, and other devices. The SPI protocol uses a master-slave architecture, where one device (the master) initiates and controls the communication, while the other devices (the slaves) respond to the master's commands.

- The SPI protocol requires four signal lines:

  - **SCLK (Serial Clock)**: Provides a clock signal generated by the master device to synchronize the data transfer.

  - **MOSI (Master Output Slave Input)**: Carries data from the master device to the slave device.

  - **MISO (Master Input Slave Output)**: Carries data from the slave device to the master device.

  - **SS (Slave Select)**: A signal that enables the master to select which slave device to communicate with.

- The SPI protocol is a full-duplex communication protocol, which means that data can be transmitted and received simultaneously. It is also a relatively fast protocol, with speeds of up to several MHz.

- The SPI protocol is commonly used in embedded systems, where it provides a simple and efficient way to communicate between different devices.
    
## Block Diagram    
![image](https://user-images.githubusercontent.com/84337196/235107685-f035ae0c-d5ef-4fec-8f0f-968d06d07747.png)


## Timing Diagram
<img src="https://user-images.githubusercontent.com/84337196/234562302-3ad1e84f-5d1d-4c05-9beb-ff4f5bf9db7c.png"  width="1000" height="200">

### Description
- Frequency of the system clock = 10MHz.
- A frequency divider clock is employed to reduce the frequency to 10KHz as required by the IC.
- The system reads the data at reset high.
- SCK, the slave clock, has a frequency that is half that of the master clock as it toggles on each negative edge of the divided clock.
- Chip Select is kept active low, so whenever the signal is low, the slave clock is turned ON and data is read.
- dataSeg[7:0] is the 7-segment display output which is obtained from the shift register.
- disp[1:0] are the 2 select lines for the respective 7-segment displays. When disp[1] is ON, disp[0] is OFF: 2'b10 = 1'd2. disp[1] is OFF, disp[0] is ON: 2'b01 = 1'd1.  



    
## Circuitry of the FPGA used for the project

### Blocks in the control logic
- Counter 
  - Code Snippet
  ``` verilog
    //5b Counter
  always @(posedge divclk or negedge RSTN)
    if (~RSTN)
       count <= `RST_COUNT;
    else if (count == `MAX_COUNT)
       count <= `RST_COUNT;
    else
       count <= count + 1'b1;
    ```
- State Machine
  - Code Snippet
  ```verilog
   // 2-spi_state (IDE, READ) spi_state-machine
   always @(posedge divclk or negedge RSTN)
   if (~RSTN)
      begin	    
        spi_state <= `SPI_IDLE;
        bcd_latch <= 4'b0000;
      end
   else if ((count >= `CS_LOW_COUNT) && (count < `CS_HIGH_COUNT) )
      begin
        spi_state <= `SPI_READ;
      end
   else if (count == `CS_HIGH_COUNT)
      begin	    
        spi_state <= `DISP_WRITE_MSB;
        bcd_latch <= bcd_msb;
      end
   else if (count == `WRITE_LSB_COUNT)
      begin	    
        spi_state <= `DISP_WRITE_LSB;
        bcd_latch <= bcd_lsb;
      end
   else 
      begin	    
        spi_state <= `SPI_IDLE;
      end
 
### Why is synchronous used in place of asynchronous counter?
- **Synchronous operation**: In synchronous counters, all flip-flops change state at the same time in response to a single clock pulse. This ensures that the outputs of the counter are synchronized and stable, eliminating the possibility of glitches or other timing errors that can occur in asynchronous counters.
- **Higher operating frequency**: Synchronous counters can operate at higher clock frequencies which makes them suitable for use in applications that require high-speed counting.
- **Simpler circuit design**: Synchronous counters have a simpler circuit design compared to asynchronous counters, as they do not require additional logic gates to generate the clock pulses that drive the flip-flops. This makes them easier to implement and troubleshoot.
- **Easier to cascade**: Synchronous counters are easier to cascade or connect in series to create larger counters. This is because the outputs of the flip-flops change state simultaneously in response to a single clock pulse, simplifying the interconnection of multiple counters.

### Shift Register Action
A SIPO Register is used here, which stores data serially on each positive clock edge. The stored data is then outputted in parallel and used as input for the Binary to BCD converter.

### Binary to BCD Converter
- Here, two 4-bit binary to BCD converters are used for MSB and LSB respectively where the 8-bit data is divided into two 4-bit datas.

### Why BCD Converter?
-  BCD is more human-readable than binary or hexadecimal codes because it directly represents decimal digits. This makes it easier for people to interpret and understand the encoded values.
-  BCD is easier to process in certain applications, such as arithmetic operations, because it preserves the decimal structure of the data.

### Use of Multiplexer
- Here, a 2:1 MUX is used where the select line is the output of the control logic and the inputs to the MUX are the 4-bit MSB and the 4-bit LSB bits obtained from the BCD outputs respectivey.

### BCD to 7-segment
- This conversion logic is obtained by solving the Karnaugh Map of the individual segments of the 7-segment display.

### 7-Segment Display
- In this project, we have used two 7-Segment displays. 
- Each of the displays is activated by a select line.
- Here logic is developed such that, if one of the select lines get a 1, the other gets a 0 and vice-versa.
- Due to the high clock frequency and rapid data change rate, the changes in the data are imperceptible to the human eye, resulting in the simultaneous and accurate display of the data on both screens.

## Implementation 
- This circuit can be implemented in 2 ways:
  - ASIC(Application Specific Integrated Circuit)
  - FPGA(Field Programmable Gate Array)

### ASIC

**ASIC (Application-Specific Integrated Circuit)** is a type of integrated circuit that is designed for a specific application or task. Unlike FPGAs, ASICs are not programmable and are custom-designed to perform a specific function, which makes them more efficient and faster than general-purpose processors. ASICs can be optimized for power consumption, speed, and size, and are commonly used in applications such as automotive, telecommunications, consumer electronics, and aerospace. However, ASICs require significant upfront design and fabrication costs, which can make them more expensive to develop than FPGAs or other off-the-shelf solutions.

### FPGA

**FPGA (Field Programmable Gate Array)** is a type of semiconductor device that can be programmed after manufacturing to perform a wide range of digital logic functions. FPGAs consist of a matrix of configurable logic blocks (CLBs) that can be programmed to perform specific logic functions. FPGAs are highly parallel and can perform multiple tasks simultaneously, which can lead to significant performance gains over traditional processors. FPGAs are used in a wide range of applications, including digital signal processing, image processing, audio and video processing, network processing, and high-performance computing. However, they also require specialized knowledge and can be more expensive than traditional processors.
- SPARTAN 6 FPGA board is used in this project

### What is SPARTAN 6?

SPARTAN 6 is a **chip** that you can program to make different electronic devices. It is an **FPGA** (field-programmable gate array).

Some benefits of SPARTAN 6 are :

- It is cheap and low-power because it uses very small parts (**45nm technology**).
- It can do many tasks (**147K logic cells**) and store a lot of data (**4.8Mb memory**).
- It can handle many kinds of signals (**40 I/O standards**) and connect with other devices easily (**PCI Express®**).
- It can use a special program (**MicroBlaze™ soft processor**) instead of another chip.
- It can protect your work from being copied or changed by others (**AES encryption** and **Device DNA**).
- It can work in very hot or cold places (extended temperature range).

SPARTAN 6 is good for making devices that need to connect with other things, like cars, TVs, cameras, and machines.

### Code
```verilog
// Code your design here
`timescale 1ns / 1ps
//DEFINES
`define RST_COUNT       5'd0
`define CS_LOW_COUNT    5'd4
`define CS_HIGH_COUNT   5'd20
`define WRITE_LSB_COUNT	5'd22
//`define WRITE_LSB_COUNT	5'd24
`define MAX_COUNT       5'd28
`define SPI_IDLE	2'b00
`define SPI_READ	2'b01
`define DISP_WRITE_MSB	2'b10
`define DISP_WRITE_LSB	2'b11

/////Verilog code for continous SPI read of LM07
module LM07_read(SYSCLK, RSTN, divclk, CS, SCK, SIO, disp, dataSeg, outreg);
  input SYSCLK;	                    //System clock from the testbench
  input RSTN;	                    //Active-low reset signal
  input SIO;	                    //Serial data output from the temp sensor.
  output reg divclk;                //Frequency Divider Clock
  output CS;	                    //Generate the Chip select for temp sensor
  output reg SCK;	            //Generate the SPI clock for temp sensor
  output [1:0] disp;	            //7-segment display select lines.
  output [7:0] dataSeg;             //7-segment data
  output [7:0] outreg;	            //the 8-bit data is latched for display
  reg SYSCLK_HALF;
  reg [7:0] shift_reg;
  reg [1:0] spi_state;
  reg [4:0] count;
  wire sysclk_gated;
  wire [7:0] temp_bin;
  wire [3:0] bcd_msb;
  wire [3:0] bcd_lsb;
  reg [3:0] bcd_latch;    
  reg [7:0] IN;
  // Frequency Divider Clock
  integer cnt;
  always @ (posedge SYSCLK) begin
     if(!RSTN) begin
      divclk = 0;
      cnt =0;
      end	
     else
      cnt = cnt+1;
      if (cnt==1000) begin
        divclk = ~divclk;
        cnt = 0;
       end
      end
  //Latch the shift register at the end of read.
  assign outreg[7:4] = bcd_msb; 
  assign outreg[3:0] = bcd_latch; 

  //BCD-to-7segment decoder
  assign dataSeg[7] = (~bcd_latch[2] && ~bcd_latch[0]) || bcd_latch[1] || bcd_latch[0] || (bcd_latch[2] && bcd_latch[0]); //a
  assign dataSeg[6] = ~bcd_latch[2] || (~bcd_latch[1] && ~bcd_latch[0]) || (bcd_latch[1] && bcd_latch[0]);  //b
  assign dataSeg[5] = ~bcd_latch[1] || bcd_latch[0] || bcd_latch[2]; //c
  assign dataSeg[4] = (~bcd_latch[2] && ~bcd_latch[0]) || (~bcd_latch[2] && bcd_latch[1]) || (bcd_latch[2] && ~bcd_latch[1] && bcd_latch[0]) || (bcd_latch[1] &&    ~bcd_latch[0]) || bcd_latch[3]; //d
  assign dataSeg[3] = (~bcd_latch[2] && ~bcd_latch[0]) || (bcd_latch[1] && ~bcd_latch[0]); //e
  assign dataSeg[2] = (~bcd_latch[1] && ~bcd_latch[0]) || (bcd_latch[2] && ~bcd_latch[1]) || (bcd_latch[2] && ~bcd_latch[0]) || bcd_latch[3]; //f
  assign dataSeg[1] = (~bcd_latch[2] && bcd_latch[1]) || (bcd_latch[2] && ~bcd_latch[1]) || bcd_latch[3] || (bcd_latch[1] && ~bcd_latch[0]); //g
  assign dataSeg[0] = 1'b0; //DP
  //assign dataSeg = 8'hDA;
  //Since LSB is 2-deg C, multiply by 2
  assign temp_bin = shift_reg<<1;

  //BCD Logic
  //Temp/10 approx. 1/16 + 1/32
  assign bcd_msb = (temp_bin + (temp_bin>>1))>>4;
  //LSB = temp - 10*MSB = temp - (8*MSB + 2*MSB)
  assign bcd_lsb = temp_bin - ((bcd_msb<<3) + (bcd_msb<<1));  
 
  //shift register for the input (SIO)
  always @(posedge SCK or negedge RSTN)
    if (~RSTN)
      shift_reg <= 8'h00;
    else
      begin
       shift_reg <= shift_reg<<1;
	shift_reg[0] <= SIO ;
      end

//SPI CLOCK SCK generator
always @(negedge divclk or negedge RSTN)
  if (~RSTN || CS)
    SCK <= 1'b0;
  else
    SCK <= ~SCK;
//input generation
always @ (posedge divclk)
begin
if (~RSTN)
 IN = 0;
 else 
 IN = 'b1101;
 //IN = IN >> 1;
 end
  
// Chip Select CS generator
assign CS = ~(spi_state == `SPI_READ);
//7-Segment select lines
 assign disp[1] =  (spi_state == `DISP_WRITE_MSB);
 assign disp[0] =  (spi_state == `DISP_WRITE_LSB);

// 2-spi_state (IDE, READ) spi_state-machine
always @(posedge divclk or negedge RSTN)
  if (~RSTN)
      begin	    
        spi_state <= `SPI_IDLE;
        bcd_latch <= 4'b0000;
      end
  else if ((count >= `CS_LOW_COUNT) && (count < `CS_HIGH_COUNT) )
      begin
        spi_state <= `SPI_READ;
      end
  else if (count == `CS_HIGH_COUNT)
      begin	    
        spi_state <= `DISP_WRITE_MSB;
        bcd_latch <= bcd_msb;
      end
  else if (count == `WRITE_LSB_COUNT)
      begin	    
        spi_state <= `DISP_WRITE_LSB;
        bcd_latch <= bcd_lsb;
      end
  else 
      begin	    
        spi_state <= `SPI_IDLE;
      end

  //5b Counter
  always @(posedge divclk or negedge RSTN)
    if (~RSTN)
       count <= `RST_COUNT;
    else if (count == `MAX_COUNT)
       count <= `RST_COUNT;
    else
       count <= count + 1'b1;
 endmodule
 ```
 
### Frequency Divider Clock
- A frequency divider clock is used here as so as to reduce the power consumption and switching. The clock frequency provided by the SPARTAN 6 FPGA is 10 Mhz and the divider clock frequency is 10 Khz.
  - Code Snippet
  ```verilog
  integer cnt;
  always @ (posedge SYSCLK) begin
     if(!RSTN) begin
      divclk = 0;
      cnt =0;
      end	
     else
      cnt = cnt+1;
      if (cnt==1000) begin
        divclk = ~divclk;
        cnt = 0;
       end
      end
  ```
### BCD Logic
- The left shift operations denotes the multiplication by 2 operation while the right shift operation denotes the division by 2 operation. 
  - Code Snippet
  ```verilog
   assign temp_bin = shift_reg<<1;
  //BCD Logic
  //Temp/10 approx. 1/16 + 1/32
  assign bcd_msb = (temp_bin + (temp_bin>>1))>>4;
  //LSB = temp - 10*MSB = temp - (8*MSB + 2*MSB)
  assign bcd_lsb = temp_bin - ((bcd_msb<<3) + (bcd_msb<<1));  
  ```	  
## User Constraint File(UCF)
A UCF file, or User Constraint File, is a text-based file used in electronic design tools for creating digital circuits. It contains settings and constraints that define how components within a circuit should behave and interact. These files are commonly used with programmable logic devices like FPGAs or CPLDs. UCF files specify details such as pin assignments, clock frequencies, input/output delays, and other design constraints. They allow designers to customize circuit behavior and ensure proper functionality and timing in the implementation process.

The UCF file used here is:
```bash
NET "SYSCLK" LOC = "P87";         //Input clock (Location P87 is fixed)
NET "CS" LOC = "P69";                //Chip Select clock at pin16 of GPIO1
NET "RSTN" LOC = "P22";          //Manual RSTN from input port 22
NET "SCK" LOC = "P60";            //Slave Clock generation from pin 16 of GPIO1
NET "disp[1]" LOC = "P102";    //Display of 1st  7-segment display
NET "disp[0]" LOC = "P101";    //Display of 2nd 7-segment display
NET "SIO" LOC = "P126";         //Output of the temperature generation from pin 14 of GPIO1


//Pins of 7-segment display (a,b,c,d,e,f,g)
NET "dataSeg[7]"  LOC = "P119";
NET "dataSeg[6]"  LOC = "P118";
NET "dataSeg[5]"  LOC = "P117";
NET "dataSeg[4]"  LOC = "P116";
NET "dataSeg[3]"  LOC = "P115";
NET "dataSeg[2]"  LOC = "P114";
NET "dataSeg[1]"  LOC = "P112";
NET "dataSeg[0]"  LOC = "P111";		
```


 
 
 


