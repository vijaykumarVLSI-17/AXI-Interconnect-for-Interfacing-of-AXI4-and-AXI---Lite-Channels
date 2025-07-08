# AXI-Interconnect-for-Interfacing-of-AXI4-and-AXI---Lite-Channels
# AXI Interconnect (Basic Router)

This repository contains a Verilog module implementing a basic AXI (Advanced eXtensible Interface) Interconnect. Its primary function is to route AXI transactions from a single AXI Master to multiple AXI Slave interfaces based on address decoding. It supports both full AXI4 and AXI-Lite protocols for the connected slaves.

## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [AXI Protocol Overview](#axi-protocol-overview)
- [Address Map](#address-map)
- [Module Parameters](#module-parameters)
- [Input/Output Ports](#inputoutput-ports)
- [Internal Logic Overview](#internal-logic-overview)
- [Usage](#usage)
- [Limitations and Future Improvements](#limitations-and-future-improvements)
- [Contributing](#contributing)
- [License](#license)

## Introduction

In complex System-on-Chip (SoC) designs, multiple masters (e.g., CPU, DMA controller) need to communicate with various slave devices (e.g., memory, peripherals). An AXI Interconnect acts as a central routing hub, directing transactions from masters to the correct slaves based on their address ranges. This module provides a foundational example of such an interconnect, handling address decoding and handshake signaling for a single master and two types of slaves.

## Features

* **Address-Based Routing:** Routes AXI transactions to different slave interfaces based on the target address.
* **AXI4 and AXI-Lite Support:** Connects to one full AXI4 slave and one AXI-Lite slave.
* **Parameterized Design:** Highly configurable parameters for data width, address width, burst length, and memory mapping.
* **Separate Channels:** Implements logic for all five AXI channels (AW, W, B, AR, R).
* **Basic Handshake Management:** Handles `VALID`/`READY` handshake signals for all channels.
* **Burst Transaction Support:** Manages `AWLEN`/`ARLEN` and `WLAST`/`RLAST` for burst operations (for AXI4 slave).

## AXI Protocol Overview

The AXI protocol is a channel-based interface. Each channel uses a `VALID`/`READY` handshake mechanism to ensure reliable data transfer. A transfer occurs only when both `VALID` (from source) and `READY` (from destination) are asserted.

* **AW Channel (Write Address):** Master sends address and control for a write.
* **W Channel (Write Data):** Master sends write data.
* **B Channel (Write Response):** Slave sends response indicating write completion.
* **AR Channel (Read Address):** Master sends address and control for a read.
* **R Channel (Read Data):** Slave sends read data and response.

## Address Map

The interconnect uses the following default address map to determine which slave receives the transaction:

| Slave Type          | Base Address       | End Address        | Size     | Notes                               |
| :------------------ | :----------------- | :----------------- | :------- | :---------------------------------- |
| **AXI4 Slave**      | `32'h0000_0000`    | `32'h0000_FFFF`    | 64 KB    | Typically for Block RAM (BRAM)      |
| **AXI-Lite Slave**  | `32'h4000_0000`    | `32'h4000_FFFF`    | 64 KB    | For AXI-Lite peripheral controller  |
| *SPI Peripheral*    | `32'h4000_1000`    | `32'h4000_1FFF`    | 4 KB     | (Within AXI-Lite range)             |
| *GPIO Peripheral*   | `32'h4000_2000`    | `32'h4000_2FFF`    | 4 KB     | (Within AXI-Lite range)             |
| *USB Peripheral*    | `32'h4000_3000`    | `32'h4000_3FFF`    | 4 KB     | (Within AXI-Lite range)             |
| *I2C Peripheral*    | `32'h4000_4000`    | `32'h4000_4FFF`    | 4 KB     | (Within AXI-Lite range)             |
| *UART Peripheral*   | `32'h4000_5000`    | `32'h4000_5FFF`    | 4 KB     | (Within AXI-Lite range)             |
| **External Memory** | `32'hA000_0000`    | `32'hA0FF_FFFF`    | 16 MB    | Optional, currently not routed      |

## Module Parameters

You can customize the interconnect by modifying the `parameter` values:

| Parameter Name           | Default Value | Description                                          |
| :----------------------- | :------------ | :--------------------------------------------------- |
| `DATA_WIDTH`             | `32`          | Width of the AXI data bus.                           |
| `STRB_WIDTH`             | `(DATA_WIDTH/8)` | Width of the write strobe (byte enable).           |
| `ADDR_WIDTH`             | `32`          | Width of the AXI address bus.                        |
| `BURST_TYPE`             | `2`           | Width of `AWBURST`/`ARBURST` (AXI burst type).       |
| `BURST_LEN`              | `8`           | Width of `AWLEN`/`ARLEN` (AXI burst length).         |
| `BEAT_SIZE`              | `3`           | Width of `AWSIZE`/`ARSIZE` (AXI beat size).          |
| `RESP_WIDTH`             | `2`           | Width of `BRESP`/`RRESP` (AXI response).             |
| `ID`                     | `5`           | Width of `AWID`/`ARID`/`BID`/`RID` (transaction ID). |
| `addr_mem_width`         | `50`          | Internal width for buffering AW channel info.        |
| `data_mem_width`         | `37`          | Internal width for buffering W channel info.         |
| `raddr_mem_width`        | `50`          | Internal width for buffering AR channel info.        |
| `BRAM_BASE_ADDR`         | `32'h0000_0000` | Base address for the AXI4 BRAM slave.              |
| `BRAM_ADDR_SIZE`         | `32'h0001_0000` | Size of the BRAM address space (64 KB).            |
| `SLAVE_CONTLR_BASE_ADDR` | `32'h4000_0000` | Base address for the AXI-Lite peripheral controller. |
| `SLAVE_CONTLR_ADDR_SIZE` | `32'h0001_0000` | Size of the AXI-Lite controller address space (64 KB). |
| `SPI_BASE_ADDR`          | `32'h4000_1000` | Base address for SPI peripheral.                     |
| `GPIO_BASE_ADDR`         | `32'h4000_2000` | Base address for GPIO peripheral.                    |
| `USB_BASE_ADDR`          | `32'h4000_3000` | Base address for USB peripheral.                     |
| `I2C_BASE_ADDR`          | `32'h4000_4000` | Base address for I2C peripheral.                     |
| `UART_BASE_ADDR`         | `32'h4000_5000` | Base address for UART peripheral.                    |
| `EXTERNAL_MEM_BASE_ADDR` | `32'hA000_0000` | Base address for optional external memory.           |
| `EXTERNAL_MEM_ADDR_SIZE` | `32'h0100_0000` | Size of external memory address space (16 MB).       |

## Input/Output Ports

The module exposes standard AXI signals for connection to a master and two slaves.

* **Master Interface (I_in_*, I_out_*)**:
    * `I_in_AWADDR`, `I_in_AWBURST`, `I_in_AWLEN`, `I_in_AWSIZE`, `I_in_AWID`, `I_in_AWVALID`
    * `I_out_AWREADY`
    * `I_in_WDATA`, `I_in_WSTRB`, `I_in_WLAST`, `I_in_WVALID`
    * `I_out_WREADY`
    * `I_out_BRESP`, `I_out_BID`, `I_out_BVALID`
    * `I_in_BREADY`
    * `I_in_ARADDR`, `I_in_ARBURST`, `I_in_ARLEN`, `I_in_ARSIZE`, `I_in_ARID`, `I_in_ARVALID`
    * `I_out_ARREADY`
    * `I_out_RDATA`, `I_out_RID`, `I_out_RLAST`, `I_out_RVALID`
    * `I_in_RREADY`

* **AXI4 Slave Interface (AXI4_out_*, AXI4_in_*)**:
    * `AXI4_out_AWADDR`, `AXI4_out_AWBURST`, `AXI4_out_AWLEN`, `AXI4_out_AWSIZE`, `AXI4_out_AWID`, `AXI4_out_AWVALID`
    * `AXI4_in_AWREADY`
    * `AXI4_out_WDATA`, `AXI4_out_WSTRB`, `AXI4_out_WLAST`, `AXI4_out_WVALID`
    * `AXI4_in_WREADY`
    * `AXI4_in_BRESP`, `AXI4_in_BID`, `AXI4_in_BVALID`
    * `AXI4_out_BREADY`
    * `AXI4_out_ARADDR`, `AXI4_out_ARBURST`, `AXI4_out_ARLEN`, `AXI4_out_ARSIZE`, `AXI4_out_ARID`, `AXI4_out_ARVALID`
    * `AXI4_in_ARREADY`
    * `AXI4_in_RDATA`, `AXI4_in_RID`, `AXI4_in_RLAST`, `AXI4_in_RVALID`
    * `AXI4_out_RREADY`

* **AXI-Lite Slave Interface (lite_out_*, lite_in_*)**:
    * `lite_out_AWADDR`, `lite_out_AWBURST`, `lite_out_AWLEN`, `lite_out_AWSIZE`, `lite_out_AWID`, `lite_out_AWVALID`
    * `lite_in_AWREADY`
    * `lite_out_WDATA`, `lite_out_WSTRB`, `lite_out_WLAST`, `lite_out_WVALID`
    * `lite_in_WREADY`
    * `lite_in_BRESP`, `lite_in_BID`, `lite_in_BVALID`
    * `lite_out_BREADY`
    * `lite_out_ARADDR`, `lite_out_ARBURST`, `lite_out_ARLEN`, `lite_out_ARSIZE`, `lite_out_ARID`, `lite_out_ARVALID`
    * `lite_in_ARREADY`
    * `lite_in_RDATA`, `lite_in_RID`, `lite_in_RLAST`, `lite_in_RVALID`
    * `lite_out_RREADY`

## Internal Logic Overview

The interconnect manages transactions using dedicated state machines for the write address, write data, write response, read address, and read data channels.

1.  **Write Address (AW) State Machine (`write_addr_state`):**
    * `AW_IDLE`: Waits for a new write address request from the master.
    * `AW_ADDR_DECODE`: Decodes the incoming address to determine the target slave (AXI4 BRAM or AXI-Lite controller).
    * `ROUTE_AXI`: Forwards the AW transaction to the AXI4 slave.
    * `ROUTE_LITE`: Forwards the AW transaction to the AXI-Lite slave.
    * Transaction details are buffered in `addr_mem` while routing occurs.

2.  **Write Data (W) State Machine (`write_data_state`):**
    * `W_IDLE`: Waits for write data beats from the master.
    * `ROUTE_AXI_DATA`: Forwards write data to the AXI4 slave.
    * `ROUTE_LITE_DATA`: Forwards write data to the AXI-Lite slave.
    * Write data and strobe information are buffered in `data_mem`.

3.  **Write Response (B) State Machine (`write_resp_state`):**
    * `B_IDLE`: Waits for write responses from the connected slaves.
    * `AXI4_RESP`: Forwards the AXI4 slave's write response to the master.
    * `LITE_RESP`: Forwards the AXI-Lite slave's write response to the master.

4.  **Read Address (AR) State Machine (`read_raddr_state`):**
    * `AR_IDLE`: Waits for a new read address request from the master.
    * `AR_ADDR_DECODE`: Decodes the incoming address to determine the target slave.
    * `AR_ROUTE_AXI`: Forwards the AR transaction to the AXI4 slave.
    * `AR_ROUTE_LITE`: Forwards the AR transaction to the AXI-Lite slave.
    * Transaction details are buffered in `raddr_mem`.

5.  **Read Data (R) State Machine (`read_data_state`):**
    * `R_IDLE`: Waits for read data beats from the connected slaves.
    * `ROUTE_AXI4`: Forwards read data from the AXI4 slave to the master.
    * `ROUTE_lite`: Forwards read data from the AXI-Lite slave to the master.
    * Read data, ID, and `RLAST` are buffered in internal registers.

## Usage

To use this AXI Interconnect module in your Verilog design:

1.  **Instantiate the module:**
    ```verilog
    Interconnect_test_dsgn #(
        .DATA_WIDTH(32),
        .ADDR_WIDTH(32),
        // ... other parameters as needed
    ) my_interconnect (
        .ACLK(sys_clk),
        .ARESETn(sys_resetn),
        // Connect AXI Master signals
        .I_in_AWADDR(master_awaddr),
        // ...
        // Connect AXI4 Slave signals
        .AXI4_out_AWADDR(bram_awaddr),
        // ...
        // Connect AXI-Lite Slave signals
        .lite_out_AWADDR(periph_awaddr)
        // ...
    );
    ```
2.  **Connect your AXI Master:** Connect the master's AXI output signals to the `I_in_*` ports and the master's AXI input signals to the `I_out_*` ports.
3.  **Connect your AXI4 Slave:** Connect the AXI4 slave's AXI input signals to the `AXI4_out_*` ports and the slave's AXI output signals to the `AXI4_in_*` ports.
4.  **Connect your AXI-Lite Slave:** Connect the AXI-Lite slave's AXI input signals to the `lite_out_*` ports and the slave's AXI output signals to the `lite_in_*` ports.

Ensure your master generates addresses that fall within the defined address map for the transactions to be routed correctly.

## Limitations and Future Improvements updates under the progress...

This basic interconnect serves as a starting point. For a production-ready design, consider these enhancements:

* **Arbitration:** Implement arbitration logic to support multiple AXI masters.
* **Quality of Service (QoS):** Add mechanisms to prioritize certain transactions.
* **Error Handling:** More robust handling of AXI error responses (e.g., generating `DECERR` for unmapped addresses, logging errors).
* **Burst Splitting/Merging:** Optimize burst transactions for different slave capabilities.
* **Deeper Buffering/FIFOs:** Implement deeper FIFOs on channels to improve throughput and decouple master/slave latencies.
* **Registering Outputs:** Registering output signals (`I_out_BRESP`, `I_out_BID`, `I_out_RDATA`, `I_out_RID`, `I_out_RLAST`) for better timing closure.
* **Clock Gating/Power Management:** Add logic for power optimization.
* **Configurable Slave Interfaces:** Make the number and type of slave interfaces configurable.

## Contributing

Contributions are welcome! If you find bugs or have suggestions for improvements, please open an issue or submit a pull request.

## License

This project is open-source and available under the [MIT License](LICENSE).
