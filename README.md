# HiveMatrix Documentation

Official documentation for the HiveMatrix MSP platform, built with MkDocs Material.

## 📖 View Documentation

Documentation is hosted at: **https://ruapotato.github.io/hivematrix-docs/**

## 🚀 Local Development

### Prerequisites

- Python 3.x
- pip

### Setup

```bash
# Clone the repository
git clone https://github.com/ruapotato/hivematrix-docs.git
cd hivematrix-docs

# Install MkDocs and Material theme
pip install mkdocs-material

# Serve locally
mkdocs serve
```

Visit http://127.0.0.1:8000 to view the documentation locally.

### Building

```bash
# Build static site
mkdocs build

# Deploy to GitHub Pages (done automatically via GitHub Actions)
mkdocs gh-deploy
```

## 📁 Structure

```
hivematrix-docs/
├── docs/                 # Documentation source files
│   ├── index.md         # Homepage
│   ├── ARCHITECTURE.md  # Technical architecture
│   └── CLAUDE.md        # AI development guide
├── mkdocs.yml           # MkDocs configuration
└── .github/workflows/   # GitHub Actions for auto-deployment
```

## 🔄 Automatic Deployment

Documentation is automatically built and deployed to GitHub Pages when changes are pushed to the `main` branch.

## 📝 Contributing

1. Make changes to files in the `docs/` directory
2. Test locally with `mkdocs serve`
3. Commit and push to `main` branch
4. GitHub Actions will automatically deploy

## 🎨 Theme

This documentation uses [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/), a beautiful and feature-rich theme.

## 📄 License

MIT License - See [LICENSE](LICENSE) file for details
