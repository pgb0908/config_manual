# iPaaS Flow Integration Configuration Manual

본 문서는 iPaaS 환경에서 사용되는 Flow Integration Configuration에 대한 상세 매뉴얼입니다.
각 설정 항목은 기능별로 구분되어 있으며, JSON 스키마, 항목별 설명, 그리고 샘플 구성을 포함합니다.

---

## 1. Trigger (트리거 설정)

플로우(Flow) 실행을 시작하게 하는 이벤트 소스 및 관련 설정입니다.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "trigger_config": {
      "type": "object",
      "properties": {
        "polling": {
          "type": "object",
          "properties": {
            "interval": { "type": "integer", "description": "폴링 주기(초)입니다." },
            "batch_size": { "type": "integer", "description": "한 번에 처리할 최대 레코드 수입니다." },
            "paging_token": { "type": "string", "description": "다음 페이지 조회를 위한 토큰 키입니다." },
            "deduplication_key": { "type": "string", "description": "중복 처리를 방지하기 위한 고유 키 필드입니다." }
          }
        },
        "webhook": {
          "type": "object",
          "properties": {
            "webhook_subscribe": { "type": "string", "description": "웹훅 구독 등록을 위한 URL 또는 설정입니다." },
            "webhook_unsubscribe": { "type": "string", "description": "웹훅 구독 해제를 위한 URL 또는 설정입니다." },
            "webhook_response_type": { "type": "string", "enum": ["sync", "async"], "description": "웹훅 수신 시 응답 방식입니다." }
          }
        },
        "scheduler": {
          "type": "object",
          "description": "CRON 표현식 등을 이용한 정기 실행 스케줄러 설정입니다."
        },
        "event_listener": {
          "type": "object",
          "description": "실시간 이벤트 스트림(Kafka, AMQP 등)을 수신하기 위한 리스너 설정입니다."
        }
      }
    }
  }
}
```

### 항목 설명

| 필드명 (Field) | 데이터 타입 (Type) | 설명 (Description) |
| :--- | :--- | :--- |
| **Polling** | | 주기적으로 외부 시스템을 조회하여 변경 사항을 감지하는 방식입니다. |
| `interval` | Integer | 폴링 간격(초 단위)을 설정합니다. |
| `batch_size` | Integer | 한 번의 폴링 사이클에서 가져올 최대 데이터 개수입니다. |
| `paging_token` | String | 대량 데이터 조회 시 페이지네이션을 처리하기 위한 토큰의 필드명입니다. |
| `deduplication_key` | String | 이미 처리된 데이터를 식별하고 중복 실행을 막기 위해 사용하는 고유 키(ID) 필드명입니다. |
| **Webhook** | | 외부 시스템에서 발생하는 이벤트를 HTTP 요청으로 수신하는 방식입니다. |
| `webhook_subscribe` | String | 웹훅을 활성화하기 위해 호출해야 할 API 정보나 구독 URL입니다. |
| `webhook_unsubscribe` | String | 웹훅 구독을 중지하거나 해제할 때 사용하는 설정입니다. |
| `webhook_response_type` | String | 웹훅 수신 시 동기(Sync)로 결과를 반환할지, 비동기(Async)로 수신 확인만 보낼지 결정합니다. |
| **Scheduler** | Object | 특정 시간이나 주기에 맞춰 플로우를 실행합니다. (예: `cron: "0 0 * * *"`) |
| **Event Listener** | Object | 메시지 큐나 이벤트 버스로부터 메시지를 실시간으로 소비(Consume)하는 리스너 설정입니다. |

### Sample Configuration

```json
{
  "trigger_config": {
    "polling": {
      "interval": 60,
      "batch_size": 100,
      "paging_token": "next_page_cursor",
      "deduplication_key": "order_id"
    },
    "webhook": {
      "webhook_subscribe": "https://api.external.com/hooks/subscribe",
      "webhook_unsubscribe": "https://api.external.com/hooks/unsubscribe",
      "webhook_response_type": "async"
    }
  }
}
```

---

## 2. Orchestration (오케스트레이션)

플로우 내부의 로직 제어, 반복, 트랜잭션 관리 등을 담당하는 설정입니다.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "orchestration_config": {
      "type": "object",
      "properties": {
        "condition": {
          "type": "object",
          "properties": {
            "expression": { "type": "string", "description": "조건 판단을 위한 표현식입니다." },
            "route_order": { "type": "array", "items": { "type": "string" }, "description": "조건 평가 순서입니다." },
            "otherwise": { "type": "object", "description": "모든 조건이 일치하지 않을 때 실행할 경로입니다." }
          }
        },
        "iteration": {
          "type": "object",
          "properties": {
            "for_each": { "type": "object", "description": "순차적 반복 처리를 위한 설정입니다." },
            "parallel_for_each": { "type": "object", "description": "병렬 반복 처리를 위한 설정입니다." },
            "batch_job": { "type": "object", "description": "대용량 배치 처리를 위한 설정입니다." }
          }
        },
        "execution_exception_policies": {
          "type": "object",
          "properties": {
            "retry_strategy": { "type": "string", "description": "예외 발생 시 재시도 전략입니다." },
            "max_retries": { "type": "integer", "description": "최대 재시도 횟수입니다." },
            "max_failed_records": { "type": "integer", "description": "허용되는 최대 실패 레코드 수입니다." },
            "error_behavior": { "type": "string", "enum": ["continue", "fail"], "description": "오류 발생 시 플로우 진행 여부입니다." }
          }
        },
        "distributed_transactions_saga": {
          "type": "object",
          "properties": {
            "saga_coordination": { "type": "boolean", "description": "SAGA 패턴 사용 여부입니다." },
            "compensation_flow_id": { "type": "string", "description": "롤백 시 실행할 보상 트랜잭션 플로우 ID입니다." },
            "is_pivot_step": { "type": "boolean", "description": "트랜잭션의 커밋/롤백 기준이 되는 피벗 단계 여부입니다." },
            "txn_timeout_ms": { "type": "integer", "description": "트랜잭션 타임아웃 시간(ms)입니다." }
          }
        },
        "idempotency_control": {
          "type": "object",
          "properties": {
            "idempotency_key": { "type": "string", "description": "멱등성을 보장하기 위한 유니크 키입니다." },
            "state_store_ref": { "type": "string", "description": "상태 저장을 위한 저장소 참조값입니다." },
            "cache_previous_response": { "type": "boolean", "description": "이전 응답 캐싱 사용 여부입니다." }
          }
        }
      }
    }
  }
}
```

### 항목 설명

| 필드명 (Field) | 데이터 타입 (Type) | 설명 (Description) |
| :--- | :--- | :--- |
| **Condition** | | 조건 분기 처리를 위한 설정입니다. |
| `expression` | String | 데이터 값을 기반으로 참/거짓을 판별하는 표현식(Expression)입니다. |
| `route_order` | Array | 여러 조건이 있을 때 평가할 우선순위를 정의합니다. |
| `otherwise` | Object | `expression`이 모두 `false`일 때 실행될 기본 경로(Default Route)입니다. |
| **Iteration** | | 데이터 컬렉션을 반복 처리하는 방식입니다. |
| `for_each` | Object | 데이터를 하나씩 순차적으로 처리합니다. |
| `parallel_for_each`| Object | 데이터를 병렬 스레드로 분산하여 처리합니다. |
| `batch_job` | Object | 대량의 데이터를 청크(Chunk) 단위로 나누어 처리하는 배치 작업 설정입니다. |
| **Execution Exception Policies** | | 실행 중 오류가 발생했을 때의 처리 정책입니다. |
| `retry_strategy` | String | 재시도 간격 알고리즘(예: `fixed`, `exponential`)을 설정합니다. |
| `max_retries` | Integer | 오류 발생 시 최대 몇 번까지 재시도할지 설정합니다. |
| `max_failed_records`| Integer | 배치 처리 등에서 허용할 수 있는 최대 실패 건수입니다. 이 수를 넘으면 전체 작업을 실패 처리합니다. |
| `error_behavior` | String | 오류 발생 시 플로우를 즉시 중단(`fail`)할지, 오류를 무시하고 다음 단계로 진행(`continue`)할지 설정합니다. |
| **Distributed Transactions & SAGA** | | 분산 환경에서의 트랜잭션 관리 설정입니다. |
| `saga_coordination`| Boolean | SAGA 패턴을 통한 분산 트랜잭션 코디네이션을 활성화합니다. |
| `compensation_flow_id` | String | 트랜잭션 실패(롤백) 시 실행해야 할 보상(Compensation) 플로우의 ID입니다. |
| `is_pivot_step` | Boolean | 이 단계가 성공하면 전체 트랜잭션이 성공한 것으로 간주하는 피벗 포인트인지 여부입니다. |
| `txn_timeout_ms` | Integer | 전체 트랜잭션이 완료되어야 하는 제한 시간입니다. |
| **Idempotency Control** | | 중복 실행 방지(멱등성) 제어 설정입니다. |
| `idempotency_key` | String | 요청의 고유성을 식별하는 키입니다. |
| `state_store_ref` | String | 멱등성 체크를 위해 처리 상태를 저장할 저장소(Redis, DB 등)의 참조 이름입니다. (스키마에서는 `상태저장소 참조`를 `state_store_ref`로 영문 표기) |
| `cache_previous_response` | Boolean | 동일한 키로 요청이 오면 로직을 재실행하지 않고 저장된 응답을 반환할지 여부입니다. |

### Sample Configuration

```json
{
  "orchestration_config": {
    "condition": {
      "expression": "#[payload.amount > 1000]",
      "otherwise": { "action": "default_logger" }
    },
    "iteration": {
      "parallel_for_each": {
        "max_concurrency": 10
      }
    },
    "execution_exception_policies": {
      "retry_strategy": "exponential_backoff",
      "max_retries": 3,
      "error_behavior": "fail"
    },
    "distributed_transactions_saga": {
      "saga_coordination": true,
      "compensation_flow_id": "flow_refund_process",
      "txn_timeout_ms": 5000
    },
    "idempotency_control": {
      "idempotency_key": "#[header.request_id]",
      "state_store_ref": "redis_store_1",
      "cache_previous_response": true
    }
  }
}
```

---

## 3. Resiliency & Retry (복원력 및 재시도)

일시적인 장애 상황에서 시스템의 안정성을 유지하기 위한 재시도 설정입니다.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "resiliency_config": {
      "type": "object",
      "properties": {
        "max_retries": {
          "type": "integer",
          "description": "최대 재시도 횟수입니다."
        },
        "interval": {
          "type": "integer",
          "description": "재시도 사이의 대기 시간(밀리초)입니다."
        },
        "retry_if": {
          "type": "string",
          "description": "재시도를 수행할 특정 조건이나 예외 유형입니다."
        }
      }
    }
  }
}
```

### 항목 설명

| 필드명 (Field) | 데이터 타입 (Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `max_retries` | Integer | (스키마의 `maxRetries`를 `max_retries`로 통일) 실패 시 재시도할 최대 횟수입니다. |
| `interval` | Integer | 각 재시도 시도 사이의 간격(ms)입니다. |
| `retry_if` | String | 어떤 종류의 에러나 조건에서 재시도를 할지 정의합니다. (예: `NetworkException`, `HTTP 5xx`) |

### Sample Configuration

```json
{
  "resiliency_config": {
    "max_retries": 5,
    "interval": 2000,
    "retry_if": "ConnectivityError"
  }
}
```
