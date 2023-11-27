# Other Services

이번 장에서는 **SysOps Administrator**를 준비하며 **기타 서비스들**에 대해서 알아보도록 한다.

---

### Amazon ElasticSearch

- Amazon ElasticSearch 서비스는 이제 Amazon OpenSearch 서비스이며 "Amazon ES"로 불리기도 한다.
- 시험 관점에서 보면 둘 다 동일하며 둘 다 OpenSearch용으로 이름이 변경된 ElasticSearch를 활용한다.
- 오픈 소스 프로젝트인 ElasticSearch의 관리형 버전이다.
- 서버에서 실행해야 하며 서버리스 제품이 아니다.
- 대표적인 사용 사례는 아래와 같다.
  - 로그 분석
  - 실시간 애플리케이션 모니터링
  - 보안 분석
  - 전체 텍스트 검색
  - 클릭스트림 분석
  - 인덱싱

- DynamoDB와 통합되어 사용될 수 있다.

![1-elastic-search-pattern-dynamodb.png](images%2F1-elastic-search-pattern-dynamodb.png)

#### Access Policy

![2-elasticsearch-access-policy-ip-based-1.png](images%2F2-elasticsearch-access-policy-ip-based-1.png)

- **IP-based Policy**
  - ES 도메인에 대한 액세스를 IP 주소 또는 CIDR 블록으로 제한하는 데 사용되는 리소스 기반 정책이다.
  - ES 도메인(예. CURL, Kibana 등)에 대한 서명되지 않은 요청을 허용한다.

![3-elasticsearch-access-policy-ip-based-2.png](images%2F3-elasticsearch-access-policy-ip-based-2.png)

#### Kibana Authentication

- Kibana는 기본적으로 IAM 사용자 및 역할을 지원하지 않는다.
- 아래의 항목을 사용하여 Kibana에 대한 액세스를 제어할 수 있다.
  - **HTTP Basic Authentication**
    - ElasticSearch Index에 저장된 내부 사용자 데이터베이스
  - **SAML**
    - 기존 제3자 ID 공급자를 사용하여 Kibana에 로그인
    - SAML 2.0 지원 (예. Active Directory Federation Service - ADFS, Auth0 등..)
  - **Amazon Cognito**
    - MS Active Directory(AD) 통합이 용이하다.

![4-kibana-authentication.png](images%2F4-kibana-authentication.png)

#### Production Setup

- 권장되는 사항은 아래와 같다.
  - 3개의 전용 마스터 노드 사용
  - AZ당 최소 2개의 데이터 노드 사용(복제본)
  - 3개의 가용 영역에 걸쳐 도메인 배포
  - 클러스터의 각 인덱스에 대해 하나 이상의 복제본을 생성

![5-elasticsearch-production-setup.png](images%2F5-elasticsearch-production-setup.png)

---

### AWS X-Ray

- 기존 방식의 프로덕션 디버깅
  - 로콜에서 테스트
  - 모든 위치에 로그 추가
  - 프로덕션 환경에 재배포
- 로그 형식은 애플리케이션마다 다르며 로그 분석이 어렵다.
- 디버깅: 하나의 큰 단일체는 쉽지만, 분산 서비스는 어려움이 있다.
- 전체 아키텍처에 대한 공통된 관점이 없다.

![6-xray-visual-analysis-our-application.png](images%2F6-xray-visual-analysis-our-application.png)

- X-Ray를 이용하면 응용 프로그램을 추적하고 시각적 분석을 할 수 있다.
- 서비스 상에서 활성화하면 각 서비스에 무슨 일이 일어나는지 온전한 그림을 확인할 수 있다.
- 어디가 문제인지, 성능이 어떤지 확인할 수 있고 요청 하나가 잘못되면 X-Ray 콘솔에서 직접 시각화할 수 있다.

#### X-Ray Advantage

- 보틀넥을 확인하여 성능 문제를 해결할 수 있다.
- 마이크로서비스 아키텍처의 종속성을 이해할 수 있다.
- 서비스 문제를 파악할 수 있다.
- 요청 행동을 검토할 수 있다.
- 오류 및 예외를 찾을 수 있다.
- 시간 SLA를 충족하고 있는지 확인할 수 있다.
- 어디에서 쓰로틀되어 있는지 확인할 수 있다.
- 영향을 받는 사용자를 식별할 수 있다.

---

### AWS Amplify

- 확장 가능한 풀 스택 웹 및 모바일 애플리케이션을 개발하고 배포하는 데 도움이 되는 도구 및 서비스 모음이다.
- 인증, 스토리지 API(REST, GraphQL), CI/CD, PubSub, 분석, AI/ML Predictions, 모니터링 등을 지원한다.
- GitHub, AWS CodeCommit, Bitbucket, GitLab에서 소스 코드를 연결하거나 직접 업로드할 수 있다.

![7-aws-amplify.png](images%2F7-aws-amplify.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
