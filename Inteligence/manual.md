# Intelligence Configuration

Intelligence 모듈은 iPaaS 내에서 AI 에이전트, 모델 거버넌스, 자동화된 운영 등을 관리하는 설정입니다. 모든 설정은 `intelligence`라는 최상위 객체 아래에 구성됩니다.

## 1. MCP Server & Tool Catalog (`mcp_server`)

Model Context Protocol (MCP) 서버 및 도구 카탈로그에 대한 설정입니다. 외부 도구를 에이전트가 사용할 수 있도록 등록하고 내보내는 기능을 제어합니다.

| 필드명 | 타입 | 설명 |
| :--- | :--- | :--- |
| `mcp_export_enabled` | Boolean | MCP 서버 내보내기 기능 활성화 여부입니다. `true`일 경우 외부에서 이 서버의 도구를 사용할 수 있습니다. |
| `tool_description` | String | 도구에 대한 설명입니다. LLM이 도구의 용도를 이해하는 데 사용됩니다. |
| `input_output_schema` | Object | 도구의 입력 및 출력 데이터 구조를 정의하는 JSON 스키마입니다. |
| `tool_registry_scope` | String | 도구가 등록될 범위입니다. (`GLOBAL`, `PROJECT`, `PRIVATE`) |

#### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "mcp_export_enabled": { "type": "boolean" },
    "tool_description": { "type": "string" },
    "input_output_schema": { "type": "object" },
    "tool_registry_scope": {
      "type": "string",
      "enum": ["GLOBAL", "PROJECT", "PRIVATE"]
    }
  }
}
```

#### Sample
```json
{
  "mcp_export_enabled": true,
  "tool_description": "고객 데이터 조회 및 업데이트를 위한 도구 모음",
  "input_output_schema": {
    "input": { "type": "object", "properties": { "user_id": { "type": "string" } } },
    "output": { "type": "object", "properties": { "user_data": { "type": "object" } } }
  },
  "tool_registry_scope": "PROJECT"
}
```

---

## 2. Agent Control Plane (`agent_control_plane`)

AI 에이전트의 실행 범위, 안전 장치(Safe-guard), 오케스트레이션 및 로깅을 관리하는 제어 평면입니다.

| 필드명 | 타입 | 설명 |
| :--- | :--- | :--- |
| `scope.agent_scopes` | Array | 에이전트가 접근 가능한 리소스 또는 작업의 범위 목록입니다. |
| `safe_guard.idempotency_mode` | String | 멱등성 보장 모드입니다. (`AT_LEAST_ONCE`, `EXACTLY_ONCE`) |
| `safe_guard.dry_run_policy.enabled` | Boolean | 드라이 런(가상 실행) 모드 활성화 여부입니다. |
| `safe_guard.dry_run_policy.notification_channel` | String | 드라이 런 결과가 전송될 알림 채널입니다. |
| `safe_guard.approval_threshold` | Number | 에이전트 실행 승인을 위한 신뢰도 임계값(0.0 ~ 1.0)입니다. |
| `orchestration.plan_execute_verify` | Boolean | '계획 -> 실행 -> 검증' 패턴의 오케스트레이션 활성화 여부입니다. |
| `log.level` | String | 로그 레벨입니다. (`DEBUG`, `INFO`, `WARN`, `ERROR`) |
| `log.destination` | String | 로그가 저장될 목적지입니다. (예: `S3`, `ELASTICSEARCH`) |
| `log.retention_days` | Integer | 로그 보관 기간(일)입니다. |

#### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "scope": {
      "type": "object",
      "properties": {
        "agent_scopes": { "type": "array", "items": { "type": "string" } }
      }
    },
    "safe_guard": {
      "type": "object",
      "properties": {
        "idempotency_mode": { "type": "string", "enum": ["AT_LEAST_ONCE", "EXACTLY_ONCE"] },
        "dry_run_policy": {
          "type": "object",
          "properties": {
            "enabled": { "type": "boolean" },
            "notification_channel": { "type": "string" }
          }
        },
        "approval_threshold": { "type": "number", "minimum": 0, "maximum": 1 }
      }
    },
    "orchestration": {
      "type": "object",
      "properties": {
        "plan_execute_verify": { "type": "boolean" }
      }
    },
    "log": {
      "type": "object",
      "properties": {
        "level": { "type": "string", "enum": ["DEBUG", "INFO", "WARN", "ERROR"] },
        "destination": { "type": "string" },
        "retention_days": { "type": "integer" }
      }
    }
  }
}
```

#### Sample
```json
{
  "scope": {
    "agent_scopes": ["read_database", "call_external_api"]
  },
  "safe_guard": {
    "idempotency_mode": "EXACTLY_ONCE",
    "dry_run_policy": {
      "enabled": true,
      "notification_channel": "slack-ops-channel"
    },
    "approval_threshold": 0.85
  },
  "orchestration": {
    "plan_execute_verify": true
  },
  "log": {
    "level": "INFO",
    "destination": "ELASTICSEARCH",
    "retention_days": 30
  }
}
```

---

## 3. AI-driven SDLC Assist (`ai_driven_sdlc_assist`)

소프트웨어 개발 생명주기(SDLC)를 보조하는 AI 기능 설정입니다. 코드 초안 생성, 테스트 생성, 배포 점검 등을 포함합니다.

| 필드명 | 타입 | 설명 |
| :--- | :--- | :--- |
| `draft_gen_mode` | String | 코드 초안 생성 모드입니다. (`AUTO` - 자동 생성, `MANUAL` - 수동 요청 시 생성) |
| `test_gen_mode` | String | 테스트 코드 생성 범위입니다. (`UNIT`, `INTEGRATION`, `E2E`) |
| `check_deploy` | Boolean | 배포 전 AI 기반 코드 및 설정 점검 활성화 여부입니다. |

#### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "draft_gen_mode": { "type": "string", "enum": ["AUTO", "MANUAL"] },
    "test_gen_mode": { "type": "string", "enum": ["UNIT", "INTEGRATION", "E2E"] },
    "check_deploy": { "type": "boolean" }
  }
}
```

#### Sample
```json
{
  "draft_gen_mode": "MANUAL",
  "test_gen_mode": "UNIT",
  "check_deploy": true
}
```

---

## 4. AI Layer Governance (`ai_layer_governance`)

AI 모델 사용, 데이터 보호, 비용 및 성능 평가를 위한 거버넌스 설정입니다.

| 필드명 | 타입 | 설명 |
| :--- | :--- | :--- |
| `model_registry.model_provider_groups` | Array | 사용할 AI 모델 제공자 그룹 목록입니다. |
| `model_registry.fallback_routing.enabled` | Boolean | 주 모델 실패 시 예비 모델로 라우팅할지 여부입니다. |
| `model_registry.fallback_routing.strategy` | String | 라우팅 전략입니다. (`LATENCY`, `COST`, `AVAILABILITY`) |
| `dlp.pii_masking.enabled` | Boolean | 개인정보(PII) 마스킹 기능 활성화 여부입니다. |
| `dlp.prompt_check.enabled` | Boolean | 프롬프트 내 금지어 또는 보안 위반 내용 검사 활성화 여부입니다. |
| `showback_budget.limit` | Number | AI 사용 비용 한도입니다. |
| `showback_budget.alert_threshold` | Integer | 예산의 몇 퍼센트 도달 시 알림을 보낼지 설정합니다. |
| `eval.metrics` | Array | 모델 평가 지표 목록입니다. (`ACCURACY`, `HALLUCINATION`, `LATENCY` 등) |
| `eval.sample_rate` | Number | 전체 요청 중 평가를 위해 샘플링할 비율(0.0 ~ 1.0)입니다. |

#### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "model_registry": {
      "type": "object",
      "properties": {
        "model_provider_groups": { "type": "array", "items": { "type": "string" } },
        "fallback_routing": {
          "type": "object",
          "properties": {
            "enabled": { "type": "boolean" },
            "strategy": { "type": "string", "enum": ["LATENCY", "COST", "AVAILABILITY"] }
          }
        }
      }
    },
    "dlp": {
      "type": "object",
      "properties": {
        "pii_masking": {
          "type": "object",
          "properties": { "enabled": { "type": "boolean" } }
        },
        "prompt_check": {
          "type": "object",
          "properties": { "enabled": { "type": "boolean" } }
        }
      }
    },
    "showback_budget": {
      "type": "object",
      "properties": {
        "limit": { "type": "number" },
        "alert_threshold": { "type": "integer" }
      }
    },
    "eval": {
      "type": "object",
      "properties": {
        "metrics": { "type": "array", "items": { "type": "string" } },
        "sample_rate": { "type": "number" }
      }
    }
  }
}
```

#### Sample
```json
{
  "model_registry": {
    "model_provider_groups": ["openai-gpt-4", "anthropic-claude-3"],
    "fallback_routing": {
      "enabled": true,
      "strategy": "AVAILABILITY"
    }
  },
  "dlp": {
    "pii_masking": { "enabled": true },
    "prompt_check": { "enabled": true }
  },
  "showback_budget": {
    "limit": 1000.00,
    "alert_threshold": 80
  },
  "eval": {
    "metrics": ["ACCURACY", "LATENCY"],
    "sample_rate": 0.1
  }
}
```

---

## 5. Autonomous Operations (`autonomous_operations`)

AI를 활용한 자율 운영 기능 설정입니다. 자동화된 근본 원인 분석(RCA), 런북 실행 등을 포함합니다.

| 필드명 | 타입 | 설명 |
| :--- | :--- | :--- |
| `automated_rca.enabled` | Boolean | 장애 발생 시 자동 근본 원인 분석 활성화 여부입니다. |
| `runbook_ops.auto_execute` | Boolean | 식별된 문제에 대해 런북을 자동으로 실행할지 여부입니다. |
| `hitl.required_severity` | String | 사람의 개입(Human-In-The-Loop)이 필요한 장애 심각도 레벨입니다. |

#### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "automated_rca": {
      "type": "object",
      "properties": {
        "enabled": { "type": "boolean" }
      }
    },
    "runbook_ops": {
      "type": "object",
      "properties": {
        "auto_execute": { "type": "boolean" }
      }
    },
    "hitl": {
      "type": "object",
      "properties": {
        "required_severity": { "type": "string", "enum": ["HIGH", "CRITICAL"] }
      }
    }
  }
}
```

#### Sample
```json
{
  "automated_rca": {
    "enabled": true
  },
  "runbook_ops": {
    "auto_execute": false
  },
  "hitl": {
    "required_severity": "CRITICAL"
  }
}
```

---

## 6. Legacy MCP Integration (`legacy_mcp_integration`)

기존 레거시 시스템과의 MCP 통합 설정입니다. 어댑터 모드 및 메타데이터 수집을 관리합니다.

| 필드명 | 타입 | 설명 |
| :--- | :--- | :--- |
| `adapter_type_setting.mode` | String | 어댑터 작동 모드입니다. (`PROXY`, `BRIDGE`, `AGENT`) |
| `adapter_type_setting.config` | Object | 선택된 모드에 대한 추가 설정입니다. |
| `metadata_collector.auto_harvest_enabled` | Boolean | 레거시 시스템의 메타데이터 자동 수집 활성화 여부입니다. |

#### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "adapter_type_setting": {
      "type": "object",
      "properties": {
        "mode": { "type": "string", "enum": ["PROXY", "BRIDGE", "AGENT"] },
        "config": { "type": "object" }
      }
    },
    "metadata_collector": {
      "type": "object",
      "properties": {
        "auto_harvest_enabled": { "type": "boolean" }
      }
    }
  }
}
```

#### Sample
```json
{
  "adapter_type_setting": {
    "mode": "PROXY",
    "config": {
      "target_url": "http://legacy-system.internal:8080"
    }
  },
  "metadata_collector": {
    "auto_harvest_enabled": true
  }
}
```
