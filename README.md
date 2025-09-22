# Teltonika GPS Collector

Collects AVL data from Teltonika GPS devices using Java / Quarkus.

---

## What It Does

- Listens on a configured TCP port for incoming AVL packets from Teltonika devices.  
- Parses multiple AVL records per packet.  
- Saves parsed location (latitude, longitude, timestamp, etc.).  
- Sends the required acknowledgment back to the device (number of records parsed), so that duplicates are avoided.  
- Supports raw logging of incoming AVL payloads (hex) for debugging.  
- Validates fields (lat/lon bounds, reasonable timestamps).

---

## Setup & Configuration

### Configuration properties

| Property | Default | Description |
|---|---|---|
| `receiver.port` | `1111` | TCP port your server listens for Teltonika AVL data |
| `receiver.ip` | `127.0.0.1` | IP (usually 0.0.0.0 or host IP) to bind socket if needed |

You can set these via `application.properties` or environment variables as usual in Quarkus.

---

## How to Run

### Local / Dev mode

```bash
./mvnw clean package
java -jar target/teltonika-gps-collector.jar
