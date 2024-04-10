---
hide:
  - toc
---


```bash
mkdir BYOS & cd BYOS
```


```nano sat.py```


```python
import socket
import random
import struct

# UDP IP and Ports
LISTEN_IP = "0.0.0.0"
COMMAND_PORT = 1235
TELEMETRY_PORT = 1234
TELEMETRY_IP = "openc3-operator"  # Change if the client is on a different machine

# Simulated satellite state
satellite_mode = "NORMAL"

# Create a UDP socket for commands
sock_command = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock_command.bind((LISTEN_IP, COMMAND_PORT))

# Create a UDP socket for telemetry
sock_telemetry = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

tlm_ids = {
    "STATUS": 0x4320,
    "PING": 0x4321,
    "GET_TEMP": 0x4322,
    "SET_MODE": 0x4323,
    "REBOOT": 0x4324,
    "ERROR": 0x4325
}

print("Satellite simulation started. Waiting for commands...")

def send_telemetry(command,message):
    id = tlm_ids[command]
    fmt = '>h12s'
    packed = struct.pack(fmt, id, message.encode())
    sock_telemetry.sendto(packed, (TELEMETRY_IP, TELEMETRY_PORT))
    print(f"Sent telemetry: {message}")

while True:
    # Listen for commands
    data, addr = sock_command.recvfrom(1024)  # buffer size is 1024 bytes
    command = data.decode()
    print(f"Received command: {command} from {addr}")

    # Process commands
    if command == "STATUS":
        send_telemetry(command,"STATUS:OK")
    elif command == "PING":
        send_telemetry(command,"PING:ACK")
    elif command == "GET_TEMP":
        # Simulate a temperature reading
        temp = random.randint(-50, 50)
        send_telemetry(command,f"TEMP:{temp}C")
    elif command == "SET_MODE":
        # Set mode and confirm
        satellite_mode = "SAFE"
        send_telemetry(command,f"MODE:{satellite_mode}")
    elif command == "REBOOT":
        # Simulate a reboot sequence
        send_telemetry(command,"REBOOTING")
        satellite_mode = "NORMAL"  # Reset to NORMAL mode after reboot
        send_telemetry(command,"REBOOTED:OK")
    else:
        # Unknown command
        send_telemetry("ERROR","UNKNOWN_CMD")
```


```bash
nano Dockerfile
```


```Dockerfile
# Use an official Python runtime as a parent image
FROM python:3.8-slim

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy the current directory contents into the container at /usr/src/app
COPY . .

# Run the server when the container launches
CMD ["python","-u", "./sat.py"]
```

```bash
docker build -t byos .
```


```bash
docker run --net=openc3-cosmos-network --name byos -p1234:1234/udp -p1235:1235 --rm byos
```


```bash
cd ~/cosmos
```

```bash
./openc3.sh cli generate plugin BYOS
```

```bash
cd openc3-cosmos-byos
```

```bash
../openc3.sh cli generate target BYOS
```

```nano plugin.txt```


```
VARIABLE ip 127.0.0.1
VARIABLE port_tm 1234
VARIABLE port_tc 1235
VARIABLE byos_target_name BYOS

TARGET BYOS <%= byos_target_name %>
INTERFACE <%= byos_target_name %>_INT udp_interface.rb <%= ip %> <%= port_tc %> <%= port_tm %> nil nil 128 nil nil
  MAP_TARGET <%= byos_target_name %>
```


```bash
cd targets/BYOS/cmd_tlm/
```

```bash
nano cmd.txt
```

```
COMMAND BYOS PING BIG_ENDIAN "Ping Satellite"
  APPEND_PARAMETER CMD_STRING 0 STRING "PING" "Ping Command"

COMMAND BYOS STATUS BIG_ENDIAN "Get Status"
  APPEND_PARAMETER CMD_STRING 0 STRING "STATUS" "Command to Query Status"

COMMAND BYOS GET_TEMP BIG_ENDIAN "Get TEMP"
  APPEND_PARAMETER CMD_STRING 0 STRING "GET_TEMP" "Command to Query Temp"
  
COMMAND BYOS SET_MODE BIG_ENDIAN "Set Satellite Mode"
  APPEND_PARAMETER CMD_STRING 0 STRING "SET_MODE" "Command to Set Mode of Satellite"

COMMAND BYOS REBOOT BIG_ENDIAN "REBOOT Satellite"
  APPEND_PARAMETER CMD_STRING 0 STRING "REBOOT" "Reboot Satellite"
```

```bash
nano tlm.txt
```

```
TELEMETRY BYOS STAUS BIG_ENDIAN "STATUS PKT"
  APPEND_ID_ITEM PACKET_ID 16 UINT 0x4320 "PACKET ID"
    FORMAT_STRING "0X%04X"
  APPEND_ITEM RESULT 96 STRING "RESPONSE"

TELEMETRY BYOS PING BIG_ENDIAN "PING PKT"
  APPEND_ID_ITEM PACKET_ID 16 UINT 0x4321 "PACKET ID"
    FORMAT_STRING "0X%04X"
  APPEND_ITEM RESULT 96 STRING "RESPONSE"

TELEMETRY BYOS TEMP BIG_ENDIAN "TEMP PKT"
  APPEND_ID_ITEM PACKET_ID 16 UINT 0x4322 "PACKET ID"
    FORMAT_STRING "0X%04X"
  APPEND_ITEM RESULT 96 STRING "RESPONSE"

TELEMETRY BYOS MODE BIG_ENDIAN "MODE PKT"
  APPEND_ID_ITEM PACKET_ID 16 UINT 0x4323 "PACKET ID"
    FORMAT_STRING "0X%04X"
  APPEND_ITEM RESULT 96 STRING "RESPONSE"

TELEMETRY BYOS REBOOT BIG_ENDIAN "REBOOT PKT"
  APPEND_ID_ITEM PACKET_ID 16 UINT 0x4324 "PACKET ID"
    FORMAT_STRING "0X%04X"
  APPEND_ITEM RESULT 96 STRING "RESPONSE"

TELEMETRY BYOS ERROR BIG_ENDIAN "ERROR PKT"
  APPEND_ID_ITEM PACKET_ID 16 UINT 0x4325 "PACKET ID"
    FORMAT_STRING "0X%04X"
  APPEND_ITEM RESULT 96 STRING "RESPONSE"
  ```


```bash
cd ../../../
../openc3.sh cli rake build VERSION=1.0.0 .
```


