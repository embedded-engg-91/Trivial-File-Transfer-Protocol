# TFTP (Trivial File Transfer Protocol)

A client-server implementation of TFTP in C using UDP socket programming.

## Features

### Client

* Interactive command-line client.
* `connect <IP>` — connects to the TFTP server.
* `get <filename>` — downloads a file from the server.
* `put <filename>` — uploads a file to the server.
* Uses `octet` mode for file transfers.
* `quit` / `bye` — exits the client.
* Validates the server IP address before connecting.
* Prevents file transfer operations until a server connection is established.

### Server

* UDP-based TFTP server.
* Continuously listens for incoming client requests.
* Handles both **RRQ (Read Request)** and **WRQ (Write Request)**.
* Creates a separate process for handling client transfers.
* Uses a separate transfer port for each client connection.
* Supports simultaneous transfer handling through process-based execution.

### TFTP Packet Handling

The implementation supports the following packet types:

| Opcode | Packet               |
| -----: | -------------------- |
|    `1` | RRQ — Read Request   |
|    `2` | WRQ — Write Request  |
|    `3` | DATA                 |
|    `4` | ACK — Acknowledgment |
|    `5` | ERROR                |

Packets are formed and parsed separately using dedicated functions for each packet type.

### File Transfer

* Transfers files in **512-byte data blocks**.
* Uses sequential block numbers to maintain transfer order.
* Each DATA block is acknowledged before the next block is transmitted.
* Detects the final block when the transferred data is less than 512 bytes.
* Supports both file upload and download.
* Handles duplicate DATA packets by retransmitting the corresponding ACK.
* Detects unexpected block numbers.

### Timeout & Retransmission

* UDP receive timeout is configured for transfer operations.
* Retransmission is performed when a timeout occurs.
* Maximum retry attempts are limited to **3**.
* RRQ and WRQ requests can be retransmitted if the initial response is not received.
* DATA packets are retransmitted when the expected ACK is not received.

### Transfer ID Handling

* The server assigns a separate port for individual transfers.
* The client records the transfer address after the initial server response.
* Packets received from an unexpected IP address or port are rejected/ignored.
* This helps ensure that packets belong to the correct transfer.


## Project Structure

| File               | Responsibility                                                         |
| ------------------ | ---------------------------------------------------------------------- |
| `client.c`         | Client interface and GET/PUT transfer logic                            |
| `server.c`         | Server loop, request handling, and transfer logic                      |
| `tftp.h`           | TFTP constants, packet definitions, opcodes, and error codes           |
| `tftp_pkt_form.c`  | Forms RRQ, WRQ, DATA, ACK, and ERROR packets                           |
| `tftp_pkt_parse.c` | Parses and validates received TFTP packets                             |
| `udp.c`            | UDP socket creation, binding, sending, receiving, and timeout handling |
| `udp.h`            | UDP function declarations                                              |
| `file.c`           | File open, read, write, and close operations                           |
| `file.h`           | File operation declarations                                            |

## Build

Compile the client:

```bash
gcc client.c udp.c file.c tftp_pkt_form.c tftp_pkt_parse.c -o client
```

Compile the server:

```bash
gcc server.c udp.c file.c tftp_pkt_form.c tftp_pkt_parse.c -o server
```

## Run

Start the server:

```bash
./server
```

Start the client in another terminal:

```bash
./client
```

Example:

```text
connect 127.0.0.1
put test.txt
get test.txt
quit
```

## Author

**Manoj B.**
