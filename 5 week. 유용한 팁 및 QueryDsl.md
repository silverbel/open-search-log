# AWS opensearch에서 사용하는 유용한 팁

### 클러스터에 어떤 플러그인이 기본으로 설치되어 있는지 확인하는 명령어
```json
GET _cat/plugins
```
- id 또는 node.name: 노드 식별자 또는 이름
- component: 플러그인의 이름 (예: opensearch-security, performance-analyzer 등)
- version: 플러그인의 버전

> 📍 추가 설명
> AWS OpenSearch의 특성: AWS OpenSearch Service는 관리형 서비스이기 때문에, 사용자가 임의로 플러그인을 설치할 수 없습니다.  
> 기본적으로 AWS에서 제공하는 필수 플러그인(예: 보안, 성능 분석 등)만이 설치되어 있어, 출력 결과에는 이러한 기본 플러그인들만 나타나게 됩니다.
>  
> 플러그인 버전별 호환성 참고 : https://docs.aws.amazon.com/ko_kr/opensearch-service/latest/developerguide/supported-plugins.html


### 클러스터 헬스 체크
```json
GET _cluster/health
```

## Query Dsl
> 참고 : https://opensearch.org/docs/latest/query-dsl/

### multi-match
```json
GET _search
{
  "query": {
    "multi_match": {
      "query": "wind",
      "fields": ["제목^4", "줄거리"]
    }
  }
}
```
- 제목 또는 줄거리 필드에 wind가 있으면 검색
- "^4"의 뜻은 결과 리스트가 나올때 제목 필드에 4배의 비중을 더 높게 두어 제목에 wind가 있는것들이 살위에 노출

### ragne
```json
GET products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2025/01/01",
        "lte": "2025/01/31"
      }
    }
  }
}
```
- created 필드 값이 2025/01/01 ~ 2025/01/31 사이의 값인 데이터들 조회

> queryDsl의 경우 필요한 함수들을 직접 찾으면서 해보면 좋을거 같습니다.
