# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive Docker Compose setup for running Ghost CMS in production with optional analytics and ActivityPub support. The repository orchestrates multiple services including Ghost, MySQL, and optional Tinybird analytics and ActivityPub federation. An external Nginx (installed on the host) is used as the reverse proxy.

## Architecture

The project uses Docker Compose to orchestrate these services:

1. **Ghost** - The main CMS application (exposed on `127.0.0.1:2368`)
2. **MySQL** - Database backend with health checks and support for multiple databases
3. **Traffic Analytics** (optional profile) - Tinybird integration for web analytics
4. **ActivityPub** (optional profile) - Federated social networking support
5. **Supporting services** - Tinybird setup tools and ActivityPub migrations

**Nginx** runs on the host (outside Docker) as a reverse proxy, handling HTTPS/SSL termination and proxying requests to Ghost on `127.0.0.1:2368`. See `nginx.conf.example` for a sample configuration.

## Common Commands

```bash
# Core operations
docker compose up -d                    # Start Ghost + MySQL
docker compose down                     # Stop all services
docker compose logs -f [service]        # View logs (e.g., ghost, mysql)
docker compose ps                       # Check service status
docker compose pull                     # Update all images
docker compose restart ghost            # Restart just Ghost

# With optional profiles
docker compose --profile=analytics up -d     # Include analytics services
docker compose --profile=activitypub up -d   # Include ActivityPub services
COMPOSE_PROFILES=analytics,activitypub docker compose up -d  # Start everything

# Tinybird analytics setup (if using analytics profile)
docker compose run --rm tinybird-login       # Interactive Tinybird login
docker compose --profile=analytics up tinybird-sync   # Sync datasources/pipes
docker compose --profile=analytics up tinybird-deploy # Deploy configuration

# Development & debugging
docker compose exec ghost sh            # Access Ghost container shell
docker compose exec mysql mysql -u root -p  # Access MySQL CLI
```

## Configuration

All configuration is done via environment variables. Key patterns:

- **Required variables**: `DOMAIN`, `DATABASE_PASSWORD`, `DATABASE_ROOT_PASSWORD`
- **Ghost config pattern**: `section__subsection__key=value` (e.g., `mail__options__service=Mailgun`)
- **Developer experiments**: Must set `labs__publicAPI=true` for analytics/ActivityPub features
- **Data persistence**: Volumes stored in `./data/ghost` and `./data/mysql`

### Key configuration files:
- `.env` - Main environment configuration (create from `.env.example`)
- `compose.yml` - Docker Compose service definitions
- `nginx.conf.example` - Example Nginx reverse proxy configuration
- `mysql-init/create-multiple-databases.sh` - MySQL multi-database initialization

## Migration from Ghost CLI

The repository includes comprehensive migration tools:

- `scripts/migrate.sh` - Main migration script that:
  - Backs up existing Ghost installation
  - Automatically tries Ghost's database credentials first
  - Only prompts for alternative credentials if needed
  - Uses `--no-tablespaces` flag to avoid PROCESS privilege requirements
  - Converts config.json to environment variables
  - Preserves content and database
  - Creates recovery script with clear restoration instructions
  - Sets up Docker Compose environment

- `scripts/config-to-env.js` - Converts Ghost JSON config to .env format

## Development Workflow

1. Clone repository and copy `.env.example` to `.env`
2. Configure required environment variables (domain, passwords)
3. Run `docker compose up -d` to start services
4. Configure Nginx on the host using `nginx.conf.example` as a template
5. Access Ghost at `https://DOMAIN` (Nginx handles SSL)
6. Monitor logs with `docker compose logs -f ghost`

For analytics setup, see `TINYBIRD.md` for detailed instructions.

## Important Notes

- Ghost is exposed on `127.0.0.1:2368`; Nginx on the host proxies external traffic to it
- Email configuration is critical even without newsletter features (used for admin notifications)
- MySQL health checks ensure database is ready before Ghost starts
- The compose file uses yaml-language-server schema for IDE completion support
- For production, always use strong passwords and consider additional security measures
