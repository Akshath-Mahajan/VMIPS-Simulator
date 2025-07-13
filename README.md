# Vector-Simulator

This is a functional simulator for the presented vector ISA following the prescribed specifications. The machine is reminiscent of VMIPS but we will not consider off-chip data movement for simplicity.

The simulator is implemented in Python 3, without using any external libraries. To limit engineering effort, we are directly processing assembly instructions, and hence there is no need to consider actual instruction encodings and machine code.

## Input

The simulator takes the following files as inputs:
- `Code.asm`: The file contains the assembly code for the test function.
- `SDMEM.txt`: The file contains the initial state of the SDMEM containing the data required for the
test function in integer format. Each line in this file represents one word (32 bit) of data in the SDMEM.
- `VDMEM.txt`: The file contains the initial state of the VDMEM containing the data required for the
test function in integer format. Each line in this file represents one word (32 bit) of data in the VDMEM.

## Output
The simulator outputs these files:
- `VRF.txt`: The file shows the final state of the Vector Register File after the execution of all the
instructions in the input `Code.asm` file. Each line contains a comma separated list of integer values showing all the elements of a vector register.
- `SRF.txt`: The file shows the final state of the Scalar Register File after the execution of all the instructions in the input `Code.asm` file. Each line contains an integer value showing a scalar register data.
- `SDMEMOP.txt`: The file contains the final state of the SDMEM after the execution of the `Code.asm` file.
- `VDMEMOP.txt`: The file contains the final state of the VDMEM after the execution of the `Code.asm` file.

## ISA Specification

Vector length of 64, 32b integer elements.

The architectural state of the vector processor has the following components: 
- VDMEM: Vector Data Memory with a capacity of 512 KB, word addressable. 
- SDMEM: Scalar Data Memory with a capacity of 32 KB, word addressable. 

Register Files:

- Scalar Register File: 8 Scalar Registers of 32 bits each.
- Vector Register File: 8 Vector Registers each with a maximum capacity of 2048 bits or 256 bytes or 64
32-bit elements. Maximum Vector Length is set to 64.
- Vector Mask Register: 1 Vector Mask Register also known as the Flag Register. Contains 64 1 bit values.
The vector operations are valid only for the elements with corresponding flag register value set. Clear this
register if all the elements are valid.
- Vector Length Register: 1 Vector Length Register of size 32 bits to contain the number of vector element
operations. Set this to MVL if all the elements of the vector register inputs are to be evaluated.

# Vector Processor - Timing Simulator

__Authors:__

- Akshath Mahajan (avm6288)
- Rugved Mhatre (rrm9598)

## VMIPS ISA Specifications

- Vector Length - 64
- Vector Data Memory - 512KB, word addressable
- Scalar Data Memory  - 32KB, word addressable
- Scalar Register File - 8 registers, 32-bit elements
- Vector Register File - 8 registers, 64 32-bit elements
- Vector Lenth Register - 1 register, 32-bit
- Vector Mask Register - 1 register, 64 1-bit elements

## VMIPS Architecture

![Vector Processor Block Diagram](https://github.com/rugvedmhatre/Vector-Timing-Simulator/blob/main/images/Vector-Block-Diagram.jpg?raw=true)

*Vector Processor Block Diagram*

- 1 Scalar Functional Unit
- 1 Vector Load/Store Functional Unit
- 1 Vector Add/Subtract Functional Unit
- 1 Vector Multiply Functional Unit
- 1 Vector Divide Functional Unit
- 1 Vector Shuffle Functional Unit

### Base Configuration
- Dispatch Queue Configuration:
    - Vector Data Queue Depth - 4
    - Vector Compute Queue Depth - 4
    - Scalar Compute Queue Depth - 4
- Vector Data Memory Configuration:
    - Vector Data Memory Banks - 16
    - Vector Data Memory Bank Busy Time - 2
    - Vector Load/Store Pipeline Depth - 11
- Vector Compute Configuration:
    - Vector Lanes - 4
    - Vector Add/Sub Pipeline Depth - 2
    - Vector Multiply Pipeline Depth - 12
    - Vector Divide Pipeline Depth - 8
    - Vector Shuffle Pipeline Depth - 5

Furthermore, the VRFs have only 1 Read and 1 Write port. Hence, two instructions simultaneously reading the same Vector Register is not supported.

### Example 1

Here is an example of the execution time of a simple load vector program.

#### Code

```
LV VR1 SR0
HALT
```

#### Execution Flow

- In the $1^{st}$ cycle, `LV VR1 SR0` is fetched. 
- In the $2^{nd}$ cycle, `HALT` is fetched, and `LV VR1 SR0` is decoded, checked for strucutral hazards, and since there are no hazards, it is sent to the dispatch queue.
- In the $3^{rd}$ cycle, `HALT` is decoded, and is stalled, as it will wait for all the current instructions to be completed. `LV VR1 SR0` instruction is popped off the queue and starts executing. The pipeline depth for vector load instruction is 11 cycles, so 1 execution cycle completes in this cycle and now 10 cycles remain.
- In the $13^{th}$ cycle, `LV VR1 SR0` instruction triggers the Vector Data Memory Bank for the $1^{st}$ element address. The bank busy time is 2 cycles, so 1 cycle completes in this cycle, and now in next cycle the $1^{st}$ element is populated in the `VR1` register.
- In the $14^{th}$ cycle, we have populated the $1^{st}$ element in the `VR1` register, and triggered the Vector Data Memory Bank for the $2^{nd}$ element address.
- In the $77^{th}$ cycle, we have populated the $64^{th}$ element in the `VR1` register, and it is cleared off the busy board. `HALT` instruction stall is removed, and is moved to the dispatch queue.
- In the $78^{th}$ cycle, the `HALT` instruction is executed and the program stops.

![Example 1 Timing Diagram](https://github.com/rugvedmhatre/Vector-Timing-Simulator/blob/main/images/Example1_Timing_Diagram.png?raw=true)

*Example 1 : Timing Diagram*

![Example 1 Timing Diagram - Memory Bank View](https://github.com/rugvedmhatre/Vector-Timing-Simulator/blob/main/images/Example1_Timing_Diagram_Memory_Bank_View.png?raw=true)

*Example 1 : Timing Diagram - Memory Bank View*

## Running Timing Simulator

1. Execute the functional simulator to generate the resolved code flow, and to verify the functioning of the Vector Processor.
    
    ```
    python rrm9598_avm6288_funcsimulator.py --iodir test_cases/test_0
    ```

2. Execute the timing simulator to verify the timing performance of the Vector Processor.

    ```
    python rrm9598_avm6288_timingsimulator.py --iodir test_cases/test_0
    ```

3. To generate the Timing Diagram CSV file, use the following code. However, note that, for large programs like fully connected layer and convolution layer, the CSV file size is very large (in GB), and the code execution takes much longer.

    ```
    python rrm9598_avm6288_timingsimulator.py --iodir test_cases/test_0 --timing Y
    ```