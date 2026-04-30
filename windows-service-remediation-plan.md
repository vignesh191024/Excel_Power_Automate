# Windows Service Remediation - AI Agent Architecture Plan

## Executive Summary

An intelligent, multi-channel AI agent system that automatically detects and remediates Windows service failures by monitoring ServiceNow tickets, eliminating manual intervention for common service restart scenarios.

## System Architecture

```mermaid
graph TB
    subgraph "Input Channels"
        A[ServiceNow API Poller]
        B[Email Monitor]
        C[Webhook Receiver]
    end
    
    subgraph "AI Agent Core"
        D[Ticket Parser LLM]
        E[Decision Engine]
        F[Orchestrator]
    end
    
    subgraph "Execution Layer"
        G[Service Manager]
        H[PowerShell Remoting]
        I[WinRM Client]
    end
    
    subgraph "Data & Config"
        J[Config Store]
        K[Credential Vault]
        L[Audit Database]
    end
    
    subgraph "Monitoring & Alerts"
        M[Dashboard]
        N[Alert System]
        O[Metrics Collector]
    end
    
    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I --> |Execute on Target| P[Windows Servers]
    J --> F
    K --> I
    F --> L
    F --> N
    O --> M
    L --> M
```

## Core Components

### 1. Input Channels (Multi-Channel Approach)

#### ServiceNow API Integration
- **Purpose**: Primary channel for ticket monitoring
- **Technology**: REST API with OAuth 2.0
- **Polling Interval**: 30-60 seconds
- **Features**:
  - Query for new/updated incidents
  - Filter by assignment group, category, service
  - Extract ticket metadata (number, description, affected CI)
  - Update ticket with remediation status

#### Email Monitor
- **Purpose**: Backup channel for SNOW notifications
- **Technology**: IMAP/Exchange Web Services
- **Features**:
  - Monitor dedicated mailbox for SNOW alerts
  - Parse email body for ticket details
  - Extract server name and service information
  - Handle various email formats

#### Webhook Receiver
- **Purpose**: Real-time event processing
- **Technology**: FastAPI/Flask REST endpoint
- **Features**:
  - Receive POST requests from ServiceNow
  - Validate webhook signatures
  - Queue events for processing
  - Return acknowledgment immediately

### 2. AI Agent Core

#### Ticket Parser (LLM-Powered)
- **Technology**: OpenAI GPT-4 / Azure OpenAI / Local LLM
- **Responsibilities**:
  - Extract structured data from unstructured ticket text
  - Identify: server name, service name, issue type, priority
  - Handle variations in ticket descriptions
  - Classify issue severity and urgency

**Example Prompt Template**:
```
Extract the following information from this ServiceNow ticket:
- Server/Hostname
- Service Name
- Issue Type (stopped, crashed, not responding)
- Priority Level

Ticket: {ticket_description}

Return as JSON.
```

#### Decision Engine
- **Technology**: Rule-based + ML hybrid
- **Responsibilities**:
  - Validate extracted information
  - Check if service is in monitored list
  - Determine remediation action (restart, stop/start, investigate)
  - Apply business rules and safety checks
  - Escalate if outside automation scope

#### Orchestrator
- **Technology**: LangGraph / CrewAI
- **Responsibilities**:
  - Coordinate multi-step workflows
  - Manage state across remediation process
  - Handle retries and error recovery
  - Update ticket status at each step
  - Generate audit logs

### 3. Execution Layer

#### Service Manager Module
- **Technology**: Python with [`pypsrp`](https://github.com/jborean93/pypsrp) or [`winrm`](https://github.com/diyan/pywinrm)
- **Responsibilities**:
  - Connect to remote Windows servers
  - Query service status
  - Execute service commands (start, stop, restart)
  - Verify service state after action
  - Collect service logs

**PowerShell Commands**:
```powershell
# Check service status
Get-Service -Name "DatadogAgent" | Select-Object Status, StartType

# Restart service
Restart-Service -Name "DatadogAgent" -Force

# Verify service is running
Get-Service -Name "DatadogAgent" | Where-Object {$_.Status -eq 'Running'}
```

#### WinRM Configuration
- Enable WinRM on target servers
- Configure TrustedHosts or use HTTPS with certificates
- Use service accounts with minimal required permissions
- Implement connection pooling for efficiency

### 4. Configuration Management

#### Service Configuration File
**Format**: YAML/JSON per server or server group

```yaml
servers:
  - hostname: "prod-app-01.domain.com"
    services:
      - name: "DatadogAgent"
        display_name: "Datadog Agent"
        restart_allowed: true
        max_restart_attempts: 3
        health_check_url: "http://localhost:5000/health"
      
      - name: "splunkforwarder"
        display_name: "Splunk Universal Forwarder"
        restart_allowed: true
        dependencies: ["DatadogAgent"]
    
    credentials_ref: "prod_service_account"
    notification_email: "team@company.com"

  - hostname: "prod-db-01.domain.com"
    services:
      - name: "MSSQLSERVER"
        display_name: "SQL Server"
        restart_allowed: false  # Requires approval
        escalation_required: true
```

### 5. Security & Credentials

#### Credential Vault Integration
- **Options**: 
  - Azure Key Vault
  - HashiCorp Vault
  - CyberArk
  - Windows Credential Manager (for local testing)

- **Stored Credentials**:
  - ServiceNow API tokens
  - Email account credentials
  - Windows service account credentials per server
  - Webhook signing secrets

#### Security Best Practices
- Use service accounts with least privilege
- Rotate credentials regularly
- Encrypt sensitive data at rest and in transit
- Implement audit logging for all credential access
- Use certificate-based authentication where possible

### 6. Logging & Audit Trail

#### Audit Database Schema
```sql
CREATE TABLE remediation_events (
    id INT PRIMARY KEY,
    ticket_number VARCHAR(50),
    server_name VARCHAR(255),
    service_name VARCHAR(255),
    action_taken VARCHAR(50),
    status VARCHAR(50),
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    error_message TEXT,
    executed_by VARCHAR(100)
);
```

#### Log Levels
- **INFO**: Normal operations, successful remediations
- **WARNING**: Retries, non-critical issues
- **ERROR**: Failed remediations, connection issues
- **CRITICAL**: System failures, security events

### 7. Monitoring & Alerting

#### Dashboard Metrics
- Total tickets processed (last 24h, 7d, 30d)
- Success rate by service type
- Average remediation time
- Failed remediation count
- Active agent status
- Server connectivity status

#### Alert Conditions
- Remediation failure after max retries
- Service restart loop detected (>3 restarts in 1 hour)
- Unable to connect to target server
- AI agent service down
- Configuration file errors

#### Notification Channels
- Email alerts to operations team
- Slack/Teams integration
- Update ServiceNow ticket with status
- PagerDuty for critical failures

## Technology Stack

### Recommended Stack

**Backend Framework**: Python 3.11+
- **FastAPI**: Webhook receiver, REST API
- **LangChain/LangGraph**: AI agent orchestration
- **OpenAI API**: Ticket parsing and decision support

**Windows Integration**:
- **pywinrm**: Windows Remote Management
- **pypsrp**: PowerShell Remoting Protocol

**Data Storage**:
- **PostgreSQL**: Audit logs, configuration
- **Redis**: Task queue, caching

**Monitoring**:
- **Prometheus**: Metrics collection
- **Grafana**: Visualization dashboard
- **ELK Stack**: Log aggregation (optional)

**Deployment**:
- **Docker**: Containerization
- **Kubernetes/Docker Compose**: Orchestration
- **GitHub Actions**: CI/CD

## Implementation Workflow

### Ticket Processing Flow

```mermaid
sequenceDiagram
    participant SNOW as ServiceNow
    participant Agent as AI Agent
    participant LLM as LLM Parser
    participant Server as Windows Server
    
    SNOW->>Agent: New ticket via API/Email/Webhook
    Agent->>LLM: Parse ticket description
    LLM->>Agent: Extract: server, service, issue
    Agent->>Agent: Validate against config
    Agent->>Server: Check service status via WinRM
    Server->>Agent: Service status: Stopped
    Agent->>Server: Execute: Restart-Service
    Server->>Agent: Service restarted successfully
    Agent->>Server: Verify service running
    Server->>Agent: Status: Running
    Agent->>SNOW: Update ticket: Resolved
    Agent->>Agent: Log audit event
```

### Remediation Steps

1. **Ticket Received**
   - Capture from API/Email/Webhook
   - Assign unique tracking ID
   - Log receipt timestamp

2. **Parse & Extract**
   - Use LLM to extract structured data
   - Validate required fields present
   - Enrich with configuration data

3. **Pre-Flight Checks**
   - Verify server in managed list
   - Check service in allowed list
   - Validate restart permissions
   - Confirm credentials available

4. **Connect to Server**
   - Establish WinRM session
   - Authenticate with service account
   - Test connectivity

5. **Check Service Status**
   - Query current service state
   - Get service start type
   - Check dependencies

6. **Execute Remediation**
   - Stop service (if running but unhealthy)
   - Start/Restart service
   - Wait for service to stabilize
   - Verify service is running

7. **Post-Remediation Validation**
   - Check service status
   - Test health endpoint (if configured)
   - Verify dependencies started
   - Collect service logs

8. **Update & Notify**
   - Update ServiceNow ticket
   - Send notification to team
   - Log audit event
   - Update metrics

9. **Error Handling**
   - Retry with exponential backoff
   - Escalate if max retries exceeded
   - Create detailed error report
   - Alert on-call engineer

## Configuration Examples

### Environment Variables
```bash
# ServiceNow
SNOW_INSTANCE_URL=https://yourinstance.service-now.com
SNOW_API_USERNAME=api_user
SNOW_API_PASSWORD=stored_in_vault

# Email
EMAIL_SERVER=imap.company.com
EMAIL_USERNAME=snow-alerts@company.com
EMAIL_PASSWORD=stored_in_vault

# Webhook
WEBHOOK_SECRET=stored_in_vault
WEBHOOK_PORT=8443

# AI/LLM
OPENAI_API_KEY=stored_in_vault
LLM_MODEL=gpt-4-turbo

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/remediation

# Monitoring
PROMETHEUS_PORT=9090
GRAFANA_PORT=3000
```

### Service Configuration Example
```yaml
# config/services.yaml
global_settings:
  max_restart_attempts: 3
  restart_timeout_seconds: 60
  health_check_retries: 3
  notification_email: "ops-team@company.com"

server_groups:
  production_monitoring:
    servers:
      - "prod-app-01.domain.com"
      - "prod-app-02.domain.com"
      - "prod-web-01.domain.com"
    
    services:
      - name: "DatadogAgent"
        display_name: "Datadog Agent"
        restart_allowed: true
        priority: "high"
        
      - name: "splunkforwarder"
        display_name: "Splunk Forwarder"
        restart_allowed: true
        priority: "medium"

  production_databases:
    servers:
      - "prod-db-01.domain.com"
    
    services:
      - name: "MSSQLSERVER"
        restart_allowed: false
        escalation_required: true
        approval_needed: true
```

## Safety Mechanisms

### Safeguards
1. **Rate Limiting**: Max 5 restarts per service per hour
2. **Restart Loop Detection**: Alert if service restarts >3 times in 1 hour
3. **Approval Required**: Critical services require human approval
4. **Dry Run Mode**: Test mode that simulates actions without executing
5. **Rollback Capability**: Restore previous service state on failure
6. **Circuit Breaker**: Pause automation if error rate exceeds threshold

### Business Rules
- Never restart database services without approval
- Escalate if service fails to start after 3 attempts
- Notify team lead for production server actions
- Require change ticket for non-standard services
- Implement maintenance windows (no auto-restart during backups)

## Deployment Strategy

### Phase 1: Development & Testing (Weeks 1-2)
- Set up development environment
- Implement core components
- Create unit tests
- Test with mock ServiceNow data

### Phase 2: Pilot (Weeks 3-4)
- Deploy to test environment
- Monitor 5-10 non-critical servers
- Validate ticket parsing accuracy
- Tune LLM prompts
- Gather feedback

### Phase 3: Production Rollout (Weeks 5-6)
- Deploy to production environment
- Start with monitoring agents only
- Gradually expand service coverage
- Monitor success rates
- Adjust configurations

### Phase 4: Optimization (Ongoing)
- Analyze patterns and improve AI models
- Add predictive capabilities
- Expand to additional services
- Implement advanced features

## Success Metrics

### Key Performance Indicators (KPIs)
- **Mean Time to Remediation (MTTR)**: Target <5 minutes
- **Automation Success Rate**: Target >95%
- **Manual Intervention Reduction**: Target >80%
- **False Positive Rate**: Target <5%
- **Ticket Auto-Resolution Rate**: Target >90%

### Monitoring Dashboards
1. **Operational Dashboard**
   - Active tickets being processed
   - Success/failure rates
   - Average remediation time
   - Server connectivity status

2. **Trend Analysis**
   - Most frequently failing services
   - Peak incident times
   - Server reliability scores
   - Cost savings from automation

## Risk Mitigation

### Potential Risks
1. **Service Restart Loops**: Implement cooldown periods
2. **Incorrect Ticket Parsing**: Use confidence scores, escalate low confidence
3. **Credential Compromise**: Rotate regularly, use vault, audit access
4. **Network Connectivity Issues**: Implement retry logic, fallback mechanisms
5. **Unintended Service Disruption**: Dry run mode, approval workflows

## Cost Estimation

### Infrastructure Costs (Monthly)
- **Cloud VM** (4 vCPU, 16GB RAM): $150-200
- **Database** (PostgreSQL managed): $50-100
- **OpenAI API** (GPT-4): $100-300 (based on volume)
- **Monitoring Tools**: $50-100
- **Total**: ~$350-700/month

### ROI Calculation
- **Manual Intervention Time**: 15 min/ticket × 100 tickets/month = 25 hours
- **Engineer Cost**: $50/hour × 25 hours = $1,250/month
- **Automation Cost**: $500/month
- **Monthly Savings**: $750
- **Annual Savings**: $9,000

## Next Steps

1. **Review and approve this plan**
2. **Set up development environment**
3. **Obtain ServiceNow API credentials**
4. **Configure test servers with WinRM**
5. **Begin implementation in Code mode**

---

## Questions for Consideration

Before implementation, please confirm:

1. **ServiceNow Access**: Do you have API credentials and permissions?
2. **Server Access**: Can we set up a service account with WinRM access?
3. **Credential Storage**: Do you have Azure Key Vault or similar?
4. **LLM Preference**: OpenAI API or self-hosted model?
5. **Deployment Target**: On-premises VM, Azure, AWS, or other?
6. **Initial Scope**: Start with Datadog only or multiple services?
7. **Approval Process**: Who approves critical service restarts?