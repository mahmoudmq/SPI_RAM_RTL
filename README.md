# SPI Slave with Synchronous RAM

## Overview
This project implements a Serial Peripheral Interface (SPI) Slave wrapper around a synchronous RAM. The SPI Slave interface allows an external SPI Master to read from and write to the internal RAM using a standard SPI communication protocol. The design is targeted and constrained for the Basys 3 FPGA board (as indicated by the `Constraints_basys3.xdc` file).

## Project Architecture
The project is composed of three main Verilog modules:
1. **`spi_wrapper.v`** (`spi_wrapper`): The top-level module that instantiates and connects the SPI Slave and the Synchronous RAM.
2. **`spi_slave.v`** (`spi_slave`): Implements the SPI protocol FSM (Finite State Machine). It interprets the serial `MOSI` input to capture 10-bit packets (2 command bits + 8 data/address bits) and drives the control signals (`rx_valid`, `tx_valid`) and parallel data buses for the RAM. When reading, it also serializes the output data from the RAM over the `MISO` line.
3. **`sync_ram.v`** (`ram`): A 256x8 bit synchronous block RAM module. It receives a 10-bit input (`din`) where the most significant 2 bits define the operation:
   - `00`: Store Write Address
   - `01`: Write Data (to the stored write address)
   - `10`: Store Read Address
   - `11`: Read Data (from the stored read address)

### FSM Design (`spi_slave.v`)
The SPI slave uses an FSM with the following states to process SPI transactions:
- `IDLE`: Waits for the Slave Select (`SS_n`) to be pulled low.
- `CHK_CMD`: Evaluates the first bit on `MOSI` to determine if the upcoming operation is a Write (`0`) or a Read (`1`).
- `WRITE`: Captures 10 bits of data for write operations (either Write Address or Write Data).
- `READ_ADD`: Captures 10 bits of data for the Read Address operation.
- `READ_DATA`: Captures 10 bits of data for the Read Data operation and simultaneously shifts out the requested read data on `MISO`.

## How to Run the Simulation
The project includes a testbench (`spi_wrapper_tb.v`) and a ModelSim/QuestaSim DO script (`run.do`) to automate the simulation process.

### Running with ModelSim/QuestaSim
1. Open ModelSim or QuestaSim.
2. Navigate to the project directory in the transcript/console:
   ```tcl
   cd d:/Digital_Design/Project2_SPI
   ```
3. Execute the provided simulation script:
   ```tcl
   do run.do
   ```
This script will automatically:
- Create a working library (`work`).
- Compile all Verilog source files (`spi_slave.v`, `sync_ram.v`, `spi_wrapper.v`, and `spi_wrapper_tb.v`).
- Load the simulation for the testbench (`spi_wrapper_tb`).
- Add relevant signals to the waveform viewer.
- Run the simulation to completion.

### Testbench Operations
The testbench (`spi_wrapper_tb.v`) simulates an SPI Master by driving the `clk`, `rst_n`, `MOSI`, and `SS_n` signals. It contains a self-checking sequence that performs the following:
1. **Write Address**: Sets the RAM write address to `0xF1`.
2. **Write Data**: Writes the data value `0x77` to the previously set address.
3. **Read Address**: Sets the RAM read address to `0xF1`.
4. **Read Data**: Reads the data from `0xF1` over `MISO` and verifies that it is `0x77`.

The testbench prints success or error messages to the simulation console using `$display`.

## Synthesis and Implementation
The directory contains a `Constraints_basys3.xdc` file, meaning this design can be readily synthesized and implemented on a Digilent Basys 3 FPGA using Xilinx Vivado.
