# 🚀 Tunnel System - Self-Hosted ngrok Alternative

A complete, production-ready solution for exposing local applications to the internet via SSH reverse tunnels with automatic HTTPS certificates and a modern web dashboard.

## Features

✅ **SSH Reverse Tunnels** - No third-party SaaS, full control  
✅ **Auto HTTPS** - Caddy generates certificates via Let's Encrypt  
✅ **Web Dashboard** - Beautiful UI to manage tunnels  
✅ **JWT Authentication** - Secure API with token-based auth  
✅ **Auto-Reconnect** - Tunnel client auto-reconnects on disconnect  
✅ **Wildcard Domains** - Route subdomains to different local ports  
✅ **Docker Ready** - One-command deployment  

## Project Structure

```
tunnel-system/
├── vps-server/                 # VPS deployment (Docker)
│   ├── api/                    # Node.js/Express backend
│   ├── ui/                     # Next.js dashboard
│   └── caddy/                  # Caddy reverse proxy config
├── client/                     # Local tunnel client
├── shared/                     # Shared TypeScript types
├── docs/                       # Documentation
│   ├── setup.md               # Setup guide
│   ├── api.md                 # API reference
│   └── architecture.md        # Architecture details
├── scripts/                    # Deployment scripts
│   ├── setup-vps.sh          # Initial VPS setup
│   └── deploy.sh             # Deploy to VPS
├── docker-compose.yml         # VPS services orchestration
└── README.md                  # This file
```

## Quick Start

### Prerequisites

- **VPS**: Ubuntu 20.04+ (DigitalOcean, Hetzner, Linode, etc.)
- **Domain**: With wildcard DNS support
- **Local Machine**: Node.js 18+, Docker (optional)
- **SSH Key**: For secure authentication

### 1️⃣ VPS Setup (5 minutes)

```bash
# SSH into your VPS
ssh root@your-vps-ip

# Download and run setup script
curl -fsSL https://raw.githubusercontent.com/yourusername/tunnel-system/main/scripts/setup-vps.sh | sudo bash

# Edit configuration
nano .env
nano vps-server/caddy/Caddyfile
```

### 2️⃣ Deploy Services

```bash
cd /root/tunnel-system

# Make deploy script executable
chmod +x scripts/deploy.sh

# Deploy
./scripts/deploy.sh

# Verify
docker-compose logs -f
```

### 3️⃣ Configure Local Client

```bash
# On your local machine
cd client
cp .env.example .env

# Edit with your VPS details
nano .env
# VPS_HOST=your-vps-ip.com
# VPS_USER=tunnel
# LOCAL_PORT_1=3000
# SUBDOMAIN_1=myapp

# Install and start
npm install
npm run dev
```

### 4️⃣ Access Dashboard

```
https://yourdomain.com
Username: admin
Password: (from .env)
```

## Architecture

```
┌─ Your Machine (localhost:3000)
│  └─ SSH Reverse Tunnel (encrypted)
│     └─ VPS:4000
│        ├─ Caddy (HTTPS proxy)
│        └─ Routes to https://myapp.yourdomain.com
```

## Creating Tunnels

### Via UI Dashboard
1. Go to `https://yourdomain.com`
2. Login with credentials
3. Click "Create Tunnel"
4. Enter app name, local port
5. Subdomain auto-generated (or custom)
6. Done! Your app is live

### Via API
```bash
curl -X POST http://localhost:3000/api/tunnels \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My App",
    "localPort": 3000,
    "subdomain": "myapp"
  }'
```

### Via CLI (Future)
```bash
tunnel create --name myapp --port 3000
```

## Configuration

### Environment Variables

**VPS (.env)**
```env
JWT_SECRET=your-secret-key
AUTH_USERNAME=admin
AUTH_PASSWORD=your-password
CLOUDFLARE_API_TOKEN=optional
```

**Client (client/.env)**
```env
VPS_HOST=your-vps-ip.com
VPS_USER=tunnel
VPS_PORT=22
SSH_PRIVATE_KEY_PATH=~/.ssh/tunnel_key

LOCAL_PORT_1=3000
REMOTE_PORT_1=4000
SUBDOMAIN_1=myapp

LOG_LEVEL=info
```

## Documentation

- **[Setup Guide](docs/setup.md)** - Detailed setup instructions
- **[API Reference](docs/api.md)** - Complete API documentation
- **[Architecture](docs/architecture.md)** - System design & flow

## Troubleshooting

### SSH Connection Fails
```bash
# Test connection
ssh -i ~/.ssh/tunnel_key tunnel@your-vps-ip

# Check SSH config
sudo sshd -T | grep GatewayPorts
# Should show: GatewayPorts yes
```

### HTTPS Certificate Not Working
```bash
# Check Caddy logs
docker-compose logs caddy

# Test DNS
nslookup myapp.yourdomain.com
```

### Tunnel Keeps Disconnecting
- Increase SSH keep-alive in client
- Check firewall (port 22 open)
- Review client logs for errors

See [Setup Guide](docs/setup.md#troubleshooting) for more details.

## Security

### SSH Authentication
```bash
# Generate key
ssh-keygen -t ed25519 -f ~/.ssh/tunnel_key -N ""

# Add to VPS
ssh-copy-id -i ~/.ssh/tunnel_key.pub tunnel@vps-ip
```

### API Security
- JWT tokens (24h expiration)
- Environment variable credentials
- HTTPS only (via Caddy + Let's Encrypt)

### Network Security
- UFW firewall (ports 80, 443, 22)
- fail2ban protection
- SSH restricted to `tunnel` user

### Checklist
- [ ] Strong JWT secret (32+ chars)
- [ ] Strong auth password
- [ ] SSH keys only (no password auth)
- [ ] Firewall enabled
- [ ] fail2ban running
- [ ] Cloudflare DDoS protection (optional)

## Comparison with ngrok

| Feature | This | ngrok |
|---------|------|-------|
| **Control** | Full ✅ | Limited |
| **Cost** | VPS only | Expensive SaaS |
| **Privacy** | Full ✅ | Third-party |
| **Custom Domains** | ✅ | Paid only |
| **HTTPS** | Auto ✅ | Auto |
| **UI** | Modern ✅ | Basic |
| **Setup** | 15 min | 5 min |

## Advanced Usage

### Multiple Local Applications
```typescript
// client/src/config.ts
tunnels: [
  { name: 'Frontend', localPort: 3000, ... },
  { name: 'API', localPort: 8000, ... },
  { name: 'Admin', localPort: 5000, ... },
]
```

### Custom Caddy Rules
```caddy
# vps-server/caddy/Caddyfile

app.yourdomain.com {
  reverse_proxy localhost:3000
  
  header / {
    X-Frame-Options "SAMEORIGIN"
    Strict-Transport-Security "max-age=31536000"
  }
  
  # Rate limiting
  rate_limit {
    zone dynamic {
      key {http.request.remote}
      events 100
      window 1m
    }
  }
}
```

### Persistent Storage
Use a database instead of in-memory:

```typescript
// Replace in-memory TunnelModel with MongoDB/PostgreSQL
import { Tunnel } from './models/Tunnel';

// In API
const tunnels = await Tunnel.find();
```

## Monitoring

### Health Checks
```bash
# API health
curl http://localhost:3000/health

# Full status
curl http://localhost:3000/status
```

### Docker Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f api
docker-compose logs -f caddy
```

### Tunnel Status
Access via dashboard or API:
```bash
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3000/api/tunnels
```

## Contributing

Contributions welcome! Please:
1. Fork the repo
2. Create a feature branch
3. Submit a pull request

## License

MIT License - see LICENSE file

## Support

- 📖 [Documentation](docs/)
- 🐛 [Issues](https://github.com/yourusername/tunnel-system/issues)
- 💬 [Discussions](https://github.com/yourusername/tunnel-system/discussions)

## Roadmap

- [ ] CLI tool for tunnel management
- [ ] Database persistence (MongoDB/PostgreSQL)
- [ ] Webhooks for events
- [ ] Analytics & monitoring
- [ ] Multi-user support
- [ ] Rate limiting per tunnel
- [ ] Custom SSL certificates
- [ ] Tunnel sharing/collaboration

## FAQ

**Q: Can I run multiple apps?**  
A: Yes! Configure multiple tunnels with different local ports.

**Q: Is it secure?**  
A: Yes! SSH encryption + HTTPS + firewall. More details in [Security section](#security).

**Q: How much does it cost?**  
A: Just VPS cost (~$5-20/month). No third-party fees.

**Q: Can I use my own domain?**  
A: Yes! Must support wildcard DNS (*.yourdomain.com).

**Q: What if my connection drops?**  
A: Client auto-reconnects with exponential backoff.

---

**Made with ❤️ for developers who want control of their infrastructure**
