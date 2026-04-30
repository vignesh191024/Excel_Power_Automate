# Windows Service Remediation - AI Agent

An intelligent, multi-channel AI agent system that automatically detects and remediates Windows service failures by monitoring ServiceNow tickets, eliminating manual intervention for common service restart scenarios.

## 🎯 Overview

This system monitors ServiceNow tickets through multiple channels (API, Email, Webhook) and uses AI to automatically:
- Parse ticket descriptions to extract server and service information
- Connect to Windows servers via WinRM
- Check service status and restart if needed
- Update tickets with remediation results
- Log all actions for audit purposes

## 🏗️ Architecture

The system uses a multi-channel approach with AI-powered decision making:

- **Input Channels**: ServiceNow API, Email Monitor, Webhook Receiver
- **AI Core**: LLM-based ticket parsing, Decision engine, LangGraph orchestration
- **Execution**: PowerShell remoting via WinRM
- **Monitoring**: Prometheus metrics, Grafana dashboards

See [`windows-service-remediation-plan.md`](windows-service-remediation-plan.md) for detailed architecture.

## 📋 Documentation

- **[Architecture Plan](windows-service-remediation-plan.md)** - Complete system design and architecture
- **[Project Structure](project-structure.md)** - Technical specifications and code structure
- **[Implementation Guide](implementation-guide.md)** - Step-by-step implementation instructions

## 🚀 Quick Start

### Prerequisites

- Python 3.11+
- ServiceNow instance with API access
- Windows servers with WinRM enabled
- OpenAI API key (or Azure OpenAI)

### Installation

```bash
# Clone repository
git clone <repository-url>
cd windows-service-remediation

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your credentials
```

### Configuration

1. Set up environment variables in `.env`:
```env
SNOW_INSTANCE_URL=https://yourinstance.service-now.com
SNOW_USERNAME=your_username
SNOW_PASSWORD=your_password
OPENAI_API_KEY=sk-your-key-here
WIN_USERNAME=domain\serviceaccount
WIN_PASSWORD=your_password
```

2. Configure services in `config.yaml`:
```yaml
servers:
  - hostname: "prod-app-01.domain.com"
    services:
      - name: "DatadogAgent"
        restart_allowed: true
```

3. Enable WinRM on target servers:
```powershell
Enable-PSRemoting -Force
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force
Restart-Service WinRM
```

### Running the Agent

```bash
# Run the agent
python main.py

# Or with Docker
docker-compose up -d
```

## 📊 Features

### Current Features (MVP)
- ✅ ServiceNow API integration
- ✅ AI-powered ticket parsing (GPT-4)
- ✅ Windows service management via WinRM
- ✅ Automatic service restart
- ✅ Ticket status updates
- ✅ Basic logging

### Planned Features
- 🔄 Email monitoring
- 🔄 Webhook receiver
- 🔄 LangGraph orchestration
- 🔄 Azure Key Vault integration
- 🔄 Prometheus metrics
- 🔄 Grafana dashboards
- 🔄 Slack/Teams notifications
- 🔄 Approval workflows

## 🔒 Security

- Credentials stored in Azure Key Vault or environment variables
- WinRM connections use NTLM/Kerberos authentication
- Service accounts with least privilege
- Comprehensive audit logging
- Signature validation for webhooks

## 📈 Monitoring

The system exposes Prometheus metrics on port 9090:
- `tickets_processed_total` - Total tickets processed
- `remediation_success_total` - Successful remediations
- `remediation_failure_total` - Failed remediations
- `remediation_duration_seconds` - Remediation duration

## 🧪 Testing

```bash
# Run unit tests
pytest tests/unit/

# Run integration tests
pytest tests/integration/

# Run with coverage
pytest --cov=src tests/
```

## 📦 Deployment

### Docker Deployment
```bash
docker-compose up -d
```

### Systemd Service (Linux)
```bash
sudo cp remediation-agent.service /etc/systemd/system/
sudo systemctl enable remediation-agent
sudo systemctl start remediation-agent
```

## 💰 ROI

- **Manual Intervention Time**: 15 min/ticket × 100 tickets/month = 25 hours
- **Engineer Cost**: $50/hour × 25 hours = $1,250/month
- **Automation Cost**: $500/month
- **Monthly Savings**: $750
- **Annual Savings**: $9,000

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

For issues and questions:
- Create an issue in the repository
- Check the [troubleshooting guide](implementation-guide.md#troubleshooting)
- Review the [documentation](windows-service-remediation-plan.md)

## 🗺️ Roadmap

### Phase 1: MVP (Weeks 1-2) ✅
- Core functionality with ServiceNow API
- AI ticket parsing
- Basic service management

### Phase 2: Enhanced Features (Weeks 3-4)
- Email monitoring
- Configuration management
- Database logging

### Phase 3: Production Ready (Weeks 5-6)
- Webhook support
- Monitoring dashboard
- Advanced security features

## 📞 Contact

Project Maintainer: [Your Name]
Email: [your.email@company.com]

---

**Note**: This is an AI-powered automation system. Always test thoroughly in a non-production environment before deploying to production servers.