# API Gateway

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "API 게이트웨이"에 대해서 알아보도록 한다.

---

### API Gateway

- Lambda, HTTP 및 AWS 서비스를 API로 노출할 수 있도록 지원한다.
- API 버전 관리, 원한 부여, 트래픽 관리(API 키, 쓰로틀링), 대규모, 서버리스, 요청/응답 변환, OpenAPI 사양, CORS를 지원한다.
- **29초의 타임아웃이 있다.**
    - Lambda Function의 최대 작동 시간은 15분이므로 Lambda Function의 작업이 진행 중일 때, 클라이언트에게 Timeout 오류를 전달할 수 있다.
- **최대 페이로드의 크기는 10MB다.**
    - 큰 용량의 파일을 전송하고자할 때는 API 게이트웨이가 적합하지 않다.

![1-api-gateway-overview.png](images%2F1-api-gateway-overview.png)

- AWS Lambda + API Gateway: 관리할 인프라가 없다.
- WebSocket 프로토콜을 지원한다.
- API 버전 관리를 할 수 있다.
- 여러가지 다양한 환경(Dev, Test, Prod)를 처리할 수 있다.
- 보안(Authentication, Authorization)을 처리할 수 있다.
- API 키를 생성하거나, 요청을 조절하여 처리할 수 있다.
- Swagger / Open API를 가져와서 신속하게 API를 정의할 수 있다.
- 요청 및 응답을 변환하거나 검증할 수 있다.
- SDK 및 API 규격을 생성한다.
- API 응답 캐시를 사용할 수 있다.

#### Integration

- **HTTP**
    - 백엔드에서 HTTP 엔드포인트를 노출한다.
    - 예를 들어, On-Premise의 HTTP API를 노출하거나, ALB를 노출할 수 있다.
    - 속도 제한, 캐싱, 사용자 인증, API 키 추가 등의 기능을 제공받을 수 있다.
- **Lambda Function**
    - Lambda Function을 호출할 수 있다.
    - AWS Lambda에서 지원하는 REST API를 손쉽게 호출할 수 있는 방법이다.
- **AWS Service**
    - API 게이트웨이를 통해 AWS API를 노출할 수 있다.
    - 예를 들어, AWS Step Function 워크플로를 실행하거나, SQS에 메시지를 게시할 수 있다.
    - 인증을 추가하고, 공개적으로 배포하고, 속도 제어를 추가할 수 있다.

- 아래와 같이 API Gateway와 Kineses 및 S3를 통합할 수 있다.

![2-integration-kinesis-data-streams.png](images%2F2-integration-kinesis-data-streams.png)

#### 엔드포인트 유형

- **Edge-Optimized (기본값)**: 글로벌 클라이언트용이다.
    - 요청은 CloudFront Edge 로케이션을 통해 라우팅되어 지연 시간을 향상시킨다.
    - API 게이트웨이는 여전히 한 리전에서만 작동한다.
- **Regional**
    - 동일한 리전 내의 클라이언트의 경우에 사용된다.
    - CloudFront와 수동으로 결합 가능하다. (캐싱 전략 및 배포 제어 강화)
- **Private**
    - 인터페이스 VPC 엔드포인트(ENI)을 사용하여 VPC에서만 액세스할 수 있다.
    - 리소스 정책을 사용하여 액세스를 정의한다.

#### Security

- 아래의 항목을 통해서 사용자 인증(Authentication)을 진행할 수 있다.
    - IAM 역할(내부 응용프로그램에 유용)
    - Cognito(외부 사용자를 위한 ID - 예를 들어, 모바일 사용자)
    - Custom Authorizer (직접 개발한 로직)
- AWS ACM(Certificate Manager)과의 통합을 통한 **Custom Domain Name HTTPS** 보안
    - **Edge-Optimized Endpoint**를 사용하는 경우 인증서가 "us-east-1"에 있어야 한다.
    - **Regional Endpoint**를 사용하는 경우 인증서가 API Gateway 영역에 있어야 한다.
    - "Route 53"에서 CNAME 또는 A-Alias 레코드를 설정해야 한다.

#### Deployment Stages

- API 게이트웨이를 변경하더라도 배포를 하기 전까지는 변화가 없다.
- 변경 사항이 적용되려면 "배포"를 해야 한다.
- 만약 API가 작동하는 방식을 변경하고 배포를 하지 않는다면 API는 변경되지 않은 상태로 남게된다.
- 변경 사항은 스테이지 별로 배포된다.
  - 스테이지는 원하는 만큼 생성할 수 있다.
- "dev", "test", "prod"와 같이 원하는 이름을 사용할 수 있다.
  - "v1", "v2", "v3"와 같은 이름도 사용할 수 있다.
- 각 스테이지에는 고유한 구성 파라미터가 있으며 원활하게 롤백할 수 있다.
- 각 스테이지에서 발생하는 모든 배포의 전체 이력이 유지된다.

![4-api-breaking-change.png](images%2F4-api-breaking-change.png)

- v1 스테이지가 있고 v1 함수를 호출하고 있다.
  - v1 클라이언트는 URL을 통해서 API에 액세스할 수 있다.
- 새로운 버전인 v2 람다 함수를 배포하려하지만 동일한 데이터 형식을 따르지 않는다.
  - 따라서 변경 사항을 v1 스테이지에 배포하면 v1 클라이언트에 손상이 발생된다.
- 배포를 위해서 새로운 스테이지인 v2 스테이지를 생성하고 v2 람다를 가리키도록 구성한다.
  - 클라이언트는 새로운 URL을 통해서 v2 스테이지에 접근한다.

#### Stage Variables

- 스테이지 변수는 API 게이트웨이의 환경 변수와 같다.
- 자주 변경되는 구성 값을 변경하는데 사용된다.
- 다음의 용도로 사용할 수 있다.
  - 람다 함수 ARN
  - HTTP 엔드포인트
  - 파라미터 매핑 템플릿
- 아래와 같은 사용 사례가 있다.
  - 스테이지에서 통신하는 HTTP 엔드포인트 구성한다. (dev, test, prod 등..)
  - 매핑 템플릿을 통해 구성 파라미터를 AWS 람다에 전달한다.
- 스테이지 변수는 AWS 람다의 "컨텍스트"객체로 전달된다.
- `${stageVariables.variableName}`과 같은 형태로 작성된다.

#### Stage Variable & Lambda Alias

- 해당 람다 별칭을 나타내는 스테이지 변수를 생성한다.
- API 게이트웨이는 올바른 람다 함수를 자동으로 호출한다.

![5-stage-variable-lambda-alias.png](images%2F5-stage-variable-lambda-alias.png)

- Dev 스테이지는 DEV 별칭을 가리키고 DEV 별칭은 `$LATEST` 람다 함수를 호출하고 있다.
- Test 스테이지는 TEST 별칭을 가리키고 트래픽의 100%를 V2 람다 함수로 전달하고 있다.
- Prod 스테이지는 PROD 별칭을 가리키고 트래픽의 95%를 V1 람다 함수로, 트래픽의 5%를 V2 람다 함수로 전달한다.

#### Open API Spec

- API 정의를 코드로 사용하여 REST API를 정의하는 일반적인 방법이다.
- 기존 OpenAPI 3.0 사양을 API 게이트웨이로 가져올 수 있다.
  - Method
  - Method Request
  - Integration Request
  - Method Response
  - API 게이트웨이용 AWS 확장 및 모든 단일 옵션 설정
- 현재 API를 OpenAPI 스펙으로 내보낼 수 있다.
- Open API 스펙은 YAML 또는 JSON으로 작성할 수 있다.
- Open API를 사용하여 애플리케이션용 SDK를 생성할 수 있다.

#### REST API - Request Validation

- 통합 요청을 진행하기 전에 API 요청에 대한 기본 검증을 수행하도록 API 게이트웨이를 구성할 수 있다.
- 검증에 실패하면 API 게이트웨이는 즉시 요청을 실패한다.
  - 호출자에게 400 오류 응답을 반환한다.
- 이를 통해 백엔드에 대한 불필요한 호출이 줄어든다.
- 아래의 항목을 확인해야 한다.
  - 들어오는 요청의 URI, 쿼리 문자열 및 헤더에 필수 요청 파라미터가 포함되어 있으며 비어 있지 않다.
  - 해동 요청 페이로드는 메소드의 구성된 JSON 스키마 요청 모델을 준수한다.
- OpenAPI 정의 파일을 가져와서 설정 요청을 검증할 수 있다.

![6-request-validation-openapi.png](images%2F6-request-validation-openapi.png)

- 리소스를 만들고 이 리소스의 이름을 `stagevariable`이라고 지정할 수 있다.
- `stagevariable`는 스테이지 변수 기능을 데모로 보여준다.
- "Create method"를 통해서 GET 메소드를 생성한다.

#### Caching API Response

- 캐싱을 통해 백엔드에 대한 호출 수를 감소시킬 수 있다.
- 기본 TTL(라이브 시간)은 300초(최소: 0s, 최대: 3600s)다.
- 캐시는 단계별로 정의된다.
- 메서드별로 캐시 설정을 재정의할 수 있다.
- 클라이언트는 헤더에 `Cache-Control:max-age=0`을 추가하여 캐시를 무효화할 수 있다.
- 전체 캐시를 즉시 플러시(무효화)할 수 있다.
- 캐시 암호화 옵션을 제공한다.
- 0.5GB에서 237GB 사이의 캐시 용량을 제공한다.

![7-caching-api-response.png](images%2F7-caching-api-response.png)

- 전체 캐시를 즉시 플러시(무효화)할 수 있다.
- 클라이언트는 `Cache-Control: max-age=0` 헤더를 사용하여 캐시를 무효화할 수 있다.
  - 적절한 IAM 인증 사용이 필요하다.
- InvalidateCache 정책을 적용하지 않은 경우 모든 클라이언트가 API 캐시를 무효화할 수 있다.
  - 콘솔에서 인증 필요 확인란을 선택하지 않은 경우에도 모든 클라이언트가 API 캐시를 무효화할 수 있다.

![8-cache-invalidation.png](images%2F8-cache-invalidation.png)

#### Canary Deployment

- 모든 스테이지(일반적으로 운영)에서 카나리아 배포를 활성화할 수 있다.
- 카나리아 채널이 수신하는 트래픽 비율을 선택할 수 있다.
- 더 나은 모니터링을 위해서 측정 항목과 로그가 분리되어 있다.
- 카나리아의 스테이지 변수를 재정의할 수 있는 기능을 제공한다.
- AWS 람다 및 API 게이트웨이를 사용한 Blue/Green 배포다.

![9-canary-deployment.png](images%2F9-canary-deployment.png)

- Prod 스테이지는 v1을 카리키고 있고 클라이언트가 새 버전을 테스트하려한다.
- v2에 대한 "Prod Stage Canary"를 생성한다.
  - 트래픽의 95%는 잘 작동하는 기존 Prod 스테이지로 전달한다.
  - 백그라운드에서 자동으로 트래픽의 5%는 Canary 스테이지로 전달한다.

#### Logging & Tracing

- **CloudWatch Logs**
  - 로그에는 요청/응답 바디에 대한 정보가 포함된다.
  - 스테이지 수준에서 CloudWatch 로깅을 활성화할 수 있다. (로그 수준: ERROR, DEBUG, INFO)
  - API별로 설정을 재정의할 수 있다.

![10-logging-tracing.png](images%2F10-logging-tracing.png)

- **X-Ray**
  - 추적을 활성화하여 API 게이트웨이의 요청에 대한 추가 정보를 얻는다.
  - X-Ray API 게이트웨이 + AWS 람다가 전체 아키텍처를 제공할 수 있다.

#### CloudWatch Metrics

- 측정항목은 스테이지별로 제공되며 세부 측정항목을 활성화할 수 있다.
- `CacheHitCount`와 `CacheMissCount` 항목을 통해 캐시 효율성을 확인할 수 있다.
- `Count` 항목을 통해 특정 기간 동안의 총 API 요청 수를 확인할 수 있다.
- `IntegrationLatency` 항목을 통해 API 게이트웨이가 클라이언트로부터 요청을 수신한 시점과 클라이언트에 응답을 반환하는 시점 사이의 시간을 확인할 수 있다.
  - 지연 시간에는 통합 지연 시간과 기타 API 게이트웨이 오버헤드가 포함된다.
- `4XXError`는 클라이언트 요청 오류이며 `5XXError`는 서버 쪽 오류다.

#### Throttling

- Account Limit
  - API 게이트웨이는 모든 API에서 10,000rps로 요청을 제한한다.
  - 요청 시 늘릴 수 있는 소프트 제한이다.
- 제한이 발생하는 경우 `429 Too Many Requests`가 발생하며 재시도 가능한 오류다.
- 성능 향상을 위해 "스테이지 제한" 및 "메소드 제한"을 설정하 ㄹ수 있다.
  - 고객별로 제한할 사용량 계획을 정의할 수 있다.
- 람다 동시성과 마찬가지로 하나의 API가 과부하되면 다른 API가 제한될 수 있다.

#### Error

- 4xx는 클라이언트 오류를 의미한다.
    - 400: Bad Request
    - 403: Access Denied, WAF 필터링
    - 429: Quota exceeded, 쓰로틀링

- 5xx는 서버측 오류를 의미한다.
    - 502: Bad Gateway Exception, 일반적으로 Lambda 프록시 통합 백엔드에서 반환되는 호환되지 않은 출력의 경우와 때때로 과중한 부하로 인해 주문이 맞지 않는 호출의 경우에 발생한다.
    - 503: Service Unavailable Exception
    - 504: Integration Failure - Endpoint 요청 시간 초과 예외 API 게이트웨이 요청 최대 29초 후 시간 초과가 발생한다.

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)