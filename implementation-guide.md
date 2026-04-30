# Implementation Guide

## Quick Start Implementation Path

This guide provides a phased approach to implementing the Windows Service Remediation system, starting with a minimal viable product (MVP) and progressively adding features.

## Phase 1: MVP - Core Functionality (Week 1-2)

### Goal
Create a basic working system that can:
- Monitor ServiceNow tickets via API
- Parse ticket information using AI
- Restart services on Windows servers
- Update tickets with results

### Implementation Steps

#### Step 1: Environment Setup

1. **Create Project Directory**
```bash
mkdir windows-service-remediation
cd windows-service-remediation
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

2. **Install Core Dependencies**
```bash
pip install fastapi uvicorn langchain langchain-openai pywinrm pyyaml python-dotenv pydantic-settings
```

3. **Create `.env` File**
```env
# ServiceNow
SNOW_INSTANCE_URL=https://yourinstance.service-now.com
SNOW_USERNAME=your_username
SNOW_PASSWORD=your_password
SNOW_ASSIGNMENT_GROUP=Windows Support

# OpenAI
OPENAI_API_KEY=sk-your-key-here

# Windows Credentials
WIN_USERNAME=domain\serviceaccount
WIN_PASSWORD=your_password

# Settings
LOG_LEVEL=INFO
MAX_RESTART_ATTEMPTS=3
```

#### Step 2: Create Minimal Service Manager

**File**: `service_manager.py`

```python
import winrm
from typing import Dict
import logging

logger = logging.getLogger(__name__)

class ServiceManager:
    def __init__(self, hostname: str, username: str, password: str):
        self.hostname = hostname
        self.session = winrm.Session(
            f'http://{hostname}:5985/wsman',
            auth=(username, password),
            transport='ntlm'
        )
    
    def get_service_status(self, service_name: str) -> Dict:
        """Get service status"""
        ps_script = f"""
        $service = Get-Service -Name '{service_name}' -ErrorAction SilentlyContinue
        if ($service) {{
            @{{
                Status = $service.Status.ToString()
                StartType = $service.StartType.ToString()
            }} | ConvertTo-Json
        }} else {{
            @{{Error = "Service not found"}} | ConvertTo-Json
        }}
        """
        
        result = self.session.run_ps(ps_script)
        if result.status_code == 0:
            import json
            return json.loads(result.std_out)
        return {"Error": result.std_err}
    
    def restart_service(self, service_name: str) -> Dict:
        """Restart service"""
        ps_script = f"""
        try {{
            Restart-Service -Name '{service_name}' -Force -ErrorAction Stop
            Start-Sleep -Seconds 5
            $service = Get-Service -Name '{service_name}'
            @{{
                Success = $true
                Status = $service.Status.ToString()
            }} | ConvertTo-Json
        }} catch {{
            @{{
                Success = $false
                Error = $_.Exception.Message
            }} | ConvertTo-Json
        }}
        """
        
        result = self.session.run_ps(ps_script)
        if result.status_code == 0:
            import json
            return json.loads(result.std_out)
        return {"Success": False, "Error": result.std_err}
```

#### Step 3: Create AI Ticket Parser

**File**: `ticket_parser.py`

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from pydantic import BaseModel, Field
import os

class TicketInfo(BaseModel):
    server_name: str = Field(description="Server hostname")
    service_name: str = Field(description="Windows service name")
    issue_type: str = Field(description="Issue type")
    confidence: float = Field(description="Confidence score 0-1")

class TicketParser:
    def __init__(self):
        self.llm = ChatOpenAI(
            api_key=os.getenv("OPENAI_API_KEY"),
            model="gpt-4-turbo",
            temperature=0
        )
        
        self.prompt = ChatPromptTemplate.from_messages([
            ("system", """Extract server name, service name, and issue type from the ticket.
            
            Common services:
            - Datadog Agent, DatadogAgent -> DatadogAgent
            - Splunk Forwarder -> splunkforwarder
            - SQL Server -> MSSQLSERVER
            
            Return confidence based on clarity."""),
            ("user", "Ticket: {ticket_number}\n\nDescription: {description}")
        ])
    
    def parse(self, ticket_number: str, description: str) -> TicketInfo:
        """Parse ticket and extract information"""
        chain = self.prompt | self.llm.with_structured_output(TicketInfo)
        return chain.invoke({
            "ticket_number": ticket_number,
            "description": description
        })
```

#### Step 4: Create Simple ServiceNow Client

**File**: `snow_client.py`

```python
import requests
from typing import List, Dict
import os

class ServiceNowClient:
    def __init__(self):
        self.instance_url = os.getenv("SNOW_INSTANCE_URL")
        self.username = os.getenv("SNOW_USERNAME")
        self.password = os.getenv("SNOW_PASSWORD")
        self.assignment_group = os.getenv("SNOW_ASSIGNMENT_GROUP")
        
        self.base_url = f"{self.instance_url}/api/now/table/incident"
        self.auth = (self.username, self.password)
        self.headers = {
            "Content-Type": "application/json",
            "Accept": "application/json"
        }
    
    def get_new_tickets(self) -> List[Dict]:
        """Get new tickets"""
        params = {
            "sysparm_query": f"assignment_group={self.assignment_group}^state=1",
            "sysparm_limit": 10
        }
        
        response = requests.get(
            self.base_url,
            auth=self.auth,
            headers=self.headers,
            params=params
        )
        
        if response.status_code == 200:
            return response.json().get("result", [])
        return []
    
    def update_ticket(self, sys_id: str, work_notes: str, state: int = None):
        """Update ticket"""
        url = f"{self.base_url}/{sys_id}"
        data = {"work_notes": work_notes}
        if state:
            data["state"] = state
        
        response = requests.patch(
            url,
            auth=self.auth,
            headers=self.headers,
            json=data
        )
        
        return response.status_code == 200
```

#### Step 5: Create Main Application

**File**: `main.py`

```python
import os
import time
import logging
from dotenv import load_dotenv
from ticket_parser import TicketParser
from service_manager import ServiceManager
from snow_client import ServiceNowClient

# Load environment variables
load_dotenv()

# Setup logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

class RemediationAgent:
    def __init__(self):
        self.snow_client = ServiceNowClient()
        self.ticket_parser = TicketParser()
        self.win_username = os.getenv("WIN_USERNAME")
        self.win_password = os.getenv("WIN_PASSWORD")
        self.max_attempts = int(os.getenv("MAX_RESTART_ATTEMPTS", 3))
    
    def process_ticket(self, ticket: dict):
        """Process a single ticket"""
        ticket_number = ticket.get("number")
        description = ticket.get("short_description", "") + " " + ticket.get("description", "")
        sys_id = ticket.get("sys_id")
        
        logger.info(f"Processing ticket {ticket_number}")
        
        try:
            # Parse ticket
            ticket_info = self.ticket_parser.parse(ticket_number, description)
            logger.info(f"Parsed: {ticket_info}")
            
            # Check confidence
            if ticket_info.confidence < 0.7:
                self.snow_client.update_ticket(
                    sys_id,
                    f"AI Agent: Low confidence ({ticket_info.confidence:.2f}) in parsing. Manual review required."
                )
                return
            
            # Connect to server
            service_mgr = ServiceManager(
                ticket_info.server_name,
                self.win_username,
                self.win_password
            )
            
            # Check service status
            status = service_mgr.get_service_status(ticket_info.service_name)
            logger.info(f"Service status: {status}")
            
            if "Error" in status:
                self.snow_client.update_ticket(
                    sys_id,
                    f"AI Agent: {status['Error']}"
                )
                return
            
            # Restart if not running
            if status.get("Status") != "Running":
                logger.info(f"Restarting service {ticket_info.service_name}")
                result = service_mgr.restart_service(ticket_info.service_name)
                
                if result.get("Success"):
                    self.snow_client.update_ticket(
                        sys_id,
                        f"AI Agent: Successfully restarted {ticket_info.service_name}. Status: {result.get('Status')}",
                        state=6  # Resolved
                    )
                    logger.info(f"Successfully remediated {ticket_number}")
                else:
                    self.snow_client.update_ticket(
                        sys_id,
                        f"AI Agent: Failed to restart service. Error: {result.get('Error')}"
                    )
            else:
                self.snow_client.update_ticket(
                    sys_id,
                    f"AI Agent: Service {ticket_info.service_name} is already running. No action needed."
                )
        
        except Exception as e:
            logger.error(f"Error processing ticket {ticket_number}: {e}")
            self.snow_client.update_ticket(
                sys_id,
                f"AI Agent: Error during remediation: {str(e)}"
            )
    
    def run(self):
        """Main loop"""
        logger.info("Starting Remediation Agent")
        
        while True:
            try:
                # Get new tickets
                tickets = self.snow_client.get_new_tickets()
                logger.info(f"Found {len(tickets)} new tickets")
                
                # Process each ticket
                for ticket in tickets:
                    self.process_ticket(ticket)
                
                # Wait before next poll
                time.sleep(60)  # Poll every minute
            
            except KeyboardInterrupt:
                logger.info("Shutting down...")
                break
            except Exception as e:
                logger.error(f"Error in main loop: {e}")
                time.sleep(60)

if __name__ == "__main__":
    agent = RemediationAgent()
    agent.run()
```

#### Step 6: Test the MVP

1. **Setup WinRM on Target Server**
```powershell
# Run on Windows server as Administrator
Enable-PSRemoting -Force
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force
Restart-Service WinRM
```

2. **Test Connection**
```python
# test_connection.py
from service_manager import ServiceManager
import os
from dotenv import load_dotenv

load_dotenv()

mgr = ServiceManager(
    "your-server.domain.com",
    os.getenv("WIN_USERNAME"),
    os.getenv("WIN_PASSWORD")
)

status = mgr.get_service_status("DatadogAgent")
print(status)
```

3. **Run the Agent**
```bash
python main.py
```

## Phase 2: Enhanced Features (Week 3-4)

### Add Configuration Management

**File**: `config.yaml`

```yaml
servers:
  - hostname: "prod-app-01.domain.com"
    services:
      - name: "DatadogAgent"
        restart_allowed: true
        max_attempts: 3
      - name: "splunkforwarder"
        restart_allowed: true
        max_attempts: 3

  - hostname: "prod-db-01.domain.com"
    services:
      - name: "MSSQLSERVER"
        restart_allowed: false
        escalation_required: true
```

**File**: `config_manager.py`

```python
import yaml
from typing import Dict, Optional

class ConfigManager:
    def __init__(self, config_file: str = "config.yaml"):
        with open(config_file, 'r') as f:
            self.config = yaml.safe_load(f)
    
    def get_server_config(self, hostname: str) -> Optional[Dict]:
        """Get server configuration"""
        for server in self.config.get("servers", []):
            if server["hostname"] == hostname:
                return server
        return None
    
    def is_restart_allowed(self, hostname: str, service_name: str) -> bool:
        """Check if restart is allowed"""
        server = self.get_server_config(hostname)
        if not server:
            return False
        
        for service in server.get("services", []):
            if service["name"] == service_name:
                return service.get("restart_allowed", False)
        
        return False
```

### Add Database Logging

**File**: `database.py`

```python
from sqlalchemy import create_engine, Column, Integer, String, DateTime, Boolean
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime

Base = declarative_base()

class RemediationLog(Base):
    __tablename__ = 'remediation_logs'
    
    id = Column(Integer, primary_key=True)
    ticket_number = Column(String(50))
    server_name = Column(String(255))
    service_name = Column(String(255))
    action = Column(String(50))
    success = Column(Boolean)
    error_message = Column(String(500))
    timestamp = Column(DateTime, default=datetime.utcnow)

# Create engine
engine = create_engine('sqlite:///remediation.db')
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)
```

### Add Email Monitoring

**File**: `email_monitor.py`

```python
import imaplib
import email
from email.header import decode_header
import re

class EmailMonitor:
    def __init__(self, server: str, username: str, password: str):
        self.server = server
        self.username = username
        self.password = password
    
    def check_new_tickets(self):
        """Check for new ticket emails"""
        mail = imaplib.IMAP4_SSL(self.server)
        mail.login(self.username, self.password)
        mail.select("INBOX")
        
        # Search for unread emails from ServiceNow
        _, messages = mail.search(None, 'UNSEEN FROM "servicenow"')
        
        tickets = []
        for num in messages[0].split():
            _, msg = mail.fetch(num, "(RFC822)")
            email_body = msg[0][1]
            email_message = email.message_from_bytes(email_body)
            
            # Extract ticket number and description
            subject = email_message["Subject"]
            ticket_match = re.search(r'INC\d+', subject)
            
            if ticket_match:
                tickets.append({
                    "number": ticket_match.group(),
                    "description": self._get_email_body(email_message)
                })
        
        mail.close()
        mail.logout()
        
        return tickets
    
    def _get_email_body(self, email_message):
        """Extract email body"""
        if email_message.is_multipart():
            for part in email_message.walk():
                if part.get_content_type() == "text/plain":
                    return part.get_payload(decode=True).decode()
        else:
            return email_message.get_payload(decode=True).decode()
```

## Phase 3: Production Ready (Week 5-6)

### Add Webhook Support

**File**: `webhook_server.py`

```python
from fastapi import FastAPI, HTTPException, Header
import hmac
import hashlib
import os

app = FastAPI()

def verify_signature(payload: str, signature: str) -> bool:
    """Verify webhook signature"""
    secret = os.getenv("WEBHOOK_SECRET").encode()
    expected = hmac.new(secret, payload.encode(), hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, signature)

@app.post("/webhook/servicenow")
async def servicenow_webhook(
    ticket_data: dict,
    x_signature: str = Header(None)
):
    """Receive ServiceNow webhooks"""
    # Verify signature
    if not verify_signature(str(ticket_data), x_signature):
        raise HTTPException(status_code=401, detail="Invalid signature")
    
    # Process ticket
    # ... (integrate with RemediationAgent)
    
    return {"status": "accepted"}

@app.get("/health")
async def health():
    return {"status": "healthy"}
```

### Add Monitoring Dashboard

**File**: `dashboard.py`

```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server

# Metrics
tickets_processed = Counter('tickets_processed_total', 'Total tickets processed')
remediation_success = Counter('remediation_success_total', 'Successful remediations')
remediation_failure = Counter('remediation_failure_total', 'Failed remediations')
remediation_duration = Histogram('remediation_duration_seconds', 'Remediation duration')
active_tickets = Gauge('active_tickets', 'Currently processing tickets')

# Start metrics server
start_http_server(9090)
```

## Deployment

### Docker Deployment

**File**: `Dockerfile`

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "main.py"]
```

**File**: `docker-compose.yml`

```yaml
version: '3.8'

services:
  agent:
    build: .
    env_file: .env
    restart: unless-stopped
    volumes:
      - ./config.yaml:/app/config.yaml
      - ./logs:/app/logs
    ports:
      - "8443:8443"  # Webhook
      - "9090:9090"  # Metrics

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: remediation
      POSTGRES_USER: agent
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Systemd Service (Linux)

**File**: `/etc/systemd/system/remediation-agent.service`

```ini
[Unit]
Description=Windows Service Remediation Agent
After=network.target

[Service]
Type=simple
User=remediation
WorkingDirectory=/opt/remediation-agent
ExecStart=/opt/remediation-agent/venv/bin/python main.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

## Testing Strategy

### Unit Tests

**File**: `test_service_manager.py`

```python
import pytest
from unittest.mock import Mock, patch
from service_manager import ServiceManager

def test_get_service_status():
    with patch('winrm.Session') as mock_session:
        mock_result = Mock()
        mock_result.status_code = 0
        mock_result.std_out = '{"Status": "Running"}'
        mock_session.return_value.run_ps.return_value = mock_result
        
        mgr = ServiceManager("test-server", "user", "pass")
        status = mgr.get_service_status("TestService")
        
        assert status["Status"] == "Running"
```

### Integration Tests

**File**: `test_integration.py`

```python
import pytest
from main import RemediationAgent

@pytest.mark.integration
def test_full_remediation_flow():
    """Test complete remediation workflow"""
    agent = RemediationAgent()
    
    # Mock ticket
    ticket = {
        "number": "INC0001234",
        "description": "Datadog agent stopped on prod-app-01",
        "sys_id": "test123"
    }
    
    # Process
    agent.process_ticket(ticket)
    
    # Verify ticket updated
    # ... assertions
```

## Monitoring & Alerting

### Grafana Dashboard JSON

Create dashboards to monitor:
- Tickets processed per hour
- Success/failure rates
- Average remediation time
- Service restart frequency
- Agent health status

### Alert Rules

```yaml
# alerts.yml
groups:
  - name: remediation_alerts
    rules:
      - alert: HighFailureRate
        expr: rate(remediation_failure_total[5m]) > 0.5
        annotations:
          summary: "High remediation failure rate"
      
      - alert: AgentDown
        expr: up{job="remediation-agent"} == 0
        for: 5m
        annotations:
          summary: "Remediation agent is down"
```

## Troubleshooting

### Common Issues

1. **WinRM Connection Failed**
   - Check firewall rules
   - Verify TrustedHosts configuration
   - Test with `Test-WSMan -ComputerName server`

2. **Service Restart Failed**
   - Check service account permissions
   - Verify service dependencies
   - Review Windows Event Logs

3. **Ticket Parsing Low Confidence**
   - Review LLM prompt
   - Add more examples to training
   - Implement fallback to manual review

## Next Steps

After MVP is working:
1. Add Azure Key Vault integration
2. Implement LangGraph orchestration
3. Add predictive analytics
4. Create web dashboard
5. Implement approval workflows
6. Add Slack/Teams notifications

This guide provides a practical, step-by-step approach to building the system incrementally.