cat > /tmp/README.md << 'EOFREADME'
# AI-OTDR Analysis Platform

**Local AI-powered OTDR report analysis for fiber optic networks**

A complete, self-contained system for analyzing EXFO OTDR files using local Ollama AI. Includes Docker stack, Flask API, modern web UI, OpenOTDR integration, and Netbird mesh connectivity.

## Quick Start

```bash
git clone git@github.com:cpradmin/OpenOTDR.git
cd OpenOTDR
chmod +x start.sh
./start.sh
```

Then open: **http://localhost**

## Features

- ✅ **Local AI** - Ollama inference, no cloud dependencies
- ✅ **OTDR Parsing** - OpenOTDR library for EXFO .trc/.sor files
- ✅ **Web Interface** - Modern, responsive UI for operators
- ✅ **REST API** - Full programmatic access
- ✅ **Job Organization** - Structure by project/task
- ✅ **Online/Offline Modes** - Works connected or disconnected
- ✅ **Docker Stack** - Complete containerized solution
- ✅ **Netbird Integration** - Secure mesh networking

## Architecture

```
┌─────────────────────────────────────┐
│  Nginx (80/443)                     │
│  ├─ Web UI (index.html)             │
│  └─ API Proxy                       │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│  Flask API Server (5000)            │
│  ├─ File Upload                     │
│  ├─ OTDR Parsing (OpenOTDR)         │
│  └─ Result Storage                  │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│  Ollama (11437)                     │
│  └─ Local AI Inference              │
└─────────────────────────────────────┘
```

## Docker Services

| Service | Purpose | Port |
|---------|---------|------|
| **otdr-ollama** | Local AI inference | 11437 |
| **otdr-api** | Flask API server | 5000 |
| **otdr-nginx** | Web UI + reverse proxy | 80/443 |
| **otdr-netbird** | Mesh VPN (optional) | - |

## Directory Structure

```
OpenOTDR/
├── docker-compose.yml      # Complete stack definition
├── Dockerfile.api          # Python API container
├── nginx.conf              # Reverse proxy config
├── README.md               # This file
├── NETBIRD_CONFIG.md       # Mesh setup guide
├── STRUCTURE.md            # Folder details
├── .env.example            # Configuration template
├── .gitignore
├── start.sh                # Startup script
│
├── api/
│   ├── server.py           # Flask API (main logic)
│   └── requirements.txt     # Python dependencies
│
├── web/
│   └── index.html          # Web interface
│
└── data/ (created at runtime)
    ├── online/             # Connected mode files
    ├── offline/            # Disconnected mode files
    └── processed/          # Analysis results (JSON)
```

## Getting Started

### 1. Clone Repository

```bash
git clone git@github.com:cpradmin/OpenOTDR.git
cd OpenOTDR
```

### 2. Start Services

```bash
chmod +x start.sh
./start.sh
```

This will:
- Build Docker containers
- Start Ollama, API, Nginx, Netbird
- Pull AI model (mistral, ~5GB on first run)
- Display status

### 3. Access Web UI

Open: **http://localhost**

## Usage

### Upload OTDR File

1. Enter Job ID (optional)
2. Enter Task ID (optional)
3. Drag OTDR file (.trc, .sor) or click to select
4. Click "Upload & Analyze"
5. View results below

### Via API

**Check Status**
```bash
curl http://localhost/api/status
```

**Upload & Analyze**
```bash
curl -X POST http://localhost/api/analyze \
  -F "file=@measurement.trc" \
  -F "job_id=myproject" \
  -F "task_id=section-a"
```

**Get Results**
```bash
# All results
curl http://localhost/api/results

# By job
curl http://localhost/api/results?job_id=myproject

# Specific result
curl http://localhost/api/results/<result-id>

# All jobs
curl http://localhost/api/jobs
```

## Configuration

### Environment Variables

Create `.env` from `.env.example`:

```bash
cp .env.example .env
```

Edit for your setup:
- `FLASK_ENV=production`
- `OTDR_MODE=online` (or `offline`)
- `OLLAMA_HOST=http://otdr-ollama:11434`
- `DEFAULT_MODEL=mistral` (or `llama2`, `neural-chat`, etc.)

### Docker Compose Customization

Edit `docker-compose.yml` to:
- Change ports
- Adjust resource limits
- Add Netbird setup key

## Online vs Offline Modes

### Online Mode (Default)
- System connected to network/Netbird
- OTDR files analyzed immediately
- Real-time results
- Results synced across mesh

### Offline Mode
- System operates without connectivity
- Files stored locally
- Analysis happens on local Ollama
- Results cached
- Auto-sync on reconnect

Switch modes: Edit `docker-compose.yml`, change `OTDR_MODE=offline`

## Netbird Integration

Connect AI-OTDR to your fortress mesh network:

1. Get setup key from https://app.netbird.io
2. Edit `docker-compose.yml`:
   ```yaml
   NETBIRD_SETUP_KEY=<your-setup-key>
   ```
3. Start: `docker-compose up -d otdr-netbird`
4. Access via: `http://10.x.x.x/` (Netbird IP)

See `NETBIRD_CONFIG.md` for full setup.

## API Reference

### Health & Status
- `GET /health` - Basic health check
- `GET /api/status` - System status + file counts
- `GET /api/models` - Available Ollama models

### Analysis
- `POST /api/analyze` - Upload & analyze OTDR
  - Params: `file`, `job_id` (opt), `task_id` (opt)
- `POST /api/models/pull` - Pull Ollama model
  - JSON: `{"model": "model-name"}`

### Results
- `GET /api/results` - List results
  - Query: `job_id`, `task_id`, `limit` (default 50)
- `GET /api/results/<result-id>` - Get specific result
- `GET /api/jobs` - List all jobs with task counts

## Performance Tuning

### Model Selection

**For Speed** (embedded/low-power):
```bash
docker exec otdr-ollama ollama pull orca-mini
```

**For Accuracy** (recommended):
```bash
docker exec otdr-ollama ollama pull mistral
```

**For Advanced Analysis**:
```bash
docker exec otdr-ollama ollama pull neural-chat
```

### Resource Limits

Edit `docker-compose.yml` to set limits:
```yaml
otdr-api:
  deploy:
    resources:
      limits:
        cpus: '2'
        memory: 4G
```

## Troubleshooting

### Ollama Not Healthy

```bash
# Check logs
docker logs otdr-ollama

# Restart
docker restart otdr-ollama

# Pull model
docker exec otdr-ollama ollama pull mistral
```

### API Server Errors

```bash
# Check logs
docker logs otdr-api

# Test health
curl http://localhost:5000/health

# Restart
docker restart otdr-api
```

### File Upload Issues

```bash
# Check permissions
ls -la data/

# Fix if needed
chmod -R 755 data/

# Check disk space
df -h
```

### Netbird Connection Failed

```bash
# Check logs
docker logs otdr-netbird

# Verify setup key in docker-compose.yml
# Check Netbird dashboard for peer status
```

## Security

- **Local Processing**: All OTDR files stay local
- **No Cloud**: Zero external API calls (except Ollama model pulls)
- **Encryption**: Netbird uses WireGuard encryption
- **Isolated**: All services in Docker containers
- **Private**: Access control via Netbird policies

## Development

### Add Custom OTDR Analysis

Edit `api/server.py` - `_create_analysis_prompt()` method to customize AI prompts for specific analysis needs.

### Extend OTDR Parsing

The system uses OpenOTDR. Extend `_parse_otdr_file()` to add custom metrics or processing.

### Use Custom Models

Point to any Ollama-compatible model:
```python
self.model = 'your-custom-model'
```

## Deployment Variants

### Windows 8 Embedded
1. Install WSL2
2. Clone repo
3. Run `./start.sh`
4. Access via http://localhost:3000 (port forward if needed)

### Linux Server
1. Install Docker/Docker Compose
2. Clone repo
3. Run `./start.sh`
4. Access via server IP

### Docker Swarm/Kubernetes
Use provided `docker-compose.yml` as base for orchestration configs.

## Future Enhancements

- [ ] Database backend (PostgreSQL)
- [ ] Historical trend analysis
- [ ] Real-time monitoring mode
- [ ] Multi-user access control
- [ ] PDF/Excel report export
- [ ] Mobile app for field technicians
- [ ] Integration with FDOT traffic systems
- [ ] Advanced anomaly detection

## License & Attribution

- **OpenOTDR**: https://github.com/BaldrAI/OpenOTDR
- **Ollama**: https://ollama.ai
- **Netbird**: https://netbird.io
- **Flask**: https://flask.palletsprojects.com
- **Nginx**: https://nginx.org

## Support

### Common Issues

1. **Port already in use**: Change ports in `docker-compose.yml`
2. **Out of memory**: Reduce `DEFAULT_MODEL` or set resource limits
3. **Slow analysis**: Use lighter model or increase CPU allocation
4. **Cannot connect to Ollama**: Wait for container health check (40s)

### Debug Mode

```bash
docker-compose logs -f otdr-api
docker-compose logs -f otdr-ollama
docker-compose logs -f otdr-nginx
```

### Check Status

```bash
docker-compose ps
curl http://localhost/api/status | jq
```

## Contributing

Build on this foundation! The system is designed to be:
- **Modular**: Replace/extend any component
- **Portable**: Runs anywhere with Docker
- **Private**: No external dependencies
- **Secure**: Netbird mesh encryption

---

**Built for ungovernable, family-serving infrastructure** 🏛️

For FDOT District 3 - 250 miles of highway monitoring, emergency response coordination, and AI-assisted traffic management.
EOFREADME

cat /tmp/README.md
Output

# AI-OTDR Analysis Platform

**Local AI-powered OTDR report analysis for fiber optic networks**

A complete, self-contained system for analyzing EXFO OTDR files using local Ollama AI. Includes Docker stack, Flask API, modern web UI, OpenOTDR integration, and Netbird mesh connectivity.

## Quick Start

```bash
git clone git@github.com:cpradmin/OpenOTDR.git
cd OpenOTDR
chmod +x start.sh
./start.sh
```

Then open: **http://localhost**

## Features

- ✅ **Local AI** - Ollama inference, no cloud dependencies
- ✅ **OTDR Parsing** - OpenOTDR library for EXFO .trc/.sor files
- ✅ **Web Interface** - Modern, responsive UI for operators
- ✅ **REST API** - Full programmatic access
- ✅ **Job Organization** - Structure by project/task
- ✅ **Online/Offline Modes** - Works connected or disconnected
- ✅ **Docker Stack** - Complete containerized solution
- ✅ **Netbird Integration** - Secure mesh networking

## Architecture

```
┌─────────────────────────────────────┐
│  Nginx (80/443)                     │
│  ├─ Web UI (index.html)             │
│  └─ API Proxy                       │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│  Flask API Server (5000)            │
│  ├─ File Upload                     │
│  ├─ OTDR Parsing (OpenOTDR)         │
│  └─ Result Storage                  │
└────────────┬────────────────────────┘
             │
┌────────────▼────────────────────────┐
│  Ollama (11437)                     │
│  └─ Local AI Inference              │
└─────────────────────────────────────┘
```

## Docker Services

| Service | Purpose | Port |
|---------|---------|------|
| **otdr-ollama** | Local AI inference | 11437 |
| **otdr-api** | Flask API server | 5000 |
| **otdr-nginx** | Web UI + reverse proxy | 80/443 |
| **otdr-netbird** | Mesh VPN (optional) | - |

## Directory Structure

```
OpenOTDR/
├── docker-compose.yml      # Complete stack definition
├── Dockerfile.api          # Python API container
├── nginx.conf              # Reverse proxy config
├── README.md               # This file
├── NETBIRD_CONFIG.md       # Mesh setup guide
├── STRUCTURE.md            # Folder details
├── .env.example            # Configuration template
├── .gitignore
├── start.sh                # Startup script
│
├── api/
│   ├── server.py           # Flask API (main logic)
│   └── requirements.txt     # Python dependencies
│
├── web/
│   └── index.html          # Web interface
│
└── data/ (created at runtime)
    ├── online/             # Connected mode files
    ├── offline/            # Disconnected mode files
    └── processed/          # Analysis results (JSON)
```

## Getting Started

### 1. Clone Repository

```bash
git clone git@github.com:cpradmin/OpenOTDR.git
cd OpenOTDR
```

### 2. Start Services

```bash
chmod +x start.sh
./start.sh
```

This will:
- Build Docker containers
- Start Ollama, API, Nginx, Netbird
- Pull AI model (mistral, ~5GB on first run)
- Display status

### 3. Access Web UI

Open: **http://localhost**

## Usage

### Upload OTDR File

1. Enter Job ID (optional)
2. Enter Task ID (optional)
3. Drag OTDR file (.trc, .sor) or click to select
4. Click "Upload & Analyze"
5. View results below

### Via API

**Check Status**
```bash
curl http://localhost/api/status
```

**Upload & Analyze**
```bash
curl -X POST http://localhost/api/analyze \
  -F "file=@measurement.trc" \
  -F "job_id=myproject" \
  -F "task_id=section-a"
```

**Get Results**
```bash
# All results
curl http://localhost/api/results

# By job
curl http://localhost/api/results?job_id=myproject

# Specific result
curl http://localhost/api/results/<result-id>

# All jobs
curl http://localhost/api/jobs
```

## Configuration

### Environment Variables

Create `.env` from `.env.example`:

```bash
cp .env.example .env
```

Edit for your setup:
- `FLASK_ENV=production`
- `OTDR_MODE=online` (or `offline`)
- `OLLAMA_HOST=http://otdr-ollama:11434`
- `DEFAULT_MODEL=mistral` (or `llama2`, `neural-chat`, etc.)

### Docker Compose Customization

Edit `docker-compose.yml` to:
- Change ports
- Adjust resource limits
- Add Netbird setup key

## Online vs Offline Modes

### Online Mode (Default)
- System connected to network/Netbird
- OTDR files analyzed immediately
- Real-time results
- Results synced across mesh

### Offline Mode
- System operates without connectivity
- Files stored locally
- Analysis happens on local Ollama
- Results cached
- Auto-sync on reconnect

Switch modes: Edit `docker-compose.yml`, change `OTDR_MODE=offline`

## Netbird Integration

Connect AI-OTDR to your fortress mesh network:

1. Get setup key from https://app.netbird.io
2. Edit `docker-compose.yml`:
   ```yaml
   NETBIRD_SETUP_KEY=<your-setup-key>
   ```
3. Start: `docker-compose up -d otdr-netbird`
4. Access via: `http://10.x.x.x/` (Netbird IP)

See `NETBIRD_CONFIG.md` for full setup.

## API Reference

### Health & Status
- `GET /health` - Basic health check
- `GET /api/status` - System status + file counts
- `GET /api/models` - Available Ollama models

### Analysis
- `POST /api/analyze` - Upload & analyze OTDR
  - Params: `file`, `job_id` (opt), `task_id` (opt)
- `POST /api/models/pull` - Pull Ollama model
  - JSON: `{"model": "model-name"}`

### Results
- `GET /api/results` - List results
  - Query: `job_id`, `task_id`, `limit` (default 50)
- `GET /api/results/<result-id>` - Get specific result
- `GET /api/jobs` - List all jobs with task counts

## Performance Tuning

### Model Selection

**For Speed** (embedded/low-power):
```bash
docker exec otdr-ollama ollama pull orca-mini
```

**For Accuracy** (recommended):
```bash
docker exec otdr-ollama ollama pull mistral
```

**For Advanced Analysis**:
```bash
docker exec otdr-ollama ollama pull neural-chat
```

### Resource Limits

Edit `docker-compose.yml` to set limits:
```yaml
otdr-api:
  deploy:
    resources:
      limits:
        cpus: '2'
        memory: 4G
```

## Troubleshooting

### Ollama Not Healthy

```bash
# Check logs
docker logs otdr-ollama

# Restart
docker restart otdr-ollama

# Pull model
docker exec otdr-ollama ollama pull mistral
```

### API Server Errors

```bash
# Check logs
docker logs otdr-api

# Test health
curl http://localhost:5000/health

# Restart
docker restart otdr-api
```

### File Upload Issues

```bash
# Check permissions
ls -la data/

# Fix if needed
chmod -R 755 data/

# Check disk space
df -h
```

### Netbird Connection Failed

```bash
# Check logs
docker logs otdr-netbird

# Verify setup key in docker-compose.yml
# Check Netbird dashboard for peer status
```

## Security

- **Local Processing**: All OTDR files stay local
- **No Cloud**: Zero external API calls (except Ollama model pulls)
- **Encryption**: Netbird uses WireGuard encryption
- **Isolated**: All services in Docker containers
- **Private**: Access control via Netbird policies

## Development

### Add Custom OTDR Analysis

Edit `api/server.py` - `_create_analysis_prompt()` method to customize AI prompts for specific analysis needs.

### Extend OTDR Parsing

The system uses OpenOTDR. Extend `_parse_otdr_file()` to add custom metrics or processing.

### Use Custom Models

Point to any Ollama-compatible model:
```python
self.model = 'your-custom-model'
```

## Deployment Variants

### Windows 8 Embedded
1. Install WSL2
2. Clone repo
3. Run `./start.sh`
4. Access via http://localhost:3000 (port forward if needed)

### Linux Server
1. Install Docker/Docker Compose
2. Clone repo
3. Run `./start.sh`
4. Access via server IP

### Docker Swarm/Kubernetes
Use provided `docker-compose.yml` as base for orchestration configs.

## Future Enhancements

- [ ] Database backend (PostgreSQL)
- [ ] Historical trend analysis
- [ ] Real-time monitoring mode
- [ ] Multi-user access control
- [ ] PDF/Excel report export
- [ ] Mobile app for field technicians
- [ ] Integration with FDOT traffic systems
- [ ] Advanced anomaly detection

## License & Attribution

- **OpenOTDR**: https://github.com/BaldrAI/OpenOTDR
- **Ollama**: https://ollama.ai
- **Netbird**: https://netbird.io
- **Flask**: https://flask.palletsprojects.com
- **Nginx**: https://nginx.org

## Support

### Common Issues

1. **Port already in use**: Change ports in `docker-compose.yml`
2. **Out of memory**: Reduce `DEFAULT_MODEL` or set resource limits
3. **Slow analysis**: Use lighter model or increase CPU allocation
4. **Cannot connect to Ollama**: Wait for container health check (40s)

### Debug Mode

```bash
docker-compose logs -f otdr-api
docker-compose logs -f otdr-ollama
docker-compose logs -f otdr-nginx
```

### Check Status

```bash
docker-compose ps
curl http://localhost/api/status | jq
```

## Contributing

Build on this foundation! The system is designed to be:
- **Modular**: Replace/extend any component
- **Portable**: Runs anywhere with Docker
- **Private**: No external dependencies
- **Secure**: Netbird mesh encryption

---

**Built for ungovernable, family-serving infrastructure** 🏛️

For FDOT District 3 - 250 miles of highway monitoring, emergency response coordination, and AI-assisted traffic management.
