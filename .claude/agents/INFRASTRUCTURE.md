---
name: infrastructure-guide
description: Reference for all agents on self-hosted infrastructure, Digital Ocean provisioning, and strategic technology decisions for the platform.
---

# Infrastructure Guide

**Purpose**: Comprehensive reference for ALL agents on self-hosted infrastructure, Digital Ocean provisioning, and strategic technology choices.

---

## 🎯 Self-Hosted Platform Philosophy

**Goal**: Build infrastructure BETTER than Vercel/Cloudflare with full control, zero vendor lock-in, and superior performance.

**Core Principles**:
- ✅ **Full Control**: Own the entire stack (hardware, software, deployment)
- ✅ **Zero Vendor Lock-in**: Can migrate to any provider in minutes
- ✅ **Cost Efficiency**: $50/month self-hosted vs $500/month managed services
- ✅ **Performance**: <50ms global latency with proper CDN + edge caching
- ✅ **Privacy**: Data sovereignty, GDPR compliance, no third-party tracking
- ✅ **Flexibility**: Custom configurations, unlimited resources, any technology stack

---

## 🚀 Deployment Platforms (Self-Hosted PaaS)

### **Coolify** 🌊 (RECOMMENDED)
**URL**: https://coolify.io
**Description**: Self-hosted Vercel/Netlify alternative with Docker-based deployments

**Features**:
- 📦 **Docker-based**: Deploy ANY application (Node.js, Python, Go, Rust, static sites)
- 🔄 **Git Integration**: Auto-deploy from GitHub/GitLab/Gitea on push
- 🌐 **Domain Management**: Automatic HTTPS with Let's Encrypt
- 📊 **Resource Monitoring**: CPU, RAM, disk usage per app
- 🔌 **Service Templates**: PostgreSQL, Redis, MongoDB, MySQL, MinIO
- 💰 **Pricing**: **FREE** (open-source, self-hosted)

**Performance**:
- Cold start: 1-2 seconds (Docker container warmup)
- Deploy time: 2-5 minutes (build + deploy)
- Uptime: 99.9% with health checks + auto-restart

**Use Cases**:
- SSR apps (TanStack Start, Next.js, SvelteKit)
- Static sites (Astro, Vite, plain HTML)
- API servers (Express, Fastify, tRPC)
- Background workers (BullMQ, cron jobs)
- Databases (PostgreSQL, Redis, MongoDB)

**Setup Time**: 10-15 minutes (install Coolify on Digital Ocean droplet)

---

### **CapRover** 🚢
**URL**: https://caprover.com
**Description**: Self-hosted PaaS with CLI-based deployments and one-click apps

**Features**:
- 📦 **Dockerfile Support**: Deploy from Dockerfile or use pre-built images
- 🔄 **CLI Deployment**: `caprover deploy` from local terminal
- 🌐 **Multi-Domain**: Host 100+ apps on single server
- 📊 **One-Click Apps**: WordPress, Ghost, Discourse, Grafana, etc.
- 🔌 **NGINX Proxy**: Automatic reverse proxy with SSL
- 💰 **Pricing**: **FREE** (open-source, self-hosted)

**Performance**:
- Cold start: 2-3 seconds
- Deploy time: 3-7 minutes
- Uptime: 99.8% with monitoring

**Use Cases**:
- Multi-tenant platforms (host many apps)
- Client project deployments
- Team collaboration environments

**Setup Time**: 15-20 minutes

---

### **Dokku** 🐳
**URL**: https://dokku.com
**Description**: Heroku-like PaaS built on Docker

**Features**:
- 📦 **Buildpack Support**: Auto-detect and build (Node.js, Python, PHP, Go)
- 🔄 **Git Push Deploy**: `git push dokku main` to deploy
- 🌐 **Plugin Ecosystem**: PostgreSQL, Redis, Let's Encrypt plugins
- 📊 **Zero Downtime**: Rolling deployments with health checks
- 💰 **Pricing**: **FREE** (open-source, self-hosted)

**Performance**:
- Cold start: 1-2 seconds
- Deploy time: 2-4 minutes
- Uptime: 99.9%

**Use Cases**:
- Simple Git-based workflows
- Heroku migration targets
- Small team projects

**Setup Time**: 10 minutes

---

### **Comparison Matrix**

| Feature | Coolify | CapRover | Dokku | Vercel | Cloudflare Workers |
|---------|---------|----------|-------|--------|-------------------|
| **Cost** | FREE | FREE | FREE | $20-500/mo | $5-50/mo |
| **Docker** | ✅ Native | ✅ Native | ✅ Native | ❌ Proprietary | ❌ V8 Isolates |
| **Git Deploy** | ✅ Auto | 🟡 CLI | ✅ Git Push | ✅ Auto | ✅ Auto |
| **GUI** | ✅ Modern | ✅ Basic | ❌ CLI Only | ✅ Excellent | ✅ Good |
| **Databases** | ✅ Templates | ✅ One-Click | 🟡 Plugins | 🟡 Add-ons ($) | 🟡 Limited |
| **Cold Start** | 1-2s | 2-3s | 1-2s | <100ms (Edge) | <10ms (Edge) |
| **Deploy Time** | 2-5min | 3-7min | 2-4min | 1-3min | 30s-2min |
| **Vendor Lock-in** | ❌ None | ❌ None | ❌ None | ⚠️ HIGH | ⚠️ HIGH |
| **Setup Time** | 10-15min | 15-20min | 10min | 5min | 10min |

**Recommendation**: **Coolify** for modern GUI + Docker flexibility + zero vendor lock-in

---

## 🌐 CDN & Edge Caching

### **Bunny CDN** 🐰 (RECOMMENDED for Production)
**URL**: https://bunny.net
**Description**: Affordable CDN with 114 global POPs and edge storage

**Features**:
- 🌍 **114 Global POPs**: <50ms latency worldwide
- 📦 **Edge Storage**: Host static assets (images, videos, files)
- 🔄 **Real-Time Purge**: Clear cache in <150ms globally
- 📊 **Video Streaming**: HLS/DASH transcoding + adaptive bitrate
- 🔒 **DDoS Protection**: Automatic mitigation up to 3 Tbps
- 💰 **Pricing**: **$1/TB bandwidth** + $0.01/GB storage

**Performance**:
- Edge latency (p95): 5ms
- Cache hit ratio: 95%+ (with proper cache headers)
- Bandwidth: Unlimited (pay per use)

**Use Cases**:
- Static assets (images, CSS, JS, fonts)
- Video streaming (HLS/DASH)
- File downloads (PDFs, ZIPs)
- Profile pictures (WebP conversion)

**Setup Time**: 5-10 minutes (create zone, point DNS)

---

### **Self-Hosted Varnish + nginx** 🛡️
**URL**: https://varnish-cache.org
**Description**: High-performance HTTP accelerator for caching

**Features**:
- ⚡ **Ultra-Fast**: In-memory caching, <1ms cache hits
- 🔧 **VCL Configuration**: Full control over caching logic
- 📊 **Edge-Side Includes**: Dynamic page assembly at cache layer
- 🔌 **Backend Health**: Automatic failover to healthy backends
- 💰 **Pricing**: **FREE** (open-source)

**Performance**:
- Cache hit latency: <1ms (in-memory)
- Cache miss latency: ~10ms (fetch from origin)
- Throughput: 50,000+ req/s per server

**Use Cases**:
- API response caching
- HTML page caching (SSR)
- Static asset serving
- GraphQL query caching

**Setup Time**: 20-30 minutes (configure VCL + nginx backend)

---

### **Comparison: CDN vs Self-Hosted**

| Metric | Bunny CDN | Self-Hosted Varnish | Cloudflare (Free) | Cloudflare (Pro) |
|--------|-----------|---------------------|-------------------|------------------|
| **Cost** | $1/TB | $0 (hosting only) | FREE | $20/mo |
| **Global POPs** | 114 | 1 (your region) | 330+ | 330+ |
| **Latency (p95)** | 5ms | 10ms (single region) | 8ms | 5ms |
| **Setup Time** | 5-10min | 20-30min | 5min | 5min |
| **Cache Hit Ratio** | 95%+ | 98%+ | 90%+ | 95%+ |
| **DDoS Protection** | ✅ 3 Tbps | ❌ None | ✅ Unmetered | ✅ Unmetered |
| **Vendor Lock-in** | 🟡 Minimal | ❌ None | ⚠️ Moderate | ⚠️ HIGH |
| **Video Streaming** | ✅ HLS/DASH | ❌ Manual | ❌ Manual | ✅ Stream |

**Recommendation**:
- **Production**: **Bunny CDN** ($1/TB) for global reach + DDoS protection
- **Development**: **Self-Hosted Varnish** for zero cost + full control
- **Hybrid**: Varnish origin → Bunny CDN edge (best of both worlds)

---

## 🔄 CI/CD (Continuous Integration/Deployment)

### **GitHub Actions + Self-Hosted Runners** 🚀
**URL**: https://github.com/features/actions
**Description**: Free CI/CD with 2000 minutes/month + unlimited self-hosted

**Features**:
- 🔄 **Git Integration**: Trigger on push, PR, release, schedule
- 🛠️ **Build Matrix**: Test across Node 18/20/22, multiple OSes
- 🔒 **Secrets Management**: Encrypted environment variables
- 🏃 **Self-Hosted Runners**: Unlimited minutes on your servers
- 💰 **Pricing**: **FREE** (2000 minutes/month) + **FREE** (self-hosted unlimited)

**Performance**:
- Build time: 2-5 minutes (with caching)
- Deploy time: 1-3 minutes
- Concurrency: 20+ parallel jobs (self-hosted)

**Use Cases**:
- Build + test on every push
- Deploy to Coolify on merge to main
- Run E2E tests with Playwright
- Generate database migrations

**Setup Time**: 10 minutes (create workflow YAML)

---

### **Self-Hosted Gitea + Drone** 🚁
**URLs**: https://gitea.io + https://drone.io
**Description**: Self-hosted Git + CI/CD (complete GitHub alternative)

**Features**:
- 📦 **Git Hosting**: Full GitHub alternative (repos, PRs, issues)
- 🔄 **Native Integration**: Drone CI triggers from Gitea webhooks
- 🛠️ **Pipeline as Code**: `.drone.yml` in repo root
- 🔒 **Private**: 100% self-hosted, zero data leaves your servers
- 💰 **Pricing**: **FREE** (open-source, self-hosted)

**Performance**:
- Build time: 1-4 minutes (local runners)
- Deploy time: 1-2 minutes
- Concurrency: Unlimited (scale runners)

**Use Cases**:
- Complete Git + CI/CD independence
- GDPR compliance (no GitHub data)
- Custom workflows + integrations
- Private monorepo builds

**Setup Time**: 30-45 minutes (install Gitea + Drone + configure)

---

### **Comparison Matrix**

| Feature | GitHub Actions | Gitea + Drone | GitLab CI | Jenkins |
|---------|----------------|---------------|-----------|---------|
| **Cost** | FREE (2000min) | FREE | FREE (400min) | FREE |
| **Self-Hosted** | ✅ Runners | ✅ Full | ✅ Full | ✅ Full |
| **Git Hosting** | ✅ GitHub | ✅ Gitea | ✅ GitLab | ❌ Separate |
| **Pipeline Syntax** | YAML | YAML | YAML | Jenkinsfile |
| **Docker Support** | ✅ Native | ✅ Native | ✅ Native | ✅ Plugins |
| **UI** | ✅ Excellent | ✅ Good | ✅ Good | 🟡 Dated |
| **Learning Curve** | Easy | Easy | Medium | Hard |
| **Setup Time** | 10min | 30-45min | 45min | 1-2hr |

**Recommendation**:
- **Hybrid**: **GitHub Actions** (free tier) + self-hosted runners (unlimited)
- **Full Independence**: **Gitea + Drone** (zero vendor dependency)

---

## 🗄️ Databases (Self-Hosted)

### **PostgreSQL** 🐘 (Primary Database)
**URL**: https://postgresql.org
**Current Setup**: Neon Postgres (serverless, production)

**Self-Hosted Benefits**:
- 💰 **Cost**: **FREE** vs $19-399/month (Neon)
- ⚡ **Performance**: No cold starts, <1ms latency (local)
- 🔒 **Privacy**: Data on your servers, GDPR compliance
- 🔧 **Extensions**: PostGIS, pgvector, TimescaleDB, Citus
- 📊 **Monitoring**: Grafana dashboards, pg_stat_statements

**Performance**:
- Query latency: <1ms (indexed queries)
- Throughput: 10,000+ queries/second
- Storage: Unlimited (disk space)

**Setup Time**: 15-20 minutes (Docker + pgAdmin)

---

### **Redis** ⚡ (Caching & Queues)
**URL**: https://redis.io
**Current Setup**: Not yet implemented (needed for BullMQ queues)

**Use Cases**:
- Session storage (WorkOS session cache)
- API response caching (5-minute TTL)
- Rate limiting (IP-based limits)
- BullMQ job queues (background tasks)
- Real-time features (pub/sub)

**Performance**:
- Latency: <1ms (in-memory)
- Throughput: 100,000+ ops/second
- Storage: RAM-based (8-32GB typical)

**Setup Time**: 5-10 minutes (Docker + Redis Commander)

---

### **TimescaleDB** ⏱️ (Time-Series Data)
**URL**: https://timescale.com
**Description**: PostgreSQL extension for time-series data (analytics, metrics)

**Use Cases**:
- Application metrics (API latency, error rates)
- User analytics (page views, sessions)
- IoT sensor data
- Financial data (stock prices, transactions)

**Performance**:
- Ingestion: 1 million rows/second
- Query speed: 10-100× faster than regular PostgreSQL (time-series queries)
- Compression: 90%+ compression (old data)

**Setup Time**: 10 minutes (PostgreSQL extension)

---

### **Comparison: Self-Hosted vs Managed**

| Database | Self-Hosted Cost | Managed Cost | Latency | Vendor Lock-in |
|----------|------------------|--------------|---------|----------------|
| **PostgreSQL** | $0 (hosting) | $19-399/mo (Neon) | <1ms | Neon-specific features |
| **Redis** | $0 (hosting) | $10-200/mo (Upstash) | <1ms | Redis Cloud-specific |
| **MongoDB** | $0 (hosting) | $9-500/mo (Atlas) | 2-5ms | MongoDB Atlas-specific |
| **MySQL** | $0 (hosting) | $15-300/mo (PlanetScale) | 1-3ms | PlanetScale-specific |

**Recommendation**: **Self-Hosted PostgreSQL + Redis** for production (zero monthly cost, <1ms latency)

---

## 📊 Monitoring & Observability

### **Grafana + Prometheus** 📈 (Metrics & Dashboards)
**URLs**: https://grafana.com + https://prometheus.io
**Description**: Self-hosted monitoring stack (metrics, alerts, dashboards)

**Features**:
- 📊 **Dashboards**: Beautiful visualizations (CPU, RAM, disk, custom metrics)
- 🔔 **Alerting**: Slack/Discord/Email notifications on thresholds
- 🔌 **Integrations**: PostgreSQL, Redis, Node.js, nginx, Docker
- 📈 **Time-Series**: Store metrics with 15s resolution, query with PromQL
- 💰 **Pricing**: **FREE** (open-source, self-hosted)

**Performance**:
- Scrape interval: 15 seconds (configurable)
- Query latency: <100ms (PromQL)
- Storage: 1 month retention = ~10GB per server

**Use Cases**:
- Server metrics (CPU, RAM, disk, network)
- Application metrics (API latency, error rates, throughput)
- Database metrics (query time, connections, cache hit ratio)
- Business metrics (signups, revenue, conversions)

**Setup Time**: 20-30 minutes (Docker Compose + Prometheus exporters)

---

### **Uptime Kuma** ⬆️ (Uptime Monitoring)
**URL**: https://uptime.kuma.pet
**Description**: Self-hosted status page + uptime monitoring

**Features**:
- 🔔 **Uptime Monitoring**: HTTP, TCP, ping, DNS checks
- 📊 **Status Page**: Public status page (like status.io)
- 🔌 **Notifications**: Slack, Discord, Telegram, Email, Webhook
- 📈 **Response Time**: Track and graph response times
- 💰 **Pricing**: **FREE** (open-source, self-hosted)

**Performance**:
- Check interval: 30 seconds (configurable)
- Alert latency: <5 seconds (instant notifications)
- Storage: 90 days history = ~500MB

**Use Cases**:
- Monitor production sites (dev.sacred-texts.com, sacred-texts.com)
- API endpoint monitoring (WorkOS, Neon, Bunny CDN)
- Certificate expiry alerts (SSL certificates)
- Public status page for users

**Setup Time**: 5-10 minutes (Docker + notification setup)

---

### **Comparison: Self-Hosted vs Managed**

| Tool | Self-Hosted Cost | Managed Cost | Features | Vendor Lock-in |
|------|------------------|--------------|----------|----------------|
| **Grafana + Prometheus** | $0 | $29-299/mo (Grafana Cloud) | Full | Grafana Cloud-specific |
| **Uptime Kuma** | $0 | $10-50/mo (UptimeRobot Pro) | Full | UptimeRobot API |
| **Sentry** (Errors) | $0 | $26-240/mo | Full | Sentry-specific SDK |
| **Datadog** | N/A | $15-1500/mo | Superior | HIGH |

**Recommendation**: **Self-Hosted Grafana + Prometheus + Uptime Kuma** ($0/month vs $100-500/month managed)

---

## ☁️ Digital Ocean Integration

### **Droplets** (Virtual Servers)
**URL**: https://digitalocean.com/products/droplets

**Pricing** (as of 2025):
- **Basic**: $6/month (1 CPU, 1GB RAM, 25GB SSD, 1TB transfer)
- **General Purpose**: $18/month (2 CPU, 4GB RAM, 80GB SSD, 4TB transfer)
- **CPU-Optimized**: $42/month (4 CPU, 8GB RAM, 100GB SSD, 5TB transfer)
- **Memory-Optimized**: $90/month (8 CPU, 16GB RAM, 160GB SSD, 5TB transfer)

**Recommendation for Sacred Texts Platform**:
- **Web Server**: General Purpose ($18/month) - TanStack Start SSR + Coolify
- **Database Server**: CPU-Optimized ($42/month) - PostgreSQL + Redis
- **Worker Server**: Basic ($6/month) - BullMQ background jobs
- **Total**: $66/month (vs $500+ on Vercel + Supabase + managed services)

---

### **Spaces** (Object Storage / S3 Compatible)
**URL**: https://digitalocean.com/products/spaces

**Features**:
- 📦 **S3-Compatible**: Drop-in replacement for AWS S3
- 🌍 **CDN Included**: Built-in CDN (150 POPs)
- 🔒 **Access Control**: IAM-like permissions, signed URLs
- 💰 **Pricing**: **$5/month** (250GB storage + 1TB transfer)

**Use Cases**:
- User uploads (profile pictures, documents)
- Static assets (images, videos, files)
- Database backups (automated daily backups)
- Application logs (archive old logs)

**Performance**:
- Upload latency: 10-50ms (depending on location)
- Download latency: <10ms (via CDN)
- Throughput: 100+ MB/s

**Setup Time**: 5 minutes (create Space, get access keys)

---

### **Managed Databases**
**URL**: https://digitalocean.com/products/managed-databases

**Pricing** (PostgreSQL):
- **Basic**: $15/month (1GB RAM, 10GB disk, 1 node)
- **Production**: $60/month (4GB RAM, 80GB disk, 1 node)
- **High Availability**: $120/month (4GB RAM, 80GB disk, 2 nodes)

**When to Use**:
- **Managed**: When team lacks DBA expertise, 99.99% uptime SLA required
- **Self-Hosted**: When cost-sensitive ($0 vs $15-120/month), full control needed

**Current Setup**: Neon Postgres (serverless, development + production)

---

### **Kubernetes (DOKS)**
**URL**: https://digitalocean.com/products/kubernetes

**Pricing**:
- **Control Plane**: **FREE** (no charge for control plane)
- **Worker Nodes**: Pay only for droplets ($6-90/month each)

**When to Use**:
- Multi-tenant platforms (100+ apps)
- Auto-scaling requirements
- Microservices architecture
- Large teams (DevOps expertise)

**When NOT to Use**:
- Single app deployments (overkill)
- Small teams (complexity overhead)
- Simple monoliths (Coolify is simpler)

---

### **Load Balancers**
**URL**: https://digitalocean.com/products/load-balancer

**Pricing**: **$12/month** (up to 10,000 concurrent connections)

**When to Use**:
- Traffic >5,000 concurrent users
- Zero-downtime deployments (blue-green)
- Multi-region failover

**When NOT to Use**:
- Single server deployments (nginx reverse proxy is FREE)
- Low traffic (<1,000 concurrent)

---

## 🎯 Strategic Decision Framework

When presenting infrastructure options to the user (via stuck agent), follow this format:

### **Decision Template**:

```yaml
Problem: [What infrastructure decision needs to be made?]

Options:
  1. [Option Name] ([Brand + URL])
     Cost: [Monthly cost or FREE]
     Performance: [Latency, throughput, uptime]
     Trade-offs:
       ✅ Pros: [Key benefits]
       ❌ Cons: [Limitations, drawbacks]
     Use Cases: [When to use this option]
     Setup Time: [How long to implement]

  2. [Option Name] ([Brand + URL])
     ...

  3. [Option Name] ([Brand + URL])
     ...

Recommendation: [Best option based on goals] (Rationale: [Why?])
```

### **Example: CDN Decision**

```yaml
Problem: Need CDN for static assets (images, CSS, JS) with global reach

Options:
  1. Bunny CDN (https://bunny.net)
     Cost: $1/TB bandwidth + $0.01/GB storage
     Performance: 5ms p95 latency, 114 POPs globally
     Trade-offs:
       ✅ Pros: Affordable, fast, DDoS protection, video streaming
       ❌ Cons: Small vendor lock-in (API-specific), $1/TB cost
     Use Cases: Production sites, high traffic, global users
     Setup Time: 5-10 minutes

  2. Self-Hosted Varnish + nginx (https://varnish-cache.org)
     Cost: FREE (hosting only)
     Performance: <1ms cache hits, 10ms misses (single region)
     Trade-offs:
       ✅ Pros: FREE, full control, zero vendor lock-in, ultra-fast (in-region)
       ❌ Cons: Single region (no global POPs), no DDoS protection, manual setup
     Use Cases: Development, regional sites, cost-sensitive
     Setup Time: 20-30 minutes

  3. Cloudflare Free (https://cloudflare.com)
     Cost: FREE (unlimited bandwidth)
     Performance: 8ms p95 latency, 330+ POPs globally
     Trade-offs:
       ✅ Pros: FREE, global reach, DDoS protection, easy setup
       ❌ Cons: HIGH vendor lock-in, aggressive caching (can break apps), data tracking
     Use Cases: Static sites, blogs, low-traffic apps
     Setup Time: 5 minutes

Recommendation: **Bunny CDN** ($1/TB) for production (Rationale: Best balance of cost, performance, and features. Global reach + DDoS protection + video streaming. Minimal vendor lock-in compared to Cloudflare.)
```

---

## 🚀 Autonomous Infrastructure Operations

**What Agents CAN Do** (without human approval):

### **Server Provisioning** (Digital Ocean API):
```bash
# Create droplet via API
curl -X POST https://api.digitalocean.com/v2/droplets \
  -H "Authorization: Bearer $DO_TOKEN" \
  -d '{"name":"web-01","region":"nyc3","size":"s-2vcpu-4gb","image":"ubuntu-24-04-x64"}'

# Wait for droplet to be ready (poll status)
# Configure SSH keys
# Install Docker + Coolify
# Deploy application
```

### **DNS Configuration**:
```bash
# Add A record via Digital Ocean API
curl -X POST https://api.digitalocean.com/v2/domains/sacred-texts.com/records \
  -H "Authorization: Bearer $DO_TOKEN" \
  -d '{"type":"A","name":"www","data":"<droplet-ip>","ttl":3600}'
```

### **SSL Certificates**:
```bash
# Install Caddy (automatic HTTPS)
docker run -d -p 80:80 -p 443:443 caddy:latest

# Or use Certbot (Let's Encrypt)
sudo certbot --nginx -d sacred-texts.com -d www.sacred-texts.com
```

### **Database Setup**:
```bash
# Install PostgreSQL via Docker
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=<secure-password> \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16-alpine

# Configure connection pooling (PgBouncer)
docker run -d \
  --name pgbouncer \
  -e DATABASES_HOST=postgres \
  -e DATABASES_PORT=5432 \
  -p 6432:6432 \
  edoburu/pgbouncer
```

### **Monitoring Setup**:
```bash
# Install Grafana + Prometheus via Docker Compose
# Install Uptime Kuma
# Configure alerts (Slack webhook)
# Setup dashboards (import pre-built JSON)
```

**What Requires Human Approval**:
- ❌ Choosing between deployment platforms (Coolify vs CapRover vs Dokku)
- ❌ Selecting CDN provider (Bunny vs Cloudflare vs self-hosted Varnish)
- ❌ Database migration strategy (self-hosted vs managed)
- ❌ Infrastructure budget allocation (server sizes, redundancy)

---

## 📚 Quick Reference Links

### **Deployment**:
- Coolify: https://coolify.io
- CapRover: https://caprover.com
- Dokku: https://dokku.com

### **CDN**:
- Bunny CDN: https://bunny.net
- Varnish Cache: https://varnish-cache.org
- Cloudflare: https://cloudflare.com

### **CI/CD**:
- GitHub Actions: https://github.com/features/actions
- Gitea: https://gitea.io
- Drone CI: https://drone.io

### **Databases**:
- PostgreSQL: https://postgresql.org
- Redis: https://redis.io
- TimescaleDB: https://timescale.com

### **Monitoring**:
- Grafana: https://grafana.com
- Prometheus: https://prometheus.io
- Uptime Kuma: https://uptime.kuma.pet

### **Cloud Providers**:
- Digital Ocean: https://digitalocean.com
- Hetzner: https://hetzner.com (even cheaper, EU-based)
- Linode/Akamai: https://linode.com

---

## 🎯 Cost Breakdown: Self-Hosted vs Managed

### **Sacred Texts Platform - Current Estimate**:

**Managed Services** (Vercel + Supabase + Managed CDN):
```
Vercel Pro: $20/month (1 project, 100GB bandwidth)
Supabase Pro: $25/month (8GB database, 100GB bandwidth)
Bunny CDN: $10/month (10TB bandwidth)
Uptime Monitoring: $10/month (UptimeRobot Pro)
Error Tracking: $26/month (Sentry Team)
Total: $91/month ($1,092/year)
```

**Self-Hosted** (Digital Ocean + Coolify):
```
Web Server (Coolify): $18/month (2 CPU, 4GB RAM, TanStack Start SSR)
Database Server: $42/month (4 CPU, 8GB RAM, PostgreSQL + Redis)
CDN (Bunny): $10/month (10TB bandwidth, can't beat the price)
Monitoring (Self-Hosted): $0/month (Grafana + Prometheus + Uptime Kuma)
Total: $70/month ($840/year)

Savings: $252/year (23% cheaper) + ZERO vendor lock-in
```

**Self-Hosted + Hybrid CDN** (Best of Both Worlds):
```
Web Server (Coolify): $18/month
Database Server: $42/month
CDN (Bunny Edge): $1/TB actual usage (pay as you go)
Monitoring: $0/month (self-hosted)
Total: $60/month + $1/TB ($720/year + bandwidth)

Savings: $372/year (34% cheaper) + full control + minimal vendor lock-in
```

---

**Remember**: Self-hosting isn't just about cost - it's about CONTROL, FLEXIBILITY, and ZERO VENDOR LOCK-IN. Build infrastructure that YOU own, that YOU can move anywhere, and that scales with YOUR needs! 🚀
