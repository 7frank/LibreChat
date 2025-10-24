# LibreChat Development Environment

This guide explains how to run LibreChat in full development mode using Tilt.dev with live reload support.

## Overview

The development setup provides:
- **Frontend**: Vite dev server with Hot Module Replacement (HMR)
- **Backend**: Node.js with Nodemon for auto-restart
- **Live Sync**: Changes to source files automatically sync to the container
- **Full Stack**: All dependencies (MongoDB, Meilisearch, RAG API) running in containers

## Prerequisites

1. **Docker** (or compatible container runtime)
2. **Tilt.dev** - Install from https://tilt.dev
3. **kubectl** (optional, for Kubernetes mode)

### Install Tilt

```bash
# macOS
curl -fsSL https://raw.githubusercontent.com/tilt-dev/tilt/master/scripts/install.sh | bash

# Linux
curl -fsSL https://raw.githubusercontent.com/tilt-dev/tilt/master/scripts/install.sh | bash

# Windows (PowerShell)
iex ((new-object net.webclient).DownloadString('https://raw.githubusercontent.com/tilt-dev/tilt/master/scripts/install.ps1'))
```

## Quick Start

1. **Clone and navigate to the project**:
   ```bash
   cd LibreChat
   ```

2. **Copy environment file**:
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

3. **Start development environment**:
   ```bash
   tilt up
   ```

4. **Access the application**:
   - **Frontend**: http://localhost:3090 (Vite dev server with HMR)
   - **Backend API**: http://localhost:3080
   - **Tilt UI**: http://localhost:10350 (opens automatically)

5. **Press `space`** in the terminal to open Tilt web UI in your browser

## What's Running

When you run `tilt up`, the following services start:

| Service | Port | Description |
|---------|------|-------------|
| **Frontend** | 3090 | Vite dev server with HMR |
| **Backend** | 3080 | Node.js API with Nodemon |
| MongoDB | 27017 | Database |
| Meilisearch | 7700 | Search engine |
| RAG API | 8000 | Retrieval-Augmented Generation |
| VectorDB | 5432 | PostgreSQL with pgvector |

## Development Workflow

### Making Changes

1. **Frontend Changes** (`client/`):
   - Edit React components, TypeScript files, or styles
   - Changes are synced to the container
   - Vite HMR updates the browser instantly (no page reload)

2. **Backend Changes** (`api/`):
   - Edit server files, routes, or controllers
   - Changes are synced to the container
   - Nodemon detects changes and restarts the server automatically

3. **Package Changes** (`packages/`):
   - Edit shared packages (data-provider, data-schemas, api)
   - Changes are synced to the container
   - May require manual rebuild: `npm run build:packages` inside container

### Viewing Logs

In the Tilt UI (press `space` or visit http://localhost:10350):
- Click on a service to view its logs
- Filter logs by service
- See build status and resource health

Or use the terminal:
```bash
# View all logs
tilt logs

# View specific service
tilt logs librechat-dev
```

### Restarting Services

```bash
# Restart all services
tilt down && tilt up

# Rebuild and restart
tilt up --rebuild
```

### Stopping Development Environment

```bash
# Stop all services
tilt down

# Clean up volumes and data
docker compose down -v
```

## File Structure

```
LibreChat/
├── Dockerfile.dev          # Development Docker image
├── .dockerignore.dev       # Files to exclude in dev builds
├── Tiltfile                # Tilt orchestration configuration
├── docker-compose.yml      # Docker Compose for dependencies
├── api/                    # Backend source (live synced)
├── client/                 # Frontend source (live synced)
├── packages/               # Shared packages (live synced)
└── DEV.md                  # This file
```

## Architecture

### Dockerfile.dev

The development Dockerfile:
- Installs **all dependencies** (including devDependencies)
- Copies **source files** (not built assets)
- Builds **only packages** (data-provider, data-schemas, api)
- Runs **both services** concurrently:
  - `npm run backend:dev` → Nodemon on port 3080
  - `npm run frontend:dev` → Vite on port 3090

### Tiltfile

The Tilt configuration:
- Builds the dev Docker image with `Dockerfile.dev`
- Sets up **live_update** rules to sync code changes
- Configures port forwarding (3080, 3090)
- Orchestrates dependencies (MongoDB, Meilisearch, etc.)
- Provides resource ordering and health checks

## Differences from Production

| Aspect | Production | Development |
|--------|-----------|-------------|
| **Frontend** | Static build (`vite build`) | Dev server (`vite dev`) |
| **Backend** | Production mode | Development mode with nodemon |
| **Dependencies** | Production only | All (including dev) |
| **Source Maps** | Disabled | Enabled |
| **Hot Reload** | None | Full HMR (frontend) + auto-restart (backend) |
| **File Sync** | None | Live sync via Tilt |
| **Build Time** | ~5-10 min | ~2-3 min (no frontend build) |

## Troubleshooting

### Port Already in Use

If ports 3080 or 3090 are already in use:
```bash
# Find process using the port
lsof -i :3080
lsof -i :3090

# Kill the process
kill -9 <PID>
```

### Node Modules Out of Sync

If dependencies are out of sync:
```bash
# Rebuild the dev image
tilt down
docker rmi librechat-dev
tilt up
```

### Changes Not Reflecting

1. Check Tilt UI for sync errors
2. Verify file is not in `.dockerignore.dev`
3. Restart the service: `tilt up --rebuild`

### Frontend Not Loading

Ensure Vite proxy is configured correctly in `client/vite.config.ts`:
```typescript
proxy: {
  '/api': { target: 'http://localhost:3080', changeOrigin: true },
  '/oauth': { target: 'http://localhost:3080', changeOrigin: true }
}
```

### Database Connection Issues

Ensure MongoDB is running:
```bash
docker ps | grep mongodb
```

Check connection string in `.env`:
```
MONGO_URI=mongodb://mongodb:27017/LibreChat
```

## Advanced Usage

### Running Commands Inside Container

```bash
# Get container ID
docker ps | grep librechat

# Execute shell
docker exec -it <container_id> sh

# Run npm commands
docker exec -it <container_id> npm run build:packages
```

### Custom Tilt Configuration

Edit `Tiltfile` to customize:
- Port mappings
- Live sync rules
- Resource dependencies
- Environment variables

### Using with Local Backend

If you want to run backend locally but frontend in container:

1. Comment out backend in `Dockerfile.dev`
2. Run backend locally: `npm run backend:dev`
3. Update Vite proxy to point to `localhost:3080`

## Performance Tips

1. **Use .dockerignore.dev**: Exclude unnecessary files to speed up syncing
2. **Limit live_update scope**: Only sync changed directories
3. **Disable unused services**: Comment out services in `docker-compose.yml` if not needed
4. **Use volumes for node_modules**: Prevent syncing large dependency folders

## Resources

- **Tilt Documentation**: https://docs.tilt.dev
- **LibreChat Docs**: https://librechat.ai/docs
- **Vite Documentation**: https://vitejs.dev
- **Docker Documentation**: https://docs.docker.com

## Support

For issues or questions:
- **GitHub Issues**: https://github.com/danny-avila/LibreChat/issues
- **Discord**: https://discord.librechat.ai
- **Documentation**: https://librechat.ai/docs

---

**Happy Developing! 🚀**
