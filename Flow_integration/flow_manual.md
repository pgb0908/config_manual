# Flow Integration Configuration Manual

이 문서는 Flow Integration 설정에 대한 상세 매뉴얼입니다. 각 설정 항목은 기능별로 분류되어 있으며, 필드명은 `snake_case` 규칙을 따르는 영어로, 설명은 한국어로 작성되었습니다.

## 목차
1. [Trigger (트리거)](#1-trigger-트리거)
    - [Polling](#11-polling)
    - [Webhook](#12-webhook)
    - [Scheduler](#13-scheduler)
    - [Event Listener](#14-event-listener)
2. [Orchestration (오케스트레이션)](#2-orchestration-오케스트레이션)
    - [Condition](#21-condition)
    - [Iteration](#22-iteration)
    - [Execution Exception Policies](#23-execution-exception-policies)
    - [Distributed Transactions & SAGA](#24-distributed-transactions--saga)
    - [Idempotency Control](#25-idempotency-control)
3. [Resiliency & Retry (복원력 및 재시도)](#3-resiliency--retry-복원력-및-재시도)

---

## 1. Trigger (트리거)
Flow가 실행되는 시작점을 정의합니다.

### 1.1 Polling
주기적으로 소스 시스템을 조회하여 데이터를 가져오는 방식입니다.

**Configuration Fields**

| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `interval` | Integer | 폴링 주기 (초 단위). |
| `batch_size` | Integer | 한 번의 폴링으로 가져올 최대 레코드 수. |
| `paging_token` | String | 페이징 처리를 위한 토큰 식별자. 다음 페이지를 요청할 때 사용됩니다. |
| `deduplication_key` | String | 중복 데이터를 방지하기 위한 고유 키 필드명. |

**JSON Schema Sample**
```json
{
  "trigger": {
    "type": "polling",
    "config": {
      "interval": 60,
      "batch_size": 100,
      "paging_token": "next_page_cursor",
      "deduplication_key": "order_id"
    }
  }
}
```

### 1.2 Webhook
외부 시스템으로부터 HTTP 요청을 수신하여 Flow를 트리거합니다.

**Configuration Fields**

| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `webhook_subscribe` | Object | 웹훅 등록을 위한 설정 객체. (자동 등록이 필요한 경우) |
| `webhook_subscribe.url` | String | 구독 요청을 보낼 대상 URL. |
| `webhook_subscribe.method` | String | 구독 요청 HTTP 메서드 (GET, POST 등). |
| `webhook_unsubscribe` | Object | 웹훅 구독 해제를 위한 설정 객체. |
| `webhook_response_type` | String | 웹훅 수신 시 응답 방식 (`sync`: 동기, `async`: 비동기). |

**JSON Schema Sample**
```json
{
  "trigger": {
    "type": "webhook",
    "config": {
      "webhook_subscribe": {
        "url": "https://api.external.com/webhooks/register",
        "method": "POST"
      },
      "webhook_unsubscribe": {
        "url": "https://api.external.com/webhooks/unregister",
        "method": "DELETE"
      },
      "webhook_response_type": "async"
    }
  }
}
```

### 1.3 Scheduler
정해진 일정에 따라 Flow를 실행합니다.

**Configuration Fields**

| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `cron_expression` | String | 실행 주기를 정의하는 Cron 표현식. |
| `timezone` | String | 스케줄 기준 시간대 (예: "Asia/Seoul"). |
| `start_date` | String | 스케줄 시작 일시 (ISO8601 형식). |
| `end_date` | String | 스케줄 종료 일시 (ISO8601 형식). |

**JSON Schema Sample**
```json
{
  "trigger": {
    "type": "scheduler",
    "config": {
      "cron_expression": "0 0 12 * * ?",
      "timezone": "Asia/Seoul",
      "start_date": "2023-01-01T00:00:00Z",
      "end_date": "2023-12-31T23:59:59Z"
    }
  }
}
```

### 1.4 Event Listener
메시지 큐나 이벤트 스트림으로부터 이벤트를 수신합니다.

**Configuration Fields**

| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `source_type` | String | 이벤트 소스 유형 (예: "kafka", "rabbitmq"). |
| `topic_name` | String | 구독할 토픽 또는 큐 이름. |
| `consumer_group` | String | 컨슈머 그룹 ID. |
| `ack_mode` | String | 메시지 수신 확인 모드 (`auto`, `manual`). |

**JSON Schema Sample**
```json
{
  "trigger": {
    "type": "event_listener",
    "config": {
      "source_type": "kafka",
      "topic_name": "user-events",
      "consumer_group": "flow-integration-group",
      "ack_mode": "manual"
    }
  }
}
```

---

## 2. Orchestration (오케스트레이션)
데이터의 흐름 제어, 반복, 예외 처리 등을 담당합니다.

### 2.1 Condition
조건에 따라 실행 경로를 분기합니다.

**Configuration Fields**

| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `expression` | String | 조건을 평가할 표현식 (예: `input.amount > 1000`). |
| `route_order` | Integer | 조건 평가 순서. |
| `otherwise` | Boolean | 상위 조건들이 모두 거짓일 경우 실행되는 기본 경로 여부. |

**JSON Schema Sample**
```json
{
  "step": "choice",
  "conditions": [
    {
      "expression": "payload.status == 'active'",
      "route_order": 1,
      "next_step": "process_active"
    },
    {
      "otherwise": true,
      "next_step": "process_default"
    }
  ]
}
```

### 2.2 Iteration
컬렉션 데이터를 반복 처리합니다.

**Configuration Fields**

**For-Each / Parallel For-Each**
| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `type` | String | 반복 유형 (`for_each`, `parallel_for_each`, `batch_job`). |
| `collection_expression` | String | 반복할 대상 컬렉션을 가리키는 표현식. |
| `item_variable` | String | 각 항목을 참조할 변수명. |
| `max_concurrency` | Integer | (Parallel For-Each 전용) 최대 동시 실행 스레드 수. |

**Batch Job**
| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `batch_block_size` | Integer | 배치 처리를 위한 블록 크기. |
| `aggregator_size` | Integer | 처리 결과를 집계할 크기. |

**JSON Schema Sample**
```json
{
  "step": "iteration",
  "config": {
    "type": "parallel_for_each",
    "collection_expression": "payload.items",
    "item_variable": "item",
    "max_concurrency": 10
  }
}
```

### 2.3 Execution Exception Policies
실행 중 오류 발생 시 처리 정책을 정의합니다.

**Configuration Fields**

| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `retry_strategy` | String | 재시도 전략 (`fixed`, `exponential_backoff`). |
| `max_retries` | Integer | 최대 재시도 횟수. |
| `max_failed_records` | Integer | 허용 가능한 최대 실패 레코드 수 (배치 처리 시). |
| `error_behavior` | String | 오류 발생 시 동작 (`propagate`: 오류 전파, `continue`: 무시하고 계속, `dead_letter`: 별도 저장). |

**JSON Schema Sample**
```json
{
  "exception_policy": {
    "retry_strategy": "exponential_backoff",
    "max_retries": 3,
    "max_failed_records": 0,
    "error_behavior": "propagate"
  }
}
```

### 2.4 Distributed Transactions & SAGA
분산 트랜잭션 및 SAGA 패턴을 관리합니다.

**Configuration Fields**

| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `saga_coordination` | Boolean | SAGA 패턴 사용 여부. |
| `compensation_flow_id` | String | 실패 시 실행할 보상 트랜잭션(Rollback) Flow ID. |
| `is_pivot_step` | Boolean | 트랜잭션의 결정적 단계(Pivot) 여부. 이 단계 성공 후에는 보상이 불가능할 수 있음. |
| `txn_timeout_ms` | Integer | 트랜잭션 전체 타임아웃 (밀리초). |

**JSON Schema Sample**
```json
{
  "transaction": {
    "saga_coordination": true,
    "compensation_flow_id": "flow_rollback_order",
    "is_pivot_step": false,
    "txn_timeout_ms": 5000
  }
}
```

### 2.5 Idempotency Control
동일한 요청의 중복 처리를 방지합니다.

**Configuration Fields**

| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `idempotency_key` | String | 멱등성 보장을 위한 고유 키 (Header 또는 Payload 필드). |
| `store_reference` | String | 키와 응답을 저장할 상태 저장소 참조명. |
| `cache_previous_response` | Boolean | 이전 응답을 캐싱하여 동일 요청 시 반환할지 여부. |

**JSON Schema Sample**
```json
{
  "idempotency": {
    "idempotency_key": "header.X-Request-ID",
    "store_reference": "redis_store_01",
    "cache_previous_response": true
  }
}
```

---

## 3. Resiliency & Retry (복원력 및 재시도)
시스템 안정성을 위한 재시도 및 복원 설정을 정의합니다. Trigger나 개별 Step의 하위 설정으로 사용될 수 있습니다.

**Configuration Fields**

| 필드명 (Field Name) | 데이터 타입 (Data Type) | 설명 (Description) |
| :--- | :--- | :--- |
| `max_retries` | Integer | 최대 재시도 횟수. |
| `interval` | Integer | 재시도 간격 (밀리초). |
| `retry_if` | String/Array | 재시도를 수행할 조건 또는 에러 코드 목록. |

**JSON Schema Sample**
```json
{
  "resiliency": {
    "max_retries": 5,
    "interval": 2000,
    "retry_if": ["TIMEOUT", "503_SERVICE_UNAVAILABLE"]
  }
}
```
