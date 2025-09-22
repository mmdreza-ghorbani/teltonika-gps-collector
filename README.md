# Teltonika GPS Collector (GatewayService)

This service collects AVL data packets from **Teltonika GPS devices** using the Codec8/Codec8E protocol.  
It is implemented in **Java (Quarkus/JVM)** and runs as a Docker container.

---

## Features

- Listens on **TCP port 9999** for AVL data from Teltonika devices.  
- Handles device identification (IMEI handshake).  
- Parses AVL packets (Codec8 / Codec8 Extended).  
- Correctly extracts:
  - Latitude & longitude (converted from raw integer / 10,000,000)  
  - Altitude, speed, angle, satellites  
  - Timestamp (epoch millis → UTC time)  
  - IO elements (basic + extended)  
- Sends back **ACK** with number of parsed records (mandatory to prevent duplicates).  
- Saves raw data (hex string) into log files under `/home/java/projects/ReceiverLogs/`.  
- Provides debug logging for each record.

---

## How It Works

1. Device connects → sends **IMEI**.  
   - Server replies `0x01` (accepted).  
2. Device sends AVL packet:  
   - Preamble (4 bytes)  
   - Data length (4 bytes)  
   - Codec ID (8 or 8E)  
   - Record count + records  
   - CRC (4 bytes)  
3. Server parses **all records**:  
   - Each record contains timestamp, priority, GPS (lon, lat, altitude, speed, angle, satellites).  
   - Longitude & latitude are integers → divide by `10,000,000` to get decimal degrees.  
   - Example: `340364556 → 34.0364556`.  
4. Server sends back **ACK** = 4-byte integer with the record count.  
   - Without this, device will **resend the same packet** → causing repeated data.  
5. Logs are written both as structured output and raw hex (for debugging CRC mismatches).  

---

## Example Log Output

2025-09-20 07:43:32 INFO [ReceiverService] Receive IMEI: [357544370758399]
2025-09-20 07:43:32 INFO [ReceiverService] IMEI=357544370758399 CodecId=8 Records=28
2025-09-20 07:43:32 INFO [ReceiverService] IMEI=357544370758399 Record 1: lat=34.360455 lon=47.074262 in time=2025-09-20 09:23:51
2025-09-20 07:43:32 INFO [ReceiverService] IMEI=357544370758399 Record 2: lat=34.360437 lon=47.074667 in time=2025-09-20 09:23:54
2025-09-20 07:43:32 INFO [ReceiverService] Sent ACK with 28 records

yaml
Copy code

---

## Running with Docker

1. Build the JAR:
   ```bash
   mvn clean package
Build Docker image:

bash
Copy code
docker build -t gatewayservice-jvm .
Run container (with port mapping + log directory mount):

bash
Copy code
mkdir -p /path/on/host/ReceiverLogs

docker run -d \
  --name gatewayservice-jvm \
  -p 9999:9999 \
  -p 8089:8089 \
  -v /path/on/host/ReceiverLogs:/home/java/projects/ReceiverLogs \
  gatewayservice-jvm
Now your GPS devices can connect to server-ip:9999.

Troubleshooting
Problem	Cause	Fix
CRC mismatch	Usually caused by parsing error or incorrect raw data logging	Ensure correct byte offsets and CRC32 check
Same data repeated	No ACK sent, device keeps resending	Always send 4-byte ACK = record count
Coordinates look wrong (too many digits)	Not divided by 10^7	Divide lon/lat integers by 10,000,000.0
Log file not created	Directory missing inside container	Create /home/java/projects/ReceiverLogs or mount it from host

Notes
If device shows a different location on another platform (e.g. Teltonika tracking server) but not here, check:

ACK is sent properly.

Timestamp parsing (must read full 8 bytes, big-endian).

Ensure you are not stuck re-parsing the same packet.

To request the last known position from the device, use Teltonika FMB remote commands via SMS, GPRS, or FOTA Web — not currently implemented in this service.

License
MIT License. See LICENSE.

yaml
Copy code

---

👉 Do you want me to also include a **sample hex dump of raw AVL data** (like the ones you logged) in the README so that future devs can test parsing against it?
