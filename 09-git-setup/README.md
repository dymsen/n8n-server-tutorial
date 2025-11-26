# Chapter 9: Git Repository Setup

**Set up version control for your server configuration and documentation using Git and GitHub. This chapter covers local repository management, remote synchronization, and collaborative workflows for your N8N server infrastructure.**

## 🎯 What You'll Learn

By the end of this chapter, you'll have:
- Local Git repository for server files
- GitHub remote repository setup
- Automated file synchronization
- Version control best practices
- Team collaboration workflows

## 📋 Prerequisites

- Git installed on server
- GitHub account
- SSH keys configured for GitHub
- Basic Git knowledge

## 9.1 Local Git Repository

### Initialize Repository

**Create repository in home directory:**
```bash
cd ~
mkdir n8n-server-repo
cd n8n-server-repo
git init
```

**Configure Git user:**
```bash
git config user.name "Your Name"
git config user.email "your-email@example.com"
```

### Repository Structure

**Create organized directory structure:**
```bash
mkdir -p n8n-workflows server-configs documentation logs scripts
```

**Add documentation:**
```bash
cat > README.md << 'EOF'
# N8N Server Repository

This repository contains configuration, documentation, and automation scripts for the N8N server.

## Structure

- `n8n-workflows/` - N8N workflow exports and documentation
- `server-configs/` - Server configuration backups
- `documentation/` - Setup guides and procedures
- `logs/` - System logs and monitoring data
- `scripts/` - Automation and maintenance scripts

## Quick Start

1. Clone this repository
2. Review documentation in `documentation/`
3. Run setup scripts in `scripts/`
4. Check logs in `logs/` for monitoring data
EOF
```

## 9.2 GitHub Repository Setup

### Create GitHub Repository

**Create repository on GitHub:**
1. Go to GitHub.com
2. Click "New repository"
3. Name: `n8n-server-tutorial` (or your preferred name)
4. Make it private for security
5. Don't initialize with README

### Connect Local Repository

**Add GitHub remote:**
```bash
git remote add origin git@github.com:your-username/n8n-server-tutorial.git
```

**Push initial commit:**
```bash
git add .
git commit -m "Initial server repository setup"
git branch -M main
git push -u origin main
```

## 9.3 Automated File Management

### Create Auto-Backup Script

**Script to automatically add server files:**
```bash
cat > ~/auto-git-backup.sh << 'EOF'
#!/bin/bash
# Automated git backup for server files

REPO_DIR="$HOME/n8n-server-repo"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

cd "$REPO_DIR"

# Backup server configurations (safe files only)
echo "Backing up server configurations..."
sudo cp -r /etc/caddy/Caddyfile server-configs/ 2>/dev/null || true
sudo cp -r /etc/ssh/sshd_config server-configs/ 2>/dev/null || true

# Backup scripts
echo "Backing up scripts..."
cp ~/backup-n8n.sh scripts/ 2>/dev/null || true
cp ~/update-checker.sh scripts/ 2>/dev/null || true

# Backup logs (recent entries only)
echo "Backing up recent logs..."
tail -100 ~/update_check.log > "logs/update_check_recent.log" 2>/dev/null || true

# Create status summary
echo "# Server Status - $TIMESTAMP" > status.md
echo "" >> status.md
echo "## Services" >> status.md
echo "- N8N: $(podman ps | grep n8n | wc -l) running" >> status.md
echo "- Caddy: $(sudo systemctl is-active caddy)" >> status.md
echo "" >> status.md
echo "## Recent Backups" >> status.md
ls -la ~/ | grep backup | head -5 >> status.md

# Git operations
if git diff --quiet && git diff --staged --quiet; then
    echo "No changes to commit"
else
    git add .
    git commit -m "Automated backup: $TIMESTAMP
    
- Server configurations
- Scripts and logs
- Status summary"
    echo "Changes committed to local repository"
    
    # Push to GitHub (optional, uncomment if desired)
    # git push origin main
fi

echo "Git backup completed: $TIMESTAMP"
EOF

chmod +x ~/auto-git-backup.sh
```

### Schedule Automated Commits

**Add to cron for daily commits:**
```bash
(crontab -l ; echo "# Git automated backup - daily at 3 AM") | crontab
(crontab -l ; echo "0 3 * * * $HOME/auto-git-backup.sh") | crontab
```

## 9.4 N8N Workflow Management

### Export Workflows

**Manual workflow export:**
```bash
# From N8N web interface
# 1. Go to Workflows
# 2. Select workflows to export
# 3. Click "Export"
# 4. Save JSON files to n8n-workflows/
```

**Automated export (advanced):**
```bash
# Requires N8N API access
curl -X GET "http://localhost:5678/rest/workflows" \
     -H "X-N8N-API-KEY: your-api-key" \
     > n8n-workflows/all-workflows.json
```

### Workflow Documentation

**Create workflow documentation:**
```bash
cat > n8n-workflows/README.md << 'EOF'
# N8N Workflows

This directory contains exported N8N workflows and their documentation.

## Workflows

### Booking Automation
- **File:** booking-automation.json
- **Purpose:** Automate booking confirmations and reminders
- **Triggers:** New booking webhook
- **Actions:** Email notifications, calendar updates

### Data Synchronization
- **File:** data-sync.json
- **Purpose:** Sync data between systems
- **Schedule:** Every 15 minutes
- **Error Handling:** Retry with exponential backoff

## Export Instructions

1. Open N8N web interface
2. Go to Workflows
3. Select workflow
4. Click "Export" → "Download"
5. Save to this directory
6. Update this README with workflow details
EOF
```

## 9.5 Documentation Management

### Server Documentation

**Create comprehensive documentation:**
```bash
cat > documentation/server-setup.md << 'EOF'
# Server Setup Guide

## Initial Setup
- VPS: [Provider, specs]
- OS: Ubuntu 24.04 LTS
- Domain: n8n.yourdomain.com

## Installed Software
- N8N: Containerized with Podman
- Caddy: Reverse proxy with SSL
- Monitoring: Automated backups and alerts

## Security Measures
- SSH key authentication only
- No root login allowed
- Firewall configured
- Automatic updates enabled

## Backup Strategy
- Daily automated backups
- FTPS secure storage
- 7-day daily retention
- Monthly archives

## Monitoring
- Google Chat notifications
- System update checks
- Storage monitoring
- Error alerting
EOF
```

### Update Procedures

**Document maintenance procedures:**
```bash
cat > documentation/maintenance.md << 'EOF'
# Maintenance Procedures

## Weekly Tasks
- [ ] Review backup logs
- [ ] Check storage usage
- [ ] Verify SSL certificate renewal
- [ ] Update system packages

## Monthly Tasks
- [ ] Review N8N workflows
- [ ] Audit user access
- [ ] Check monitoring alerts
- [ ] Update documentation

## Emergency Procedures
- [ ] Server restart: `sudo systemctl restart n8n caddy`
- [ ] Data recovery: Follow backup restoration guide
- [ ] Security incident: Isolate server, change credentials
EOF
```

## 9.6 Collaboration Workflows

### Team Access Setup

**Add team members to GitHub repository:**
1. Go to repository Settings → Collaborators
2. Add team members with appropriate permissions
3. Use "Read" for documentation access
4. Use "Write" for configuration changes

### Pull Request Workflow

**For configuration changes:**
1. Create feature branch: `git checkout -b feature/new-config`
2. Make changes and test
3. Commit changes: `git commit -m "Add new configuration"`
4. Push branch: `git push origin feature/new-config`
5. Create pull request on GitHub
6. Review and merge

### Documentation Updates

**Keep documentation current:**
```bash
# After any server changes
cd ~/n8n-server-repo
nano documentation/server-setup.md  # Update as needed
git add documentation/
git commit -m "Update documentation for recent changes"
git push origin main
```

## 9.7 Git Best Practices

### Commit Messages

**Use descriptive commit messages:**
```bash
git commit -m "feat: Add automated backup script

- Creates daily backups of N8N data
- Uploads to secure FTP storage
- Includes rotation policy
- Sends notification on completion"
```

### Branching Strategy

**Keep main branch stable:**
- Use `main` for production configurations
- Create feature branches for changes
- Test changes before merging
- Use pull requests for review

### File Organization

**Keep repository clean:**
- Use `.gitignore` for temporary files
- Organize by function (scripts/, configs/, docs/)
- Include README files in each directory
- Document all procedures

## 9.8 Git Repository Checklist

### Local Repository
- [ ] Git initialized and configured
- [ ] Directory structure created
- [ ] Initial documentation added
- [ ] User information configured

### GitHub Integration
- [ ] Repository created on GitHub
- [ ] SSH keys configured for access
- [ ] Remote origin added
- [ ] Initial push successful

### Automation
- [ ] Auto-backup script created
- [ ] Cron job scheduled
- [ ] File synchronization working
- [ ] Git operations tested

### Documentation
- [ ] Repository structure documented
- [ ] Setup procedures documented
- [ ] Maintenance guides created
- [ ] Workflow documentation added

### Collaboration
- [ ] Team access configured (if applicable)
- [ ] Branching strategy defined
- [ ] Pull request process documented
- [ ] Code review procedures established

## 🏁 Chapter Summary

Your server configuration is now under version control! You've implemented:
- Local Git repository for server files
- GitHub remote repository for collaboration
- Automated file synchronization
- Comprehensive documentation
- Team collaboration workflows

**Your repository includes:**
- Server configurations and scripts
- N8N workflow documentation
- Maintenance procedures
- Monitoring and backup logs
- Team collaboration guidelines

**Manage your repository:**
```bash
cd ~/n8n-server-repo

# Check status
git status

# Commit changes
git add .
git commit -m "Update server configuration"

# Sync with GitHub
git push origin main

# View history
git log --oneline
```

## 📚 Additional Resources

- [Git Documentation](https://git-scm.com/doc)
- [GitHub Guides](https://guides.github.com/)
- [Git Workflow Best Practices](https://www.atlassian.com/git/tutorials/comparing-workflows)
- [Repository Organization](https://github.com/18F/github-repo-template)

---

**Next:** [Chapter 10: Maintenance & Updates →](./../10-maintenance/README.md)
