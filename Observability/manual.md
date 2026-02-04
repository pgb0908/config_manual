# iPaaS Observability 설정 매뉴얼

이 매뉴얼은 iPaaS 플랫폼의 가시성(Observability) 설정을 위한 가이드입니다. 텔레메트리 수집, 추적 정책, 이상 탐지 및 운영 분석에 대한 상세 설정을 다룹니다.

모든 설정 필드는 `snake_case`를 따르며, 설명은 한글로 작성되었습니다.

---

## 1. 텔레메트리 (Telemetry)

OpenTelemetry 기반의 데이터 수집 엔드포인트 및 기본 정책을 설정합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `otel_collector_url` | String | Yes | OpenTelemetry Collector의 URL입니다. |
| `service_namespace` | String | Yes | 메트릭 및 트레이스에 태깅될 서비스 네임스페이스입니다. |
| `trace_sampling_ratio` | Number | No | 트레이스 데이터 수집 샘플링 비율(0.0 ~ 1.0)입니다. (기본값: 1.0) |
| `export_protocol` | String | No | 데이터 전송 프로토콜입니다. (`grpc`, `http/protobuf`, `http/json` 중 선택) |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "otel_collector_url": { "type": "string", "format": "uri" },
    "service_namespace": { "type": "string" },
    "trace_sampling_ratio": { "type": "number", "minimum": 0.0, "maximum": 1.0, "default": 1.0 },
    "export_protocol": { "type": "string", "enum": ["grpc", "http/protobuf", "http/json"], "default": "grpc" }
  },
  "required": ["otel_collector_url", "service_namespace"]
}
```

### Sample Configuration

```json
{
  "otel_collector_url": "grpc://otel-collector.monitoring.svc:4317",
  "service_namespace": "ipaas-prod-cluster-1",
  "trace_sampling_ratio": 0.5,
  "export_protocol": "grpc"
}
```

---

## 2. 수집 대상 정책 (Collection Target Policy)

Gateway 및 Flow 레벨에서의 상세 데이터 수집 항목을 제어합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `gateway` | Object | Yes | API Gateway 관련 수집 설정입니다. |
| `gateway.access_logging` | Boolean | Yes | API 접근 로그(Access Log) 수집 활성화 여부입니다. |
| `gateway.policy_metrics` | Boolean | No | 적용된 정책(Rate Limit 등) 관련 메트릭 수집 여부입니다. |
| `gateway.payload_capture` | Object | No | 요청/응답 페이로드 캡처 설정입니다. |
| `gateway.payload_capture.enabled` | Boolean | Yes | 페이로드 캡처 활성화 여부입니다. |
| `gateway.payload_capture.max_size_bytes` | Integer | No | 캡처할 최대 페이로드 크기(바이트)입니다. |
| `flow` | Object | Yes | 통합 플로우(Integration Flow) 관련 수집 설정입니다. |
| `flow.execution_tracing` | Boolean | Yes | 플로우 실행 단계별 트레이싱 활성화 여부입니다. |
| `flow.component_metrics` | Boolean | No | 각 컴포넌트(노드)별 처리 시간 및 성공/실패 메트릭 수집 여부입니다. |
| `flow.custom_logs` | Boolean | No | 사용자가 정의한 커스텀 로그 수집 허용 여부입니다. |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "gateway": {
      "type": "object",
      "properties": {
        "access_logging": { "type": "boolean" },
        "policy_metrics": { "type": "boolean" },
        "payload_capture": {
          "type": "object",
          "properties": {
            "enabled": { "type": "boolean" },
            "max_size_bytes": { "type": "integer" }
          },
          "required": ["enabled"]
        }
      },
      "required": ["access_logging"]
    },
    "flow": {
      "type": "object",
      "properties": {
        "execution_tracing": { "type": "boolean" },
        "component_metrics": { "type": "boolean" },
        "custom_logs": { "type": "boolean" }
      },
      "required": ["execution_tracing"]
    }
  },
  "required": ["gateway", "flow"]
}
```

### Sample Configuration

```json
{
  "gateway": {
    "access_logging": true,
    "policy_metrics": true,
    "payload_capture": {
      "enabled": true,
      "max_size_bytes": 1024
    }
  },
  "flow": {
    "execution_tracing": true,
    "component_metrics": true,
    "custom_logs": false
  }
}
```

---

## 3. 분산 추적 (Distributed Tracing)

마이크로서비스 간의 트랜잭션 추적을 위한 컨텍스트 전파 및 ID 패턴을 설정합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `context_propagation` | Array | Yes | 사용할 컨텍스트 전파 표준입니다. (예: `w3c`, `b3`) |
| `correlation_id_pattern` | String | No | 로그 및 트레이스 연결을 위한 상관관계 ID(Correlation ID)의 헤더명 또는 패턴입니다. |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "context_propagation": {
      "type": "array",
      "items": { "type": "string", "enum": ["w3c", "b3", "jaeger"] }
    },
    "correlation_id_pattern": { "type": "string" }
  },
  "required": ["context_propagation"]
}
```

### Sample Configuration

```json
{
  "context_propagation": ["w3c", "b3"],
  "correlation_id_pattern": "X-Correlation-ID"
}
```

---

## 4. 탐지 및 예측 (Detection & Prediction)

이상 징후 탐지 알고리즘 및 예측 분석을 위한 파라미터를 설정합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `analyzer_type` | String | Yes | 이상 탐지에 사용할 알고리즘 유형입니다. (예: `statistical`, `ai_based`) |
| `fluctuation_multiplier` | Number | No | 통계적 이상치 판단을 위한 변동폭 배수(Multiplier)입니다. (일반적으로 표준편차의 배수) |
| `risk_score_threshold` | Integer | Yes | 알림을 발생시킬 위험 점수 임계값(0~100)입니다. |
| `prediction_horizon` | String | No | 미래 상태를 예측할 시간 범위입니다. (예: `1h`, `24h`) |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "analyzer_type": { "type": "string", "enum": ["statistical", "ai_based", "fixed_threshold"] },
    "fluctuation_multiplier": { "type": "number", "minimum": 1.0 },
    "risk_score_threshold": { "type": "integer", "minimum": 0, "maximum": 100 },
    "prediction_horizon": { "type": "string", "pattern": "^\\d+[hmd]$" }
  },
  "required": ["analyzer_type", "risk_score_threshold"]
}
```

### Sample Configuration

```json
{
  "analyzer_type": "ai_based",
  "fluctuation_multiplier": 3.0,
  "risk_score_threshold": 80,
  "prediction_horizon": "6h"
}
```

---

## 5. 운영 분석 (Operational Analytics)

시스템 성능 병목 분석 및 상관관계 분석을 위한 기준을 설정합니다.

### Configuration Fields

| 필드명 (Field) | 타입 (Type) | 필수 (Required) | 설명 (Description) |
| :--- | :--- | :--- | :--- |
| `bottleneck_criteria` | Object | Yes | 병목 현상 판단 기준입니다. |
| `bottleneck_criteria.latency_threshold_ms` | Integer | Yes | 병목으로 간주할 응답 지연 시간(ms)입니다. |
| `bottleneck_criteria.cpu_usage_percent` | Integer | No | 병목으로 간주할 CPU 사용률(%)입니다. |
| `top_n_analysis` | Integer | No | 분석 리포트에서 보여줄 상위 N개의 문제 항목 수입니다. (기본값: 10) |
| `correlation_with_error` | Boolean | Yes | 성능 저하와 에러 발생 간의 상관관계 분석 활성화 여부입니다. |

### JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "bottleneck_criteria": {
      "type": "object",
      "properties": {
        "latency_threshold_ms": { "type": "integer" },
        "cpu_usage_percent": { "type": "integer", "maximum": 100 }
      },
      "required": ["latency_threshold_ms"]
    },
    "top_n_analysis": { "type": "integer", "default": 10 },
    "correlation_with_error": { "type": "boolean" }
  },
  "required": ["bottleneck_criteria", "correlation_with_error"]
}
```

### Sample Configuration

```json
{
  "bottleneck_criteria": {
    "latency_threshold_ms": 2000,
    "cpu_usage_percent": 85
  },
  "top_n_analysis": 5,
  "correlation_with_error": true
}
```
