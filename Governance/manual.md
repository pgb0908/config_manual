# iPaaS Governance 설정 매뉴얼

이 매뉴얼은 iPaaS 플랫폼의 거버넌스(Governance) 설정을 위한 가이드입니다. 각 설정은 기능별로 분류되어 있으며, 상세한 Configuration 필드 설명, JSON 스키마(Schema), 그리고 샘플 설정(Sample)을 포함하고 있습니다.

모든 설정 필드는 `snake_case`를 따르며, 설명은 한글로 작성되었습니다.

---

## 1. 조직 계층 (Org Hierarchy)

조직의 기본 식별 정보와 세션 정책, 상위 정책 상속 여부를 정의합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `org_name` | String | Yes | 조직의 공식 명칭입니다. |
| `org_domain` | String | Yes | 조직이 사용하는 기본 도메인입니다. |
| `session_timeout` | Integer | No | 사용자 세션 만료 시간(분 단위)입니다. (기본값: 60) |
| `inherit_parent_policy` | Boolean | No | 상위 조직의 거버넌스 정책을 상속받을지 여부입니다. |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "org_name": { "type": "string" },
    "org_domain": { "type": "string", "format": "hostname" },
    "session_timeout": { "type": "integer", "minimum": 5, "default": 60 },
    "inherit_parent_policy": { "type": "boolean", "default": false }
  },
  "required": ["org_name", "org_domain"]
}
```

### Sample Configuration

```json
{
  "org_name": "Tech Corp Global",
  "org_domain": "techcorp.com",
  "session_timeout": 30,
  "inherit_parent_policy": true
}
```

---

## 2. 사용자 관리 (User Management)

사용자 인증 소스, 프로비저닝 방식, MFA(다중 인증) 정책을 설정합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `auth_origin` | String | Yes | 인증 원천을 지정합니다. (예: `local`, `ldap`, `saml`, `oidc`) |
| `jit_provisioning` | Object | No | Just-In-Time 사용자 프로비저닝 설정입니다. |
| `jit_provisioning.enabled` | Boolean | Yes | JIT 활성화 여부입니다. |
| `jit_provisioning.default_group` | String | No | 신규 사용자가 할당될 기본 그룹입니다. |
| `mfa_policy` | Object | No | 다중 인증(MFA) 정책 설정입니다. |
| `mfa_policy.enforce_all` | Boolean | Yes | 모든 사용자에게 MFA를 강제할지 여부입니다. |
| `mfa_policy.allowed_methods` | Array | No | 허용된 인증 수단 목록입니다. (예: `sms`, `totp`, `email`) |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "auth_origin": { "type": "string", "enum": ["local", "ldap", "saml", "oidc"] },
    "jit_provisioning": {
      "type": "object",
      "properties": {
        "enabled": { "type": "boolean" },
        "default_group": { "type": "string" }
      },
      "required": ["enabled"]
    },
    "mfa_policy": {
      "type": "object",
      "properties": {
        "enforce_all": { "type": "boolean" },
        "allowed_methods": {
          "type": "array",
          "items": { "type": "string", "enum": ["sms", "totp", "email", "biometric"] }
        }
      },
      "required": ["enforce_all"]
    }
  },
  "required": ["auth_origin"]
}
```

### Sample Configuration

```json
{
  "auth_origin": "saml",
  "jit_provisioning": {
    "enabled": true,
    "default_group": "developers"
  },
  "mfa_policy": {
    "enforce_all": true,
    "allowed_methods": ["totp", "sms"]
  }
}
```

---

## 3. 역할 기반 접근 제어 (RBAC)

사용자 역할(Role) 정의와 외부 그룹(Team) 간의 매핑을 관리합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `role_definitions` | Array | Yes | 사용자 정의 역할 목록입니다. |
| `role_definitions[].role_name` | String | Yes | 역할 이름입니다. |
| `role_definitions[].permissions` | Array | Yes | 해당 역할이 가지는 권한 목록입니다. |
| `team_mapping` | Array | No | 외부 IDP 그룹과 내부 역할 간의 매핑입니다. |
| `team_mapping[].idp_group` | String | Yes | 외부 IDP(Identity Provider)의 그룹명입니다. |
| `team_mapping[].assigned_roles` | Array | Yes | 할당할 역할 이름 목록입니다. |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "role_definitions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "role_name": { "type": "string" },
          "permissions": { "type": "array", "items": { "type": "string" } }
        },
        "required": ["role_name", "permissions"]
      }
    },
    "team_mapping": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "idp_group": { "type": "string" },
          "assigned_roles": { "type": "array", "items": { "type": "string" } }
        },
        "required": ["idp_group", "assigned_roles"]
      }
    }
  },
  "required": ["role_definitions"]
}
```

### Sample Configuration

```json
{
  "role_definitions": [
    {
      "role_name": "FlowAdmin",
      "permissions": ["flow.create", "flow.delete", "flow.publish"]
    },
    {
      "role_name": "Viewer",
      "permissions": ["flow.read", "logs.read"]
    }
  ],
  "team_mapping": [
    {
      "idp_group": "DevOps-Team",
      "assigned_roles": ["FlowAdmin"]
    }
  ]
}
```

---

## 4. 보안 및 개인정보 (Security & Privacy)

시크릿 관리, 데이터 마스킹, BYOK(Bring Your Own Key) 등 데이터 보안 관련 설정을 정의합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `secret_management` | Object | Yes | 시크릿(Credential) 관리 설정입니다. |
| `secret_management.provider` | String | Yes | 시크릿 저장소 제공자입니다. (예: `vault`, `aws_secrets_manager`) |
| `secret_management.rotation_days` | Integer | No | 시크릿 자동 교체 주기(일)입니다. |
| `data_masking` | Array | No | 로그 및 모니터링 시 마스킹할 데이터 패턴입니다. |
| `data_masking[].field_pattern` | String | Yes | 마스킹할 필드명 패턴(Regex)입니다. |
| `data_masking[].mask_char` | String | No | 마스킹에 사용할 문자입니다. (기본값: `*`) |
| `byok_encryption` | Object | No | 고객 관리 키(BYOK) 암호화 설정입니다. |
| `byok_encryption.enabled` | Boolean | Yes | BYOK 활성화 여부입니다. |
| `byok_encryption.key_arn` | String | No | 암호화 키의 식별자(ARN 등)입니다. |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "secret_management": {
      "type": "object",
      "properties": {
        "provider": { "type": "string", "enum": ["vault", "aws_secrets_manager", "azure_key_vault"] },
        "rotation_days": { "type": "integer" }
      },
      "required": ["provider"]
    },
    "data_masking": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "field_pattern": { "type": "string" },
          "mask_char": { "type": "string", "maxLength": 1 }
        },
        "required": ["field_pattern"]
      }
    },
    "byok_encryption": {
      "type": "object",
      "properties": {
        "enabled": { "type": "boolean" },
        "key_arn": { "type": "string" }
      },
      "required": ["enabled"]
    }
  },
  "required": ["secret_management"]
}
```

### Sample Configuration

```json
{
  "secret_management": {
    "provider": "vault",
    "rotation_days": 90
  },
  "data_masking": [
    {
      "field_pattern": "password|ssn|credit_card",
      "mask_char": "*"
    }
  ],
  "byok_encryption": {
    "enabled": true,
    "key_arn": "arn:aws:kms:us-east-1:123456789012:key/example-key"
  }
}
```

---

## 5. 설계 가이드라인 (Design Guardrails)

리소스 명명 규칙, 플로우 복잡도 제한, 보안 스캔 규칙 등을 강제하여 품질을 유지합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `naming_conventions` | Object | No | 리소스별 명명 규칙(Regex)입니다. |
| `max_flow_depth` | Integer | No | 통합 플로우(Flow)의 최대 허용 깊이(Depth)입니다. |
| `security_scan_rules` | Array | No | 배포 전 필수 보안 스캔 규칙 목록입니다. |
| `deployment_gate` | Object | Yes | 배포 승인 게이트 설정입니다. |
| `deployment_gate.require_peer_review` | Boolean | Yes | 동료 리뷰 필수 여부입니다. |
| `deployment_gate.block_on_high_severity` | Boolean | Yes | 심각한 보안 취약점 발견 시 배포 차단 여부입니다. |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "naming_conventions": {
      "type": "object",
      "additionalProperties": { "type": "string" },
      "description": "Key is resource type, Value is regex pattern"
    },
    "max_flow_depth": { "type": "integer" },
    "security_scan_rules": {
      "type": "array",
      "items": { "type": "string" }
    },
    "deployment_gate": {
      "type": "object",
      "properties": {
        "require_peer_review": { "type": "boolean" },
        "block_on_high_severity": { "type": "boolean" }
      },
      "required": ["require_peer_review", "block_on_high_severity"]
    }
  },
  "required": ["deployment_gate"]
}
```

### Sample Configuration

```json
{
  "naming_conventions": {
    "flow": "^flow-[a-z0-9-]+$",
    "api": "^api-[a-z0-9-]+$"
  },
  "max_flow_depth": 10,
  "security_scan_rules": ["check-sql-injection", "check-hardcoded-secrets"],
  "deployment_gate": {
    "require_peer_review": true,
    "block_on_high_severity": true
  }
}
```

---

## 6. 자산 템플릿 (Asset Templates)

표준화된 개발을 위한 템플릿 제공 및 배포 허용 범위를 설정합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `scaffolding_templates` | Array | No | 프로젝트 생성 시 사용할 수 있는 스캐폴딩 템플릿 목록입니다. |
| `canonical_models` | Array | No | 전사 표준 데이터 모델(Canonical Model) 정의 목록입니다. |
| `deployment_scope_settings` | Object | Yes | 자산이 배포될 수 있는 환경 및 리전 설정입니다. |
| `deployment_scope_settings.allowed_environments` | Array | Yes | 배포 허용 환경 목록입니다. (예: `dev`, `stage`, `prod`) |
| `deployment_scope_settings.restricted_regions` | Array | No | 배포가 제한된 리전 목록입니다. |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "scaffolding_templates": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "repo_url": { "type": "string" }
        }
      }
    },
    "canonical_models": {
      "type": "array",
      "items": { "type": "string" }
    },
    "deployment_scope_settings": {
      "type": "object",
      "properties": {
        "allowed_environments": {
          "type": "array",
          "items": { "type": "string" }
        },
        "restricted_regions": {
          "type": "array",
          "items": { "type": "string" }
        }
      },
      "required": ["allowed_environments"]
    }
  },
  "required": ["deployment_scope_settings"]
}
```

### Sample Configuration

```json
{
  "scaffolding_templates": [
    { "name": "REST API Service", "repo_url": "git://templates/rest-api" },
    { "name": "Event Consumer", "repo_url": "git://templates/event-consumer" }
  ],
  "canonical_models": ["Customer", "Order", "Product"],
  "deployment_scope_settings": {
    "allowed_environments": ["dev", "qa", "prod"],
    "restricted_regions": ["ap-northeast-3"]
  }
}
```

---

## 7. AI 관리 (AI Management)

iPaaS 내 AI 서비스 활용 시의 모델 제약, 사용량 제한 및 데이터 보호 정책을 설정합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `allowed_models` | Array | Yes | 사용 가능한 AI 모델 목록입니다. (예: `gpt-4`, `claude-3`) |
| `usage_limits` | Object | Yes | 사용량 제한 설정입니다. |
| `usage_limits.max_tokens_per_request` | Integer | No | 요청당 최대 토큰 수입니다. |
| `usage_limits.monthly_budget` | Number | No | 월간 AI 사용 예산($)입니다. |
| `data_privacy` | Object | Yes | AI 데이터 프라이버시 설정입니다. |
| `data_privacy.filter_sensitive_data` | Boolean | Yes | 프롬프트 전송 전 민감 데이터 필터링 여부입니다. |
| `data_privacy.log_prompts` | Boolean | Yes | 프롬프트 내용 로깅 여부입니다. (보안상 False 권장) |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "allowed_models": {
      "type": "array",
      "items": { "type": "string" }
    },
    "usage_limits": {
      "type": "object",
      "properties": {
        "max_tokens_per_request": { "type": "integer" },
        "monthly_budget": { "type": "number" }
      }
    },
    "data_privacy": {
      "type": "object",
      "properties": {
        "filter_sensitive_data": { "type": "boolean" },
        "log_prompts": { "type": "boolean" }
      },
      "required": ["filter_sensitive_data", "log_prompts"]
    }
  },
  "required": ["allowed_models", "data_privacy"]
}
```

### Sample Configuration

```json
{
  "allowed_models": ["gpt-4o", "claude-3-5-sonnet"],
  "usage_limits": {
    "max_tokens_per_request": 8000,
    "monthly_budget": 500.00
  },
  "data_privacy": {
    "filter_sensitive_data": true,
    "log_prompts": false
  }
}
```

---

## 8. 변경 승인 (Change Approval)

변경 사항 적용 시의 승인 프로세스, 트리거 조건 및 위반 시 처리 방식을 정의합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `approval_flow_id` | String | Yes | 변경 승인을 처리할 워크플로우 또는 시스템 ID입니다. |
| `validation_triggers` | Array | Yes | 승인 프로세스가 트리거되는 이벤트 목록입니다. (예: `deploy_prod`, `delete_resource`) |
| `policy_definitions` | Object | No | 상세 승인 정책 정의입니다. |
| `policy_definitions.auto_approve_minor` | Boolean | No | 마이너 변경 사항 자동 승인 여부입니다. |
| `violation_behavior` | String | Yes | 정책 위반 시 동작입니다. (`block`, `warn` 중 하나) |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "approval_flow_id": { "type": "string" },
    "validation_triggers": {
      "type": "array",
      "items": { "type": "string" }
    },
    "policy_definitions": {
      "type": "object",
      "properties": {
        "auto_approve_minor": { "type": "boolean" }
      }
    },
    "violation_behavior": { "type": "string", "enum": ["block", "warn"] }
  },
  "required": ["approval_flow_id", "validation_triggers", "violation_behavior"]
}
```

### Sample Configuration

```json
{
  "approval_flow_id": "flow-approval-process-v1",
  "validation_triggers": ["deploy_production", "modify_security_policy"],
  "policy_definitions": {
    "auto_approve_minor": true
  },
  "violation_behavior": "block"
}
```
