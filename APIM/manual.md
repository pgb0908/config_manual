# APIM Configuration Manual

본 매뉴얼은 iPaaS의 APIM(API Management) 모듈에 적용되는 설정(Configuration)에 대한 상세 가이드입니다. 각 기능 그룹별로 설정 필드에 대한 설명(한국어)과 JSON 스키마, 그리고 적용 예시를 제공합니다.

설정 필드명은 `snake_case` 규칙을 따릅니다.

---

## 목차
1. [API Design](#1-api-design)
2. [Mocking Service](#2-mocking-service)
3. [Lifecycle](#3-lifecycle)
4. [Developer Portal](#4-developer-portal)
5. [Traffic Policies](#5-traffic-policies)
6. [Security Policies](#6-security-policies)
7. [Transformation Policy](#7-transformation-policy)
8. [Reliability](#8-reliability)

---

## 1. API Design
API의 기본 명세 및 유효성 검증 정책을 설정합니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `spec_url` | String | API 사양(OpenAPI Specification 등) 파일이 위치한 URL입니다. |
| `oas_validation` | Boolean | 들어오는 요청이 정의된 OAS 스펙을 준수하는지 실시간으로 검증할지 여부입니다. |
| `validate_body` | Boolean | 요청의 Body 내용이 정의된 스키마와 일치하는지 엄격하게 검증합니다. |
| `base_path` | String | API 게이트웨이에서 해당 API 서비스로 라우팅하기 위한 기본 경로(Prefix)입니다. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "spec_url": { "type": "string", "format": "uri" },
    "oas_validation": { "type": "boolean" },
    "validate_body": { "type": "boolean" },
    "base_path": { "type": "string" }
  },
  "required": ["base_path"]
}
```

### Sample Configuration
```json
{
  "spec_url": "https://repo.example.com/specs/payment-api-v1.yaml",
  "oas_validation": true,
  "validate_body": true,
  "base_path": "/api/v1/payments"
}
```

---

## 2. Mocking Service
백엔드 서비스 없이 API 응답을 시뮬레이션하기 위한 설정입니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `mock_rules` | Array | 특정 조건에 따라 다른 응답을 주기 위한 규칙 목록입니다. |
| `stub_payload` | Object/String | 기본적으로 반환할 모의 응답 데이터(JSON 또는 문자열)입니다. |
| `random_delay_range` | Object | 응답 지연 시간을 랜덤하게 부여하여 네트워크 지연을 시뮬레이션합니다. |
| `random_delay_range.min` | Integer | 최소 지연 시간 (밀리초). |
| `random_delay_range.max` | Integer | 최대 지연 시간 (밀리초). |
| `error_simulation_rate` | Number | 인위적으로 에러를 발생시킬 확률입니다 (0.0 ~ 1.0). 예: 0.1은 10% 확률. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "mock_rules": {
      "type": "array",
      "items": { "type": "object" }
    },
    "stub_payload": { "type": ["object", "string"] },
    "random_delay_range": {
      "type": "object",
      "properties": {
        "min": { "type": "integer", "minimum": 0 },
        "max": { "type": "integer", "minimum": 0 }
      }
    },
    "error_simulation_rate": { "type": "number", "minimum": 0.0, "maximum": 1.0 }
  }
}
```

### Sample Configuration
```json
{
  "stub_payload": {
    "message": "This is a mocked response",
    "status": "success"
  },
  "random_delay_range": {
    "min": 100,
    "max": 500
  },
  "error_simulation_rate": 0.05
}
```

---

## 3. Lifecycle
API의 생명주기 및 버전 관리를 위한 설정입니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `api_version` | String | API의 버전 식별자입니다 (예: "1.0.0"). |
| `lifecycle_status` | String | API의 현재 상태입니다. (`CREATED`, `PUBLISHED`, `DEPRECATED`, `RETIRED`) |
| `sunset_date` | String | API 서비스가 종료될 예정인 날짜입니다 (YYYY-MM-DD 형식). |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "api_version": { "type": "string" },
    "lifecycle_status": {
      "type": "string",
      "enum": ["CREATED", "PUBLISHED", "DEPRECATED", "RETIRED"]
    },
    "sunset_date": { "type": "string", "format": "date" }
  },
  "required": ["api_version", "lifecycle_status"]
}
```

### Sample Configuration
```json
{
  "api_version": "2.1.0",
  "lifecycle_status": "DEPRECATED",
  "sunset_date": "2024-12-31"
}
```

---

## 4. Developer Portal
개발자 포털에서의 API 노출 및 접근 권한을 설정합니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `portal_visibility` | String | 포털에 API를 공개할 범위입니다. (`PUBLIC`, `PRIVATE`, `INTERNAL`) |
| `self_registration` | Boolean | 개발자가 관리자 개입 없이 스스로 API 사용 신청을 할 수 있는지 여부입니다. |
| `approval_type` | String | API 사용 신청에 대한 승인 방식입니다. (`MANUAL`: 수동 승인, `AUTO`: 자동 승인) |
| `usage_analytics_access` | Boolean | API 소비자가 자신의 사용량 통계에 접근할 수 있는지 여부입니다. |
| `code_snippets_languages` | Array | 포털에서 예제 코드로 제공할 프로그래밍 언어 목록입니다. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "portal_visibility": { "type": "string", "enum": ["PUBLIC", "PRIVATE", "INTERNAL"] },
    "self_registration": { "type": "boolean" },
    "approval_type": { "type": "string", "enum": ["MANUAL", "AUTO"] },
    "usage_analytics_access": { "type": "boolean" },
    "code_snippets_languages": {
      "type": "array",
      "items": { "type": "string" }
    }
  }
}
```

### Sample Configuration
```json
{
  "portal_visibility": "PUBLIC",
  "self_registration": true,
  "approval_type": "AUTO",
  "usage_analytics_access": true,
  "code_snippets_languages": ["curl", "python", "javascript", "java"]
}
```

---

## 5. Traffic Policies
트래픽 제어 및 SLA(Service Level Agreement) 관련 설정입니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `rate_limit` | Object | API 요청 속도 제한 설정입니다. |
| `rate_limit.quota` | Object | 일정 기간 동안 허용되는 총 요청 수(Quota) 설정입니다. |
| `rate_limit.quota.limit_count` | Integer | 허용되는 최대 요청 수입니다. |
| `rate_limit.quota.interval` | String | 제한 기간 단위입니다 (예: "1h", "1d", "1m"). |
| `rate_limit.burst_limit` | Object | 순간적인 트래픽 폭주를 제어하기 위한 버스트 제한 설정입니다. |
| `rate_limit.burst_limit.rate` | Integer | 초당 허용 가능한 최대 버스트 요청 수입니다. |
| `rate_limit.burst_limit.identifier` | String | 제한을 적용할 기준 키입니다 (예: "ip", "client_id"). |
| `max_concurrency` | Object | 동시 처리 가능한 최대 요청 수를 제한합니다. |
| `max_concurrency.limit` | Integer | 동시에 처리할 수 있는 최대 연결 수입니다. |
| `sla_tiers` | Array | 클라이언트 등급별 차등 정책 목록입니다. |
| `sla_tiers[].client_id` | String | 정책을 적용할 클라이언트 ID입니다. |
| `sla_tiers[].tier_name` | String | 적용할 SLA 등급 이름입니다 (예: "GOLD", "SILVER"). |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "rate_limit": {
      "type": "object",
      "properties": {
        "quota": {
          "type": "object",
          "properties": {
            "limit_count": { "type": "integer" },
            "interval": { "type": "string" }
          }
        },
        "burst_limit": {
          "type": "object",
          "properties": {
            "rate": { "type": "integer" },
            "identifier": { "type": "string" }
          }
        }
      }
    },
    "max_concurrency": {
      "type": "object",
      "properties": {
        "limit": { "type": "integer" }
      }
    },
    "sla_tiers": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "client_id": { "type": "string" },
          "tier_name": { "type": "string" }
        }
      }
    }
  }
}
```

### Sample Configuration
```json
{
  "rate_limit": {
    "quota": {
      "limit_count": 10000,
      "interval": "1d"
    },
    "burst_limit": {
      "rate": 50,
      "identifier": "client_id"
    }
  },
  "max_concurrency": {
    "limit": 200
  },
  "sla_tiers": [
    { "client_id": "vip-client-001", "tier_name": "GOLD" },
    { "client_id": "partner-123", "tier_name": "SILVER" }
  ]
}
```

---

## 6. Security Policies
API 보안 및 접근 제어를 위한 다양한 정책 설정입니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `jwt_validation` | Object | JWT(Json Web Token) 유효성 검증 설정입니다. |
| `jwt_validation.jwks_url` | String | 서명 검증을 위한 공개키(JWKS)를 제공하는 URL입니다. |
| `jwt_validation.issuer` | String | 토큰 발급자(iss) 검증 값입니다. |
| `jwt_validation.audiences` | Array | 토큰 수신자(aud) 검증 값 목록입니다. |
| `jwt_validation.clock_skew` | Integer | 토큰 만료 시간 검증 시 허용할 시간 오차(초)입니다. |
| `oauth2` | Object | OAuth 2.0 인증/인가 설정입니다. |
| `oauth2.grant_types` | Array | 허용할 권한 부여 방식 목록입니다 (예: "authorization_code", "client_credentials"). |
| `oauth2.scopes` | Array | API 접근에 필요한 스코프 목록입니다. |
| `oauth2.token_introspection_url`| String | 토큰의 유효성을 검증하기 위한 Introspection 엔드포인트 URL입니다. |
| `mtls` | Object | 상호 TLS(mTLS) 인증 설정입니다. |
| `mtls.client_cert_ca` | String | 클라이언트 인증서를 검증하기 위한 CA 인증서(PEM 형식 또는 파일 경로)입니다. |
| `mtls.verify_depth` | Integer | 인증서 체인 검증 깊이입니다. |
| `ip_allowlist` | Object | IP 기반 접근 제어 설정입니다. |
| `ip_allowlist.allowed_cidrs` | Array | 접근이 허용된 IP 대역(CIDR) 목록입니다. |
| `json_threat_protection` | Object | 악의적인 JSON 요청으로부터 시스템을 보호하기 위한 설정입니다. |
| `json_threat_protection.max_container_depth` | Integer | JSON 객체/배열의 최대 중첩 깊이입니다. |
| `json_threat_protection.max_string_length` | Integer | JSON 문자열 값의 최대 길이입니다. |
| `cors` | Object | 교차 출처 리소스 공유(CORS) 설정입니다. |
| `cors.allow_origins` | Array | 허용할 출처(Origin) 목록입니다 (`*` 허용). |
| `cors.allow_methods` | Array | 허용할 HTTP 메서드 목록입니다. |
| `cors.allow_headers` | Array | 허용할 HTTP 헤더 목록입니다. |
| `cors.expose_headers` | Array | 브라우저에 노출할 응답 헤더 목록입니다. |
| `cors.max_age` | Integer | Preflight 요청 결과를 캐시할 시간(초)입니다. |
| `cors.allow_credentials`| Boolean | 쿠키 등 인증 정보를 포함한 요청을 허용할지 여부입니다. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "jwt_validation": {
      "type": "object",
      "properties": {
        "jwks_url": { "type": "string" },
        "issuer": { "type": "string" },
        "audiences": { "type": "array", "items": { "type": "string" } },
        "clock_skew": { "type": "integer" }
      }
    },
    "oauth2": {
      "type": "object",
      "properties": {
        "grant_types": { "type": "array", "items": { "type": "string" } },
        "scopes": { "type": "array", "items": { "type": "string" } },
        "token_introspection_url": { "type": "string" }
      }
    },
    "mtls": {
      "type": "object",
      "properties": {
        "client_cert_ca": { "type": "string" },
        "verify_depth": { "type": "integer" }
      }
    },
    "ip_allowlist": {
      "type": "object",
      "properties": {
        "allowed_cidrs": { "type": "array", "items": { "type": "string" } }
      }
    },
    "json_threat_protection": {
      "type": "object",
      "properties": {
        "max_container_depth": { "type": "integer" },
        "max_string_length": { "type": "integer" }
      }
    },
    "cors": {
      "type": "object",
      "properties": {
        "allow_origins": { "type": "array", "items": { "type": "string" } },
        "allow_methods": { "type": "array", "items": { "type": "string" } },
        "allow_headers": { "type": "array", "items": { "type": "string" } },
        "expose_headers": { "type": "array", "items": { "type": "string" } },
        "max_age": { "type": "integer" },
        "allow_credentials": { "type": "boolean" }
      }
    }
  }
}
```

### Sample Configuration
```json
{
  "jwt_validation": {
    "jwks_url": "https://auth.example.com/.well-known/jwks.json",
    "issuer": "https://auth.example.com",
    "audiences": ["my-api-service"],
    "clock_skew": 60
  },
  "ip_allowlist": {
    "allowed_cidrs": ["192.168.1.0/24", "10.0.0.5/32"]
  },
  "cors": {
    "allow_origins": ["https://app.example.com"],
    "allow_methods": ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
    "allow_headers": ["Content-Type", "Authorization"],
    "allow_credentials": true,
    "max_age": 3600
  }
}
```

---

## 7. Transformation Policy
요청/응답 메시지의 변환 및 데이터 마스킹 관련 설정입니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `header_param_manipulation` | Object | 헤더 및 쿼리 파라미터 조작 설정입니다. |
| `header_param_manipulation.inject_headers` | Object | 요청/응답에 추가할 헤더 키-값 쌍입니다. |
| `header_param_manipulation.remove_headers` | Array | 요청/응답에서 제거할 헤더 이름 목록입니다. |
| `header_param_manipulation.rename_query_params` | Object | 쿼리 파라미터 이름을 변경할 매핑 정보입니다 (OldName: NewName). |
| `type_transformation` | Object | 데이터 포맷(JSON/XML) 변환 설정입니다. |
| `type_transformation.json_to_xml` | Boolean | JSON 요청/응답을 XML로 변환할지 여부입니다. |
| `type_transformation.xml_to_json` | Boolean | XML 요청/응답을 JSON으로 변환할지 여부입니다. |
| `data_masking` | Object | 민감 정보 마스킹 설정입니다. |
| `data_masking.masking_rules` | Array | 마스킹 규칙 목록입니다. |
| `data_masking.masking_rules[].path` | String | 마스킹할 데이터의 JSON 경로(JSONPath) 또는 필드명입니다. |
| `data_masking.masking_rules[].regex_pattern` | String | 마스킹할 패턴을 지정하는 정규 표현식입니다. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "header_param_manipulation": {
      "type": "object",
      "properties": {
        "inject_headers": { "type": "object", "additionalProperties": { "type": "string" } },
        "remove_headers": { "type": "array", "items": { "type": "string" } },
        "rename_query_params": { "type": "object", "additionalProperties": { "type": "string" } }
      }
    },
    "type_transformation": {
      "type": "object",
      "properties": {
        "json_to_xml": { "type": "boolean" },
        "xml_to_json": { "type": "boolean" }
      }
    },
    "data_masking": {
      "type": "object",
      "properties": {
        "masking_rules": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "path": { "type": "string" },
              "regex_pattern": { "type": "string" }
            }
          }
        }
      }
    }
  }
}
```

### Sample Configuration
```json
{
  "header_param_manipulation": {
    "inject_headers": {
      "X-Api-Source": "Gateway"
    },
    "remove_headers": ["Server", "X-Powered-By"],
    "rename_query_params": {
      "q": "query"
    }
  },
  "type_transformation": {
    "xml_to_json": true
  },
  "data_masking": {
    "masking_rules": [
      {
        "path": "$.body.user.ssn",
        "regex_pattern": "\\d{3}-\\d{2}-\\d{4}"
      }
    ]
  }
}
```

---

## 8. Reliability
시스템 안정성 및 성능 보장을 위한 설정입니다.

### Configuration Fields
| Field Name | Type | Description |
| :--- | :--- | :--- |
| `caching` | Object | 응답 캐싱 설정입니다. |
| `caching.ttl` | Integer | 캐시 유지 시간(초)입니다 (Time To Live). |
| `caching.cache_key_params` | Array | 캐시 키 생성 시 포함할 쿼리 파라미터 목록입니다. |
| `caching.invalidation_headers` | Array | 캐시를 무효화(삭제)하기 위한 요청 헤더 목록입니다. |
| `load_balancing` | Object | 백엔드 서버 부하 분산 설정입니다. |
| `load_balancing.algorithm` | String | 부하 분산 알고리즘입니다 (`ROUND_ROBIN`, `LEAST_CONN`, `IP_HASH`). |
| `load_balancing.targets` | Array | 트래픽을 분산할 대상 백엔드 서버 목록입니다. |
| `circuit_breaker` | Object | 장애 확산 방지를 위한 서킷 브레이커 설정입니다. |
| `circuit_breaker.failure_threshold` | Integer | 서킷을 열기(Open) 위한 실패 횟수 또는 비율 임계값입니다. |
| `circuit_breaker.reset_timeout` | Integer | 서킷이 열린 후 다시 닫힘(Closed) 상태로 전환을 시도하기 전 대기 시간(초)입니다. |

### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "caching": {
      "type": "object",
      "properties": {
        "ttl": { "type": "integer" },
        "cache_key_params": { "type": "array", "items": { "type": "string" } },
        "invalidation_headers": { "type": "array", "items": { "type": "string" } }
      }
    },
    "load_balancing": {
      "type": "object",
      "properties": {
        "algorithm": { "type": "string", "enum": ["ROUND_ROBIN", "LEAST_CONN", "IP_HASH"] },
        "targets": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "host": { "type": "string" },
              "port": { "type": "integer" },
              "weight": { "type": "integer" }
            }
          }
        }
      }
    },
    "circuit_breaker": {
      "type": "object",
      "properties": {
        "failure_threshold": { "type": "integer" },
        "reset_timeout": { "type": "integer" }
      }
    }
  }
}
```

### Sample Configuration
```json
{
  "caching": {
    "ttl": 300,
    "cache_key_params": ["category", "page"]
  },
  "load_balancing": {
    "algorithm": "ROUND_ROBIN",
    "targets": [
      { "host": "10.0.1.10", "port": 8080, "weight": 1 },
      { "host": "10.0.1.11", "port": 8080, "weight": 2 }
    ]
  },
  "circuit_breaker": {
    "failure_threshold": 5,
    "reset_timeout": 60
  }
}
```
