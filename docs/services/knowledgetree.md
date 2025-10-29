# KnowledgeTree - Knowledge Base System

**Port**: 5020
**Database**: Neo4j (graph database)
**Repository**: [hivematrix-knowledgetree](https://github.com/ruapotato/hivematrix-knowledgetree)

## Overview

KnowledgeTree is HiveMatrix's knowledge base system that organizes company knowledge in a hierarchical, filesystem-like structure. It's designed to store and manage all institutional knowledge - documentation, procedures, troubleshooting guides, and best practices.

## Purpose

KnowledgeTree serves as the centralized repository for:
- Technical documentation
- Standard operating procedures (SOPs)
- Troubleshooting guides
- Best practices and policies
- Client-specific documentation
- Internal company knowledge

Think of it as a Wiki combined with a file system, where knowledge is organized hierarchically and can be easily searched and referenced.

## Hierarchical Structure

KnowledgeTree uses a three-level hierarchy similar to a filesystem:

```
Sections (folders)
  └── Categories (subfolders)
        └── Topics (files/articles)
```

### Example Structure

```
IT Operations (Section)
  ├── Network (Category)
  │     ├── Router Configuration (Topic)
  │     ├── VLAN Setup (Topic)
  │     └── Firewall Rules (Topic)
  │
  ├── Server Management (Category)
  │     ├── Windows Updates (Topic)
  │     ├── Backup Procedures (Topic)
  │     └── Active Directory (Topic)
  │
  └── End User Support (Category)
        ├── Password Resets (Topic)
        ├── Email Configuration (Topic)
        └── VPN Setup (Topic)
```

## Key Features

### Hierarchical Organization
- **Sections**: Top-level organization (like IT, HR, Finance)
- **Categories**: Mid-level grouping (like Network, Server, Desktop)
- **Topics**: Individual knowledge articles with content

### Markdown Support
- Rich text formatting
- Code blocks with syntax highlighting
- Tables, lists, and links
- Embedded images

### Full-Text Search
- Search across all topics
- Find knowledge by keyword
- Filter by section or category

### Company Scoping
- Associate knowledge with specific companies
- General knowledge available to all
- Client-specific procedures

### Graph Relationships
Neo4j enables tracking relationships between knowledge items:
- Related topics
- Prerequisites
- See also links
- Knowledge dependencies

## Why Neo4j?

KnowledgeTree uses Neo4j graph database because:

1. **Hierarchical Data**: Natural fit for Section → Category → Topic structure
2. **Relationships**: Easy to link related knowledge articles
3. **Fast Traversal**: Quick navigation through knowledge tree
4. **Flexible Schema**: Easy to add new relationship types
5. **Graph Queries**: Find connections between topics

## Data Model

### Node Types

**Section**
```
{
  id: UUID,
  name: "IT Operations",
  description: "Technical documentation",
  created_at: timestamp
}
```

**Category**
```
{
  id: UUID,
  name: "Network",
  description: "Network configuration and management",
  section_id: Section UUID,
  created_at: timestamp
}
```

**Topic**
```
{
  id: UUID,
  title: "Router Configuration",
  content: "Markdown content...",
  category_id: Category UUID,
  company_id: Company ID (optional),
  created_at: timestamp,
  updated_at: timestamp,
  author: User email
}
```

### Relationships

```
(Section)-[:HAS_CATEGORY]->(Category)
(Category)-[:HAS_TOPIC]->(Topic)
(Topic)-[:RELATED_TO]->(Topic)
(Topic)-[:PREREQUISITE]->(Topic)
(Topic)-[:APPLIES_TO]->(Company)
```

## API Endpoints

### Sections

**List all sections**
```bash
GET /api/sections
```

**Create section**
```bash
POST /api/sections
{
  "name": "IT Operations",
  "description": "Technical documentation"
}
```

### Categories

**List categories in a section**
```bash
GET /api/sections/{section_id}/categories
```

**Create category**
```bash
POST /api/categories
{
  "section_id": "...",
  "name": "Network",
  "description": "Network docs"
}
```

### Topics

**List topics in a category**
```bash
GET /api/categories/{category_id}/topics
```

**Get topic by ID**
```bash
GET /api/topics/{topic_id}
```

**Create topic**
```bash
POST /api/topics
{
  "category_id": "...",
  "title": "Router Configuration",
  "content": "# How to configure...",
  "company_id": null  // Optional
}
```

**Update topic**
```bash
PUT /api/topics/{topic_id}
{
  "content": "Updated markdown content..."
}
```

**Search topics**
```bash
GET /api/search?q=password+reset
```

## Common Use Cases

### Standard Operating Procedures
Store SOPs that technicians follow for common tasks:
- Password reset procedures
- New user onboarding
- System maintenance checklists

### Troubleshooting Guides
Document solutions to common problems:
- "Printer not responding"
- "Email synchronization issues"
- "VPN connection failures"

### Client-Specific Documentation
Track unique client configurations:
- Special network setups
- Custom software configurations
- Client-specific access procedures

### Internal Knowledge
Document internal processes:
- Billing procedures
- Escalation paths
- Tool usage guides

## Integration with Other Services

### Codex Integration
- Links topics to specific companies
- Queries company information for context
- Associates knowledge with client accounts

### Brainhair Integration
- AI assistant searches KnowledgeTree
- Suggests relevant articles based on questions
- Can create new topics through approval flow

### Nexus Integration
- Provides navigation to knowledge base
- Renders Markdown content in UI

## Configuration

KnowledgeTree configuration in `instance/knowledgetree.conf`:

```ini
[database]
neo4j_uri = bolt://localhost:7687
neo4j_user = neo4j
neo4j_password = your-password

[services]
codex_url = http://localhost:5010
core_url = http://localhost:5000
```

## Installation

### Prerequisites

Install Neo4j:
```bash
cd hivematrix-helm
sudo bash install_neo4j.sh
```

### Install KnowledgeTree

```bash
cd hivematrix-helm
sudo ./install_manager.py install knowledgetree
./start.sh
```

## Common Operations

### Create Knowledge Structure

1. **Create Section**
   - Navigate to KnowledgeTree
   - Click "New Section"
   - Enter name and description

2. **Add Categories**
   - Open section
   - Click "New Category"
   - Group related topics

3. **Write Topics**
   - Open category
   - Click "New Topic"
   - Write content in Markdown
   - Save

### Search Knowledge

```bash
# Via UI: Use search box in KnowledgeTree

# Via API:
curl -H "Authorization: Bearer $TOKEN" \
  "http://localhost:5020/api/search?q=password"
```

### Link Related Topics

```bash
POST /api/topics/{topic_id}/relate
{
  "related_topic_id": "...",
  "relationship_type": "related_to"
}
```

## Best Practices

### Organizing Knowledge

1. **Use Clear Hierarchies**: Create logical section/category structure
2. **Consistent Naming**: Use verb phrases for procedures ("How to..." or "Troubleshooting...")
3. **Company Scoping**: Mark client-specific topics appropriately
4. **Cross-Linking**: Link related topics for easy navigation

### Writing Topics

1. **Clear Titles**: Descriptive and searchable
2. **Step-by-Step**: Number procedural steps
3. **Examples**: Include command examples and screenshots
4. **Keep Updated**: Review and update regularly

### Maintenance

1. **Regular Review**: Audit topics quarterly
2. **Remove Outdated**: Delete obsolete procedures
3. **Consolidate Duplicates**: Merge similar topics
4. **Improve Search**: Add keywords to content

## Troubleshooting

### Neo4j Connection Issues

```bash
# Check Neo4j status
sudo systemctl status neo4j

# Test connection
cypher-shell -u neo4j -p your-password -d system

# Restart Neo4j
sudo systemctl restart neo4j
```

### Slow Search

Neo4j indexes can be created for faster searches:

```cypher
// Create full-text index on topics
CREATE FULLTEXT INDEX topic_search FOR (t:Topic) ON EACH [t.title, t.content]
```

### Missing Data

Verify database connection in config:
```bash
cd hivematrix-knowledgetree
cat instance/knowledgetree.conf
```

## Backup & Restore

### Backup Neo4j

```bash
sudo systemctl stop neo4j
sudo neo4j-admin database dump --database=neo4j --to=/backup/neo4j.dump
sudo systemctl start neo4j
```

### Restore Neo4j

```bash
sudo systemctl stop neo4j
sudo neo4j-admin database load --from=/backup/neo4j.dump --database=neo4j --overwrite-destination
sudo systemctl start neo4j
```

## Security

- All endpoints require JWT authentication
- Permission checks for write operations
- Company-scoped topics only visible to associated users
- Markdown sanitization to prevent XSS

## Performance

KnowledgeTree is optimized for read-heavy workloads:
- Neo4j graph queries are very fast
- Hierarchical navigation is efficient
- Full-text search uses indexes
- Caching for frequently accessed topics

## See Also

- [Services Overview](../services-overview.md)
- [Neo4j Documentation](https://neo4j.com/docs/)
- [Markdown Guide](https://www.markdownguide.org/)
- [Configuration Guide](../configuration.md)
