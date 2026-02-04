# iPaaS Connector Configuration Manual

본 문서는 iPaaS 환경에서 사용되는 Connector의 Connectivity Configuration에 대한 상세 매뉴얼입니다.
각 설정 항목은 기능별로 구분되어 있으며, JSON 스키마, 항목별 설명, 그리고 샘플 구성을 포함합니다.

---

## 1. Authentication (인증 관리)

커넥터가 외부 시스템과 통신하기 위해 필요한 인증 정보를 설정합니다.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "authentication": {
      "type": "object",
      "properties": {
        "auth_type": {
          "type": "string",
          "enum": ["oauth2", "api_key", "basic", "bearer"],
          "description": "사용할 인증 유형을 지정합니다."
        },
        "client_id": {
          "type": "string",
          "description": "OAuth2 등의 인증에서 사용되는 클라이언트 식별자입니다."
        },
        "access_token_url": {
          "type": "string",
          "format": "uri",
          "description": "액세스 토큰을 발급받기 위한 URL입니다."
        },
        "refresh_strategy": {
          "type": "object",
          "description": "토큰 만료 시 갱신 전략을 정의합니다.",
          "properties": {
            "method": { "type": "string" },
            "refresh_token_url": { "type": "string" }
          }
        },
        "scopes": {
          "type": "array",
          "items": { "type": "string" },
          "description": "요청할 권한 범위를 목록으로 지정합니다."
        }
      },
      "required": ["auth_type"]
    }
  }
}
```

### 항목 설명

| 필드명 (Field) | 데이터 타입 (Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `auth_type` | String | 인증 방식을 선택합니다. (예: `oauth2`, `api_key`, `basic`) |
| `client_id` | String | 외부 시스템에서 발급받은 클라이언트 ID입니다. |
| `access_token_url` | String | 토큰 발급 엔드포인트 URL입니다. OAuth2 흐름에서 주로 사용됩니다. |
| `refresh_strategy` | Object | 토큰 만료 시 자동 갱신을 위한 설정입니다. 갱신 방식 및 URL 등을 포함할 수 있습니다. |
| `scopes` | Array (String) | 인증 시 요청할 권한(Scope)의 목록입니다. |

### Sample Configuration

```json
{
  "authentication": {
    "auth_type": "oauth2",
    "client_id": "client_12345",
    "access_token_url": "https://api.example.com/oauth/token",
    "refresh_strategy": {
      "method": "refresh_token",
      "refresh_token_url": "https://api.example.com/oauth/refresh"
    },
    "scopes": ["read_user", "write_order"]
  }
}
```

---

## 2. Endpoint & Protocol (엔드포인트 및 프로토콜)

연결할 대상 시스템의 호스트 정보 및 통신 관련 설정을 관리합니다.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "endpoint_config": {
      "type": "object",
      "properties": {
        "host": {
          "type": "string",
          "description": "연결할 대상 시스템의 기본 호스트 주소입니다."
        },
        "timeout": {
          "type": "integer",
          "description": "요청 타임아웃 시간(밀리초)입니다."
        },
        "proxy": {
          "type": "object",
          "description": "프록시 서버 사용 시 설정 정보입니다.",
          "properties": {
            "enabled": { "type": "boolean" },
            "url": { "type": "string" },
            "port": { "type": "integer" }
          }
        }
      },
      "required": ["host"]
    }
  }
}
```

### 항목 설명

| 필드명 (Field) | 데이터 타입 (Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `host` | String | API 요청을 보낼 기본 호스트 URL (Base URL)입니다. |
| `timeout` | Integer | 응답을 기다리는 최대 시간(ms)입니다. 이 시간이 지나면 요청을 중단합니다. |
| `proxy` | Object | 네트워크 환경에 따라 프록시 서버를 경유해야 할 경우 설정합니다. |

### Sample Configuration

```json
{
  "endpoint_config": {
    "host": "https://api.example.com/v1",
    "timeout": 5000,
    "proxy": {
      "enabled": true,
      "url": "http://proxy.internal.com",
      "port": 8080
    }
  }
}
```

---

## 3. Connector SDK & Extensibility (SDK 및 확장성)

커넥터가 사용하는 SDK 버전 및 확장 기능에 대한 설정입니다.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "sdk_config": {
      "type": "object",
      "properties": {
        "sdk_version": {
          "type": "string",
          "description": "사용 중인 커넥터 SDK의 버전입니다."
        },
        "extension_type": {
          "type": "string",
          "description": "커넥터의 확장 유형을 정의합니다 (예: standard, custom)."
        },
        "dependencies": {
          "type": "array",
          "items": { "type": "string" },
          "description": "커넥터 실행에 필요한 추가 라이브러리나 의존성 목록입니다."
        },
        "handler_mapping": {
          "type": "object",
          "description": "특정 이벤트나 요청을 처리할 핸들러 매핑 정보입니다."
        }
      }
    }
  }
}
```

### 항목 설명

| 필드명 (Field) | 데이터 타입 (Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `sdk_version` | String | 커넥터 개발 및 실행에 사용된 SDK의 버전 정보입니다. |
| `extension_type` | String | 커넥터가 표준(Standard) 타입인지, 커스텀(Custom) 확장 타입인지 명시합니다. |
| `dependencies` | Array (String) | 커넥터 로드 시 필요한 외부 라이브러리 패키지명 등을 나열합니다. |
| `handler_mapping` | Object | 특정 작업(Action)이나 트리거(Trigger)에 연결될 코드 핸들러의 매핑 정보입니다. |

### Sample Configuration

```json
{
  "sdk_config": {
    "sdk_version": "1.2.0",
    "extension_type": "custom",
    "dependencies": ["lodash", "axios"],
    "handler_mapping": {
      "on_init": "initHandler",
      "on_error": "errorHandler"
    }
  }
}
```

---

## 4. Marketplace (마켓플레이스 정보)

커넥터 마켓플레이스에 등록될 때 사용되는 메타데이터 및 배포 정보입니다.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "marketplace_info": {
      "type": "object",
      "properties": {
        "registry_visibility": {
          "type": "string",
          "enum": ["public", "private", "shared"],
          "description": "레지스트리에서의 공개 범위를 설정합니다."
        },
        "tags": {
          "type": "array",
          "items": { "type": "string" },
          "description": "검색 및 분류를 위한 태그 목록입니다."
        },
        "release_status": {
          "type": "string",
          "enum": ["alpha", "beta", "ga", "deprecated"],
          "description": "커넥터의 현재 릴리스 상태입니다."
        },
        "auth_profile_templates": {
          "type": "array",
          "items": { "type": "object" },
          "description": "사용자가 쉽게 인증을 설정할 수 있도록 제공하는 템플릿 목록입니다."
        }
      }
    }
  }
}
```

### 항목 설명

| 필드명 (Field) | 데이터 타입 (Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `registry_visibility` | String | 커넥터가 마켓플레이스 내에서 누구에게 보여질지 설정합니다 (공개, 비공개 등). |
| `tags` | Array (String) | 사용자가 커넥터를 쉽게 찾을 수 있도록 돕는 키워드 태그입니다. |
| `release_status` | String | 개발 단계(Alpha, Beta), 정식 출시(GA), 혹은 지원 중단(Deprecated) 등의 상태를 나타냅니다. |
| `auth_profile_templates` | Array (Object) | 미리 정의된 인증 프로필 템플릿을 제공하여 사용자 설정을 돕습니다. |

### Sample Configuration

```json
{
  "marketplace_info": {
    "registry_visibility": "public",
    "tags": ["crm", "sales", "customer"],
    "release_status": "ga",
    "auth_profile_templates": [
      {
        "name": "Standard OAuth",
        "description": "Standard OAuth2 flow template"
      }
    ]
  }
}
```

---

## 5. Metadata Harvesting (메타데이터 수집)

외부 시스템의 스키마나 객체 정보를 동적으로 수집하기 위한 설정입니다.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "metadata_config": {
      "type": "object",
      "properties": {
        "introspection_job": {
          "type": "object",
          "description": "메타데이터 수집 작업의 스케줄링 및 설정입니다."
        },
        "discovery_depth": {
          "type": "integer",
          "description": "메타데이터 탐색 시 하위 객체를 어디까지 탐색할지 깊이를 설정합니다."
        },
        "dynamic_schema_enabled": {
          "type": "boolean",
          "description": "실시간 스키마 변경 사항 반영 여부입니다."
        },
        "metadata_ttl": {
          "type": "integer",
          "description": "수집된 메타데이터의 유효 기간(TTL, 초 단위)입니다."
        }
      }
    }
  }
}
```

### 항목 설명

| 필드명 (Field) | 데이터 타입 (Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `introspection_job` | Object | 주기적으로 메타데이터를 갱신하기 위한 백그라운드 작업 설정입니다. |
| `discovery_depth` | Integer | 연관된 객체를 검색할 때 몇 단계 깊이까지 조회할지 결정합니다. 깊을수록 성능에 영향을 줄 수 있습니다. |
| `dynamic_schema_enabled` | Boolean | `true`일 경우, 런타임에 동적으로 스키마를 확인하여 변경사항을 반영합니다. |
| `metadata_ttl` | Integer | 캐시된 메타데이터가 유지되는 시간(초)입니다. 이 시간이 지나면 다시 수집합니다. |

### Sample Configuration

```json
{
  "metadata_config": {
    "introspection_job": {
      "schedule": "0 0 * * *",
      "enabled": true
    },
    "discovery_depth": 2,
    "dynamic_schema_enabled": true,
    "metadata_ttl": 3600
  }
}
```

---

## 6. Health Check (상태 점검)

연결 상태를 주기적으로 확인하고 장애를 감지하기 위한 설정입니다.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "health_check": {
      "type": "object",
      "properties": {
        "heartbeat_interval": {
          "type": "integer",
          "description": "상태 점검 주기(초 단위)입니다."
        },
        "failure_threshold": {
          "type": "integer",
          "description": "연결 실패로 간주하기 전 허용되는 연속 실패 횟수입니다."
        },
        "alarm": {
          "type": "object",
          "description": "장애 발생 시 알림 설정입니다.",
          "properties": {
            "enabled": { "type": "boolean" },
            "channels": { "type": "array", "items": { "type": "string" } }
          }
        },
        "validate_endpoint": {
          "type": "string",
          "description": "상태 점검을 위해 호출할 API 엔드포인트입니다."
        }
      }
    }
  }
}
```

### 항목 설명

| 필드명 (Field) | 데이터 타입 (Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `heartbeat_interval` | Integer | 헬스 체크 요청을 보내는 간격(초)입니다. |
| `failure_threshold` | Integer | 이 횟수만큼 연속으로 실패하면 커넥터 상태를 'Down'으로 변경합니다. |
| `alarm` | Object | 장애 감지 시 관리자에게 알림을 보낼지 여부와 채널(이메일, 슬랙 등)을 설정합니다. |
| `validate_endpoint` | String | 실제 연결 가능한지 확인하기 위해 가볍게 호출할 수 있는 API 경로(예: `/ping`, `/health`)입니다. |

### Sample Configuration

```json
{
  "health_check": {
    "heartbeat_interval": 60,
    "failure_threshold": 3,
    "alarm": {
      "enabled": true,
      "channels": ["email", "slack"]
    },
    "validate_endpoint": "/v1/system/ping"
  }
}
```

---

## 7. System Adapter Wrapping (시스템 어댑터 래핑)

레거시 시스템이나 복잡한 프로토콜을 추상화하여 연동하기 위한 설정입니다.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "system_adapter_config": {
      "type": "object",
      "properties": {
        "bridge_mode": {
          "type": "boolean",
          "description": "브리지 모드 활성화 여부입니다."
        },
        "virtual_url": {
          "type": "string",
          "description": "내부 시스템에 매핑되는 가상 URL입니다."
        }
      }
    }
  }
}
```

### 항목 설명

| 필드명 (Field) | 데이터 타입 (Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `bridge_mode` | Boolean | 직접 연결이 어려운 시스템을 위해 중간 브리지를 사용할지 여부를 설정합니다. |
| `virtual_url` | String | 외부에서 접근할 때 사용되는 가상의 엔드포인트 주소입니다. 실제 내부 주소로 라우팅됩니다. |

### Sample Configuration

```json
{
  "system_adapter_config": {
    "bridge_mode": true,
    "virtual_url": "https://connector-hub.internal/legacy-erp"
  }
}
```
