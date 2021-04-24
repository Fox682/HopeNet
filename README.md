# HopeNet
### Mesh Network made of HopeRF Radio Transcievers

A Mesh Network using HopeRF Radios. Self organizing network of nodes with software written in Python. HopeRF Modems are treated as Modems only, Packet based communication. All routing and logic is done by the Nodes and their own Internal State. This keeps the system easy to work with and makes swapping or upgrading radios easier.



### Parts List (BOM):
- HopeRF RM* Transcievers (Variety)
  - Soldering Wires/Headers needed
- RaspberryPi Zero W/WH
  - Solder Headers to the Pi
- Wiring to connect SPI lines
- Power Supply (Can be tied in with Pi?)
- Protoboard to wire components together?
- USB/SPI Adapter (Adafruit has one) for PC (Dev.)

### Software Used:
Python3
- Easy to use and maintain
- Ncurses for User Interface (depends on complexity)
- Find Library that makes sense (SpiDev, Python3-spi, etc)
- SQLite module

SQLite
- Lightweight database where all information will be stored and drawn from

### Multiple Python apps
- Radio Interface
  -Wire SPI
  - Using Python3-spi
  - 1Mhz SPI speed
  - Python3-spi interface
- Database Manager
  - SQLite works
- User Interface
  - Ncurses or Menu Based
- Client/Server Modes
  - Client Only
    - Request based, does not route or relay (Small Microcontrollers)
  - Relay Only
    - Relays Packets from one place to another (Gateway/Router) (More powerful Microcontrollers)
  - Client/Relay Hybrid
    - Access Services on other nodes while still relaying Packets as possible (*Best mode!* for RPi Zero)
  - BBS mode (Server)
    - Accepts Client Requests
    - Stateless system [Asset List & Request]

#### Network Protocol Concepts

- Determine Packet Format (KISS Method, minimal information)
- Basic Routing APRS style Routing [Node1,Node2,Node3]/[Wide,Node10] (Backup/Manual Mode)
  - B.A.T.M.A.N. is better!
    - Basic BATMAN Info [Batman @ Poly.edu](https://witestlab.poly.edu/blog/batman/)
  - >BATMAN nodes don't have to calculate the whole routes for their outgoing packets, and they don't have to know the full topology of the mesh network. In BATMAN, all nodes periodically broadcasts "hello" signals, also known as originator messages (OGM), to its neighbors. Each originator messages consists of an originator address, a sending node address and a unique sequence number. When an OGM is received, the receiving node changes the sending address to its own address and re-broadcasts the message. The sequence number is used to identify which of a pair of messages is newer. With this process, each node in the network learns its own direct neighbors, but also learns about other nodes that are not in range through a direct link but can be reached by hopping through a neighbor.
- Batman OGM will keep the routing table small for Microcontrollers 
- Link Quality can be determined by a combination of Response (ping), Packet Loss (PL) and RSSI.
  - Basic Math needed for this
  - eg. Link with very low response time with *some* PL may be preferred over a link that has no PL but a very high response time
  - Stronger Links Preferred (Lower RSSI)
-----------

- Originator massages
-- Use these to build routes

- Prefer Strong Links vs
- Return to sender only

- Link Grading Algo.
  - Packet loss higher weighted
  - Response Time lower rated
  - otherwise if 
    - N1 PL is 0% and ping is 2000ms
    - N2 PL is 50% and ping is < 1000ms
    - Both are equal in capability (maybe)

Ranked Algo by points 8-bit value
- PL by % 
- Ping (unknown)
  - can be determined by:
    - Transmit time of PING pkt
      - Full packet + preamble & crc
      - Plus recovery time of radio
      - Plus transmit time of PONG
  - Ping packet x2 + recovery time
    - Find packet size


------------------------

#### Packet Config
  - See Radio documentation
- Preamble can be short
  - A few bytes!
- SyncWord can be the same for all radios
  - 1 Byte is ok
- AddressByte is optional
- Payload max 66 Bytes (aim is ~50)
- CRC is 2 bytes

[Preamble][SyncWord][AddressByte][Payload][CRC]

- AddressByte is optional 
  - Put Address in Payload(!)

[Preamble][SyncWord][Length][Payload][CRC]

[101010... 3Bytes][d204 0xCC 2Bytes][Length Byte][1 Byte Min. To 50 bytes] [CRC 2Bytes]

Overhead packet size
50 bytes payload = 58 Bytes max.
1 byte payload = 9 bytes min.
  - Time @ 1200bps/150Bps
  - 1 bit = 0.833ms
  - Max pkt = ~48ms
  - Min pkt = ~7.5ms
  - Min Overhead 8 Bytes = ~6.7ms
    - Not including payload
    - For calculating packet transit times

Packets are variable in size
- Add length byte (needed)
  - Assume length byte counts bytes

------------------------

Packet types:
[Pkt Type] = 8 bits / 1 byte
- 0-255 types available

OGM
- Advertisement of node existence
- Send: [Pkt. Type][Node Add.][Node Type]
- Send Intermittently
  - After receiving OGM an message
    - Only if no OGM sent for >5 mins
  - Max. Every 5 mins +/- random 30 secs + queue (adjust higher?)
  - Max pkt size = 5 bytes to advertise
Time (pkt+ovh)= 33.3ms + 6.7ms = ~40ms per node

PING 
- [Pkt. Type][Dest. Addr][Src. Addr]
- Type: Ping Req. / Ping Resp.
- Max. Pkt size for ping = 5 Bytes
- Time (pkt+ovh)= 33.3ms + 6.7ms = ~40ms x2 (ping/pong) is ~80ms

Data
- Need Data Queue
- Split data to multiple pkts.
- Seq# needed (8-bit 0-255)
- Packet type is Data Request + Type
- [Pkt. Type][Dest.][Src.][Data Size]
- Data Types?
  - Short data
  - Long data

Max data size (255 pkts/12.4KBs/12.2sec.) and timeout (15 secs?)

------------------------

- Need to approve data send requests
- Should not tie up the spectrum unless systems are ready

Ack. Pkts. (Acknowledgment Packets)
- Data Request TX/RX Response
  - Data TX (Ready to Send)
  - Data TX OK Standby (Medium busy)
- Receiving system
  - Data RX OK (System Ready, Go!)
  - Data RX NOK Standby (System busy)
  - Data RX NOK (Reject)

- Data Received Verification
  - Verify with Pkt Seq#'s received
  - Req. Missing packet numbers
    - Dump Duplicates
  - Modem will verify CRCs for us


------------
**NOTE:**
Email from HopeRF:
- Data Whitening **AND** CRC *may not be compatible* between different Modules! (ie RFM22 vs RFM69)
- Need to find simplest Whitening for DC-Free in Python
  - Remove CRC pad, Packet will have to contain this to verify data integrity
  - This can make the system more flexible
    - Could use other methods to verify data


#### Alpha:
- Information gathering
  - "Ping" - ICMP Style communication verification system (Can we connect to the other node?)
  - Route tracing
- Network and Link Discovery (build node list)
- Basic Routing
- Infrastructure to build services on top of

#### Beta:
- Basic Service Introduction
  - Direct Messaging (Texting)
  - Keyboard to Keyboard communication (PC or RPi Zero to another node)

#### Release Candidate
- Services Available and functional
  - BBS Style Services
    - Email (indirect messaging with storage... maybe)
    - Forum (Threaded Public Messaging)
    - Bulletin Board Messaging 
    - Guest Book Signing 

#### Advanced
- Future Features
  - Voice Messages
  - Text Only Internet Access





