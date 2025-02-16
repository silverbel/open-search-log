# AWS 람다로 opensearch에 인덱싱하기 및 autocomplete(자동완성) 기능

- 실습 overview  
  <img width="515" alt="image" src="https://github.com/user-attachments/assets/3ac50a74-d947-46d4-996c-ad56a16fc8aa" />

- 참고 문서  
  [opensearch 자습서](https://docs.aws.amazon.com/ko_kr/opensearch-service/latest/developerguide/quick-start.html)

## 실습1) 문서에 인덱스 생성

- opensearch내 dev-tool을 사용하여 메서드를 실행합니다.

#### 인덱스 ID를 명시적으로 생성할때는 PUT 메서드
```json
PUT fruit/_doc/1
{
  "name":"strawberry",
  "color":"red"
}
```
#### 자동으로 생성되는 ID 만들기는 POST 메서드
```json
POST veggies/_doc
{
  "name":"beet",
  "color":"red",
  "classification":"root"
}
```
#### AWS 람다 함수를 사용하여 인덱싱
- event-bridge를 사용해서 trigger를 만들어주면 배치형태로 사용 가능합니다.

```javascript
export const handler = async (event) => {

  /**
   * 1. POST 데이터 전송
   * 2. headers Basic Authorization
   * 3. body 
   * 4. time
   */

  const URL = "AWS opensearch end-point"
  const username = "admin"
  const password = "Admin123!"
  const res = await fetch(`${URL}/veggies/_doc`, {
    method: "POST",
    headers: {
      'Content-Type': "application/json",
      'Authorization': "Basic " + `${Buffer.from(`${username}:${password}`).toString("base64")}`
    },

    body: JSON.stringify({
      name: `kale@${Date.now()}`,
      time: Date.now(),
      color: "red",
      classification: "root"
    })
  })

  if (res.ok) {
    const json = await res.json();
    console.log(json);
  }

  // TODO implement
  const response = {
    statusCode: 200,
    body: JSON.stringify('Hello from Lambda!'),
  };


  return response;
};
```

## autocomplete(자동완성) 기능

### 인덱스 매핑 설정
- 자동 완성을 위해 completion 필드를 사용하여 인덱스를 생성합니다
```json
PUT my_index
{
  "mappings": {
    "properties": {
      "suggest": {
        "type": "completion"
      }
    }
  }
}
```

### 데이터 삽입
- 자동 완성 기능을 활용하려면 suggest 필드에 입력값을 추가해야 합니다.
```json
POST my_index/_doc/1
{
  "suggest": {
    "input": ["아이폰", "아이폰 13", "아이폰 13 프로", "아이폰 14"]
  }
}
```

### 자동 완성 검색
- 사용자가 일부 키워드를 입력하면 추천 결과를 반환하도록 /_search API에서 suggest를 활용합니다.
```json
POST my_index/_search
{
  "suggest": {
    "goods-suggest": {
      "prefix": "아이폰",
      "completion": {
        "field": "suggest",
        "size": 5
      }
    }
  }
}
```

## completion 필드 외에도 Edge NGram 분석기를 활용하여 자동 완성을 구현 가능!

### 인덱스 매핑 설정
```json
PUT my_index
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "edge_ngram_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 20,
          "token_chars": ["letter", "digit"]
        }
      },
      "analyzer": {
        "edge_ngram_analyzer": {
          "type": "custom",
          "tokenizer": "edge_ngram_tokenizer"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "edge_ngram_analyzer",
        "search_analyzer": "standard"
      }
    }
  }
}
```

### 데이터 삽입
```json
POST my_index/_doc/1
{
  "name": "아이폰 14 프로"
}
```

### 검색 쿼리
```json
POST my_index/_search
{
  "query": {
    "match": {
      "name": "아이"
    }
  }
}
```

> 📌 어떤 방식이 더 적절할까?
> | 방식            | 장점                           | 단점                         |
> |---------------|-----------------------------|-----------------------------|
> | **Completion 필드** | 빠른 성능, 자동 완성 기능 최적화 | 인덱싱 시 메모리 사용량 증가 |
> | **Edge NGram** | 검색어 분석이 용이, 커스텀 가능  | 검색 성능 저하 가능          |
> OpenSearch에서 단순 자동 완성을 원하면 completion 필드를, 더 정밀한 검색 최적화를 원하면 Edge NGram을 사용!




