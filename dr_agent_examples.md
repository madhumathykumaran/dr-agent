# Disaster Recovery Compliance Agent System - Agent Examples

This document provides comprehensive examples of request and response formats for each agent in the Disaster Recovery Compliance Agent system. These examples will help you understand how to properly format inputs and what outputs to expect from each agent.

## 1. Process Dependency Agent

This agent finds underlying dependencies and application codes which submitted appcodes rely on.

### Request Format:
```json
{
  "business_process": "Payment Processing",
  "app_codes": ["APP001", "APP002"]
}
```

### Response Format:
```json
{
  "status": "success",
  "message": "Successfully found dependencies for 2 application codes",
  "data": {
    "business_process": "Payment Processing",
    "submitted_app_codes": ["APP001", "APP002"],
    "dependent_app_codes": ["APP003", "APP004", "APP005"],
    "all_app_codes": ["APP001", "APP002", "APP003", "APP004", "APP005"],
    "dependencies": [
      {
        "app_code": "APP001",
        "depends_on": ["APP003", "APP004"]
      },
      {
        "app_code": "APP002",
        "depends_on": ["APP004", "APP005"]
      }
    ]
  }
}
```

### How to Use:
```python
from src.agents.process_dependency.agent import process_dependency_agent

# Create input message
message = {
  "business_process": "Payment Processing",
  "app_codes": ["APP001", "APP002"]
}

# Process the message
result = process_dependency_agent.process_message(message)

# Or generate a reply using the AG2 agent
reply = process_dependency_agent.generate_reply(message)
```

## 2. DR Plans Fetcher Agent

This agent fetches disaster recovery plans from Postgres database table and/or Business Continuity Plans (BCP) from ServiceNow.

### Request Format (Using Postgres):
```json
{
  "app_codes": ["APP001", "APP002", "APP003"],
  "use_servicenow": false
}
```

### Request Format (Using ServiceNow):
```json
{
  "app_codes": ["APP001", "APP002", "APP003"],
  "use_servicenow": true
}
```

### Response Format:
```json
{
  "status": "success",
  "message": "Fetched 3 plans from ServiceNow for 3 application codes",
  "data": {
    "app_codes": ["APP001", "APP002", "APP003"],
    "plans": [
      {
        "id": "plan-001",
        "name": "Payment Processing DR Plan",
        "description": "Disaster recovery plan for the payment processing application",
        "app_code": "APP001",
        "version": "1.2",
        "recovery_time_objective": 120,
        "recovery_point_objective": 15,
        "created_at": "2023-01-15T10:30:00+00:00",
        "updated_at": "2023-03-20T14:45:00+00:00",
        "tasks": [
          {
            "id": "task-001",
            "name": "Restore database from backup",
            "description": "Restore the payment database from the latest backup",
            "sequence": 1,
            "estimated_duration": 30,
            "owner": "Database Team",
            "dependencies": []
          },
          {
            "id": "task-002",
            "name": "Start application servers",
            "description": "Start the application servers in the DR environment",
            "sequence": 2,
            "estimated_duration": 15,
            "owner": "Application Team",
            "dependencies": ["task-001"]
          }
        ],
        "devices": [
          {
            "id": "device-001",
            "name": "Payment DB Server",
            "type": "Database Server",
            "hostname": "paydb01.example.com",
            "ip_address": "10.0.1.10",
            "operating_system": "Linux",
            "owner": "Database Team"
          }
        ]
      },
      // Additional plans...
    ],
    "plan_count": 3,
    "source": "ServiceNow",
    "servicenow_api_type": "external"
  }
}
```

### How to Use:
```python
from src.agents.dr_plans_fetcher.agent import dr_plans_fetcher_agent

# Create input message for ServiceNow
message = {
  "app_codes": ["APP001", "APP002", "APP003"],
  "use_servicenow": true
}

# Process the message
result = dr_plans_fetcher_agent.process_message(message)

# Or generate a reply using the AG2 agent
reply = dr_plans_fetcher_agent.generate_reply(message)

# You can also directly fetch plans
plans = dr_plans_fetcher_agent.fetch_plans(
  app_codes=["APP001", "APP002", "APP003"],
  use_servicenow=True  # Use ServiceNow
)
```

## 3. Description Analysis Agent

This agent analyzes each plan's description against standard plan guidelines.

### Request Format:
```json
{
  "plans_data": {
    "status": "success",
    "data": {
      "plans": [
        {
          "id": "plan-001",
          "name": "Payment Processing DR Plan",
          "description": "Disaster recovery plan for the payment processing application...",
          "app_code": "APP001",
          "version": "1.2",
          "recovery_time_objective": 120,
          "recovery_point_objective": 15,
          "tasks": [...],
          "devices": [...]
        }
      ],
      "source": "ServiceNow"
    }
  }
}
```

### Response Format:
```json
{
  "status": "success",
  "message": "Successfully analyzed descriptions for 1 plan",
  "data": {
    "analysis_results": [
      {
        "plan_id": "plan-001",
        "app_code": "APP001",
        "plan_name": "Payment Processing DR Plan",
        "summary": "The plan description provides a basic overview of the disaster recovery process for the payment processing application.",
        "gaps": [
          "Missing detailed recovery objectives",
          "No mention of communication procedures",
          "Lacks testing schedule information"
        ],
        "improvements": [
          "Add specific RTO/RPO targets in the description",
          "Include communication procedures and contact information",
          "Add information about testing schedule and results"
        ],
        "compliance_score": 65
      }
    ],
    "overall_summary": "The analyzed plan descriptions have an average compliance score of 65%. Most plans lack detailed recovery objectives and testing information."
  }
}
```

### How to Use:
```python
from src.agents.description_analysis.agent import description_analysis_agent

# Create input message using output from DR Plans Fetcher Agent
message = {
  "plans_data": dr_plans_fetcher_result
}

# Process the message
result = description_analysis_agent.process_message(message)

# Or generate a reply using the AG2 agent
reply = description_analysis_agent.generate_reply(message)
```

## 4. Tasks Analysis Agent

This agent analyzes each plan's recovery tasks against standard plan guidelines.

### Request Format:
```json
{
  "plans_data": {
    "status": "success",
    "data": {
      "plans": [
        {
          "id": "plan-001",
          "name": "Payment Processing DR Plan",
          "description": "...",
          "app_code": "APP001",
          "version": "1.2",
          "recovery_time_objective": 120,
          "recovery_point_objective": 15,
          "tasks": [
            {
              "id": "task-001",
              "name": "Restore database from backup",
              "description": "Restore the payment database from the latest backup",
              "sequence": 1,
              "estimated_duration": 30,
              "owner": "Database Team",
              "dependencies": []
            },
            {
              "id": "task-002",
              "name": "Start application servers",
              "description": "Start the application servers in the DR environment",
              "sequence": 2,
              "estimated_duration": 15,
              "owner": "Application Team",
              "dependencies": ["task-001"]
            }
          ],
          "devices": [...]
        }
      ],
      "source": "ServiceNow"
    }
  }
}
```

### Response Format:
```json
{
  "status": "success",
  "message": "Successfully analyzed recovery tasks for 1 plan",
  "data": {
    "analysis_results": [
      {
        "plan_id": "plan-001",
        "app_code": "APP001",
        "plan_name": "Payment Processing DR Plan",
        "task_count": 2,
        "total_estimated_duration": 45,
        "critical_path_duration": 45,
        "task_analysis": [
          {
            "task_id": "task-001",
            "task_name": "Restore database from backup",
            "gaps": ["No verification step", "No fallback procedure"],
            "improvements": ["Add verification step", "Include fallback procedure"]
          },
          {
            "task_id": "task-002",
            "task_name": "Start application servers",
            "gaps": ["No health check procedure"],
            "improvements": ["Add health check verification"]
          }
        ],
        "overall_gaps": [
          "Missing network configuration tasks",
          "No validation tasks for application functionality"
        ],
        "overall_improvements": [
          "Add network configuration tasks",
          "Include validation tasks for application functionality"
        ],
        "compliance_score": 70
      }
    ],
    "overall_summary": "The analyzed plan tasks have an average compliance score of 70%. Most plans have well-defined task sequences but lack verification steps and fallback procedures."
  }
}
```

### How to Use:
```python
from src.agents.tasks_analysis.agent import tasks_analysis_agent

# Create input message using output from DR Plans Fetcher Agent
message = {
  "plans_data": dr_plans_fetcher_result
}

# Process the message
result = tasks_analysis_agent.process_message(message)

# Or generate a reply using the AG2 agent
reply = tasks_analysis_agent.generate_reply(message)
```

## 5. Device Reconciliation Agent

This agent matches devices in the plan with devices that actually belong to the app.

### Request Format:
```json
{
  "plans_data": {
    "status": "success",
    "data": {
      "plans": [
        {
          "id": "plan-001",
          "name": "Payment Processing DR Plan",
          "app_code": "APP001",
          "devices": [
            {
              "id": "device-001",
              "name": "Payment DB Server",
              "type": "Database Server",
              "hostname": "paydb01.example.com",
              "ip_address": "10.0.1.10",
              "operating_system": "Linux",
              "owner": "Database Team"
            }
          ]
        }
      ]
    }
  }
}
```

### Response Format:
```json
{
  "status": "success",
  "message": "Successfully reconciled devices for 1 plan",
  "data": {
    "reconciliation_results": [
      {
        "plan_id": "plan-001",
        "app_code": "APP001",
        "plan_name": "Payment Processing DR Plan",
        "devices_in_plan": 1,
        "devices_in_inventory": 3,
        "matching_devices": 1,
        "missing_devices": [
          {
            "id": "inv-device-002",
            "name": "Payment App Server",
            "type": "Application Server",
            "hostname": "payapp01.example.com",
            "ip_address": "10.0.1.20",
            "operating_system": "Linux",
            "owner": "Application Team"
          },
          {
            "id": "inv-device-003",
            "name": "Payment Load Balancer",
            "type": "Load Balancer",
            "hostname": "paylb01.example.com",
            "ip_address": "10.0.1.5",
            "operating_system": "Linux",
            "owner": "Network Team"
          }
        ],
        "extra_devices": [],
        "compliance_score": 33
      }
    ],
    "overall_summary": "The analyzed plans have an average device compliance score of 33%. Most plans are missing critical devices from their inventory."
  }
}
```

### How to Use:
```python
from src.agents.device_reconciliation.agent import device_reconciliation_agent

# Create input message using output from DR Plans Fetcher Agent
message = {
  "plans_data": dr_plans_fetcher_result
}

# Process the message
result = device_reconciliation_agent.process_message(message)

# Or generate a reply using the AG2 agent
reply = device_reconciliation_agent.generate_reply(message)
```

## 6. IIPM Analysis Agent

This agent fetches required details for all supporting appcodes for the business process and dependent appcodes.

### Request Format:
```json
{
  "business_process": "Payment Processing",
  "dependencies_data": {
    "status": "success",
    "data": {
      "business_process": "Payment Processing",
      "submitted_app_codes": ["APP001", "APP002"],
      "dependent_app_codes": ["APP003", "APP004", "APP005"],
      "all_app_codes": ["APP001", "APP002", "APP003", "APP004", "APP005"]
    }
  }
}
```

### Response Format:
```json
{
  "status": "success",
  "message": "Successfully fetched IIPM data for 5 application codes",
  "data": {
    "business_process": "Payment Processing",
    "app_codes": ["APP001", "APP002", "APP003", "APP004", "APP005"],
    "iipm_data": [
      {
        "app_code": "APP001",
        "name": "Payment Processing Application",
        "description": "Core payment processing application",
        "business_criticality": "High",
        "recovery_time_objective": 120,
        "recovery_point_objective": 15,
        "business_impact": "Severe",
        "owner": "Payment Team",
        "support_team": "Payment Support",
        "dependencies": ["APP003", "APP004"]
      },
      {
        "app_code": "APP002",
        "name": "Payment Gateway",
        "description": "Payment gateway integration",
        "business_criticality": "High",
        "recovery_time_objective": 60,
        "recovery_point_objective": 5,
        "business_impact": "Severe",
        "owner": "Gateway Team",
        "support_team": "Gateway Support",
        "dependencies": ["APP004", "APP005"]
      },
      // Additional app data...
    ],
    "summary": {
      "total_apps": 5,
      "high_criticality_apps": 3,
      "medium_criticality_apps": 2,
      "low_criticality_apps": 0,
      "average_rto": 96,
      "average_rpo": 10
    }
  }
}
```

### How to Use:
```python
from src.agents.iipm_analysis.agent import iipm_analysis_agent

# Create input message using output from Process Dependency Agent
message = {
  "business_process": "Payment Processing",
  "dependencies_data": process_dependency_result
}

# Process the message
result = iipm_analysis_agent.process_message(message)

# Or generate a reply using the AG2 agent
reply = iipm_analysis_agent.generate_reply(message)
```

## 7. Reasoning Agent

This agent judges overall quality of Disaster recovery readiness of the business process.

### Request Format:
```json
{
  "business_process": "Payment Processing",
  "dependencies_data": {
    "status": "success",
    "data": {
      "business_process": "Payment Processing",
      "all_app_codes": ["APP001", "APP002", "APP003", "APP004", "APP005"]
    }
  },
  "plans_data": {
    "status": "success",
    "data": {
      "plans": [...],
      "source": "ServiceNow"
    }
  },
  "description_analysis_data": {
    "status": "success",
    "data": {
      "analysis_results": [...]
    }
  },
  "tasks_analysis_data": {
    "status": "success",
    "data": {
      "analysis_results": [...]
    }
  },
  "device_reconciliation_data": {
    "status": "success",
    "data": {
      "reconciliation_results": [...]
    }
  },
  "iipm_data": {
    "status": "success",
    "data": {
      "iipm_data": [...]
    }
  }
}
```

### Response Format:
```json
{
  "status": "success",
  "message": "Successfully analyzed DR readiness for Payment Processing business process",
  "data": {
    "business_process": "Payment Processing",
    "overall_dr_readiness_score": 68,
    "dr_readiness_rating": "Medium",
    "key_findings": [
      "3 out of 5 applications have DR plans",
      "Average plan quality score is 67%",
      "Critical dependency APP003 has no DR plan",
      "Device reconciliation shows missing critical devices in most plans",
      "Recovery tasks lack verification steps"
    ],
    "risks": [
      {
        "description": "Missing DR plan for APP003",
        "impact": "High",
        "mitigation": "Create DR plan for APP003 immediately"
      },
      {
        "description": "Missing critical devices in DR plans",
        "impact": "Medium",
        "mitigation": "Update DR plans to include all critical devices"
      }
    ],
    "rto_rpo_analysis": {
      "business_process_rto": 120,
      "business_process_rpo": 15,
      "dependent_apps_max_rto": 180,
      "dependent_apps_max_rpo": 30,
      "rto_gap": 60,
      "rpo_gap": 15,
      "rto_rpo_achievable": false
    },
    "recommendations": [
      "Create DR plans for all applications without plans",
      "Update device lists in all plans to match actual inventory",
      "Add verification steps to all recovery tasks",
      "Reduce RTO for dependent application APP003 to meet business process requirements",
      "Conduct end-to-end DR testing to validate recovery procedures"
    ]
  }
}
```

### How to Use:
```python
from src.agents.reasoning.agent import reasoning_agent

# Create input message using outputs from all previous agents
message = {
  "business_process": "Payment Processing",
  "dependencies_data": process_dependency_result,
  "plans_data": dr_plans_fetcher_result,
  "description_analysis_data": description_analysis_result,
  "tasks_analysis_data": tasks_analysis_result,
  "device_reconciliation_data": device_reconciliation_result,
  "iipm_data": iipm_analysis_result
}

# Process the message
result = reasoning_agent.process_message(message)

# Or generate a reply using the AG2 agent
reply = reasoning_agent.generate_reply(message)
```

## Complete Workflow Example

Here's how to chain all the agents together in a complete workflow:

```python
# 1. Process Dependency Agent
dependency_message = {
  "business_process": "Payment Processing",
  "app_codes": ["APP001", "APP002"]
}
dependency_result = process_dependency_agent.process_message(dependency_message)

# 2. DR Plans Fetcher Agent
plans_message = {
  "app_codes": dependency_result["data"]["all_app_codes"],
  "use_servicenow": True  # Use ServiceNow API
}
plans_result = dr_plans_fetcher_agent.process_message(plans_message)

# 3. Description Analysis Agent
description_message = {
  "plans_data": plans_result
}
description_result = description_analysis_agent.process_message(description_message)

# 4. Tasks Analysis Agent
tasks_message = {
  "plans_data": plans_result
}
tasks_result = tasks_analysis_agent.process_message(tasks_message)

# 5. Device Reconciliation Agent
device_message = {
  "plans_data": plans_result
}
device_result = device_reconciliation_agent.process_message(device_message)

# 6. IIPM Analysis Agent
iipm_message = {
  "business_process": "Payment Processing",
  "dependencies_data": dependency_result
}
iipm_result = iipm_analysis_agent.process_message(iipm_message)

# 7. Reasoning Agent
reasoning_message = {
  "business_process": "Payment Processing",
  "dependencies_data": dependency_result,
  "plans_data": plans_result,
  "description_analysis_data": description_result,
  "tasks_analysis_data": tasks_result,
  "device_reconciliation_data": device_result,
  "iipm_data": iipm_result
}
reasoning_result = reasoning_agent.process_message(reasoning_message)

# Final output with DR readiness assessment
print(f"DR Readiness Score: {reasoning_result['data']['overall_dr_readiness_score']}")
print(f"DR Readiness Rating: {reasoning_result['data']['dr_readiness_rating']}")
print("\nKey Findings:")
for finding in reasoning_result["data"]["key_findings"]:
    print(f"- {finding}")
```
