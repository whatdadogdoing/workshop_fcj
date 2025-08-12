# Enterprise CI/CD Pipeline Workshop

Visit the workshop: [GitHub Pages Site](https://yourusername.github.io/workshop-site)

## Quick Setup

1. Create new GitHub repository named `workshop-site`
2. Upload all files from this folder
3. Enable GitHub Pages in Settings → Pages
4. Source: GitHub Actions
5. Push changes to main branch
6. Site builds automatically

## Workshop Structure

```
workshop-site/
├── _config.yml              # Jekyll configuration
├── index.md                 # Main workshop page
├── 1-environment-setup/     # Module 1
├── 4-cicd-pipeline/         # Module 4
├── images/                  # Workshop images
└── .github/workflows/       # Auto-deployment
```

## Local Development

```bash
# Install Jekyll
gem install jekyll bundler

# Serve locally
jekyll serve
```

## Features

- ✅ Professional AWS FCJ-style layout
- ✅ Module-based navigation
- ✅ Automatic GitHub Pages deployment
- ✅ Mobile responsive design
- ✅ Code syntax highlighting