# Alerting & Incident Ops Configuration Manual

이 문서는 Alerting & Incident Ops 모듈의 설정 파일에 대한 상세 매뉴얼입니다.
본 설정은 iPaaS 환경에서 발생하는 알람 소스 등록, 중요도 설정, 노이즈 제거, 그리고 대응 동기화 기능을 정의합니다.

모든 설정 필드는 `snake_case`를 따르며, JSON 형식으로 작성됩니다.

---

## Configuration Structure

설정은 크게 4가지 주요 섹션으로 구분됩니다.

1.  **Alert Source Registry (`alert_source_registry`)**: 다양한 알람 소스 및 모니터링 대상을 정의합니다.
2.  **Severity (`severity`)**: 알람의 중요도 수준과 라우팅 규칙, 음성 채널 알림을 설정합니다.
3.  **De-noising (`de_noising`)**: 알람 홍수를 방지하기 위한 그룹핑 및 억제 로직을 설정합니다.
4.  **Response Sync (`response_sync`)**: ITSM 도구 연동, ChatOps, 실패 처리(DLQ), 데이터 보관 정책을 설정합니다.

---

## Detailed Configuration Fields

### 1. Alert Source Registry (`alert_source_registry`)

시스템 내외부에서 발생하는 다양한 알람 소스를 등록하고 임계값을 설정합니다.

*   **`sla`**: 서비스 수준 계약(SLA) 위반 감지 설정
    *   `enabled` (boolean): 모니터링 활성화 여부
    *   `threshold_ms` (integer): 응답 시간 임계값 (밀리초)
    *   `breach_action` (string): 위반 시 수행할 동작 (예: "alert", "log_only")
*   **`security_dlp`**: 보안 및 데이터 유출 방지(DLP) 관련 설정
    *   `enabled` (boolean): 보안 모니터링 활성화 여부
    *   `sensitivity_level` (string): 감지 민감도 ("high", "medium", "low")
    *   `target_protocols` (array): 모니터링 대상 프로토콜 (예: ["http", "ftp"])
*   **`infra_resource`**: 인프라 리소스 사용량 모니터링
    *   `cpu_threshold_percent` (integer): CPU 사용량 알람 임계값 (%)
    *   `memory_threshold_percent` (integer): 메모리 사용량 알람 임계값 (%)
    *   `disk_usage_threshold_percent` (integer): 디스크 사용량 알람 임계값 (%)
*   **`operation_asset`**: 운영 자산 상태 모니터링
    *   `monitor_active_assets` (boolean): 활성 자산 모니터링 여부
    *   `health_check_interval_seconds` (integer): 상태 확인 주기 (초)
*   **`business_cost`**: 비용 및 비즈니스 로직 관련 모니터링
    *   `daily_cost_limit_usd` (number): 일일 비용 한도 (USD)
    *   `alert_on_forecast_overrun` (boolean): 예상 비용 초과 시 알람 여부

### 2. Severity (`severity`)

발생한 알람의 중요도를 정의하고, 이에 따른 처리 규칙을 설정합니다.

*   **`severity_levels`**: 중요도 레벨 정의
    *   `definitions` (array): 레벨 객체 배열
        *   `name` (string): 레벨 이름 (예: "critical", "warning")
        *   `priority` (integer): 우선순위 (숫자가 낮을수록 높음)
*   **`alert_routing_rules`**: 알람 전파 규칙
    *   `default_channel` (string): 기본 알림 채널
    *   `rules` (array): 라우팅 규칙 배열
        *   `severity_match` (string): 매칭할 중요도 레벨
        *   `target_team` (string): 수신 팀
        *   `channel` (string): 알림 채널 (예: "slack", "email", "pagerduty")
*   **`voice_channel`**: 긴급 상황 시 음성 통화 알림 설정
    *   `enabled` (boolean): 음성 알림 활성화 여부
    *   `provider` (string): 통화 서비스 제공자 (예: "twilio", "aws_connect")
    *   `trigger_severity` (array): 음성 알림을 트리거할 중요도 레벨 목록
    *   `call_flow_id` (string): 실행할 콜 플로우 ID

### 3. De-noising (`de_noising`)

불필요한 알람을 줄이고, 의미 있는 알람만 전달하기 위한 설정입니다.

*   **`alert_grouping_logic`**: 연관된 알람 그룹핑 설정
    *   `group_by_fields` (array): 그룹핑 기준 필드 (예: ["source_ip", "error_code"])
    *   `grouping_window_seconds` (integer): 그룹핑 시간 윈도우 (초)
    *   `strategy` (string): 그룹핑 전략 ("time_based", "content_based")
*   **`suppression_window`**: 중복 알람 억제 설정
    *   `duration_seconds` (integer): 억제 기간 (초)
    *   `max_occurrences` (integer): 기간 내 최대 허용 발생 횟수 (초과 시 억제)

### 4. Response Sync (`response_sync`)

알람 발생 후 대응 프로세스 및 외부 시스템과의 동기화 설정입니다.

*   **`itsm_sync`**: ITSM 도구(Jira, ServiceNow 등) 연동
    *   `system_name` (string): 연동할 ITSM 시스템 명
    *   `auto_ticket_creation` (boolean): 티켓 자동 생성 여부
    *   `ticket_template_id` (string): 사용할 티켓 템플릿 ID
*   **`chat_ops`**: 협업 툴(ChatOps) 연동
    *   `platform` (string): 플랫폼 (예: "slack", "teams")
    *   `webhook_url` (string): 웹훅 URL
    *   `interactive_buttons` (boolean): 알람 메시지 내 버튼(Ack, Resolve 등) 포함 여부
*   **`dlq_destination`**: 처리 실패 알람(Dead Letter) 전송 대상
    *   `type` (string): 저장소 유형 ("s3", "kafka")
    *   `target_resource` (string): 버킷명 또는 토픽명
*   **`retention_policy`**: 데이터 보관 정책 (보관 정책)
    *   `log_retention_days` (integer): 로그 보관 기간 (일)
    *   `alert_history_retention_days` (integer): 알람 이력 보관 기간 (일)
    *   `archive_storage_enabled` (boolean): 장기 보관소(Cold Storage) 사용 여부

---

## JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "alert_source_registry": {
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
    },
    "severity": {
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
    },
    "de_noising": {
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
    },
    "response_sync": {
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
  },
  "required": ["alert_source_registry", "severity", "de_noising", "response_sync"]
}
```

## Configuration Sample

```json
{
  "alert_source_registry": {
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
  },
  "severity": {
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
  },
  "de_noising": {
    "alert_grouping_logic": {
      "group_by_fields": ["service_id", "error_code"],
      "grouping_window_seconds": 300,
      "strategy": "time_based"
    },
    "suppression_window": {
      "duration_seconds": 3600,
      "max_occurrences": 5
    }
  },
  "response_sync": {
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
}
```
