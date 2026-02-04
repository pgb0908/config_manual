# Multi-tenancy Configuration 매뉴얼

이 매뉴얼은 iPaaS 플랫폼의 멀티테넌시(Multi-tenancy) 및 리소스 격리(Resource Isolation)를 위한 설정(Configuration) 가이드를 제공합니다.
각 테넌트의 식별, 조직 구조, 리소스 할당, 보안 정책 및 격리 수준을 정의합니다.

---

## 1. 식별 및 기본 정보 (Identity & Basic Info)

테넌트를 고유하게 식별하고 관리하기 위한 기본적인 정보를 설정합니다.

### 설정 상세 (Configuration Details)

*   `tenant_id` (String): 테넌트를 고유하게 식별하는 ID입니다.
*   `tenant_name` (String): 테넌트의 표시 이름(Display Name)입니다.
*   `tenant_domain` (String): 테넌트 전용 도메인 또는 서브도메인입니다.
*   `admin_contact` (Object): 테넌트 관리자 연락처 정보입니다.
    *   `email` (String): 관리자 이메일 주소입니다.
    *   `phone` (String): 관리자 전화번호입니다.
*   `status` (String): 테넌트의 현재 상태입니다. (`active`, `suspended`, `terminated`)
*   `created_at` (String): 테넌트가 생성된 일시입니다. (ISO 8601 형식)

### JSON 스키마 (JSON Schema)

```json
{
  "type": "object",
  "properties": {
    "tenant_id": { "type": "string" },
    "tenant_name": { "type": "string" },
    "tenant_domain": { "type": "string", "format": "hostname" },
    "admin_contact": {
      "type": "object",
      "properties": {
        "email": { "type": "string", "format": "email" },
        "phone": { "type": "string" }
      },
      "required": ["email"]
    },
    "status": { "type": "string", "enum": ["active", "suspended", "terminated"] },
    "created_at": { "type": "string", "format": "date-time" }
  },
  "required": ["tenant_id", "tenant_name", "tenant_domain"]
}
```

### 설정 예시 (Configuration Sample)

```json
{
  "tenant_id": "tn-12345678",
  "tenant_name": "Acme Corp",
  "tenant_domain": "acme.ipaas-cloud.com",
  "admin_contact": {
    "email": "admin@acme.com",
    "phone": "+82-10-1234-5678"
  },
  "status": "active",
  "created_at": "2023-10-27T10:00:00Z"
}
```

---

## 2. 조직 구조 (Organization Structure)

기업 내의 조직 계층 및 비용 관리 정보를 정의합니다.

### 설정 상세 (Configuration Details)

*   `business_group` (Object): 테넌트가 속한 비즈니스 그룹 정보입니다.
    *   `id` (String): 비즈니스 그룹 고유 ID입니다.
    *   `name` (String): 비즈니스 그룹 이름입니다.
*   `parent_org_id` (String): 상위 조직의 ID입니다. 계층적 관리를 위해 사용됩니다.
*   `cost_center` (Object): 비용 정산을 위한 부서 정보를 정의합니다.
    *   `code` (String): 코스트 센터 코드입니다.
    *   `department` (String): 담당 부서 명칭입니다.

### JSON 스키마 (JSON Schema)

```json
{
  "type": "object",
  "properties": {
    "business_group": {
      "type": "object",
      "properties": {
        "id": { "type": "string" },
        "name": { "type": "string" }
      },
      "required": ["id"]
    },
    "parent_org_id": { "type": "string" },
    "cost_center": {
      "type": "object",
      "properties": {
        "code": { "type": "string" },
        "department": { "type": "string" }
      }
    }
  },
  "required": ["business_group"]
}
```

### 설정 예시 (Configuration Sample)

```json
{
  "business_group": {
    "id": "bg-sales-001",
    "name": "Global Sales"
  },
  "parent_org_id": "org-global-HQ",
  "cost_center": {
    "code": "CC-9921",
    "department": "Sales Operations"
  }
}
```

---

## 3. 리소스 쿼터 (Resource Quotas)

테넌트에 할당된 리소스 사용량 한도를 설정하여 안정적인 서비스 운영을 보장합니다.

### 설정 상세 (Configuration Details)

*   `api_quotas` (Object): API 호출과 관련된 쿼터 설정입니다.
    *   `max_calls_per_month` (Integer): 월간 최대 API 호출 허용 횟수입니다.
    *   `burst_limit` (Integer): 초당 순간 최대 호출 수(TPS)입니다.
*   `storage_quotas` (Object): 데이터 저장 공간 관련 쿼터 설정입니다.
    *   `limit_gb` (Integer): 최대 디스크 용량(GB)입니다.
    *   `retention_days` (Integer): 데이터 보관 기간(일)입니다.
*   `compute_quotas` (Object): 연산 자원 및 연결 관련 쿼터 설정입니다.
    *   `max_concurrent_connections` (Integer): 동시 접속 가능한 최대 연결 수입니다.
    *   `max_active_flows` (Integer): 활성화 상태로 유지할 수 있는 최대 통합 플로우 수입니다.

### JSON 스키마 (JSON Schema)

```json
{
  "type": "object",
  "properties": {
    "api_quotas": {
      "type": "object",
      "properties": {
        "max_calls_per_month": { "type": "integer", "minimum": 0 },
        "burst_limit": { "type": "integer", "minimum": 0 }
      }
    },
    "storage_quotas": {
      "type": "object",
      "properties": {
        "limit_gb": { "type": "integer", "minimum": 0 },
        "retention_days": { "type": "integer", "minimum": 0 }
      }
    },
    "compute_quotas": {
      "type": "object",
      "properties": {
        "max_concurrent_connections": { "type": "integer", "minimum": 0 },
        "max_active_flows": { "type": "integer", "minimum": 0 }
      }
    }
  }
}
```

### 설정 예시 (Configuration Sample)

```json
{
  "api_quotas": {
    "max_calls_per_month": 1000000,
    "burst_limit": 100
  },
  "storage_quotas": {
    "limit_gb": 500,
    "retention_days": 30
  },
  "compute_quotas": {
    "max_concurrent_connections": 200,
    "max_active_flows": 50
  }
}
```

---

## 4. 보안 정책 (Security Policies)

테넌트별 보안 접근 제어 및 규정 준수 정책을 설정합니다.

### 설정 상세 (Configuration Details)

*   `network_security` (Object): 네트워크 수준의 보안 설정입니다.
    *   `ip_allowlist` (Array of Strings): 접근이 허용된 IP 주소 대역(CIDR) 목록입니다.
    *   `vpn_required` (Boolean): 접근 시 VPN 연결이 필수인지 여부입니다.
*   `authentication` (Object): 인증 관련 정책입니다.
    *   `mfa_enforced` (Boolean): 다중 요소 인증(MFA) 강제 여부입니다.
    *   `sso_enabled` (Boolean): 단일 로그인(SSO) 활성화 여부입니다.
    *   `idp_metadata_url` (String): IdP(Identity Provider) 메타데이터 URL입니다.
*   `access_policy` (Object): 세션 및 접근 정책입니다.
    *   `session_timeout_minutes` (Integer): 유휴 세션 만료 시간(분)입니다.
    *   `password_rotation_days` (Integer): 비밀번호 변경 주기(일)입니다.
*   `compliance` (Object): 규정 준수 및 감사 설정입니다.
    *   `level` (String): 적용할 규정 준수 레벨입니다. (`NONE`, `SOC2`, `HIPAA`, `GDPR`)
    *   `audit_logging` (Boolean): 상세 감사 로그 기록 여부입니다.

### JSON 스키마 (JSON Schema)

```json
{
  "type": "object",
  "properties": {
    "network_security": {
      "type": "object",
      "properties": {
        "ip_allowlist": { "type": "array", "items": { "type": "string" } },
        "vpn_required": { "type": "boolean" }
      }
    },
    "authentication": {
      "type": "object",
      "properties": {
        "mfa_enforced": { "type": "boolean" },
        "sso_enabled": { "type": "boolean" },
        "idp_metadata_url": { "type": "string", "format": "uri" }
      }
    },
    "access_policy": {
      "type": "object",
      "properties": {
        "session_timeout_minutes": { "type": "integer", "minimum": 1 },
        "password_rotation_days": { "type": "integer", "minimum": 1 }
      }
    },
    "compliance": {
      "type": "object",
      "properties": {
        "level": { "type": "string", "enum": ["NONE", "SOC2", "HIPAA", "GDPR"] },
        "audit_logging": { "type": "boolean" }
      }
    }
  }
}
```

### 설정 예시 (Configuration Sample)

```json
{
  "network_security": {
    "ip_allowlist": ["192.168.1.0/24", "10.0.0.5"],
    "vpn_required": false
  },
  "authentication": {
    "mfa_enforced": true,
    "sso_enabled": true,
    "idp_metadata_url": "https://idp.acme.com/metadata"
  },
  "access_policy": {
    "session_timeout_minutes": 30,
    "password_rotation_days": 90
  },
  "compliance": {
    "level": "SOC2",
    "audit_logging": true
  }
}
```

---

## 5. 격리 설정 (Isolation Settings)

테넌트 간의 물리적 및 논리적 리소스 격리 수준을 정의합니다.

### 설정 상세 (Configuration Details)

*   `execution_isolation` (Object): 실행 환경의 격리 설정입니다.
    *   `plane_type` (String): 격리 유형입니다. (`shared`: 공유, `dedicated`: 전용)
    *   `resource_profile` (String): 할당된 자원의 규모 프로필입니다. (`small`, `medium`, `large`)
*   `data_isolation` (Object): 데이터 저장소의 격리 설정입니다.
    *   `dedicated_db` (Boolean): 테넌트 전용 데이터베이스 사용 여부입니다.
    *   `encryption_key_type` (String): 데이터 암호화 키 관리 방식입니다. (`platform`, `customer`)
*   `infrastructure` (Object): 물리적 인프라 위치 설정입니다.
    *   `region` (String): 데이터 및 서비스가 호스팅될 지리적 리전입니다. (예: `ap-northeast-2`)
    *   `availability_zones` (Array of Strings): 사용 중인 가용 영역 목록입니다.

### JSON 스키마 (JSON Schema)

```json
{
  "type": "object",
  "properties": {
    "execution_isolation": {
      "type": "object",
      "properties": {
        "plane_type": { "type": "string", "enum": ["shared", "dedicated"] },
        "resource_profile": { "type": "string", "enum": ["small", "medium", "large"] }
      }
    },
    "data_isolation": {
      "type": "object",
      "properties": {
        "dedicated_db": { "type": "boolean" },
        "encryption_key_type": { "type": "string", "enum": ["platform", "customer"] }
      }
    },
    "infrastructure": {
      "type": "object",
      "properties": {
        "region": { "type": "string" },
        "availability_zones": { "type": "array", "items": { "type": "string" } }
      }
    }
  }
}
```

### 설정 예시 (Configuration Sample)

```json
{
  "execution_isolation": {
    "plane_type": "dedicated",
    "resource_profile": "medium"
  },
  "data_isolation": {
    "dedicated_db": true,
    "encryption_key_type": "customer"
  },
  "infrastructure": {
    "region": "ap-northeast-2",
    "availability_zones": ["ap-northeast-2a", "ap-northeast-2c"]
  }
}
```
