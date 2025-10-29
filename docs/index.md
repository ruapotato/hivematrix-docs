# HiveMatrix Documentation

Welcome to the HiveMatrix platform documentation. HiveMatrix is a modular, AI-maintainable platform for MSP (Managed Service Provider) operations.

## Quick Links

- [Architecture & Development Guide](ARCHITECTURE.md) - Complete technical architecture and development guidelines
- [Claude AI Guide](CLAUDE.md) - Instructions for AI assistants working on HiveMatrix

## What is HiveMatrix?

HiveMatrix is a collection of independent, fully functional applications that work together to provide:

- **Client Management** - Company and contact tracking
- **Billing & Invoicing** - Automated billing calculations via Ledger
- **Knowledge Management** - Graph-based knowledge system with KnowledgeTree
- **AI Integration** - Claude AI integration via Brainhair
- **Document Management** - File archival with Archive
- **Ticketing** - Coming soon with Resolve

## Core Principles

1. **AI Maintainability** - Each application remains small, focused, and simple
2. **Modularity** - Independent services that can be composed together
3. **Simplicity** - Simple, explicit patterns over complex, "magical" ones

## Getting Started

### Installation

```bash
# Clone the orchestration system
git clone https://github.com/ruapotato/hivematrix-helm
cd hivematrix-helm

# Run the installation
./start.sh
```

The installation will:
1. Install system dependencies (PostgreSQL, Neo4j, Java, Python)
2. Set up Keycloak authentication server
3. Clone and configure Core and Nexus (required components)
4. Set up databases
5. Start all services

### Default Credentials

- **Username**: admin
- **Password**: admin

**Important**: Change the default password after first login!

### Access

- **Main Interface**: https://localhost:443
- **Helm Dashboard**: http://localhost:5004
- **Keycloak Admin**: http://localhost:8080

## Architecture Overview

HiveMatrix uses a **monolithic service pattern** where each module is a self-contained application with:

- Server-side rendering (returns complete HTML)
- Own database (no cross-service database access)
- Optional data APIs (JSON endpoints)

### Authentication Flow

1. User logs in via Keycloak (through Nexus proxy)
2. Keycloak validates credentials
3. Core mints a HiveMatrix JWT with user permissions
4. Nexus stores JWT in session cookie
5. Backend services verify JWT using Core's public key

### Key Components

- **Core** - Authentication, JWT management, service registry
- **Nexus** - Frontend gateway, HTTPS termination, smart proxying
- **Helm** - Orchestration system, service management
- **Codex** - Central data hub (companies, contacts, assets)
- **Ledger** - Billing calculations and invoicing
- **KnowledgeTree** - Knowledge management with Neo4j
- **Brainhair** - AI assistant with PHI/CJIS filtering

## Development

### Running the Development Environment

```bash
cd hivematrix-helm
./start.sh
```

### Stopping Services

```bash
./stop.sh
```

### Viewing Logs

```bash
# Via CLI tool
cd hivematrix-helm
source pyenv/bin/activate
python logs_cli.py --service core --tail 50

# Or check log files directly
tail -f logs/core.stdout.log
```

### Configuration

All configuration is centralized in `hivematrix-helm`:

- `apps_registry.json` - Source of truth for all apps
- `master_config.json` - System-wide configuration
- `instance/configs/` - Service-specific configs (auto-generated)

### Backup & Restore

```bash
# Create backup
sudo python3 backup.py

# Restore from backup
./stop.sh
sudo python3 restore.py /tmp/hivematrix_backup_*.zip
./start.sh
```

## Documentation Structure

- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Complete technical documentation including:
  - Core philosophy and goals
  - Service patterns and architecture
  - Authentication and authorization
  - Frontend composition
  - Configuration management
  - Development tools and debugging
  - Security architecture
  - Database best practices

- **[CLAUDE.md](CLAUDE.md)** - AI assistant guidelines for:
  - Understanding HiveMatrix structure
  - Making code changes
  - Following conventions
  - Debugging issues

## Contributing

When contributing to HiveMatrix:

1. Read [ARCHITECTURE.md](ARCHITECTURE.md) thoroughly
2. Follow the monolithic service pattern
3. Keep services small and focused
4. Use server-side rendering
5. Never access another service's database directly
6. Update documentation for architectural changes

## License

[Add license information]

## Support

For issues and questions:
- GitHub Issues: [HiveMatrix Repositories](https://github.com/ruapotato/)
- Documentation: This site

## Version

Current version: 4.0 (See [ARCHITECTURE.md](ARCHITECTURE.md) version history)
