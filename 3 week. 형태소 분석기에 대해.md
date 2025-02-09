# 형태소 분석기 (Nori 분석기)

# Term Vectors란
Term Vectors는 OpenSearch/Elasticsearch에서 특정 문서의 텍스트 필드에 포함된 토큰(단어)과 관련된 정보를 제공하는 기능입니다. 이를 통해 형태소 분석(토큰화) 결과, 빈도수, 위치 정보, 동시 발생(term positions), 오프셋(offset) 등의 메타데이터를 확인할 수 있습니다.

| **항목**                | **설명**                                               |
|-----------------------|------------------------------------------------------|
| **Terms (단어 목록)**     | 문서 내에서 분석된 단어(토큰)                              |
| **Term Frequency (빈도수, tf)** | 해당 단어가 문서에서 등장한 횟수                           |
| **Positions (위치 정보)**  | 해당 단어가 문장에서 몇 번째 위치에 있는지                  |
| **Offsets (문장 내 오프셋 정보)** | 시작/끝 위치(offset)를 포함한 정보                        |
| **Payloads (추가 메타데이터)** | 인덱싱 시 설정한 추가 데이터 (일반적으로 사용 안 함)        |

## Term Vectors API 예제

### 📌 예제 데이터 저장
#### OpenSearch에 Nori 분석기가 적용된 인덱스 생성
```json
PUT /my_nori_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "nori_analyzer": {
          "type": "custom",
          "tokenizer": "nori_tokenizer",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "nori_analyzer",
        "term_vector": "yes"  
      }
    }
  }
}
```
> nori_analyzer를 적용한 인덱스를 생성하고 문서를 추가합니다  
> "term_vector": "yes" 옵션을 설정해야 Term Vectors API를 사용할 수 있습니다.

#### 예제 문서 색인
```json
POST /my_nori_index/_doc/1
{
  "text": "나는  학교에 간다"
}
```


### 📌 Term Vectors API 사용
```json
POST /my_nori_index/_termvectors/1
{
  "fields": ["text"],
  "offsets": true,
  "positions": true,
  "term_statistics": true
}
```

#### 문서의 형태소 결과 반환
```json
{
  "_index": "my_nori_index",
  "_id": "1",
  "term_vectors": {
    "text": {
      "field_statistics": {
        "sum_doc_freq": 5,
        "doc_count": 1,
        "sum_ttf": 5
      },
      "terms": {
        "나": {
          "term_freq": 1,
          "tokens": [
            {
              "position": 0,
              "start_offset": 0,
              "end_offset": 1
            }
          ]
        },
        "는": {
          "term_freq": 1,
          "tokens": [
            {
              "position": 1,
              "start_offset": 1,
              "end_offset": 2
            }
          ]
        },
        "학교": {
          "term_freq": 1,
          "tokens": [
            {
              "position": 2,
              "start_offset": 3,
              "end_offset": 5
            }
          ]
        },
        "에": {
          "term_freq": 1,
          "tokens": [
            {
              "position": 3,
              "start_offset": 5,
              "end_offset": 6
            }
          ]
        },
        "가": {
          "term_freq": 1,
          "tokens": [
            {
              "position": 4,
              "start_offset": 7,
              "end_offset": 8
            }
          ]
        }
      }
    }
  }
}
```

#### Term Vectors 결과 해석
📍 주요 항목 설명
- "terms": 문서에서 분석된 단어 목록
- "term_freq": 해당 단어의 등장 횟수 (TF 값)
- "tokens": 단어의 위치 정보 포함
- "position": 단어의 문장 내 위치
- "start_offset", "end_offset": 문장에서 해당 단어의 시작/끝 위치

📍 예제 해석
- "나"는 문서의 첫 번째 단어 (position: 0, start_offset: 0, end_offset: 1)
- "학교"는 세 번째 단어 (position: 2, start_offset: 3, end_offset: 5)
- "가"는 다섯 번째 단어 (position: 4, start_offset: 7, end_offset: 8)

### Kotlin으로 Term Vectors API 호출
```kotlin
import org.opensearch.client.Request
import org.opensearch.client.RestHighLevelClient
import org.springframework.stereotype.Service

@Service
class TermVectorService(private val client: RestHighLevelClient) {

    fun getTermVectors(indexName: String, docId: String) {
        val request = Request("POST", "/$indexName/_termvectors/$docId")
        request.setJsonEntity(
            """
            {
              "fields": ["text"],
              "offsets": true,
              "positions": true,
              "term_statistics": true
            }
            """.trimIndent()
        )

        val response = client.lowLevelClient.performRequest(request)
        val responseBody = response.entity.content.bufferedReader().use { it.readText() }
        println("🔍 Term Vectors Result: $responseBody")
    }
}
```

# Nori 분석기의 역할
- 문서를 저장할 때(Indexing Time)  
문서를 저장하면 OpenSearch가 Nori 분석기를 사용하여 형태소 분석(Tokenization) 을 수행한 후 역색인(Inverted Index) 을 생성합니다.  
예를 들어 "나는 학교에 간다"를 저장하면, OpenSearch는 "나", "는", "학교", "에", "가", "ᆫ다" 등의 단어로 분리하여 저장합니다.

- 검색할 때(Query Time)  
사용자가 "학교 가다" 같은 검색어를 입력하면, Nori 분석기가 이를 형태소 분석하여 색인된 데이터와 비교합니다.  
검색어 "학교 가다" → 분석 결과: ["학교", "가", "다"]  
위 분석 결과를 색인된 문서의 토큰과 비교하여 가장 관련성이 높은 문서를 반환합니다.  

## Nori 분석기 적용 방식

- Nori 분석기를 저장할 때만 사용할 경우
```json
{
  "settings": {
    "analysis": {
      "analyzer": {
        "nori_index_analyzer": {
          "type": "custom",
          "tokenizer": "nori_tokenizer",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "nori_index_analyzer"
      }
    }
  }
}
```
> "index_analyzer"만 설정하여 문서를 저장할 때만 분석하도록 설정할 수 있습니다.  
> 검색할 때는 원본 텍스트와 매칭되므로, 검색어가 형태소 분석되지 않음.

- 저장과 검색 모두에 Nori 분석기 적용
```json
{
  "settings": {
    "analysis": {
      "analyzer": {
        "nori_index_analyzer": {
          "type": "custom",
          "tokenizer": "nori_tokenizer",
          "filter": ["lowercase"]
        },
        "nori_search_analyzer": {
          "type": "custom",
          "tokenizer": "nori_tokenizer",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "nori_index_analyzer",
        "search_analyzer": "nori_search_analyzer"
      }
    }
  }
}
```
> 색인할 때(Indexing)와 검색할 때(Querying) 모두 Nori 분석기를 적용하려면 "search_analyzer"를 추가해야 합니다.  
> 저장할 때(Indexing)와 검색할 때(Querying) 모두 Nori 분석기가 적용되므로, 검색 정확도가 올라갑니다.  
> 즉, 색인된 토큰과 검색어의 토큰이 같은 방식으로 변환되므로, 일관된 검색 결과를 얻을 수 있습니다.

### Nori 분석기 사용 방식 정리
| **사용 방식**                           | **설명**                                           | **검색 정확도**                       |
|-------------------------------------|--------------------------------------------------|------------------------------------|
| **저장할 때만 사용 (index_analyzer)**    | 형태소 분석을 저장할 때만 적용, 검색 시 원본과 비교            | 낮음 (검색어가 분석되지 않음)         |
| **저장과 검색 모두 사용 (index_analyzer + search_analyzer)** | 저장할 때와 검색할 때 모두 형태소 분석 수행               | 높음 (색인된 토큰과 검색어가 동일한 방식으로 분석됨) |
