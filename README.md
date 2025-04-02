# AHB to APB Bridge




## About the AMBA Buses

The Advanced Microcontroller Bus Architecture (AMBA) specification defines an
on-chip communications standard for designing high-performance embedded
microcontrollers.
Three distinct buses are defined within the AMBA specification:
- Advanced High-performance Bus (AHB)
- Advanced System Bus (ASB)
- Advanced Peripheral Bus (APB).

### Advanced High-performance Bus (AHB)

The AMBA AHB is for high-performance, high clock frequency system modules.
The AHB acts as the high-performance system backbone bus. AHB supports the
efficient connection of processors, on-chip memories and off-chip external memory
interfaces with low-power peripheral macrocell functions. AHB is also specified to
ensure ease of use in an efficient design flow using synthesis and automated test
techniques.

### Advanced System Bus (ASB)

The AMBA ASB is for high-performance system modules.
AMBA ASB is an alternative system bus suitable for use where the high-performance
features of AHB are not required. ASB also supports the efficient connection of
processors, on-chip memories and off-chip external memory interfaces with low-power
peripheral macrocell functions.

### Advanced Peripheral Bus (APB)

The AMBA APB is for low-power peripherals.
AMBA APB is optimized for minimal power consumption and reduced interface
complexity to support peripheral functions. APB can be used in conjunction with either
version of the system bus.


The overall architecture looks like the following:

![AMBA System]

## Basic Terminology

#### Bus cycle 
A bus cycle is a basic unit of one bus clock period and for the
purpose of AMBA AHB or APB protocol descriptions is defined
from rising-edge to rising-edge transitions. 

#### Bus transfer 
An AMBA ASB or AHB bus transfer is a read or write operation of a data object, which may take one or more bus cycles. The bus
transfer is terminated by a completion response from the
addressed slave.
An AMBA APB bus transfer is a read or write operation
of a data object, which always requires two bus cycles.

#### Burst operation 

A burst operation consists of one or more consecutive data transactions initiated by a bus master, where each
transaction maintains a consistent data width and sequentially accesses an incremental address space. The 
address increments are determined by the transfer size (byte, halfword, or word). Burst operations are not 
supported on the APB (Advanced Peripheral Bus).

##  AMBA Signals

### AMBA AHB Signals



| **Name** | **Source** | **Description** |  
|-----------|-----------|-----------|  
| HCLK | Clock Source | The main system clock that synchronizes all bus transfers. All signal timings are referenced to its rising edge. |  
| HRESETn | Reset Controller | An active LOW reset signal used to reset the system and the bus. This is the only active LOW signal in the system. |  
| HADDR[31:0] | Master | A 32-bit address bus used by the master to specify the memory or peripheral location for the current transfer. |  
| HTRANS[1:0] | Master | Defines the type of transfer: NONSEQUENTIAL, SEQUENTIAL, IDLE, or BUSY. |  
| HWRITE | Master | Indicates the transfer direction: HIGH for a write operation, LOW for a read operation. |  
| HSIZE[2:0] | Master | Specifies the data transfer size: byte (8-bit), halfword (16-bit), word (32-bit), or larger sizes up to 1024 bits. |  
| HBURST[2:0] | Master | Defines whether a transfer is part of a burst sequence, supporting 4, 8, or 16-beat bursts, either incrementing or wrapping. |  
| HPROT[3:0] | Master | Provides protection control information for bus access, typically used for security and privilege levels (optional). |  
| HWDATA[31:0] | Master | A 32-bit write data bus used to transfer data from the master to a slave during write operations. |  
| HSELx | Decoder | A slave select signal that identifies the target slave for the current transfer, derived through address decoding. |  
| HRDATA[31:0] | Slave | A 32-bit read data bus used to transfer data from a slave to the master during read operations. |  
| HREADY | Slave | Indicates the completion of a transfer: HIGH means the transfer is complete, while LOW extends the transfer duration. **Note:** Slaves require HREADY as both an input and output signal. |  
| HRESP[1:0] | Slave | Provides status information about a transfer with possible responses: OKAY, ERROR, RETRY, or SPLIT. |  


### Advanced Microcontroller Bus Architecture - Advanced Peripheral Bus signals


| **Name** | **Source** | **Description** |  
| ----------- | ----------- |  ----------- |
|PCLK  | Clock Source|The rising edge of PCLK is used to time all transfers on the APB.|
|PRESETn  | Reset Controller| The APB bus reset signal is active LOW and this signal will normally be connected directly to the system bus reset signal.|
|PADDR[31:0]  | Master |This is the APB address bus, which may be up to 32-bits wide and is driven by the peripheral bus bridge unit.|
|PSELx  | Decoder | A signal from the secondary decoder, within the peripheral bus bridge unit, to each peripheral bus slave x. This signal indicates that the slave device is selected and a data transfer is required. There is a PSELx signal for each bus slave.|
|PENABLE  |  Master | This strobe signal is used to time all accesses on the peripheral bus. The enable signal is used to indicate the second cycle of an APB transfer. The rising edge of PENABLE occurs in the middle of the APB transfer.|
|PWRITE  |  Master | When HIGH this signal indicates an APB write access and when LOW a read access.|
|PRDATA[31:0]  | Slave | The read data bus is driven by the selected slave during read cycles (when PWRITE is LOW). The read data bus can be up to 32-bits wide.|
|PWDATA[31:0]  | Master | The write data bus is driven by the peripheral bus bridge unit during write cycles (when PWRITE is HIGH). The write data bus can be up to 32-bits wide.|

# Implementation 

## Objective

To design and simulate a synthesizable AHB to APB bridge interface using Verilog and run single read and single write tests using AHB Master and APB Slave testbenches.
The bridge unit converts system bus transfers into APB transfers and performs the following functions: 
- Latches the address and holds it valid throughout the transfer.
- Decodes the address and generates a peripheral select, PSELx. Only one select signal can be active during a transfer.
- Drives the data onto the APB for a write transfer.
- Drives the APB data onto the system bus for a read transfer.
- Generates a timing strobe, PENABLE, for the transfer
- Can implement single read and write operations successfully.

The diagram below shows the interface:

![APB Bridge]

## Basic Implementation Tools

- HDL Used : Verilog
- Simulator Tool Used: ModelSIM
- Synthesis Tool Used: Quartus Prime
- Family: Cyclone V
- Device: 5CSXFC6D6F31I7ES

## Design Modules

### AHB Slave Interface

An AHB bus slave responds to transfers initiated by bus masters within the system. The 
slave uses a HSELx select signal from the decoder to determine when it should respond 
to a bus transfer. All other signals required for the transfer, such as the address and 
control information, will be generated by the bus master.

### APB Controller

The AHB to APB bridge comprises a state machine, which is used to control the 
generation of the APB and AHB output signals, and the address decoding logic which 
is used to generate the APB peripheral select lines.
  

## Notes  
 
**The design files are attached in the repository along with the AHB Master and APB Slave which generates the appropriate signals. Only the Bridge is synthesizable and other modules are used as testbenches only to generate the necessary read/write operations. Below are the screenshots from the synthesis and the simulator tool**

# Simulation Results

It consists of a write operation followed by a read operation on the AHB Bus which is successfully mapped to the APB bus according to the interfacing.

![simulation_AHBtoAPB] 

# Synthesis Results

RTL Model:


State Machine Viewer:








