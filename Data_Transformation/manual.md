# Data Transformation Configuration Manual

이 문서는 Data Transformation 모듈의 설정 파일에 대한 상세 매뉴얼입니다.
본 설정은 iPaaS 환경에서 데이터 스키마 정의, 필드 매핑, 그리고 데이터 유효성 검증 기능을 정의합니다.

모든 설정 필드는 `snake_case`를 따르며, JSON 형식으로 작성됩니다.

---

## Configuration Structure

설정은 크게 3가지 주요 섹션으로 구분됩니다.

1.  **Schemas (`schemas`)**: 데이터 구조, 표준 모델, 데이터 타입 변환을 정의합니다.
2.  **Field Mapping (`field_mapping`)**: 소스-타겟 간 필드 매핑, 표현식 언어, 참조 테이블, AI 기반 스마트 매핑을 설정합니다.
3.  **Validation (`validation`)**: 데이터 무결성 및 포맷 검증 규칙을 설정합니다.

---

## Detailed Configuration Fields

### 1. Schemas (`schemas`)

데이터 변환의 기준이 되는 스키마 정보를 정의합니다.

*   **`schema_definition`**: 소스 및 타겟 데이터 구조 정의
    *   `source_schema_url` (string): 소스 데이터 스키마 파일 URL (JSON Schema, AVRO 등)
    *   `target_schema_url` (string): 타겟 데이터 스키마 파일 URL
    *   `strict_mode` (boolean): 정의된 스키마와 정확히 일치해야 하는지 여부
*   **`canonical_model`**: 내부 표준 데이터 모델 (Canonical Model) 설정
    *   `use_canonical` (boolean): 표준 모델 사용 여부
    *   `model_id` (string): 사용할 표준 모델의 식별자
    *   `version` (string): 표준 모델 버전
*   **`type_casting`**: 데이터 타입 자동 변환 규칙
    *   `rules` (array): 변환 규칙 배열
        *   `source_type` (string): 원본 데이터 타입 (예: "string_date")
        *   `target_type` (string): 목표 데이터 타입 (예: "timestamp")
        *   `format` (string): 변환 포맷 (예: "yyyy-MM-dd HH:mm:ss")
        *   `on_error` (string): 변환 실패 시 처리 방식 ("fail", "set_null", "default_value")

### 2. Field Mapping (`field_mapping`)

데이터 필드 간의 매핑 로직을 설정합니다.

*   **`visual_mapping`**: UI 기반 매핑 정보 (JSON 표현)
    *   `mapping_id` (string): 매핑 정의 ID
    *   `definitions` (array): 매핑 상세 정의
        *   `source_path` (string): 소스 필드 경로 (JSONPath)
        *   `target_path` (string): 타겟 필드 경로 (JSONPath)
*   **`expression_language`**: 복잡한 변환을 위한 표현식 언어 설정
    *   `language` (string): 사용할 표현식 언어 ("spel", "jq", "javascript")
    *   `scripts` (object): 필드별 변환 스크립트 맵 (Key: target_path, Value: script)
*   **`lookup_tables`**: 코드 변환 등을 위한 참조 테이블
    *   `tables` (array): 참조 테이블 목록
        *   `name` (string): 테이블 이름
        *   `data_source` (string): 데이터 소스 ("csv", "database", "api")
        *   `cache_ttl_seconds` (integer): 캐시 유효 시간 (초)
*   **`smart_mapping`**: AI 기반 자동 매핑 추천 설정
    *   `enabled` (boolean): 스마트 매핑 활성화 여부
    *   `confidence_score_threshold` (number): 매핑 적용을 위한 최소 신뢰도 점수 (0.0 ~ 1.0)

### 3. Validation (`validation`)

변환 전후의 데이터 유효성을 검증합니다.

*   **`required_fields`**: 필수 필드 검사
    *   `fields` (array): 필수 필드 경로 목록
    *   `check_null` (boolean): Null 값 허용 여부 (False일 경우 Null 불가)
    *   `check_empty` (boolean): 빈 문자열 허용 여부 (False일 경우 빈 값 불가)
*   **`format_check`**: 데이터 포맷 유효성 검사
    *   `rules` (array): 포맷 규칙 배열
        *   `field_path` (string): 대상 필드 경로
        *   `pattern` (string): 정규식 패턴 (Regex)
        *   `type` (string): 검사 유형 ("email", "phone", "uuid", "regex")
*   **`business_rules`**: 비즈니스 로직 기반 검증
    *   `script` (string): 검증 로직 스크립트
    *   `error_policy` (string): 검증 실패 시 처리 정책 ("reject", "skip", "dead_letter")

---

## JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "schemas": {
      "type": "object",
      "properties": {
        "schema_definition": {
          "type": "object",
          "properties": {
            "source_schema_url": { "type": "string" },
            "target_schema_url": { "type": "string" },
            "strict_mode": { "type": "boolean" }
          },
          "required": ["source_schema_url", "target_schema_url"]
        },
        "canonical_model": {
          "type": "object",
          "properties": {
            "use_canonical": { "type": "boolean" },
            "model_id": { "type": "string" },
            "version": { "type": "string" }
          }
        },
        "type_casting": {
          "type": "object",
          "properties": {
            "rules": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "source_type": { "type": "string" },
                  "target_type": { "type": "string" },
                  "format": { "type": "string" },
                  "on_error": { "type": "string", "enum": ["fail", "set_null", "default_value"] }
                }
              }
            }
          }
        }
      },
      "required": ["schema_definition"]
    },
    "field_mapping": {
      "type": "object",
      "properties": {
        "visual_mapping": {
          "type": "object",
          "properties": {
            "mapping_id": { "type": "string" },
            "definitions": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "source_path": { "type": "string" },
                  "target_path": { "type": "string" }
                }
              }
            }
          }
        },
        "expression_language": {
          "type": "object",
          "properties": {
            "language": { "type": "string", "enum": ["spel", "jq", "javascript"] },
            "scripts": {
              "type": "object",
              "additionalProperties": { "type": "string" }
            }
          }
        },
        "lookup_tables": {
          "type": "object",
          "properties": {
            "tables": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "name": { "type": "string" },
                  "data_source": { "type": "string" },
                  "cache_ttl_seconds": { "type": "integer" }
                }
              }
            }
          }
        },
        "smart_mapping": {
          "type": "object",
          "properties": {
            "enabled": { "type": "boolean" },
            "confidence_score_threshold": { "type": "number", "minimum": 0, "maximum": 1 }
          }
        }
      }
    },
    "validation": {
      "type": "object",
      "properties": {
        "required_fields": {
          "type": "object",
          "properties": {
            "fields": { "type": "array", "items": { "type": "string" } },
            "check_null": { "type": "boolean" },
            "check_empty": { "type": "boolean" }
          }
        },
        "format_check": {
          "type": "object",
          "properties": {
            "rules": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "field_path": { "type": "string" },
                  "pattern": { "type": "string" },
                  "type": { "type": "string" }
                }
              }
            }
          }
        },
        "business_rules": {
          "type": "object",
          "properties": {
            "script": { "type": "string" },
            "error_policy": { "type": "string", "enum": ["reject", "skip", "dead_letter"] }
          }
        }
      }
    }
  },
  "required": ["schemas", "field_mapping", "validation"]
}
```

## Configuration Sample

```json
{
  "schemas": {
    "schema_definition": {
      "source_schema_url": "s3://schemas/order/v1/source.json",
      "target_schema_url": "s3://schemas/erp/v2/target.json",
      "strict_mode": true
    },
    "canonical_model": {
      "use_canonical": true,
      "model_id": "canonical_order",
      "version": "1.0.0"
    },
    "type_casting": {
      "rules": [
        {
          "source_type": "string_integer",
          "target_type": "integer",
          "on_error": "fail"
        },
        {
          "source_type": "string_date",
          "target_type": "iso8601_date",
          "format": "yyyyMMdd",
          "on_error": "set_null"
        }
      ]
    }
  },
  "field_mapping": {
    "visual_mapping": {
      "mapping_id": "map_001",
      "definitions": [
        {
          "source_path": "$.orderId",
          "target_path": "$.erp_header.id"
        },
        {
          "source_path": "$.customer.name",
          "target_path": "$.erp_header.customer_name"
        }
      ]
    },
    "expression_language": {
      "language": "spel",
      "scripts": {
        "$.erp_header.total_price": "#root['items'].![price * qty].stream().reduce(0, (a, b) => a + b)",
        "$.erp_header.status": "#root['paid'] ? 'CONFIRMED' : 'PENDING'"
      }
    },
    "lookup_tables": {
      "tables": [
        {
          "name": "country_code_map",
          "data_source": "database",
          "cache_ttl_seconds": 3600
        }
      ]
    },
    "smart_mapping": {
      "enabled": true,
      "confidence_score_threshold": 0.85
    }
  },
  "validation": {
    "required_fields": {
      "fields": ["$.orderId", "$.customer.email"],
      "check_null": true,
      "check_empty": true
    },
    "format_check": {
      "rules": [
        {
          "field_path": "$.customer.email",
          "type": "email"
        },
        {
          "field_path": "$.items[*].sku",
          "type": "regex",
          "pattern": "^[A-Z]{3}-[0-9]{4}$"
        }
      ]
    },
    "business_rules": {
      "script": "if (root['total_amount'] > 10000 && !root['is_vip']) return false;",
      "error_policy": "reject"
    }
  }
}
```
