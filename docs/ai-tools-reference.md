# AI Tools Reference

Brainhair includes a comprehensive set of AI tools that enable the Claude AI assistant to interact with HiveMatrix services. This guide documents all available tools.

## Overview

AI tools are Python scripts located in `hivematrix-brainhair/ai_tools/` that:
- Accept command-line arguments for parameters
- Use service-to-service authentication
- Request user approval for write operations
- Apply PHI/CJIS filtering to protect sensitive data
- Return structured output for the AI to parse

## Tool Categories

### Company Management

#### list_companies.py
**Purpose**: List all companies from Codex

**Usage**:
```bash
python list_companies.py
python list_companies.py --filter phi  # PHI filtering
python list_companies.py --filter cjis # CJIS filtering
```

**Returns**: List of companies with filtered contact information

#### set_company_plan.py
**Purpose**: Assign or change a company's billing plan

**Usage**:
```bash
python set_company_plan.py <company_name_or_account> <plan_name>
python set_company_plan.py "ACME Corp" "Managed Plus"
python set_company_plan.py 276292 "Standard Support"
```

**Approval Required**: ✅ Yes - modifies billing configuration

### Billing Management

#### get_billing.py
**Purpose**: Retrieve billing information for a company

**Usage**:
```bash
python get_billing.py <company_name_or_account>
python get_billing.py "ACME Corp"
python get_billing.py 276292
```

**Returns**: Billing plan, rates, overrides, and line items

#### update_billing.py
**Purpose**: Update per-unit rates and add line items

**Usage**:
```bash
python update_billing.py <company> --per-user 125 --per-server 125
python update_billing.py 276292 --per-workstation 0 --per-user 125
python update_billing.py "ACME" --line-item "Network Management" 200
```

**Approval Required**: ✅ Yes - modifies billing rates

**Arguments**:
- `--per-user <amount>` - Set per-user rate
- `--per-server <amount>` - Set per-server rate
- `--per-workstation <amount>` - Set per-workstation rate
- `--line-item <description> <amount>` - Add line item

#### import_billing_plans.py
**Purpose**: Import billing plans from CSV or JSON

**Usage**:
```bash
python import_billing_plans.py plans.csv
python import_billing_plans.py plans.json
```

**Approval Required**: ✅ Yes - creates/modifies billing plans

### Asset Management

#### list_devices.py
**Purpose**: List devices/assets for a company

**Usage**:
```bash
python list_devices.py <company_name_or_account>
python list_devices.py "ACME Corp"
python list_devices.py 276292 --filter phi
```

**Returns**: Devices with PHI filtering applied to user information

#### manage_network_equipment.py
**Purpose**: Add, update, or remove network equipment

**Usage**:
```bash
# Add equipment
python manage_network_equipment.py add <company> --name "Router-Main" --type router --ip 192.168.1.1

# Update equipment
python manage_network_equipment.py update <equipment_id> --name "New Name"

# Remove equipment
python manage_network_equipment.py remove <equipment_id>
```

**Approval Required**: ✅ Yes - modifies network inventory

### Ticketing

#### list_tickets.py
**Purpose**: List support tickets from Codex

**Usage**:
```bash
python list_tickets.py
python list_tickets.py --company "ACME Corp"
python list_tickets.py --status open
python list_tickets.py --limit 20 --filter phi
```

**Arguments**:
- `--company <name>` - Filter by company
- `--status <status>` - Filter by status (open, closed, pending)
- `--limit <n>` - Limit results (default: 50)
- `--filter <type>` - Apply PHI or CJIS filtering

**Returns**: Ticket list with sensitive data filtered

### Knowledge Management

#### search_knowledge.py
**Purpose**: Search the KnowledgeTree knowledge base

**Usage**:
```bash
python search_knowledge.py "password reset"
python search_knowledge.py "network troubleshooting" --filter phi
```

**Returns**: Relevant knowledge articles with PHI filtering

#### manage_knowledge.py
**Purpose**: Create, update, or delete knowledge articles

**Usage**:
```bash
# Create article
python manage_knowledge.py create --section "IT" --category "Procedures" --title "Password Reset" --content "..."

# Update article
python manage_knowledge.py update <article_id> --content "Updated content"

# Delete article
python manage_knowledge.py delete <article_id>
```

**Approval Required**: ✅ Yes - modifies knowledge base

#### update_knowledge.py
**Purpose**: Bulk update or import knowledge articles

**Usage**:
```bash
python update_knowledge.py import knowledge.json
python update_knowledge.py update <article_id> --file article.md
```

**Approval Required**: ✅ Yes - modifies knowledge base

### Feature Management

#### update_features.py
**Purpose**: Modify feature flags and overrides for companies

**Usage**:
```bash
python update_features.py <company> --enable email_hosting
python update_features.py <company> --disable backup
python update_features.py <company> --set support_hours 24x7
```

**Approval Required**: ✅ Yes - modifies service features

### Utility Tools

#### set_chat_title.py
**Purpose**: Set the title for a Brainhair chat session

**Usage**:
```bash
python set_chat_title.py <session_id> "Billing Update for ACME"
```

**Approval Required**: ❌ No - metadata only

#### test_all_endpoints.py
**Purpose**: Test connectivity to all HiveMatrix services

**Usage**:
```bash
python test_all_endpoints.py
```

**Returns**: Service health status and connectivity tests

### Helper Modules

#### approval_helper.py
**Module**: Provides the `request_approval()` function for tools

**Purpose**: Handles the approval flow for write operations

**Usage** (in tools):
```python
from approval_helper import request_approval

approved = request_approval(
    "Update billing rates for ACME Corp",
    {
        'Company': 'ACME Corp',
        'Per-User Rate': '$125.00',
        'Per-Server Rate': '$125.00'
    }
)
```

#### brainhair_auth.py
**Module**: Authentication helper for service-to-service calls

**Purpose**: Simplifies getting service tokens from Core

**Usage** (in tools):
```python
from brainhair_auth import get_auth

auth = get_auth()
response = auth.get('/api/codex/companies')
companies = response.json()
```

#### brainhair_simple.py
**Module**: Simplified API wrapper for common operations

**Purpose**: High-level API for common Brainhair tasks

## Approval Flow

Tools that modify data follow this approval flow:

1. **Tool requests approval** via `approval_helper.request_approval()`
2. **File created**: `/tmp/brainhair_approval_request_{session_id}_{timestamp}.json`
3. **Browser detects** the approval file via polling
4. **Modal displayed** to user with action details
5. **User decision**: Approve or Deny button clicked
6. **Response written**: `/tmp/brainhair_approval_response_{approval_id}.json`
7. **Tool reads response** and continues or exits

### Approval Timeout

Tools wait up to **120 seconds** (2 minutes) for user approval. If no response is received, the operation is automatically denied.

## PHI/CJIS Filtering

All tools that retrieve data support PHI (Protected Health Information) and CJIS (Criminal Justice Information Systems) filtering using Microsoft Presidio.

### Filtered Data Types

**PHI Entities**:
- Person names → "FirstName L." format
- Email addresses → `<EMAIL_ADDRESS>`
- Phone numbers → `<PHONE_NUMBER>`
- SSN → `<US_SSN>`
- Credit cards → `<CREDIT_CARD>`
- IP addresses → `<IP_ADDRESS>`
- Medical licenses, driver's licenses, passports

**CJIS Entities**:
- All PHI entities
- Additional criminal justice data types

### Using Filters

```bash
# Apply PHI filtering (default)
python list_companies.py --filter phi

# Apply CJIS filtering
python list_tickets.py --filter cjis

# No filtering (requires admin)
python list_companies.py --filter none
```

## Creating New AI Tools

To create a new AI tool:

1. **Create the script** in `hivematrix-brainhair/ai_tools/`
2. **Add shebang**: `#!/path/to/brainhair/pyenv/bin/python`
3. **Add docstring** with usage examples
4. **Import helpers**:
   ```python
   from brainhair_auth import get_auth
   from approval_helper import request_approval  # If write operation
   ```
5. **Implement tool logic**:
   - Parse command-line arguments
   - Request approval for writes
   - Make service calls with `get_auth()`
   - Return structured output (JSON or formatted text)
6. **Make executable**: `chmod +x ai_tools/your_tool.py`
7. **Test**: Run manually to verify

### Tool Template

```python
#!/path/to/brainhair/pyenv/bin/python
"""
Tool Name and Description

Usage:
    python tool_name.py <args>

Examples:
    python tool_name.py example1
    python tool_name.py example2 --option value
"""

import sys
import json
from brainhair_auth import get_auth
from approval_helper import request_approval  # If needed

def main():
    # Parse arguments
    if len(sys.argv) < 2:
        print(__doc__)
        sys.exit(1)

    # For write operations, request approval
    approved = request_approval(
        "Action description",
        {'Detail': 'Value'}
    )

    if not approved:
        print("✗ User denied the operation")
        sys.exit(1)

    # Make service call
    auth = get_auth()
    response = auth.get('/api/endpoint', params={'filter': 'phi'})

    # Return results
    print(json.dumps(response.json(), indent=2))

if __name__ == "__main__":
    main()
```

## Environment Variables

Tools automatically receive these environment variables from Brainhair:

- `BRAINHAIR_SESSION_ID` - Current chat session ID
- `CORE_SERVICE_URL` - Core service URL (default: http://localhost:5000)
- `CODEX_SERVICE_URL` - Codex service URL (default: http://localhost:5010)
- `LEDGER_SERVICE_URL` - Ledger service URL (default: http://localhost:5030)
- `KNOWLEDGETREE_SERVICE_URL` - KnowledgeTree service URL (default: http://localhost:5020)

## Debugging Tools

### View Tool Output

All tool executions are logged to Helm:

```bash
cd hivematrix-helm
source pyenv/bin/activate
python logs_cli.py brainhair --tail 100
```

### Test Tool Manually

```bash
cd hivematrix-brainhair
export BRAINHAIR_SESSION_ID="test_session_123"
./ai_tools/list_companies.py --filter phi
```

### Check Approval Files

```bash
# View pending approvals
ls -la /tmp/brainhair_approval_request_*

# View approval responses
ls -la /tmp/brainhair_approval_response_*
```

## Security Considerations

1. **Service Tokens** - Tools use service-to-service authentication, bypassing user permissions
2. **Approval Required** - All write operations must be approved by the user
3. **PHI Filtering** - Sensitive data is filtered before being shown to AI
4. **Timeout Protection** - Tools automatically fail if approval takes too long
5. **Audit Trail** - All tool executions are logged to Helm database

## See Also

- [Brainhair Architecture](ARCHITECTURE.md#16-brainhair-ai-assistant--approval-flow)
- [PHI/CJIS Filtering](ARCHITECTURE.md#phi-cjis-filtering)
- [Service Communication](ARCHITECTURE.md#4-service-to-service-communication)
