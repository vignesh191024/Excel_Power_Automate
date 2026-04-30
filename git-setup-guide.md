# Git Repository Setup Guide

## Files to Create for Git Repository

This guide lists all the files needed to set up the Git repository properly.

### 1. .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Virtual Environment
venv/
ENV/
env/

# Environment Variables
.env
.env.local
.env.*.local

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Database
*.db
*.sqlite
*.sqlite3

# Credentials
credentials.json
secrets.yaml

# Test Coverage
.coverage
htmlcov/
.pytest_cache/

# Docker
.dockerignore

# Temporary files
*.tmp
*.bak
```

### 2. .env.example

```env
# ServiceNow Configuration
SNOW_INSTANCE_URL=https://yourinstance.service-now.com
SNOW_USERNAME=your_username
SNOW_PASSWORD=your_password
SNOW_ASSIGNMENT_GROUP=Windows Support

# Email Configuration
EMAIL_SERVER=imap.company.com
EMAIL_USERNAME=snow-alerts@company.com
EMAIL_PASSWORD=your_password
EMAIL_FOLDER=INBOX

# Webhook Configuration
WEBHOOK_SECRET=your_webhook_secret
WEBHOOK_PORT=8443

# AI/LLM Configuration
OPENAI_API_KEY=sk-your-key-here
LLM_MODEL=gpt-4-turbo

# Windows Credentials
WIN_USERNAME=domain\serviceaccount
WIN_PASSWORD=your_password

# Database Configuration
DATABASE_URL=postgresql://user:password@localhost:5432/remediation
REDIS_URL=redis://localhost:6379

# Monitoring
PROMETHEUS_PORT=9090
LOG_LEVEL=INFO

# Remediation Settings
MAX_RESTART_ATTEMPTS=3
RESTART_TIMEOUT_SECONDS=60
HEALTH_CHECK_RETRIES=3
```

### 3. requirements.txt

```txt
# Web Framework
fastapi==0.109.0
uvicorn[standard]==0.27.0
pydantic==2.5.3
pydantic-settings==2.1.0

# AI/LLM
langchain==0.1.0
langchain-openai==0.0.2
langgraph==0.0.20
openai==1.10.0

# Windows Integration
pywinrm==0.4.3
pypsrp==0.8.1
requests-ntlm==1.2.0

# ServiceNow Integration
pysnow==0.7.17
requests==2.31.0

# Email
imapclient==2.3.1
email-validator==2.1.0

# Database
sqlalchemy==2.0.25
alembic==1.13.1
psycopg2-binary==2.9.9
redis==5.0.1

# Security
cryptography==42.0.0
python-jose[cryptography]==3.3.0
azure-keyvault-secrets==4.7.0
azure-identity==1.15.0

# Monitoring
prometheus-client==0.19.0
python-json-logger==2.0.7

# Notifications
slack-sdk==3.26.2
pymsteams==0.2.2

# Utilities
python-dotenv==1.0.0
pyyaml==6.0.1
tenacity==8.2.3
schedule==1.2.0
```

### 4. requirements-dev.txt

```txt
# Testing
pytest==7.4.4
pytest-asyncio==0.23.3
pytest-cov==4.1.0
pytest-mock==3.12.0

# Code Quality
black==24.1.1
flake8==7.0.0
mypy==1.8.0
pylint==3.0.3

# Documentation
mkdocs==1.5.3
mkdocs-material==9.5.3
```

### 5. LICENSE

```
MIT License

Copyright (c) 2026 [Your Organization]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Git Commands to Initialize and Push

### Step 1: Initialize Git Repository

```bash
# Navigate to project directory
cd C:/Users/VigneshB/Desktop

# Initialize git repository
git init

# Add all files
git add .

# Create initial commit
git commit -m "Initial commit: Windows Service Remediation AI Agent planning documents"
```

### Step 2: Create GitHub Repository

Option A: Using GitHub CLI
```bash
# Install GitHub CLI if not already installed
# https://cli.github.com/

# Login to GitHub
gh auth login

# Create new repository
gh repo create windows-service-remediation --public --description "AI-powered Windows service monitoring and remediation system"

# Push to GitHub
git push -u origin main
```

Option B: Using GitHub Web Interface
1. Go to https://github.com/new
2. Repository name: `windows-service-remediation`
3. Description: `AI-powered Windows service monitoring and remediation system`
4. Choose Public or Private
5. Do NOT initialize with README (we already have one)
6. Click "Create repository"

Then run:
```bash
# Add remote origin (replace YOUR_USERNAME with your GitHub username)
git remote add origin https://github.com/YOUR_USERNAME/windows-service-remediation.git

# Rename branch to main if needed
git branch -M main

# Push to GitHub
git push -u origin main
```

### Step 3: Verify Repository

```bash
# Check remote
git remote -v

# Check status
git status

# View commit history
git log --oneline
```

## Repository Structure After Push

```
windows-service-remediation/
├── README.md                              ✅ Created
├── windows-service-remediation-plan.md    ✅ Created
├── project-structure.md                   ✅ Created
├── implementation-guide.md                ✅ Created
├── git-setup-guide.md                     ✅ Created
├── .gitignore                             ⏳ To be created
├── .env.example                           ⏳ To be created
├── requirements.txt                       ⏳ To be created
├── requirements-dev.txt                   ⏳ To be created
└── LICENSE                                ⏳ To be created
```

## Next Steps After Repository Setup

1. **Clone the repository** on your development machine
2. **Create virtual environment** and install dependencies
3. **Copy `.env.example` to `.env`** and configure credentials
4. **Switch to Code mode** to implement the actual application code
5. **Create feature branches** for each component implementation
6. **Set up CI/CD pipeline** using GitHub Actions

## Branch Strategy

Recommended branching strategy:

```
main (production-ready code)
├── develop (integration branch)
│   ├── feature/servicenow-integration
│   ├── feature/ai-ticket-parser
│   ├── feature/service-manager
│   ├── feature/webhook-receiver
│   └── feature/monitoring-dashboard
```

## Commit Message Convention

Use conventional commits:
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `refactor:` Code refactoring
- `test:` Adding tests
- `chore:` Maintenance tasks

Example:
```bash
git commit -m "feat: add ServiceNow API integration module"
git commit -m "docs: update architecture diagram with webhook flow"
git commit -m "fix: resolve WinRM connection timeout issue"
```

## GitHub Repository Settings

After creating the repository, configure:

1. **Branch Protection Rules** (Settings → Branches)
   - Require pull request reviews
   - Require status checks to pass
   - Require branches to be up to date

2. **Secrets** (Settings → Secrets and variables → Actions)
   - Add `OPENAI_API_KEY`
   - Add `SNOW_API_TOKEN`
   - Add deployment credentials

3. **Topics** (About section)
   - `ai-agent`
   - `windows-automation`
   - `servicenow`
   - `langchain`
   - `devops`

4. **Description**
   - "AI-powered Windows service monitoring and remediation system that automatically detects and fixes service failures through ServiceNow integration"

## Collaboration Guidelines

1. **Fork the repository** for external contributors
2. **Create feature branches** from `develop`
3. **Submit pull requests** with clear descriptions
4. **Request code reviews** before merging
5. **Update documentation** with code changes
6. **Add tests** for new features

---

**Ready to push to Git?** Switch to Code mode to create the remaining files and execute the Git commands.