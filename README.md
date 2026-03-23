# Honeypot + Log Visualizer

A lightweight multi-service honeypot built with Python, featuring a real-time web dashboard for attack visualization and analysis. Designed to simulate SSH, HTTP, and FTP services to capture and log attacker behavior.

---

## Features

- **Multi-service honeypot** — fake SSH, HTTP, and FTP servers that mimic real services
- **Credential capture** — logs all usernames, passwords, and payloads sent by attackers
- **GeoIP lookup** — automatically resolves attacker IP to country and city
- **SQLite database** — structured storage with SQLAlchemy ORM
- **Live dashboard** — real-time web UI with auto-refresh every 5 seconds
- **Attack visualization** — donut chart (hits by service) and bar chart (top attacker IPs)
- **Service filter** — filter attack feed by SSH, HTTP, or FTP
- **Event detail modal** — view full raw payload per event
- **CSV export** — export filtered attack data to CSV

---

## Tech Stack

| Layer | Technology |
|---|---|
| Honeypot engine | Python `socket`, `threading` |
| Database | SQLite + SQLAlchemy |
| GeoIP | ip-api.com (free, no key required) |
| Backend API | FastAPI + Uvicorn |
| Frontend | HTML, CSS, JavaScript |
| Charts | Chart.js |

---

## Project Structure

```
honeypot/
├── main.py               # Entry point — starts all honeypot services
├── base_server.py        # Base TCP server class
├── logger.py             # Event logging to database
├── database.py           # SQLAlchemy models and DB setup
├── geoip.py              # GeoIP lookup via ip-api.com
├── stats.py              # Query helpers for dashboard statistics
├── services/
│   ├── __init__.py
│   ├── ssh_honeypot.py   # Fake SSH server (port 2222)
│   ├── http_honeypot.py  # Fake HTTP server (port 8080)
│   └── ftp_honeypot.py   # Fake FTP server (port 2121)
├── dashboard/
│   ├── __init__.py
│   ├── app.py            # FastAPI application and REST endpoints
│   ├── templates/
│   │   └── index.html    # Dashboard UI
│   └── static/
│       └── app.js        # Frontend logic, charts, polling
└── logs/
    └── events.json       # Legacy JSON log (Fase 1)
```

---

## Requirements

- Python 3.10+
- WSL Ubuntu (recommended) or Linux
- pip packages: `sqlalchemy`, `requests`, `fastapi`, `uvicorn`, `jinja2`, `python-multipart`

---

## Installation

**1. Clone the repository:**
```bash
git clone https://github.com/your-username/honeypot.git
cd honeypot
```

**2. Create and activate virtual environment:**
```bash
python3 -m venv venv
source venv/bin/activate
```

**3. Install dependencies:**
```bash
pip install sqlalchemy requests fastapi uvicorn jinja2 python-multipart
```

---

## Usage

**Terminal 1 — start honeypot services:**
```bash
source venv/bin/activate
python3 main.py
```

Expected output:
```
[SSH] Listening on port 2222
[HTTP] Listening on port 8080
[FTP] Listening on port 2121
Honeypot running on ports: 2222 (SSH), 8080 (HTTP), 2121 (FTP)
```

**Terminal 2 — start dashboard:**
```bash
source venv/bin/activate
uvicorn dashboard.app:app --host 0.0.0.0 --port 5000 --reload
```

Open browser at `http://localhost:5000`

---

## Testing

Generate test traffic to populate the dashboard:

```bash
# Simulate SSH connection attempt
echo "test" | nc localhost 2222

# Simulate HTTP login attempt
curl -X POST http://localhost:8080/login \
  -d "username=admin&password=secret"

# Simulate FTP brute force
ftp localhost 2121
# Name: admin
# Password: password123
```

Check database directly:
```bash
python3 -c "
from database import SessionLocal, Event
db = SessionLocal()
for e in db.query(Event).all():
    print(f'{e.service:5} | {e.src_ip} | {e.country} | {e.extra}')
db.close()
"
```

---

## Dashboard

The web dashboard at `http://localhost:5000` provides:

- **Stat cards** — total hits, unique IPs, last updated time
- **Hits by service** — donut chart showing SSH/HTTP/FTP distribution
- **Top attacker IPs** — bar chart of most active source IPs
- **Live attack feed** — table with timestamp, service badge, IP, country, and detail
- **Filter buttons** — filter feed by All / SSH / HTTP / FTP
- **Detail modal** — click Detail on any row to see full raw payload
- **Export CSV** — download current filtered data as CSV file

---

## How It Works

### Honeypot services

Each fake service sends a realistic banner to fool scanners into thinking they found a real server:

| Service | Port | Banner |
|---|---|---|
| SSH | 2222 | `SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.6` |
| HTTP | 8080 | Fake router admin login page |
| FTP | 2121 | `220 ProFTPD 1.3.8 Server ready.` |

### Data captured per event

Every connection is logged with:
- Timestamp (UTC)
- Source IP and port
- Service name
- Country and city (via GeoIP)
- Raw payload (bytes sent by attacker)
- Parsed extra data (HTTP method/path/body, FTP username/password)

### Architecture

```
Attacker → [SSH/HTTP/FTP Honeypot] → Logger → SQLite DB
                                                   ↓
Browser ← [Dashboard HTML/JS] ← FastAPI REST API ←┘
```

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | Dashboard UI |
| GET | `/api/stats` | Aggregate statistics |
| GET | `/api/events?limit=50` | Latest events |
| GET | `/api/events/{id}` | Single event detail |

---

## Notes

- Run on ports above 1024 (2222, 8080, 2121) to avoid requiring `sudo`
- For local IP addresses (127.x, 192.168.x, 10.x), GeoIP returns `Local`
- The honeypot is intended for **educational and research purposes only**
- Do not deploy on production networks without proper authorization

---

## Possible Improvements

- [ ] AbuseIPDB integration for IP reputation scoring
- [ ] Attack pattern classification (scanner, brute forcer, flooder)
- [ ] Telegram bot notifications for new attacks
- [ ] Daily PDF report export
- [ ] World map visualization of attack origins
- [ ] Docker containerization

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## Author

**Samuel Kaunang**  
S1 Informatika — Cybersecurity Project
