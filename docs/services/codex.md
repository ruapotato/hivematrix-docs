# Codex - Central Data Hub

**Port**: 5010
**Database**: PostgreSQL
**Repository**: [hivematrix-codex](https://github.com/ruapotato/hivematrix-codex)

## Overview

Codex is the central data hub and API aggregation point for HiveMatrix. It serves as the "rolodex" of the entire platform, storing and managing all company, contact, and asset information.

## Purpose

Think of Codex as the master address book and inventory system. Other services query Codex when they need to know:
- Who works at which company?
- What equipment does a client have?
- What are the contact details for a specific person?
- What's the user's preferred theme?

## What Codex Stores

### Companies
- Client and vendor information
- Company hierarchy (parent/child relationships)
- Account numbers and billing information
- Company-specific settings and preferences

### Contacts
- People associated with companies
- Contact information (email, phone)
- Roles and responsibilities
- Primary/secondary contact designation

### Assets
- Equipment inventory (servers, workstations, network devices)
- Software licenses
- Warranty information and expiration dates
- Asset assignment to companies

### Network Equipment
- Routers, switches, access points, firewalls
- IP addresses and network configuration
- Device models and firmware versions
- Network topology relationships

### User Preferences
- Theme settings (light/dark mode)
- UI preferences
- Notification settings

### Third-Party Integration Data
- **Freshservice**: Synced ticket and asset data
- **Datto RMM**: Device monitoring and management data
- Bidirectional sync to keep data current

## Key Features

### Company Management
- Create and manage client companies
- Set up company hierarchies for multi-location businesses
- Track billing accounts and contract information
- Assign contacts and assets to companies

### Contact Management
- Store contact details with PHI/CJIS filtering
- Assign contacts to specific companies
- Mark primary contacts for quick reference
- Track contact roles (billing, technical, executive)

### Asset Tracking
- Inventory management for all client equipment
- Warranty expiration tracking
- Assignment tracking (which company owns what)
- Integration with RMM tools for automatic updates

### Network Topology
- Map network equipment and connections
- Track IP addresses and subnets
- Monitor device status through RMM integration
- Visualize network infrastructure

### API First Design
Codex exposes comprehensive REST APIs that other services use:
- `/api/companies` - Company CRUD operations
- `/api/contacts` - Contact management
- `/api/assets` - Asset inventory
- `/api/network_equipment` - Network device management
- `/api/tickets` - Ticket data from integrations

## Integration Points

### Services That Use Codex

**Ledger** (Billing)
- Queries company information for invoicing
- Retrieves asset counts for billing calculations
- Uses company settings for billing plans

**Brainhair** (AI Assistant)
- Searches companies and contacts
- Queries asset information
- Updates records through approved actions

**KnowledgeTree**
- Associates knowledge articles with companies
- Retrieves company context for documentation

**Nexus**
- Fetches user theme preferences
- Provides company data for navigation

### External Systems

**Freshservice**
- Syncs companies, contacts, and tickets
- Updates asset information
- Pulls change requests and incidents

**Datto RMM**
- Syncs device inventory
- Updates asset status and monitoring data
- Pulls alert information

## Data Sync

Codex includes synchronization scripts that run periodically:

```bash
# Sync from Freshservice
cd hivematrix-codex
source pyenv/bin/activate
python pull_freshservice.py

# Sync from Datto RMM
python pull_datto.py
```

These scripts:
1. Connect to external APIs
2. Fetch updated data
3. Update or create records in Codex database
4. Log sync status to Helm

## Database Schema

### Main Tables

**companies**
- `id` - Primary key
- `name` - Company name
- `account_number` - External account ID
- `parent_id` - For company hierarchies
- `billing_contact_id` - Default billing contact

**contacts**
- `id` - Primary key
- `company_id` - Foreign key to companies
- `name` - Contact name (PHI filtered)
- `email` - Email address
- `phone` - Phone number
- `is_primary` - Primary contact flag

**assets**
- `id` - Primary key
- `company_id` - Foreign key to companies
- `name` - Asset name/description
- `type` - server, workstation, license, etc.
- `serial_number` - Device serial
- `warranty_expires` - Warranty date

**network_equipment**
- `id` - Primary key
- `company_id` - Foreign key to companies
- `name` - Device name
- `type` - router, switch, firewall, ap
- `ip_address` - Device IP
- `model` - Hardware model

**agents**
- `id` - Primary key
- `email` - User email (Keycloak username)
- `name` - Display name
- `theme` - UI theme preference
- `permission_level` - admin, technician, billing, client

## API Examples

### Get All Companies
```bash
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:5010/api/companies
```

### Get Company by ID
```bash
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:5010/api/companies/123
```

### Create New Contact
```bash
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "company_id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "555-1234",
    "is_primary": true
  }' \
  http://localhost:5010/api/contacts
```

### Search Assets
```bash
curl -H "Authorization: Bearer $TOKEN" \
  "http://localhost:5010/api/assets?company_id=123&type=server"
```

## Configuration

Codex configuration is in `instance/codex.conf`:

```ini
[database]
connection_string = postgresql://codex_user:password@localhost:5432/codex_db

[integrations]
freshservice_api_key = your-api-key
freshservice_domain = your-domain.freshservice.com
datto_api_key = your-datto-key
datto_api_secret = your-datto-secret
datto_api_url = https://your-instance.centrastage.net
```

## Common Operations

### Add a New Company
1. Navigate to Codex → Companies
2. Click "Add Company"
3. Fill in company details
4. Assign primary contact
5. Add assets and network equipment as needed

### Sync External Data
```bash
# Manual sync from Freshservice
cd hivematrix-codex
source pyenv/bin/activate
python pull_freshservice.py

# Set up automated sync with cron
0 */4 * * * cd /path/to/hivematrix-codex && pyenv/bin/python pull_freshservice.py >> /var/log/codex_sync.log 2>&1
```

### Export Company Data
```bash
# Export companies to CSV
curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:5010/api/companies?format=csv > companies.csv
```

## Troubleshooting

### Sync Failures
Check logs for API errors:
```bash
cd hivematrix-helm
source pyenv/bin/activate
python logs_cli.py codex --tail 100
```

### Missing Data
Verify external API credentials:
```bash
cd hivematrix-codex
cat instance/codex.conf
# Test API connection manually
```

### Performance Issues
Codex handles large datasets. For slow queries:
1. Check database indexes
2. Review PostgreSQL query logs
3. Consider adding pagination to API calls

## Security

- All API endpoints require JWT authentication
- PHI/CJIS filtering applied to contact data
- External API keys stored in config file (not in code)
- Database credentials in instance directory (not in git)

## See Also

- [Services Overview](../services-overview.md)
- [Configuration Guide](../configuration.md)
- [API Authentication](../ARCHITECTURE.md#4-service-to-service-communication)
