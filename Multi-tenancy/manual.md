# Multi-tenancy Configuration 매뉴얼

이 매뉴얼은 iPaaS 플랫폼의 멀티테넌시(Multi-tenancy) 및 리소스 격리(Resource Isolation)를 위한 설정(Configuration) 가이드를 제공합니다.
각 테넌트의 식별, 조직 구조, 리소스 할당, 보안 정책 및 격리 수준을 정의합니다.

## 설정 상세 (Configuration Details)

설정은 기능별로 계층 구조(Tree structure)를 가집니다.

*   `tenant_config` (Object): 테넌트 설정의 최상위 루트 객체입니다.
    *   **식별 및 기본 정보 (Identity & Basic Info)**
        *   `tenant_id` (String): 테넌트를 고유하게 식별하는 ID입니다. (시스템 자동 생성 또는 고유값)
        *   `tenant_name` (String): 테넌트의 표시 이름(Display Name)입니다.
        *   `tenant_domain` (String): 테넌트 전용 도메인 또는 서브도메인입니다. (예: `customer.ipaas.com`)
        *   `admin_email` (String): 테넌트 관리자의 연락처 이메일 주소입니다.
    *   **조직 구조 (Organization Structure)**
        *   `organization` (Object): 조직 계층 및 그룹 정보를 정의합니다.
            *   `business_group_id` (String): 해당 테넌트가 속한 비즈니스 그룹의 ID입니다.
            *   `parent_org_id` (String): 상위 조직의 ID입니다. 계층적 관리를 위해 사용됩니다.
            *   `cost_center_code` (String): 비용 정산을 위한 부서 또는 프로젝트 코드입니다.
    *   **리소스 쿼터 (Resource Quotas)**
        *   `quotas` (Object): 테넌트에 할당된 리소스 사용량 제한을 설정합니다.
            *   `max_api_calls_per_month` (Integer): 월간 최대 API 호출 허용 횟수입니다.
            *   `storage_limit_gb` (Integer): 데이터 저장을 위한 최대 디스크 용량(GB)입니다.
            *   `max_concurrent_connections` (Integer): 동시 접속 가능한 최대 연결 수입니다.
            *   `max_active_flows` (Integer): 활성화 상태로 유지할 수 있는 최대 통합 플로우(Flow) 수입니다.
    *   **보안 정책 (Security Policies)**
        *   `security` (Object): 테넌트별 보안 접근 제어 및 정책을 설정합니다.
            *   `ip_allowlist` (Array of Strings): 접근이 허용된 IP 주소 대역(CIDR) 목록입니다.
            *   `mfa_enforced` (Boolean): 모든 사용자에게 다중 요소 인증(MFA)을 강제할지 여부입니다.
            *   `session_timeout_minutes` (Integer): 유휴 세션 만료 시간(분)입니다.
            *   `compliance_level` (String): 적용할 규정 준수 레벨입니다. (예: `SOC2`, `HIPAA`, `NONE`)
    *   **격리 설정 (Isolation Settings)**
        *   `isolation` (Object): 물리적/논리적 리소스 격리 수준을 정의합니다.
            *   `execution_plane` (String): 실행 환경의 격리 유형입니다. (`shared`: 공유, `dedicated`: 전용)
            *   `dedicated_db` (Boolean): 테넌트 전용 데이터베이스 인스턴스 사용 여부입니다.
            *   `region` (String): 데이터 및 서비스가 호스팅될 지리적 리전입니다. (예: `ap-northeast-2`)

---

## JSON 스키마 (JSON Schema)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "tenant_config": {
      "type": "object",
      "properties": {
        "tenant_id": {
          "type": "string",
          "description": "Unique identifier for the tenant"
        },
        "tenant_name": {
          "type": "string",
          "description": "Display name of the tenant"
        },
        "tenant_domain": {
          "type": "string",
          "format": "hostname",
          "description": "Dedicated domain or subdomain for the tenant"
        },
        "admin_email": {
          "type": "string",
          "format": "email",
          "description": "Contact email for tenant administrator"
        },
        "organization": {
          "type": "object",
          "properties": {
            "business_group_id": {
              "type": "string",
              "description": "Identifier for the business group"
            },
            "parent_org_id": {
              "type": "string",
              "description": "Identifier for the parent organization"
            },
            "cost_center_code": {
              "type": "string",
              "description": "Code for cost allocation"
            }
          },
          "required": ["business_group_id"]
        },
        "quotas": {
          "type": "object",
          "properties": {
            "max_api_calls_per_month": {
              "type": "integer",
              "minimum": 0
            },
            "storage_limit_gb": {
              "type": "integer",
              "minimum": 0
            },
            "max_concurrent_connections": {
              "type": "integer",
              "minimum": 0
            },
            "max_active_flows": {
              "type": "integer",
              "minimum": 0
            }
          }
        },
        "security": {
          "type": "object",
          "properties": {
            "ip_allowlist": {
              "type": "array",
              "items": {
                "type": "string",
                "format": "ipv4"
              }
            },
            "mfa_enforced": {
              "type": "boolean"
            },
            "session_timeout_minutes": {
              "type": "integer",
              "minimum": 1
            },
            "compliance_level": {
              "type": "string",
              "enum": ["NONE", "SOC2", "HIPAA", "GDPR"]
            }
          }
        },
        "isolation": {
          "type": "object",
          "properties": {
            "execution_plane": {
              "type": "string",
              "enum": ["shared", "dedicated"]
            },
            "dedicated_db": {
              "type": "boolean"
            },
            "region": {
              "type": "string"
            }
          }
        }
      },
      "required": ["tenant_id", "tenant_domain", "organization"]
    }
  },
  "required": ["tenant_config"]
}
```

## 설정 예시 (Configuration Sample)

```json
{
  "tenant_config": {
    "tenant_id": "tn-12345678",
    "tenant_name": "Acme Corp",
    "tenant_domain": "acme.ipaas-cloud.com",
    "admin_email": "admin@acme.com",
    "organization": {
      "business_group_id": "bg-sales-001",
      "parent_org_id": "org-global-HQ",
      "cost_center_code": "CC-9921"
    },
    "quotas": {
      "max_api_calls_per_month": 1000000,
      "storage_limit_gb": 500,
      "max_concurrent_connections": 200,
      "max_active_flows": 50
    },
    "security": {
      "ip_allowlist": [
        "192.168.1.0/24",
        "10.0.0.5"
      ],
      "mfa_enforced": true,
      "session_timeout_minutes": 30,
      "compliance_level": "SOC2"
    },
    "isolation": {
      "execution_plane": "dedicated",
      "dedicated_db": true,
      "region": "ap-northeast-2"
    }
  }
}
```
