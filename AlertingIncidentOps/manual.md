# Alerting & Incident Ops Configuration Manual

본 매뉴얼은 iPaaS의 Alerting & Incident Ops 모듈에 적용되는 설정(Configuration)에 대한 상세 가이드입니다. 각 기능 그룹별로 설정 필드에 대한 설명(한국어)과 JSON 스키마, 그리고 적용 예시를 제공합니다.

설정 필드명은 `snake_case` 규칙을 따릅니다.

---

## 목차
1. [Alert Source Registry](#1-alert-source-registry)
2. [Severity](#2-severity)
3. [De-noising](#3-de-noising)
4. [Response Sync](#4-response-sync)

---

## 1. Alert Source Registry (`alert_source_registry`)
다양한 알람 소스 및 모니터링 대상을 정의하고 임계값을 설정합니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `sla` | Object | 서비스 수준 계약(SLA) 위반 감지 설정입니다. |
| `sla.enabled` | Boolean | 모니터링 활성화 여부입니다. |
| `sla.threshold_ms` | Integer | 응답 시간 임계값 (밀리초)입니다. |
| `sla.breach_action` | String | 위반 시 수행할 동작입니다 (예: "alert", "log_only"). |
| `security_dlp` | Object | 보안 및 데이터 유출 방지(DLP) 관련 설정입니다. |
| `security_dlp.enabled` | Boolean | 보안 모니터링 활성화 여부입니다. |
| `security_dlp.sensitivity_level` | String | 감지 민감도입니다 ("high", "medium", "low"). |
| `security_dlp.target_protocols` | Array | 모니터링 대상 프로토콜입니다 (예: ["http", "ftp"]). |
| `infra_resource` | Object | 인프라 리소스 사용량 모니터링 설정입니다. |
| `infra_resource.cpu_threshold_percent` | Integer | CPU 사용량 알람 임계값 (%)입니다. |
| `infra_resource.memory_threshold_percent` | Integer | 메모리 사용량 알람 임계값 (%)입니다. |
| `infra_resource.disk_usage_threshold_percent` | Integer | 디스크 사용량 알람 임계값 (%)입니다. |
| `operation_asset` | Object | 운영 자산 상태 모니터링 설정입니다. |
| `operation_asset.monitor_active_assets` | Boolean | 활성 자산 모니터링 여부입니다. |
| `operation_asset.health_check_interval_seconds` | Integer | 상태 확인 주기 (초)입니다. |
| `business_cost` | Object | 비용 및 비즈니스 로직 관련 모니터링 설정입니다. |
| `business_cost.daily_cost_limit_usd` | Number | 일일 비용 한도 (USD)입니다. |
| `business_cost.alert_on_forecast_overrun` | Boolean | 예상 비용 초과 시 알람 여부입니다. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "sla": {
      "type": "object",
      "properties": {
        "enabled": { "type": "boolean" },
        "threshold_ms": { "type": "integer" },
        "breach_action": { "type": "string" }
      },
      "required": ["enabled"]
    },
    "security_dlp": {
      "type": "object",
      "properties": {
        "enabled": { "type": "boolean" },
        "sensitivity_level": { "type": "string", "enum": ["low", "medium", "high"] },
        "target_protocols": { "type": "array", "items": { "type": "string" } }
      }
    },
    "infra_resource": {
      "type": "object",
      "properties": {
        "cpu_threshold_percent": { "type": "integer" },
        "memory_threshold_percent": { "type": "integer" },
        "disk_usage_threshold_percent": { "type": "integer" }
      }
    },
    "operation_asset": {
      "type": "object",
      "properties": {
        "monitor_active_assets": { "type": "boolean" },
        "health_check_interval_seconds": { "type": "integer" }
      }
    },
    "business_cost": {
      "type": "object",
      "properties": {
        "daily_cost_limit_usd": { "type": "number" },
        "alert_on_forecast_overrun": { "type": "boolean" }
      }
    }
  }
}
```

### Sample Configuration
```json
{
  "sla": {
    "enabled": true,
    "threshold_ms": 500,
    "breach_action": "alert"
  },
  "security_dlp": {
    "enabled": true,
    "sensitivity_level": "high",
    "target_protocols": ["http", "https"]
  },
  "infra_resource": {
    "cpu_threshold_percent": 85,
    "memory_threshold_percent": 90,
    "disk_usage_threshold_percent": 80
  },
  "operation_asset": {
    "monitor_active_assets": true,
    "health_check_interval_seconds": 60
  },
  "business_cost": {
    "daily_cost_limit_usd": 1000.00,
    "alert_on_forecast_overrun": true
  }
}
```

---

## 2. Severity (`severity`)
발생한 알람의 중요도를 정의하고, 이에 따른 처리 규칙을 설정합니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `severity_levels` | Object | 중요도 레벨 정의입니다. |
| `severity_levels.definitions` | Array | 레벨 객체 배열입니다. |
| `severity_levels.definitions[].name` | String | 레벨 이름입니다 (예: "critical", "warning"). |
| `severity_levels.definitions[].priority` | Integer | 우선순위입니다 (숫자가 낮을수록 높음). |
| `alert_routing_rules` | Object | 알람 전파 규칙 설정입니다. |
| `alert_routing_rules.default_channel` | String | 기본 알림 채널입니다. |
| `alert_routing_rules.rules` | Array | 라우팅 규칙 배열입니다. |
| `alert_routing_rules.rules[].severity_match` | String | 매칭할 중요도 레벨입니다. |
| `alert_routing_rules.rules[].target_team` | String | 수신 팀입니다. |
| `alert_routing_rules.rules[].channel` | String | 알림 채널입니다 (예: "slack", "email", "pagerduty"). |
| `voice_channel` | Object | 긴급 상황 시 음성 통화 알림 설정입니다. |
| `voice_channel.enabled` | Boolean | 음성 알림 활성화 여부입니다. |
| `voice_channel.provider` | String | 통화 서비스 제공자입니다 (예: "twilio", "aws_connect"). |
| `voice_channel.trigger_severity` | Array | 음성 알림을 트리거할 중요도 레벨 목록입니다. |
| `voice_channel.call_flow_id` | String | 실행할 콜 플로우 ID입니다. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "severity_levels": {
      "type": "object",
      "properties": {
        "definitions": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "name": { "type": "string" },
              "priority": { "type": "integer" }
            },
            "required": ["name", "priority"]
          }
        }
      }
    },
    "alert_routing_rules": {
      "type": "object",
      "properties": {
        "default_channel": { "type": "string" },
        "rules": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "severity_match": { "type": "string" },
              "target_team": { "type": "string" },
              "channel": { "type": "string" }
            }
          }
        }
      }
    },
    "voice_channel": {
      "type": "object",
      "properties": {
        "enabled": { "type": "boolean" },
        "provider": { "type": "string" },
        "trigger_severity": { "type": "array", "items": { "type": "string" } },
        "call_flow_id": { "type": "string" }
      }
    }
  }
}
```

### Sample Configuration
```json
{
  "severity_levels": {
    "definitions": [
      { "name": "critical", "priority": 1 },
      { "name": "major", "priority": 2 },
      { "name": "minor", "priority": 3 }
    ]
  },
  "alert_routing_rules": {
    "default_channel": "email",
    "rules": [
      {
        "severity_match": "critical",
        "target_team": "sre-oncall",
        "channel": "pagerduty"
      },
      {
        "severity_match": "major",
        "target_team": "dev-team",
        "channel": "slack"
      }
    ]
  },
  "voice_channel": {
    "enabled": true,
    "provider": "twilio",
    "trigger_severity": ["critical"],
    "call_flow_id": "flow_12345"
  }
}
```

---

## 3. De-noising (`de_noising`)
불필요한 알람을 줄이고, 의미 있는 알람만 전달하기 위한 설정입니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `alert_grouping_logic` | Object | 연관된 알람 그룹핑 설정입니다. |
| `alert_grouping_logic.group_by_fields` | Array | 그룹핑 기준 필드입니다 (예: ["source_ip", "error_code"]). |
| `alert_grouping_logic.grouping_window_seconds` | Integer | 그룹핑 시간 윈도우 (초)입니다. |
| `alert_grouping_logic.strategy` | String | 그룹핑 전략입니다 ("time_based", "content_based"). |
| `suppression_window` | Object | 중복 알람 억제 설정입니다. |
| `suppression_window.duration_seconds` | Integer | 억제 기간 (초)입니다. |
| `suppression_window.max_occurrences` | Integer | 기간 내 최대 허용 발생 횟수입니다 (초과 시 억제). |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "alert_grouping_logic": {
      "type": "object",
      "properties": {
        "group_by_fields": { "type": "array", "items": { "type": "string" } },
        "grouping_window_seconds": { "type": "integer" },
        "strategy": { "type": "string" }
      }
    },
    "suppression_window": {
      "type": "object",
      "properties": {
        "duration_seconds": { "type": "integer" },
        "max_occurrences": { "type": "integer" }
      }
    }
  }
}
```

### Sample Configuration
```json
{
  "alert_grouping_logic": {
    "group_by_fields": ["service_id", "error_code"],
    "grouping_window_seconds": 300,
    "strategy": "time_based"
  },
  "suppression_window": {
    "duration_seconds": 3600,
    "max_occurrences": 5
  }
}
```

---

## 4. Response Sync (`response_sync`)
알람 발생 후 대응 프로세스 및 외부 시스템과의 동기화 설정입니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `itsm_sync` | Object | ITSM 도구(Jira, ServiceNow 등) 연동 설정입니다. |
| `itsm_sync.system_name` | String | 연동할 ITSM 시스템 명입니다. |
| `itsm_sync.auto_ticket_creation` | Boolean | 티켓 자동 생성 여부입니다. |
| `itsm_sync.ticket_template_id` | String | 사용할 티켓 템플릿 ID입니다. |
| `chat_ops` | Object | 협업 툴(ChatOps) 연동 설정입니다. |
| `chat_ops.platform` | String | 플랫폼 명입니다 (예: "slack", "teams"). |
| `chat_ops.webhook_url` | String | 웹훅 URL입니다. |
| `chat_ops.interactive_buttons` | Boolean | 알람 메시지 내 버튼(Ack, Resolve 등) 포함 여부입니다. |
| `dlq_destination` | Object | 처리 실패 알람(Dead Letter) 전송 대상 설정입니다. |
| `dlq_destination.type` | String | 저장소 유형입니다 ("s3", "kafka", "sqs"). |
| `dlq_destination.target_resource` | String | 버킷명 또는 토픽명입니다. |
| `retention_policy` | Object | 데이터 보관 정책 설정입니다. |
| `retention_policy.log_retention_days` | Integer | 로그 보관 기간 (일)입니다. |
| `retention_policy.alert_history_retention_days` | Integer | 알람 이력 보관 기간 (일)입니다. |
| `retention_policy.archive_storage_enabled` | Boolean | 장기 보관소(Cold Storage) 사용 여부입니다. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "itsm_sync": {
      "type": "object",
      "properties": {
        "system_name": { "type": "string" },
        "auto_ticket_creation": { "type": "boolean" },
        "ticket_template_id": { "type": "string" }
      }
    },
    "chat_ops": {
      "type": "object",
      "properties": {
        "platform": { "type": "string" },
        "webhook_url": { "type": "string" },
        "interactive_buttons": { "type": "boolean" }
      }
    },
    "dlq_destination": {
      "type": "object",
      "properties": {
        "type": { "type": "string", "enum": ["s3", "kafka", "sqs"] },
        "target_resource": { "type": "string" }
      }
    },
    "retention_policy": {
      "type": "object",
      "properties": {
        "log_retention_days": { "type": "integer" },
        "alert_history_retention_days": { "type": "integer" },
        "archive_storage_enabled": { "type": "boolean" }
      }
    }
  }
}
```

### Sample Configuration
```json
{
  "itsm_sync": {
    "system_name": "jira",
    "auto_ticket_creation": true,
    "ticket_template_id": "tpl_incident_001"
  },
  "chat_ops": {
    "platform": "slack",
    "webhook_url": "https://hooks.slack.com/services/...",
    "interactive_buttons": true
  },
  "dlq_destination": {
    "type": "s3",
    "target_resource": "alert-dlq-bucket"
  },
  "retention_policy": {
    "log_retention_days": 90,
    "alert_history_retention_days": 365,
    "archive_storage_enabled": true
  }
}
```
