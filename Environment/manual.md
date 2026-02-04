# Environment Configuration Manual

본 매뉴얼은 iPaaS의 Environment(환경) 모듈에 적용되는 설정(Configuration)에 대한 상세 가이드입니다. 각 기능 그룹별로 설정 필드에 대한 설명(한국어)과 JSON 스키마, 그리고 적용 예시를 제공합니다.

설정 필드명은 `snake_case` 규칙을 따릅니다.

---

## 목차
1. [Environment Properties](#1-environment-properties)
2. [Secure Properties](#2-secure-properties)
3. [Project Properties](#3-project-properties)

---

## 1. Environment Properties
개발, 스테이징, 운영 등 각 배포 환경별로 달라지는 속성을 정의합니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `env_name` | String | 환경의 식별자 이름입니다 (예: `dev`, `staging`, `prod`). |
| `base_url` | String | 해당 환경에서 서비스들이 사용하는 기본 URL입니다. |
| `debug_mode` | Boolean | 디버깅 모드 활성화 여부입니다. 활성화 시 상세 로그가 출력됩니다. |
| `variables` | Object | 해당 환경에서만 유효한 일반 변수들의 키-값 쌍입니다. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "env_name": { "type": "string", "enum": ["dev", "staging", "prod"] },
    "base_url": { "type": "string", "format": "uri" },
    "debug_mode": { "type": "boolean" },
    "variables": {
      "type": "object",
      "additionalProperties": { "type": "string" }
    }
  },
  "required": ["env_name", "base_url"]
}
```

### Sample Configuration
```json
{
  "env_name": "staging",
  "base_url": "https://api-staging.example.com",
  "debug_mode": true,
  "variables": {
    "db_host": "10.0.5.20",
    "feature_flag_new_ui": "true"
  }
}
```

---

## 2. Secure Properties
비밀번호, API 키, 인증서 등 민감한 정보를 안전하게 관리하기 위한 설정입니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `encryption_config` | Object | 민감 정보 암호화를 위한 설정입니다. |
| `encryption_config.kms_provider` | String | 키 관리 서비스(KMS) 제공자입니다 (예: `aws_kms`, `vault`, `google_kms`). |
| `encryption_config.key_id` | String | 암호화/복호화에 사용할 키 식별자입니다. |
| `vault_integration` | Object | HashiCorp Vault와 같은 외부 보안 저장소 연동 설정입니다. |
| `vault_integration.enabled` | Boolean | Vault 연동 활성화 여부입니다. |
| `vault_integration.address` | String | Vault 서버 주소입니다. |
| `vault_integration.auth_path` | String | 인증 경로입니다 (예: `kubernetes`, `approle`). |
| `secret_variables` | Object | 값 자체가 암호화되어 저장되거나, 런타임에 보안 저장소에서 주입받는 변수들입니다. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "encryption_config": {
      "type": "object",
      "properties": {
        "kms_provider": { "type": "string", "enum": ["aws_kms", "vault", "google_kms"] },
        "key_id": { "type": "string" }
      }
    },
    "vault_integration": {
      "type": "object",
      "properties": {
        "enabled": { "type": "boolean" },
        "address": { "type": "string", "format": "uri" },
        "auth_path": { "type": "string" }
      }
    },
    "secret_variables": {
      "type": "object",
      "additionalProperties": { "type": "string" }
    }
  }
}
```

### Sample Configuration
```json
{
  "encryption_config": {
    "kms_provider": "aws_kms",
    "key_id": "arn:aws:kms:us-east-1:123456789012:key/mrk-1234abcd"
  },
  "vault_integration": {
    "enabled": true,
    "address": "https://vault.internal:8200",
    "auth_path": "kubernetes"
  },
  "secret_variables": {
    "db_password": "vault:secret/data/db/password",
    "api_key": "enc:Base64EncodedCiphertext..."
  }
}
```

---

## 3. Project Properties
모든 환경과 서비스에 공통적으로 적용되는 전역 상수 및 정책을 정의합니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `common_headers` | Object | 모든 아웃바운드 요청에 기본적으로 포함될 HTTP 헤더입니다. |
| `global_timeout` | Integer | 모든 API 호출에 대한 기본 타임아웃 시간(밀리초)입니다. |
| `retry_policy` | Object | 실패한 요청에 대한 기본 재시도 정책입니다. |
| `retry_policy.max_attempts` | Integer | 최대 재시도 횟수입니다. |
| `retry_policy.initial_interval` | Integer | 첫 재시도 전 대기 시간(밀리초)입니다. |
| `retry_policy.multiplier` | Number | 재시도 간격 증가 배수(Backoff multiplier)입니다. |
| `localization` | Object | 날짜, 시간, 언어 등 지역화 관련 기본 설정입니다. |
| `localization.timezone` | String | 기본 타임존입니다 (예: `Asia/Seoul`). |
| `localization.locale` | String | 기본 로케일입니다 (예: `ko_KR`). |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "common_headers": {
      "type": "object",
      "additionalProperties": { "type": "string" }
    },
    "global_timeout": { "type": "integer", "minimum": 0 },
    "retry_policy": {
      "type": "object",
      "properties": {
        "max_attempts": { "type": "integer" },
        "initial_interval": { "type": "integer" },
        "multiplier": { "type": "number" }
      }
    },
    "localization": {
      "type": "object",
      "properties": {
        "timezone": { "type": "string" },
        "locale": { "type": "string" }
      }
    }
  }
}
```

### Sample Configuration
```json
{
  "common_headers": {
    "X-Project-ID": "prj-alpha-2024",
    "User-Agent": "iPaaS-Runtime/1.0"
  },
  "global_timeout": 5000,
  "retry_policy": {
    "max_attempts": 3,
    "initial_interval": 1000,
    "multiplier": 1.5
  },
  "localization": {
    "timezone": "Asia/Seoul",
    "locale": "ko_KR"
  }
}
```
