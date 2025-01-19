# Elasticsearch와 OpenSearch의 차이점

원래는 open search는 무료버전이었지만 aws가 open search를 통해 돈을 벌면서 분쟁이 발생. 그래서 **7.10.2** 이후로 라이선스 변경과 함께 독자적으로 발전.

| 구분               | Elasticsearch                                                                                   | OpenSearch                                                                                   |
|--------------------|------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------|
| **1. 라이선스 차이** | 7.11 버전부터 SSPL 적용<br>상업적 사용 시 제한 가능<br>일부 기능은 유료 Elastic Stack에서 제공     | Apache 2.0 라이선스 적용 (완전 오픈소스)<br>자유로운 사용, 수정, 배포 가능<br>AWS 및 커뮤니티 주도로 개발 |
| **2. 기능 및 플러그인 차이** | Elastic Stack(ELK) 제공:<br>- Kibana, APM, ML, 보안 기능 (일부 유료)<br>- X-Pack 필요<br>- Fleet, Elastic Agent 지원 | OpenSearch Dashboards 제공<br>ML 및 Anomaly Detection 오픈소스로 제공<br>기본 보안 기능 제공<br>Observability 강화 (로그 분석, 추적) |
| **3. 클라우드 지원 차이** | Elastic Cloud에서 공식 관리형 서비스<br>AWS, GCP, Azure에서 사용 가능                            | AWS의 Amazon OpenSearch Service로 관리형 제공<br>오픈소스 기반으로 자유로운 배포 및 클라우드 사용 가능 |
| **4. API 및 호환성** | 7.10.2까지 OpenSearch와 호환<br>7.11 이후 도입된 API는 OpenSearch에서 미지원<br>vector search, ILM 정책 제공 | Elasticsearch 7.10.2까지의 API와 호환<br>일부 API는 변경되거나 제거 가능<br>대체 방식으로 벡터 검색 제공 |
| **5. 검색 및 분석 기능 차이** | 최신 검색 알고리즘 및 벡터 검색(ANN, kNN) 지원<br>Elastic APM 지원<br>Elasticsearch SQL 제공    | AI 및 검색 최적화 기능 독자 개발 중<br>k-NN 및 최적화된 분석 제공<br>다중 테넌트 분석 가능             |
| **6. 보안 및 인증 차이** | X-Pack을 통한 유료 보안 기능 제공<br>API key, LDAP, SAML 연동 필요                               | 무료 기본 보안 기능 제공<br>RBAC, LDAP, SAML, OpenID Connect 지원<br>Audit Logging 기본 제공 |
| **7. 커뮤니티 및 지원** | Elastic 사의 상업적 지원 제공<br>GitHub 리포지토리 참여 제한                                     | AWS 및 커뮤니티 중심 개발<br>GitHub에서 적극적 기여 가능<br>다양한 환경에서 커뮤니티 지원          |
| **8. 배포 및 운영 차이** | Elastic Cloud로 쉬운 운영 및 업데이트 지원<br>수동 설치 시 관리 복잡                              | 자체 클러스터 구성 용이<br>AWS OpenSearch Service로 간편한 운영 지원                        |

--------------
# (OpenSearch와 비슷한 search 서비스가 하나 더 있던데...) AWS Cloud Search 는 무엇인가?

### 도큐먼트
- https://docs.aws.amazon.com/cloudsearch/latest/developerguide/what-is-cloudsearch.html

### 장점
- domain을 생성하여 csv 파일을 통해 데이터를 밀어넣으면 간단한 색인을 빠르게 구현가능.

### 단점
- 검색엔진이 2013년 이후로 develop 되어 있지 않음. (증거사진 첨부)
  <img width="741" alt="image" src="https://github.com/user-attachments/assets/1f0b4358-4e12-4beb-bd50-76761fce9512" />

| 항목              | AWS CloudSearch                              | AWS OpenSearch Service                              |
|-------------------|----------------------------------------------|----------------------------------------------------|
| **출시 연도**      | 2011년                                       | 2015년 (초기: Amazon Elasticsearch Service)       |
| **기반 기술**      | Amazon의 자체 검색 엔진                      | OpenSearch (Elasticsearch 기반)                   |
| **운영 방식**      | 완전 관리형(서버리스 유사)                   | 관리형 클러스터                                    |
| **기능**           | 기본적인 텍스트 검색 및 필터링                | 고급 분석, 풀텍스트 검색, 로그 분석               |
| **확장성**         | 자동 확장 지원                               | 수동 또는 자동 조정 가능                           |
| **사용 용도**      | 간단한 검색 애플리케이션                     | 복잡한 검색 및 분석 워크로드                      |
| **데이터 인덱싱 방식** | Batch API 기반                               | 실시간 스트리밍 및 REST API 지원                  |
| **보안 기능**      | 기본 IAM 인증                                | IAM, VPC, TLS, RBAC 등 다양한 보안 옵션 제공      |
| **주요 사용 사례**  | 문서 검색, 카탈로그 검색                     | 로그 분석, 보안 분석, AI 기반 검색                |

### 코멘트
적은 인원의 조직이 빠른성과를 내야할때 AWS CloudSearch를 사용하면 좋을거라고 생각.(개발자 하는일이 데이터를 밀어넣는것만 하면 되고, 서버의 수직, 수평 확장까지 관리해주기 때문에 정말 편함. 하지만 비용이 비싸고, 복잡한 기능을 지원하진 않음.)
하지만 AWS 자체에서도 OpenSearch Service를 검색 및 분석을 위한 차세대 솔루션으로 강조하고 있기 때문에 대부분의 개발자들이 AWS OpenSearch Service를 채택할것으로 예상.
추가로 필자는 AWS 환경내 다른 서비스들과 연동하여 사용하기 편리한 AWS OpenSearch를 선택!!

--------------
# (강의에서 Cloud 9를 사용한다는데..) Cloud9 은 무엇인가?
- 클라우드 기반 통합 개발 환경(IDE)
- kotlin을 지원하지 않고(설치하면 사용가능 하긴 함..) 추가 비용이 들기 때문에 실습시에는 intellij를 통해 진행할 예정!


### 도큐먼트
- https://docs.aws.amazon.com/ko_kr/cloud9/latest/user-guide/welcome.html
