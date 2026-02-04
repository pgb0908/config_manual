# iPaaS CI/CD Configuration Manual

본 매뉴얼은 iPaaS 시스템의 CI/CD 파이프라인 및 배포 환경을 구성하기 위한 설정 가이드입니다.
각 설정 항목은 기능별로 분류되어 있으며, 상세한 필드 설명과 함께 JSON 스키마 및 예시를 제공합니다.

---

## 1. Version Tagging (버전 태깅)

배포 아티팩트의 버전 관리 및 태깅 전략을 정의합니다.

### Configuration Fields

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `strategy` | string | 버전 생성 전략 (예: `semantic`, `timestamp`, `git_hash`) |
| `tag_prefix` | string | 버전 태그 앞에 붙을 접두사 (예: `v`, `release-`) |
| `include_metadata` | boolean | 메타데이터(빌드 정보 등) 포함 여부 |
| `custom_format` | string | `strategy`가 `custom`일 경우 사용할 포맷 문자열 |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Version Tagging Configuration",
  "type": "object",
  "properties": {
    "strategy": {
      "type": "string",
      "enum": ["semantic", "timestamp", "git_hash", "custom"],
      "description": "버전 생성 전략"
    },
    "tag_prefix": {
      "type": "string",
      "default": "v",
      "description": "태그 접두사"
    },
    "include_metadata": {
      "type": "boolean",
      "default": false,
      "description": "메타데이터 포함 여부"
    },
    "custom_format": {
      "type": "string",
      "description": "커스텀 버전 포맷"
    }
  },
  "required": ["strategy"]
}
```

### Sample Configuration

```json
{
  "strategy": "semantic",
  "tag_prefix": "v",
  "include_metadata": true
}
```

---

## 2. Manifest File (배포 매니페스트)

배포에 사용할 매니페스트 파일의 경로 및 처리 방식을 설정합니다.

### Configuration Fields

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `file_paths` | array | 배포할 매니페스트 파일들의 경로 목록 |
| `format` | string | 매니페스트 파일 형식 (예: `kubernetes`, `helm`, `kustomize`, `docker_compose`) |
| `values_file` | string | (Helm/Kustomize) 변수 주입을 위한 값 파일 경로 |
| `validation_strictness` | string | 문법 검증 강도 (`strict`, `lenient`) |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Manifest File Configuration",
  "type": "object",
  "properties": {
    "file_paths": {
      "type": "array",
      "items": { "type": "string" },
      "description": "매니페스트 파일 경로 목록"
    },
    "format": {
      "type": "string",
      "enum": ["kubernetes", "helm", "kustomize", "docker_compose"],
      "description": "파일 형식"
    },
    "values_file": {
      "type": "string",
      "description": "값 설정 파일 경로"
    },
    "validation_strictness": {
      "type": "string",
      "enum": ["strict", "lenient"],
      "default": "strict"
    }
  },
  "required": ["file_paths", "format"]
}
```

### Sample Configuration

```json
{
  "file_paths": ["./deploy/k8s/deployment.yaml", "./deploy/k8s/service.yaml"],
  "format": "kubernetes",
  "validation_strictness": "strict"
}
```

---

## 3. Rollback Policy (롤백 정책)

배포 실패 시 또는 특정 조건 발생 시 이전 버전으로의 복구 정책을 정의합니다.

### Configuration Fields

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `auto_rollback` | boolean | 배포 실패 시 자동 롤백 수행 여부 |
| `max_retries` | integer | 배포 재시도 최대 횟수 |
| `rollback_timeout` | string | 롤백 수행 제한 시간 (예: `10m`, `300s`) |
| `notify_on_rollback` | boolean | 롤백 발생 시 알림 발송 여부 |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Rollback Policy Configuration",
  "type": "object",
  "properties": {
    "auto_rollback": {
      "type": "boolean",
      "description": "자동 롤백 활성화"
    },
    "max_retries": {
      "type": "integer",
      "minimum": 0,
      "description": "최대 재시도 횟수"
    },
    "rollback_timeout": {
      "type": "string",
      "pattern": "^[0-9]+(s|m|h)$",
      "description": "롤백 타임아웃"
    },
    "notify_on_rollback": {
      "type": "boolean",
      "description": "알림 발송 여부"
    }
  },
  "required": ["auto_rollback"]
}
```

### Sample Configuration

```json
{
  "auto_rollback": true,
  "max_retries": 3,
  "rollback_timeout": "5m",
  "notify_on_rollback": true
}
```

---

## 4. Deployment Strategy (배포 전략)

서비스 중단 없이 안정적으로 배포하기 위한 전략을 구성합니다.

### Configuration Fields

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `release_strategy` | string | 배포 방식 (`rolling_update`, `blue_green`, `canary`, `recreate`) |
| `canary_config` | object | Canary 배포 시 상세 설정 |
| &nbsp;&nbsp;`weight` | integer | 초기 트래픽 분배 가중치 (0-100) |
| &nbsp;&nbsp;`duration` | string | 단계별 지속 시간 |
| &nbsp;&nbsp;`step_weight` | integer | 단계별 가중치 증가량 |
| `rollback_criteria` | object | 롤백 트리거 조건 |
| &nbsp;&nbsp;`error_rate_threshold` | number | 에러율 임계값 (퍼센트) |
| &nbsp;&nbsp;`latency_threshold_ms` | integer | 응답 지연 임계값 (ms) |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Deployment Strategy Configuration",
  "type": "object",
  "properties": {
    "release_strategy": {
      "type": "string",
      "enum": ["rolling_update", "blue_green", "canary", "recreate"],
      "description": "배포 전략 선택"
    },
    "canary_config": {
      "type": "object",
      "properties": {
        "weight": { "type": "integer", "minimum": 0, "maximum": 100, "description": "초기 가중치" },
        "duration": { "type": "string", "description": "테스트 지속 시간" },
        "step_weight": { "type": "integer", "description": "단계별 가중치 증가분" }
      }
    },
    "rollback_criteria": {
      "type": "object",
      "properties": {
        "error_rate_threshold": { "type": "number", "description": "에러율 임계값 (%)" },
        "latency_threshold_ms": { "type": "integer", "description": "지연 시간 임계값 (ms)" }
      }
    }
  },
  "required": ["release_strategy"]
}
```

### Sample Configuration

```json
{
  "release_strategy": "canary",
  "canary_config": {
    "weight": 10,
    "duration": "15m",
    "step_weight": 20
  },
  "rollback_criteria": {
    "error_rate_threshold": 1.5,
    "latency_threshold_ms": 500
  }
}
```

---

## 5. Runtime & Infra Architecture (런타임 및 인프라 아키텍처)

애플리케이션이 실행될 인프라 환경 및 런타임 동작 방식을 정의합니다.

### Configuration Fields

#### 5.1 Deployment Target
배포 대상 환경을 설정합니다.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `deployment_target` | object | 배포 타겟 설정 |
| &nbsp;&nbsp;`deployment_type` | string | 인프라 유형 (`kubernetes`, `vm`, `serverless`) |
| &nbsp;&nbsp;`region_location` | string | 리전 정보 (예: `us-east-1`) |
| &nbsp;&nbsp;`environment_id` | string | 환경 식별자 (`dev`, `staging`, `prod`) |
| &nbsp;&nbsp;`vpc_subnet_id` | string | VPC 서브넷 ID |

#### 5.2 Runtime Profile
리소스 할당 및 트래픽 제어 정책을 설정합니다.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `runtime_profile` | object | 런타임 프로파일 설정 |
| &nbsp;&nbsp;`resource_allocation` | object | CPU/Memory 리소스 할당 |
| &nbsp;&nbsp;&nbsp;&nbsp;`cpu_request` | string | CPU 요청량 (예: `500m`) |
| &nbsp;&nbsp;&nbsp;&nbsp;`cpu_limit` | string | CPU 제한량 (예: `1000m`) |
| &nbsp;&nbsp;&nbsp;&nbsp;`memory_request` | string | 메모리 요청량 (예: `512Mi`) |
| &nbsp;&nbsp;&nbsp;&nbsp;`memory_limit` | string | 메모리 제한량 (예: `1Gi`) |
| &nbsp;&nbsp;`concurrency_control` | object | 동시성 제어 설정 |
| &nbsp;&nbsp;&nbsp;&nbsp;`max_concurrent_requests` | integer | 최대 동시 요청 수 |
| &nbsp;&nbsp;`back_pressure` | object | 배압 조절 정책 |
| &nbsp;&nbsp;&nbsp;&nbsp;`strategy` | string | 처리 전략 (`buffer`, `drop`, `reject`) |
| &nbsp;&nbsp;&nbsp;&nbsp;`buffer_size` | integer | 버퍼 크기 |
| &nbsp;&nbsp;`queue_management` | object | 내부 큐 관리 설정 |
| &nbsp;&nbsp;&nbsp;&nbsp;`type` | string | 큐 유형 (`fifo`, `priority`) |
| &nbsp;&nbsp;&nbsp;&nbsp;`capacity` | integer | 큐 용량 |

#### 5.3 Distributed & HA (고가용성)
분산 처리 및 고가용성 구성을 설정합니다.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `distributed_ha` | object | 고가용성 설정 |
| &nbsp;&nbsp;`replicas` | integer | 복제본(Pod/Instance) 수 |
| &nbsp;&nbsp;`ha_profiling` | string | HA 모드 (`active_active`, `active_standby`) |
| &nbsp;&nbsp;`cluster_architecture` | string | 클러스터 아키텍처 유형 |
| &nbsp;&nbsp;`worker_pool` | object | 워커 풀 설정 |
| &nbsp;&nbsp;&nbsp;&nbsp;`min_workers` | integer | 최소 워커 수 |
| &nbsp;&nbsp;&nbsp;&nbsp;`max_workers` | integer | 최대 워커 수 |

#### 5.4 Stateful Recovery (상태 복구)
상태 저장 및 장애 복구 정책을 설정합니다.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `stateful_recovery` | object | 상태 복구 설정 |
| &nbsp;&nbsp;`check_point` | object | 체크포인트 설정 |
| &nbsp;&nbsp;&nbsp;&nbsp;`enabled` | boolean | 활성화 여부 |
| &nbsp;&nbsp;&nbsp;&nbsp;`interval` | string | 저장 간격 |
| &nbsp;&nbsp;`state_store` | object | 상태 저장소 설정 |
| &nbsp;&nbsp;&nbsp;&nbsp;`type` | string | 저장소 유형 (`redis`, `database`, `filesystem`) |
| &nbsp;&nbsp;&nbsp;&nbsp;`config` | object | 접속 정보 등 상세 설정 |
| &nbsp;&nbsp;`recovery_policy` | object | 복구 정책 |
| &nbsp;&nbsp;&nbsp;&nbsp;`action_on_failure` | string | 실패 시 동작 (`restart`, `skip`, `manual_intervention`) |

#### 5.5 Endpoint Exposure (엔드포인트 노출)
외부 접근을 위한 네트워크 설정을 정의합니다.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `endpoint_exposure` | object | 엔드포인트 설정 |
| &nbsp;&nbsp;`ingress_config` | object | 인그레스 설정 |
| &nbsp;&nbsp;&nbsp;&nbsp;`enabled` | boolean | 인그레스 생성 여부 |
| &nbsp;&nbsp;&nbsp;&nbsp;`host` | string | 호스트 도메인 |
| &nbsp;&nbsp;&nbsp;&nbsp;`path` | string | 경로 (Path) |
| &nbsp;&nbsp;`tls_setting` | object | TLS/SSL 보안 설정 |
| &nbsp;&nbsp;&nbsp;&nbsp;`enabled` | boolean | TLS 활성화 여부 |
| &nbsp;&nbsp;&nbsp;&nbsp;`cert_secret` | string | 인증서 시크릿 명 |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Runtime & Infra Architecture Configuration",
  "type": "object",
  "properties": {
    "deployment_target": {
      "type": "object",
      "properties": {
        "deployment_type": { "type": "string", "enum": ["kubernetes", "vm", "serverless"] },
        "region_location": { "type": "string" },
        "environment_id": { "type": "string" },
        "vpc_subnet_id": { "type": "string" }
      },
      "required": ["deployment_type", "region_location"]
    },
    "runtime_profile": {
      "type": "object",
      "properties": {
        "resource_allocation": {
          "type": "object",
          "properties": {
            "cpu_request": { "type": "string" },
            "cpu_limit": { "type": "string" },
            "memory_request": { "type": "string" },
            "memory_limit": { "type": "string" }
          }
        },
        "concurrency_control": {
          "type": "object",
          "properties": {
            "max_concurrent_requests": { "type": "integer" }
          }
        },
        "back_pressure": {
          "type": "object",
          "properties": {
            "strategy": { "type": "string", "enum": ["buffer", "drop", "reject"] },
            "buffer_size": { "type": "integer" }
          }
        },
        "queue_management": {
          "type": "object",
          "properties": {
            "type": { "type": "string" },
            "capacity": { "type": "integer" }
          }
        }
      }
    },
    "distributed_ha": {
      "type": "object",
      "properties": {
        "replicas": { "type": "integer", "minimum": 1 },
        "ha_profiling": { "type": "string", "enum": ["active_active", "active_standby"] },
        "cluster_architecture": { "type": "string" },
        "worker_pool": {
          "type": "object",
          "properties": {
            "min_workers": { "type": "integer" },
            "max_workers": { "type": "integer" }
          }
        }
      }
    },
    "stateful_recovery": {
      "type": "object",
      "properties": {
        "check_point": {
          "type": "object",
          "properties": {
            "enabled": { "type": "boolean" },
            "interval": { "type": "string" }
          }
        },
        "state_store": {
          "type": "object",
          "properties": {
            "type": { "type": "string" },
            "config": { "type": "object" }
          }
        },
        "recovery_policy": {
          "type": "object",
          "properties": {
            "action_on_failure": { "type": "string" }
          }
        }
      }
    },
    "endpoint_exposure": {
      "type": "object",
      "properties": {
        "ingress_config": {
          "type": "object",
          "properties": {
            "enabled": { "type": "boolean" },
            "host": { "type": "string" },
            "path": { "type": "string" }
          }
        },
        "tls_setting": {
          "type": "object",
          "properties": {
            "enabled": { "type": "boolean" },
            "cert_secret": { "type": "string" }
          }
        }
      }
    }
  },
  "required": ["deployment_target"]
}
```

### Sample Configuration

```json
{
  "deployment_target": {
    "deployment_type": "kubernetes",
    "region_location": "ap-northeast-2",
    "environment_id": "prod",
    "vpc_subnet_id": "subnet-0123456789"
  },
  "runtime_profile": {
    "resource_allocation": {
      "cpu_request": "500m",
      "cpu_limit": "2000m",
      "memory_request": "1Gi",
      "memory_limit": "4Gi"
    },
    "concurrency_control": {
      "max_concurrent_requests": 1000
    },
    "back_pressure": {
      "strategy": "reject",
      "buffer_size": 100
    },
    "queue_management": {
      "type": "priority",
      "capacity": 5000
    }
  },
  "distributed_ha": {
    "replicas": 3,
    "ha_profiling": "active_active",
    "cluster_architecture": "standard",
    "worker_pool": {
      "min_workers": 2,
      "max_workers": 10
    }
  },
  "stateful_recovery": {
    "check_point": {
      "enabled": true,
      "interval": "5m"
    },
    "state_store": {
      "type": "redis",
      "config": {
        "host": "redis-cluster.local",
        "port": 6379
      }
    },
    "recovery_policy": {
      "action_on_failure": "restart"
    }
  },
  "endpoint_exposure": {
    "ingress_config": {
      "enabled": true,
      "host": "api.example.com",
      "path": "/v1/service"
    },
    "tls_setting": {
      "enabled": true,
      "cert_secret": "wildcard-cert"
    }
  }
}
```
