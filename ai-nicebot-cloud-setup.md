# ai.nicebot.cloud Setup Guide

Complete guide for running the NiceBot AI assistant publicly via Cloudflare Tunnel.

## Quick Start: Boot to Functional ai.nicebot.cloud

After the Jetson boots, follow these steps in order:

### 1. Start Ollama (LLM Backend)

```bash
ollama serve
```

Or if running as a service:
```bash
sudo systemctl start ollama
sudo systemctl status ollama
```

### 2. Start Open WebUI (Chat Interface)

```bash
docker start open-webui
```

Verify it's running:
```bash
docker ps | grep open-webui
```

### 3. Start Cloudflare Tunnel

If installed as a service (recommended):
```bash
sudo systemctl start cloudflared
sudo systemctl status cloudflared
```

If running manually:
```bash
cloudflared tunnel run jetson-ai
```

### 4. Verify Access

- **Local:** http://10.0.0.240:8080
- **Public:** https://ai.nicebot.cloud

---

## Service Auto-Start Configuration

For fully automatic startup on boot, enable all services:

```bash
# Ollama (usually auto-enabled on install)
sudo systemctl enable ollama

# Docker (for Open WebUI)
sudo systemctl enable docker

# Open WebUI container (set restart policy)
docker update --restart always open-webui

# Cloudflare Tunnel
sudo systemctl enable cloudflared
```

After enabling, all services will start automatically on Jetson boot.

---

## Component Details

### Ollama (LLM Runtime)

| Property | Value |
|----------|-------|
| Model | `gemma3:4b` |
| Model Size | 3.3 GB |
| Context Window | 96K tokens |
| API Endpoint | `http://10.0.0.240:11434` |

**Common Commands:**
```bash
# Start server
ollama serve

# List models
ollama list

# Run model interactively
ollama run gemma3:4b

# Pull a model
ollama pull gemma3:4b
```

### Open WebUI (Chat Interface)

| Property | Value |
|----------|-------|
| Container Name | `open-webui` |
| Local Port | 8080 |
| Data Volume | `open-webui:/app/backend/data` |

**Docker Commands:**
```bash
# Start container
docker start open-webui

# Stop container
docker stop open-webui

# View logs
docker logs open-webui

# Check status
docker ps | grep open-webui
```

**Initial Installation (if needed):**
```bash
docker run -d \
  --name open-webui \
  --network host \
  -v open-webui:/app/backend/data \
  -e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

### Cloudflare Tunnel

| Property | Value |
|----------|-------|
| Tunnel Name | `jetson-ai` |
| Tunnel ID | `6471d798-bb50-45e9-add9-ca8c451a3962` |
| Public URL | https://ai.nicebot.cloud |
| Local Service | http://localhost:8080 |

**Configuration Files:**
- Config: `/etc/cloudflared/config.yml`
- Credentials: `/etc/cloudflared/6471d798-bb50-45e9-add9-ca8c451a3962.json`

**Config File Contents (`/etc/cloudflared/config.yml`):**
```yaml
tunnel: 6471d798-bb50-45e9-add9-ca8c451a3962
credentials-file: /etc/cloudflared/6471d798-bb50-45e9-add9-ca8c451a3962.json

ingress:
  - hostname: ai.nicebot.cloud
    service: http://localhost:8080
  - service: http_status:404
```

**Service Commands:**
```bash
# Start tunnel service
sudo systemctl start cloudflared

# Stop tunnel service
sudo systemctl stop cloudflared

# Check status
sudo systemctl status cloudflared

# View logs
sudo journalctl -u cloudflared -f
```

---

## Initial Setup (One-Time)

These steps were already completed but documented for reference.

### 1. Install Cloudflared

```bash
# Download and install
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64 -o cloudflared
sudo mv cloudflared /usr/local/bin/
sudo chmod +x /usr/local/bin/cloudflared

# Verify installation
cloudflared --version
```

### 2. Authenticate with Cloudflare

```bash
cloudflared tunnel login
```

This opens a browser to authenticate with your Cloudflare account.

### 3. Create Tunnel

```bash
cloudflared tunnel create jetson-ai
```

This creates:
- Tunnel ID: `6471d798-bb50-45e9-add9-ca8c451a3962`
- Credentials file: `~/.cloudflared/<tunnel-id>.json`

### 4. Route DNS

```bash
cloudflared tunnel route dns jetson-ai ai.nicebot.cloud
```

This creates a CNAME record in Cloudflare DNS pointing to the tunnel.

### 5. Create Config File

```bash
cat > ~/.cloudflared/config.yml << 'EOF'
tunnel: 6471d798-bb50-45e9-add9-ca8c451a3962
credentials-file: /home/ironaxis/.cloudflared/6471d798-bb50-45e9-add9-ca8c451a3962.json

ingress:
  - hostname: ai.nicebot.cloud
    service: http://localhost:8080
  - service: http_status:404
EOF
```

### 6. Install as System Service

```bash
# Copy config to system location
sudo mkdir -p /etc/cloudflared
sudo cp ~/.cloudflared/config.yml /etc/cloudflared/
sudo cp ~/.cloudflared/6471d798-bb50-45e9-add9-ca8c451a3962.json /etc/cloudflared/

# Update credentials path in system config
sudo sed -i 's|/home/ironaxis/.cloudflared/|/etc/cloudflared/|g' /etc/cloudflared/config.yml

# Install and enable service
sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

---

## Troubleshooting

### Ollama Not Responding

```bash
# Check if running
sudo systemctl status ollama

# Restart
sudo systemctl restart ollama

# Check logs
sudo journalctl -u ollama -f
```

### Open WebUI Shows Error

1. Verify Ollama is running first
2. Check container logs:
   ```bash
   docker logs open-webui
   ```
3. Restart container:
   ```bash
   docker restart open-webui
   ```

### Tunnel Not Working

```bash
# Check service status
sudo systemctl status cloudflared

# View logs
sudo journalctl -u cloudflared -f

# Test config
cloudflared tunnel --config /etc/cloudflared/config.yml run jetson-ai
```

### YAML Config Errors

Common issues:
- Indentation must use spaces, not tabs
- `credentials-file` path must be on a single line
- `tunnel` and `credentials-file` should have no indentation
- `ingress` entries use 2-space indentation

### Cannot SSH After Reboot

If Jetson's host key changed (e.g., after SSD migration):
```bash
ssh-keygen -R 10.0.0.240
ssh jetson
```

---

## Architecture Overview

```
[Internet]
    ↓
[Cloudflare Edge] (ai.nicebot.cloud)
    ↓ (encrypted tunnel)
[Jetson: cloudflared] (port 8080)
    ↓
[Jetson: Open WebUI] (Docker container)
    ↓ (localhost:11434)
[Jetson: Ollama] (gemma3:4b model)
```

**Benefits:**
- No port forwarding required
- HTTPS automatically provided by Cloudflare
- DDoS protection included
- Access from anywhere with internet
- No public IP needed

---

## Hardware Reference

| Component | Specification |
|-----------|---------------|
| Device | NVIDIA Jetson Orin Nano Super Developer Kit |
| CPU | 6-core ARM Cortex-A78AE |
| GPU | 1024 CUDA cores (Ampere) |
| Memory | 8GB LPDDR5 (shared) |
| Storage | 256GB NVMe SSD |
| Network | 10.0.0.240 |
| OS | Ubuntu 22.04 (aarch64) |
| JetPack | 6.2.1 |

---

*Last updated: December 2025*
