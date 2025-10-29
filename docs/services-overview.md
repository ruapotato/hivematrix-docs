# Services Overview

HiveMatrix is composed of independent services that work together to provide a complete MSP platform. Each service is a self-contained application with its own database and user interface.

## Core Services

These services are required for HiveMatrix to function.

### Core
**Repository**: [hivematrix-core](https://github.com/ruapotato/hivematrix-core)
**Port**: 5000
**Database**: PostgreSQL

The authentication and service registry hub. Core handles:

- **JWT Token Management**: Issues and validates HiveMatrix JWTs
- **Keycloak Integration**: Exchanges Keycloak tokens for HiveMatrix tokens
- **Service Registry**: Maintains list of available services for Nexus
- **Public Key Distribution**: Provides JWKS endpoint for JWT verification
- **Session Management**: Tracks active sessions with revocation support

Core is the first service started and other services depend on it for authentication.

**Key Endpoints**:
- `/api/token/exchange` - Exchange Keycloak token for HiveMatrix JWT
- `/api/token/validate` - Validate and check JWT status
- `/api/token/revoke` - Revoke a session
- `/.well-known/jwks.json` - Public keys for JWT verification
- `/api/services` - List of registered services

### Nexus
**Repository**: [hivematrix-nexus](https://github.com/ruapotato/hivematrix-nexus)
**Port**: 443 (HTTPS)
**Database**: None (stateless proxy)

The frontend gateway and smart proxy. Nexus provides:

- **HTTPS Termination**: SSL/TLS for all services
- **Authentication Gateway**: Handles login flow with Keycloak
- **Smart Proxying**: Routes requests to backend services
- **Session Management**: Stores JWTs in encrypted session cookies
- **Service Composition**: Combines UIs from multiple services
- **Static Asset Serving**: Provides global CSS and common UI elements

Nexus is the only service accessible from the internet. All other services run on localhost.

**Key Features**:
- Automatic Keycloak proxy for authentication
- JWT validation on every request
- Dynamic service discovery from Core
- Global navigation bar with service switcher
- Responsive design system

### Helm
**Repository**: [hivematrix-helm](https://github.com/ruapotato/hivematrix-helm)
**Port**: 5004
**Database**: PostgreSQL

The orchestration and management system. Helm provides:

- **Service Management**: Start, stop, restart services
- **Log Aggregation**: Centralized logging from all services
- **Health Monitoring**: Check service status and health
- **Configuration Management**: Sync configs across services
- **Installation Management**: Add/remove services
- **Security Auditing**: Port scanning and firewall checks

Helm is admin-only and provides a dashboard for platform management.

**Key Features**:
- Real-time service status monitoring
- Centralized log viewer with search
- One-click service restart
- Automatic service discovery
- Security audit reports

## Application Services

These services provide specific business functionality.

### Codex
**Repository**: [hivematrix-codex](https://github.com/ruapotato/hivematrix-codex)
**Port**: 5010
**Database**: PostgreSQL
**[→ Detailed Documentation](services/codex.md)**

The central data hub and API aggregation point for HiveMatrix. Codex serves as the "rolodex" managing:

- **Companies**: Client and vendor information
- **Contacts**: People associated with companies
- **Assets**: Equipment, licenses, and resources
- **Network Equipment**: Routers, switches, access points
- **Third-Party Integration**: Freshservice and Datto RMM sync

Codex is the master data source that other services query for company/contact information.

**Key Features**:
- Company hierarchy support (parent/child relationships)
- Contact management with roles
- Asset tracking with warranty dates
- Network topology mapping
- Bi-directional sync with PSA/RMM tools
- Import/export capabilities

**API Endpoints**:
- `/api/companies` - Company CRUD operations
- `/api/contacts` - Contact management
- `/api/assets` - Asset tracking
- `/api/network_equipment` - Network device management

### Ledger
**Repository**: [hivematrix-ledger](https://github.com/ruapotato/hivematrix-ledger)
**Port**: 5030
**Database**: PostgreSQL

Billing calculations and invoicing system. Ledger handles:

- **Billing Plans**: Define pricing structures
- **Usage Tracking**: Record billable services
- **Invoice Generation**: Calculate and create invoices
- **Payment Tracking**: Record payments and credits
- **Reporting**: Billing analytics and insights

Ledger integrates with Codex to bill clients for services.

**Key Features**:
- Flexible billing plan system (per-user, flat-rate, tiered)
- Automated invoice generation
- Pro-rating for mid-cycle changes
- Multi-currency support
- Aging reports
- Payment history tracking

**API Endpoints**:
- `/api/plans` - Billing plan management
- `/api/invoices` - Invoice generation and retrieval
- `/api/usage` - Usage tracking
- `/api/payments` - Payment recording

### KnowledgeTree
**Repository**: [hivematrix-knowledgetree](https://github.com/ruapotato/hivematrix-knowledgetree)
**Port**: 5020
**Database**: Neo4j (graph database)
**[→ Detailed Documentation](services/knowledgetree.md)**

Knowledge base system with hierarchical organization. KnowledgeTree provides:

- **Graph-Based Knowledge**: Store information with relationships
- **Hierarchical Organization**: Sections, categories, topics
- **Rich Content**: Markdown support with attachments
- **Relationship Mapping**: Connect related information
- **Search**: Full-text search across all content
- **Company Scoping**: Knowledge tied to specific clients

KnowledgeTree uses Neo4j to represent complex relationships between knowledge items.

**Key Features**:
- Graph visualization of relationships
- Hierarchical browsing (sections > categories > topics)
- Markdown formatting with code highlighting
- Full-text search
- Company-specific knowledge bases
- Tag-based organization

**API Endpoints**:
- `/api/sections` - Top-level organization
- `/api/topics` - Knowledge items
- `/api/search` - Full-text search
- `/api/graph` - Relationship data

### Brainhair
**Repository**: [hivematrix-brainhair](https://github.com/ruapotato/hivematrix-brainhair)
**Port**: 5050
**Database**: PostgreSQL

AI assistant with PHI/CJIS filtering. Brainhair provides:

- **Claude AI Integration**: Natural language interface to HiveMatrix
- **Sensitive Data Filtering**: Microsoft Presidio for PHI/CJIS detection
- **Approval Workflow**: Review AI actions before execution
- **Tool System**: Extensible AI capabilities
- **Conversation History**: Track AI interactions

Brainhair allows administrators to manage HiveMatrix using natural language while protecting sensitive information.

**Key Features**:
- Automatic PHI/CJIS filtering (SSN, credit cards, etc.)
- Approval flow for write operations
- Integration with Codex, Ledger, and other services
- Conversation context management
- Tool framework for extending capabilities

**AI Tools**:
- Company management (create, update, search)
- Invoice generation
- Network equipment queries
- Feature flag management
- Billing plan modifications

## Optional Services

### Archive
**Repository**: [hivematrix-archive](https://github.com/ruapotato/hivematrix-archive)
**Port**: 5012
**Database**: PostgreSQL

Immutable billing snapshot storage service. Archive captures permanent records of billing data:

- **Billing Snapshots**: Stores finalized bills from Ledger as immutable records
- **Historical Lookup**: Search past bills by company, year, and month
- **CSV Export**: Download invoices for accounting and auditing
- **Automated Capture**: Scheduled monthly snapshots of all company billing
- **Audit Trail**: Complete history of accepted bills with timestamps

**Key Features**:
- Permanent, unchangeable billing records
- Integration with Ledger's "Accept Bill" button
- Automated monthly snapshot creation
- Search by company, period, or invoice number
- CSV invoice downloads
- Job tracking for scheduled snapshots

**API Endpoints**:
- `/api/snapshot` - Create billing snapshot
- `/api/snapshot/{invoice_number}` - Retrieve specific snapshot
- `/api/snapshot/{invoice_number}/csv` - Download CSV invoice
- `/api/snapshots/search` - Search historical bills
- `/api/scheduler/config` - Configure automated snapshots

### Template
**Repository**: [hivematrix-template](https://github.com/ruapotato/hivematrix-template)
**Port**: 5040
**Database**: PostgreSQL

Template for creating new HiveMatrix services. Use this as a starting point for custom services.

## Service Dependencies

```
┌─────────────┐
│  Keycloak   │ (Authentication Server)
└──────┬──────┘
       │
       ├─────► Core (Authentication & Registry)
       │          │
       │          ├─────► Codex (Master Data)
       │          │          │
       │          │          ├─────► Ledger (Billing)
       │          │          │
       │          │          ├─────► KnowledgeTree (Knowledge)
       │          │          │
       │          │          └─────► Brainhair (AI Assistant)
       │          │
       │          └─────► Helm (Management)
       │
       └─────► Nexus (Gateway)
                  │
                  └─────► All Services (Proxied Access)
```

## Service Communication

Services communicate through:

1. **HTTP APIs**: JSON REST endpoints
2. **JWT Authentication**: Bearer tokens for service-to-service calls
3. **Service Registry**: Core maintains service locations
4. **Shared Database**: Some services query Codex directly (read-only)

## Adding New Services

See the [Development Guide](development.md) for details on creating custom services using the template.

## Service Management

All services are managed through Helm:

```bash
# Via CLI
cd hivematrix-helm
source pyenv/bin/activate

# Check status
python service_manager.py status

# Start/stop services
python service_manager.py start codex
python service_manager.py stop ledger
python service_manager.py restart nexus

# View logs
python logs_cli.py --service core --tail 100
```

Or through the Helm web interface at http://localhost:5004
