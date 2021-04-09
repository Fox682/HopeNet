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
  - 4-Wire SPI
  - Using Python3-spi
  - 1Mhz SPI speed
  - Python3-spi interface
- Database Manager
  - SQLite works
- User Interface
  - Ncurses or Menu Based
- Client/Server Modes
  - Client only
    - Request based
  - Relay Only
    - Relays Packets from one place to another (Gateway/Router)
  - Client/Relay Hybrid
    - Access Services on other nodes while still relaying Packets as possible (*Best mode!*)
  - BBS mode (Server)
    - Accepts Client Requests
    - Stateless system [ClientID:Asset Request]

#### Network Protocol Concepts

#### Alpha:
- Determine Packet Format (KISS Method, minimal information)
- "Ping" - ICMP Style communication verification system (Can we connect to the other node?)
- Basic Routing APRS style Routing [Node1,Node2,Node3]/[Wide,Node10]
- RSSI Based Logic to prefer Stonger Links (Meshy!)

#### Beta:
- Message Passing (Texting, SMS Style)
- Keyboard to Keyboard communication (PC or RPi Zero to another node)







