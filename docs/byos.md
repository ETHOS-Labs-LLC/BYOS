---
hide:
  - toc
---




Client
```
import socket

# Server address and ports
SERVER_IP = "localhost"
COMMAND_PORT = 1234
TELEMETRY_PORT = 1235

# Create a UDP socket for commands
sock_command = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# Create a UDP socket for telemetry
sock_telemetry = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock_telemetry.bind((SERVER_IP, TELEMETRY_PORT))

# Send a command to the satellite
command = "STATUS"
sock_command.sendto(command.encode(), (SERVER_IP, COMMAND_PORT))
print(f"Sent command: {command}")

# Wait for telemetry
data, addr = sock_telemetry.recvfrom(1024)  # buffer size is 1024 bytes
telemetry = data.decode()
print(f"Received telemetry: {telemetry}")
```

Server
```
import socket
import random

# UDP IP and Ports
LISTEN_IP = "0.0.0.0"
COMMAND_PORT = 1234
TELEMETRY_PORT = 1235
TELEMETRY_IP = "localhost"  # Change if the client is on a different machine

# Simulated satellite state
satellite_mode = "NORMAL"

# Create a UDP socket for commands
sock_command = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock_command.bind((LISTEN_IP, COMMAND_PORT))

# Create a UDP socket for telemetry
sock_telemetry = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

print("Satellite simulation started. Waiting for commands...")

def send_telemetry(message):
    sock_telemetry.sendto(message.encode(), (TELEMETRY_IP, TELEMETRY_PORT))
    print(f"Sent telemetry: {message}")

while True:
    # Listen for commands
    data, addr = sock_command.recvfrom(1024)  # buffer size is 1024 bytes
    command = data.decode()
    print(f"Received command: {command} from {addr}")

    # Process commands
    if command == "STATUS":
        send_telemetry("STATUS:OK")
    elif command == "PING":
        send_telemetry("PING:ACK")
    elif command == "GET_TEMP":
        # Simulate a temperature reading
        temp = random.randint(-50, 50)
        send_telemetry(f"TEMP:{temp}C")
    elif command == "SET_MODE":
        # Set mode and confirm
        satellite_mode = "SAFE"
        send_telemetry(f"MODE:{satellite_mode}")
    elif command == "REBOOT":
        # Simulate a reboot sequence
        send_telemetry("REBOOTING")
        satellite_mode = "NORMAL"  # Reset to NORMAL mode after reboot
        send_telemetry("REBOOTED:OK")
    else:
        # Unknown command
        send_telemetry("ERROR:UNKNOWN_COMMAND")
```
