# Project Structure & Technical Specifications

## Project Directory Structure

```
windows-service-remediation/
├── src/
│   ├── __init__.py
│   ├── main.py                          # Application entry point
│   ├── config/
│   │   ├── __init__.py
│   │   ├── settings.py                  # Configuration management
│   │   ├── services.yaml                # Service definitions
│   │   └── servers.yaml                 # Server inventory
│   │
│   ├── integrations/
│   │   ├── __init__.py
│   │   ├── servicenow/
│   │   │   ├── __init__.py
│   │   │   ├── api_client.py           # SNOW API integration
│   │   │   ├── ticket_parser.py        # Ticket data extraction
│   │   │   └── ticket_updater.py       # Update ticket status
│   │   │
│   │   ├── email/
│   │   │   ├── __init__.py
│   │   │   ├── monitor.py              # Email monitoring
│   │   │   └── parser.py               # Email parsing
│   │   │
│   │   └── webhook/
│   │       ├── __init__.py
│   │       ├── receiver.py             # Webhook endpoint
│   │       └── validator.py            # Signature validation
│   │
│   ├── ai_agent/
│   │   ├── __init__.py
│   │   ├── orchestrator.py             # LangGraph orchestrator
│   │   ├── ticket_analyzer.py          # LLM-based ticket analysis
│   │   ├── decision_engine.py          # Decision logic
│   │   ├── prompts.py                  # LLM prompt templates
│   │   └── state.py                    # Agent state management
│   │
│   ├── windows/
│   │   ├── __init__.py
│   │   ├── service_manager.py          # Service operations
│   │   ├── winrm_client.py            # WinRM connection
│   │   ├── powershell_executor.py     # PowerShell command execution
│   │   └── health_checker.py          # Service health validation
│   │
│   ├── security/
│   │   ├── __init__.py
│   │   ├── credential_manager.py       # Credential vault integration
│   │   ├── encryption.py               # Data encryption utilities
│   │   └── audit_logger.py            # Security audit logging
│   │
│   ├── database/
│   │   ├── __init__.py
│   │   ├── models.py                   # SQLAlchemy models
│   │   ├── repository.py               # Data access layer
│   │   └── migrations/                 # Alembic migrations
│   │
│   ├── monitoring/
│   │   ├── __init__.py
│   │   ├── metrics.py                  # Prometheus metrics
│   │   ├── health_check.py            # Health endpoints
│   │   └── dashboard.py               # Grafana dashboard config
│   │
│   ├── notifications/
│   │   ├── __init__.py
│   │   ├── email_notifier.py          # Email notifications
│   │   ├── slack_notifier.py          # Slack integration
│   │   └── teams_notifier.py          # MS Teams integration
│   │
│   └── utils/
│       ├── __init__.py
│       ├── logger.py                   # Logging configuration
│       ├── retry.py                    # Retry decorators
│       └── validators.py               # Input validation
│
├── tests/
│   ├── __init__.py
│   ├── unit/
│   │   ├── test_ticket_parser.py
│   │   ├── test_service_manager.py
│   │   ├── test_decision_engine.py
│   │   └── test_winrm_client.py
│   │
│   ├── integration/
│   │   ├── test_snow_integration.py
│   │   ├── test_email_integration.py
│   │   └── test_end_to_end.py
│   │
│   └── fixtures/
│       ├── sample_tickets.json
│       └── mock_responses.py
│
├── scripts/
│   ├── setup_winrm.ps1                 # WinRM setup script
│   ├── deploy.sh                       # Deployment script
│   └── test_connection.py              # Connection testing
│
├── docker/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── .dockerignore
│
├── docs/
│   ├── architecture.md
│   ├── deployment.md
│   ├── configuration.md
│   └── troubleshooting.md
│
├── .github/
│   └── workflows/
│       ├── ci.yml                      # CI pipeline
│       └── deploy.yml                  # CD pipeline
│
├── requirements.txt                     # Python dependencies
├── requirements-dev.txt                 # Development dependencies
├── pyproject.toml                       # Project metadata
├── .env.example                         # Environment variables template
├── .gitignore
└── README.md
```

## Core Dependencies

### requirements.txt
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

# Testing (in requirements-dev.txt)
pytest==7.4.4
pytest-asyncio==0.23.3
pytest-cov==4.1.0
pytest-mock==3.12.0
black==24.1.1
flake8==7.0.0
mypy==1.8.0
```

## Key Module Specifications

### 1. ServiceNow API Client

**File**: [`src/integrations/servicenow/api_client.py`](src/integrations/servicenow/api_client.py)

```python
from typing import List, Dict, Optional
import pysnow
from datetime import datetime, timedelta

class ServiceNowClient:
    """ServiceNow API client for ticket management"""
    
    def __init__(self, instance: str, username: str, password: str):
        self.client = pysnow.Client(
            instance=instance,
            user=username,
            password=password
        )
        self.incident_resource = self.client.resource(api_path='/table/incident')
    
    async def get_new_tickets(
        self, 
        assignment_group: str,
        since_minutes: int = 5
    ) -> List[Dict]:
        """Fetch new tickets assigned to group"""
        since_time = datetime.utcnow() - timedelta(minutes=since_minutes)
        
        query = (
            pysnow.QueryBuilder()
            .field('assignment_group').equals(assignment_group)
            .AND()
            .field('state').equals('New')
            .AND()
            .field('sys_created_on').greater_than(since_time)
        )
        
        return self.incident_resource.get(query=query).all()
    
    async def update_ticket(
        self,
        ticket_number: str,
        work_notes: str,
        state: Optional[str] = None
    ) -> bool:
        """Update ticket with remediation status"""
        update_data = {'work_notes': work_notes}
        if state:
            update_data['state'] = state
        
        ticket = self.incident_resource.get(
            query={'number': ticket_number}
        ).one()
        
        return ticket.update(update_data)
```

### 2. AI Ticket Analyzer

**File**: [`src/ai_agent/ticket_analyzer.py`](src/ai_agent/ticket_analyzer.py)

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from pydantic import BaseModel, Field
from typing import Optional

class TicketInfo(BaseModel):
    """Structured ticket information"""
    server_name: str = Field(description="Server hostname or FQDN")
    service_name: str = Field(description="Windows service name")
    issue_type: str = Field(description="Type of issue: stopped, crashed, not_responding")
    priority: str = Field(description="Priority: low, medium, high, critical")
    confidence_score: float = Field(description="Confidence in extraction (0-1)")

class TicketAnalyzer:
    """LLM-powered ticket analysis"""
    
    def __init__(self, api_key: str, model: str = "gpt-4-turbo"):
        self.llm = ChatOpenAI(
            api_key=api_key,
            model=model,
            temperature=0
        )
        self.prompt = ChatPromptTemplate.from_messages([
            ("system", """You are an expert at extracting structured information 
            from IT service tickets. Extract the server name, service name, 
            issue type, and priority from the ticket description.
            
            Common service names:
            - DatadogAgent, Datadog Agent -> DatadogAgent
            - Splunk Forwarder, splunkforwarder -> splunkforwarder
            - SQL Server, MSSQLSERVER -> MSSQLSERVER
            
            Return confidence_score based on clarity of information."""),
            ("user", "Ticket Number: {ticket_number}\n\nDescription: {description}")
        ])
    
    async def analyze_ticket(
        self,
        ticket_number: str,
        description: str
    ) -> TicketInfo:
        """Extract structured information from ticket"""
        chain = self.prompt | self.llm.with_structured_output(TicketInfo)
        
        result = await chain.ainvoke({
            "ticket_number": ticket_number,
            "description": description
        })
        
        return result
```

### 3. Windows Service Manager

**File**: [`src/windows/service_manager.py`](src/windows/service_manager.py)

```python
from typing import Dict, Optional
import winrm
from tenacity import retry, stop_after_attempt, wait_exponential

class ServiceManager:
    """Manage Windows services via WinRM"""
    
    def __init__(
        self,
        hostname: str,
        username: str,
        password: str,
        transport: str = 'ntlm'
    ):
        self.hostname = hostname
        self.session = winrm.Session(
            f'http://{hostname}:5985/wsman',
            auth=(username, password),
            transport=transport
        )
    
    async def get_service_status(self, service_name: str) -> Dict:
        """Get current service status"""
        ps_script = f"""
        $service = Get-Service -Name '{service_name}' -ErrorAction SilentlyContinue
        if ($service) {{
            @{{
                Name = $service.Name
                DisplayName = $service.DisplayName
                Status = $service.Status.ToString()
                StartType = $service.StartType.ToString()
            }} | ConvertTo-Json
        }} else {{
            @{{Error = "Service not found"}} | ConvertTo-Json
        }}
        """
        
        result = self.session.run_ps(ps_script)
        return self._parse_result(result)
    
    @retry(
        stop=stop_after_attempt(3),
        wait=wait_exponential(multiplier=1, min=4, max=10)
    )
    async def restart_service(
        self,
        service_name: str,
        force: bool = True
    ) -> Dict:
        """Restart Windows service"""
        force_flag = "-Force" if force else ""
        
        ps_script = f"""
        try {{
            Restart-Service -Name '{service_name}' {force_flag} -ErrorAction Stop
            Start-Sleep -Seconds 5
            $service = Get-Service -Name '{service_name}'
            @{{
                Success = $true
                Status = $service.Status.ToString()
                Message = "Service restarted successfully"
            }} | ConvertTo-Json
        }} catch {{
            @{{
                Success = $false
                Error = $_.Exception.Message
            }} | ConvertTo-Json
        }}
        """
        
        result = self.session.run_ps(ps_script)
        return self._parse_result(result)
    
    async def verify_service_running(
        self,
        service_name: str,
        timeout_seconds: int = 30
    ) -> bool:
        """Verify service is running and stable"""
        ps_script = f"""
        $timeout = {timeout_seconds}
        $elapsed = 0
        
        while ($elapsed -lt $timeout) {{
            $service = Get-Service -Name '{service_name}'
            if ($service.Status -eq 'Running') {{
                Start-Sleep -Seconds 5
                $service = Get-Service -Name '{service_name}'
                if ($service.Status -eq 'Running') {{
                    return $true
                }}
            }}
            Start-Sleep -Seconds 2
            $elapsed += 2
        }}
        return $false
        """
        
        result = self.session.run_ps(ps_script)
        return result.status_code == 0 and 'True' in result.std_out
```

### 4. LangGraph Orchestrator

**File**: [`src/ai_agent/orchestrator.py`](src/ai_agent/orchestrator.py)

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
from enum import Enum

class RemediationState(TypedDict):
    """State for remediation workflow"""
    ticket_number: str
    ticket_description: str
    server_name: str
    service_name: str
    issue_type: str
    priority: str
    confidence_score: float
    service_status: str
    remediation_action: str
    success: bool
    error_message: str
    attempts: int

class WorkflowStage(Enum):
    PARSE_TICKET = "parse_ticket"
    VALIDATE_CONFIG = "validate_config"
    CHECK_SERVICE = "check_service"
    DECIDE_ACTION = "decide_action"
    EXECUTE_REMEDIATION = "execute_remediation"
    VERIFY_SUCCESS = "verify_success"
    UPDATE_TICKET = "update_ticket"
    ESCALATE = "escalate"

class RemediationOrchestrator:
    """LangGraph-based remediation orchestrator"""
    
    def __init__(
        self,
        ticket_analyzer,
        service_manager,
        config_manager,
        snow_client
    ):
        self.ticket_analyzer = ticket_analyzer
        self.service_manager = service_manager
        self.config_manager = config_manager
        self.snow_client = snow_client
        self.workflow = self._build_workflow()
    
    def _build_workflow(self) -> StateGraph:
        """Build the remediation workflow graph"""
        workflow = StateGraph(RemediationState)
        
        # Add nodes
        workflow.add_node("parse_ticket", self._parse_ticket)
        workflow.add_node("validate_config", self._validate_config)
        workflow.add_node("check_service", self._check_service)
        workflow.add_node("decide_action", self._decide_action)
        workflow.add_node("execute_remediation", self._execute_remediation)
        workflow.add_node("verify_success", self._verify_success)
        workflow.add_node("update_ticket", self._update_ticket)
        workflow.add_node("escalate", self._escalate)
        
        # Define edges
        workflow.set_entry_point("parse_ticket")
        workflow.add_edge("parse_ticket", "validate_config")
        workflow.add_conditional_edges(
            "validate_config",
            self._should_proceed,
            {
                "proceed": "check_service",
                "escalate": "escalate"
            }
        )
        workflow.add_edge("check_service", "decide_action")
        workflow.add_edge("decide_action", "execute_remediation")
        workflow.add_edge("execute_remediation", "verify_success")
        workflow.add_conditional_edges(
            "verify_success",
            self._check_success,
            {
                "success": "update_ticket",
                "retry": "execute_remediation",
                "escalate": "escalate"
            }
        )
        workflow.add_edge("update_ticket", END)
        workflow.add_edge("escalate", END)
        
        return workflow.compile()
    
    async def _parse_ticket(self, state: RemediationState) -> RemediationState:
        """Parse ticket using LLM"""
        ticket_info = await self.ticket_analyzer.analyze_ticket(
            state["ticket_number"],
            state["ticket_description"]
        )
        
        state.update({
            "server_name": ticket_info.server_name,
            "service_name": ticket_info.service_name,
            "issue_type": ticket_info.issue_type,
            "priority": ticket_info.priority,
            "confidence_score": ticket_info.confidence_score
        })
        
        return state
    
    async def run_remediation(self, ticket_data: Dict) -> RemediationState:
        """Execute the remediation workflow"""
        initial_state = RemediationState(
            ticket_number=ticket_data["number"],
            ticket_description=ticket_data["description"],
            attempts=0,
            success=False
        )
        
        final_state = await self.workflow.ainvoke(initial_state)
        return final_state
```

### 5. Configuration Manager

**File**: [`src/config/settings.py`](src/config/settings.py)

```python
from pydantic_settings import BaseSettings
from typing import Dict, List
import yaml

class Settings(BaseSettings):
    """Application settings"""
    
    # ServiceNow
    snow_instance_url: str
    snow_username: str
    snow_password: str
    snow_assignment_group: str
    
    # Email
    email_server: str
    email_username: str
    email_password: str
    email_folder: str = "INBOX"
    
    # Webhook
    webhook_secret: str
    webhook_port: int = 8443
    
    # AI/LLM
    openai_api_key: str
    llm_model: str = "gpt-4-turbo"
    
    # Database
    database_url: str
    redis_url: str = "redis://localhost:6379"
    
    # Monitoring
    prometheus_port: int = 9090
    log_level: str = "INFO"
    
    # Remediation
    max_restart_attempts: int = 3
    restart_timeout_seconds: int = 60
    health_check_retries: int = 3
    
    class Config:
        env_file = ".env"
        case_sensitive = False

class ServiceConfig:
    """Service configuration manager"""
    
    def __init__(self, config_path: str = "config/services.yaml"):
        with open(config_path, 'r') as f:
            self.config = yaml.safe_load(f)
    
    def get_server_config(self, hostname: str) -> Dict:
        """Get configuration for specific server"""
        for group in self.config.get('server_groups', {}).values():
            if hostname in group.get('servers', []):
                return {
                    'hostname': hostname,
                    'services': group.get('services', []),
                    'credentials_ref': group.get('credentials_ref'),
                    'notification_email': group.get('notification_email')
                }
        return None
    
    def is_service_allowed(
        self,
        hostname: str,
        service_name: str
    ) -> bool:
        """Check if service restart is allowed"""
        server_config = self.get_server_config(hostname)
        if not server_config:
            return False
        
        for service in server_config['services']:
            if service['name'] == service_name:
                return service.get('restart_allowed', False)
        
        return False
```

## Database Schema

### SQLAlchemy Models

**File**: [`src/database/models.py`](src/database/models.py)

```python
from sqlalchemy import Column, Integer, String, DateTime, Boolean, Float, Text
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class RemediationEvent(Base):
    __tablename__ = 'remediation_events'
    
    id = Column(Integer, primary_key=True)
    ticket_number = Column(String(50), nullable=False, index=True)
    server_name = Column(String(255), nullable=False, index=True)
    service_name = Column(String(255), nullable=False)
    issue_type = Column(String(50))
    priority = Column(String(20))
    confidence_score = Column(Float)
    
    action_taken = Column(String(50))
    status = Column(String(50), index=True)
    
    started_at = Column(DateTime, default=datetime.utcnow)
    completed_at = Column(DateTime)
    
    attempts = Column(Integer, default=1)
    success = Column(Boolean, default=False)
    error_message = Column(Text)
    
    executed_by = Column(String(100), default='ai_agent')
    
    def __repr__(self):
        return f"<RemediationEvent {self.ticket_number} - {self.service_name}>"

class ServiceHealth(Base):
    __tablename__ = 'service_health'
    
    id = Column(Integer, primary_key=True)
    server_name = Column(String(255), nullable=False, index=True)
    service_name = Column(String(255), nullable=False)
    status = Column(String(50))
    last_checked = Column(DateTime, default=datetime.utcnow)
    restart_count_24h = Column(Integer, default=0)
    last_restart = Column(DateTime)
    
    def __repr__(self):
        return f"<ServiceHealth {self.server_name}/{self.service_name}>"
```

## API Endpoints

### FastAPI Application

**File**: [`src/main.py`](src/main.py)

```python
from fastapi import FastAPI, HTTPException, Depends, Header
from pydantic import BaseModel
import hmac
import hashlib

app = FastAPI(title="Windows Service Remediation Agent")

class WebhookPayload(BaseModel):
    ticket_number: str
    description: str
    assignment_group: str
    priority: str

@app.post("/webhook/servicenow")
async def servicenow_webhook(
    payload: WebhookPayload,
    x_signature: str = Header(None)
):
    """Receive ServiceNow webhook events"""
    # Validate signature
    if not validate_signature(payload.dict(), x_signature):
        raise HTTPException(status_code=401, detail="Invalid signature")
    
    # Queue for processing
    await queue_remediation_task(payload)
    
    return {"status": "accepted", "ticket": payload.ticket_number}

@app.get("/health")
async def health_check():
    """Health check endpoint"""
    return {
        "status": "healthy",
        "version": "1.0.0",
        "timestamp": datetime.utcnow().isoformat()
    }

@app.get("/metrics")
async def metrics():
    """Prometheus metrics endpoint"""
    # Return Prometheus-formatted metrics
    pass
```

This structure provides a solid foundation for implementation. Would you like me to proceed with creating the actual implementation files, or would you prefer to review and adjust this plan first?