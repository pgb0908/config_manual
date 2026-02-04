# QA & TEST Configuration

QA & TEST 모듈은 iPaaS 내에서 테스트 환경, 모킹(Mocking), 테스트 슈트 및 회귀 테스트(Regression Test)를 관리하는 설정입니다. 모든 설정은 `qa_and_test`라는 최상위 객체 아래에 구성됩니다.

## 1. Test Mode & Environment (`test_mode_env`)

테스트 실행 환경 및 데이터 정책을 설정하는 섹션입니다.

| 필드명 | 타입 | 설명 |
| :--- | :--- | :--- |
| `test_env_type` | String | 테스트가 실행될 환경 유형입니다. (`LOCAL`, `STAGING`, `SANDBOX`) |
| `test_data_policy` | String | 테스트 데이터 처리 정책입니다. (`MOCK`, `SYNTHETIC`, `ANONYMIZED_PROD`) |
| `test_timeout_ms` | Integer | 전체 테스트 실행의 최대 제한 시간(밀리초)입니다. |
| `store_test_history` | Boolean | 테스트 실행 이력을 저장할지 여부입니다. |

#### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "test_env_type": { "type": "string", "enum": ["LOCAL", "STAGING", "SANDBOX"] },
    "test_data_policy": { "type": "string", "enum": ["MOCK", "SYNTHETIC", "ANONYMIZED_PROD"] },
    "test_timeout_ms": { "type": "integer" },
    "store_test_history": { "type": "boolean" }
  }
}
```

#### Sample
```json
{
  "test_env_type": "STAGING",
  "test_data_policy": "ANONYMIZED_PROD",
  "test_timeout_ms": 60000,
  "store_test_history": true
}
```

---

## 2. Mocking (`mocking`)

외부 시스템 의존성을 제거하고 테스트하기 위한 모킹 설정입니다.

| 필드명 | 타입 | 설명 |
| :--- | :--- | :--- |
| `mock_when.condition` | String | 모킹 응답을 반환할 트리거 조건 표현식입니다. (예: `header.x-test-id == '123'`) |
| `stub_payload` | Object | 모의 응답으로 반환할 JSON 데이터입니다. |
| `throw_error_code` | Integer | (선택) 강제로 에러를 발생시키고 싶을 때 설정하는 HTTP 상태 코드입니다. |
| `latency_ms` | Integer | 응답 지연 시간(밀리초)을 설정하여 네트워크 지연을 시뮬레이션합니다. |

#### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "mock_when": {
      "type": "object",
      "properties": {
        "condition": { "type": "string" }
      }
    },
    "stub_payload": { "type": "object" },
    "throw_error_code": { "type": "integer" },
    "latency_ms": { "type": "integer" }
  }
}
```

#### Sample
```json
{
  "mock_when": {
    "condition": "request.query_param.mock == 'true'"
  },
  "stub_payload": {
    "status": "success",
    "data": { "id": 1, "name": "Mock Item" }
  },
  "throw_error_code": null,
  "latency_ms": 200
}
```

---

## 3. Test Suites & Regression (`test_suites_regression`)

테스트 케이스 정의, 회귀 테스트 그룹 관리, 데이터 검증 및 파라미터 테스트를 위한 설정입니다.

| 필드명 | 타입 | 설명 |
| :--- | :--- | :--- |
| `test_case.test_case_id` | String | 테스트 케이스의 고유 식별자입니다. |
| `test_case.input_data` | Object | 테스트에 사용할 입력 데이터 세트입니다. |
| `test_case.expected_output` | Object | 예상되는 결과 데이터 세트입니다. |
| `reg_set.regression_group` | String | 이 테스트가 속한 회귀 테스트 그룹명입니다. (예: `SMOKE_TEST`, `NIGHTLY_BUILD`) |
| `data_validation.validation_rule` | String | 결과 데이터 검증 규칙입니다. (`EXACT_MATCH`, `SCHEMA_MATCH`, `CONTAINS`) |
| `data_validation.ignore_fields` | Array | 검증 시 무시할 필드 목록입니다. (예: 타임스탬프 등) |
| `parameterized_test.enabled` | Boolean | 매개변수 기반 테스트(Data-Driven Test) 활성화 여부입니다. |
| `parameterized_test.data_source` | String | 테스트 데이터를 불러올 소스입니다. (`CSV`, `JSON_FILE`, `INLINE`) |

#### JSON Schema
```json
{
  "type": "object",
  "properties": {
    "test_case": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "test_case_id": { "type": "string" },
          "input_data": { "type": "object" },
          "expected_output": { "type": "object" }
        }
      }
    },
    "reg_set": {
      "type": "object",
      "properties": {
        "regression_group": { "type": "string" }
      }
    },
    "data_validation": {
      "type": "object",
      "properties": {
        "validation_rule": { "type": "string", "enum": ["EXACT_MATCH", "SCHEMA_MATCH", "CONTAINS"] },
        "ignore_fields": { "type": "array", "items": { "type": "string" } }
      }
    },
    "parameterized_test": {
      "type": "object",
      "properties": {
        "enabled": { "type": "boolean" },
        "data_source": { "type": "string", "enum": ["CSV", "JSON_FILE", "INLINE"] }
      }
    }
  }
}
```

#### Sample
```json
{
  "test_case": [
    {
      "test_case_id": "TC-001",
      "input_data": { "amount": 100, "currency": "USD" },
      "expected_output": { "status": "approved" }
    }
  ],
  "reg_set": {
    "regression_group": "SMOKE_TEST"
  },
  "data_validation": {
    "validation_rule": "SCHEMA_MATCH",
    "ignore_fields": ["created_at", "updated_at"]
  },
  "parameterized_test": {
    "enabled": true,
    "data_source": "INLINE"
  }
}
```
