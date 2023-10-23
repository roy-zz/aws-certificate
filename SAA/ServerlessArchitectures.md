# Serverless Architectures

이번 장에서는 SAA를 준비하며 **Serverless 아키텍처를 설계 하는 방법**에 대해서 알아보도록 한다.

---

### Mobile Application: MyTodoList

- 우리는 아래와 같은 요구사항을 가지고 있는 모바일 애플리케이션을 만들어야 한다.
  - HTTPS를 사용하여 REST API를 노출한다.
  - Serverless 아키텍처로 설계되어야 한다.
  - 사용자는 S3에서 자신의 폴더와 직접 상호 작용할 수 있어야 한다.
  - 사용자는 관리형 Serverless를 통해 인증해야 한다.
  - 사용자는 할 일을 쓰고 읽을 수 있지만 대부분 읽기 작업을 수행한다.
  - 데이터베이스가 확장될 수 있어야 하며 읽기 처리량이 높아야 한다.

#### REST API 계층

![mobile-app-rest-api-layer.png](images%2FServerlessArchitectures%2Fmobile-app-rest-api-layer.png)

#### S3에 대한 사용자 접근

![mobile-app-giving-users-access-to-s3.png](images%2FServerlessArchitectures%2Fmobile-app-giving-users-access-to-s3.png)

#### 정적 데이터에 대한 높은 처리량

![mobile-app-high-read-throughput-static.png](images%2FServerlessArchitectures%2Fmobile-app-high-read-throughput-static.png)

#### API Gateway에서의 캐싱

![mobile-app-caching-api-gateway.png](images%2FServerlessArchitectures%2Fmobile-app-caching-api-gateway.png)

#### 정리

- Serverless REST API: HTTPS, API Gateway, Lambda, DynamoDB
- Cognito를 사용하여 STS로 임시 자격 증명을 생성하여 제한된 정책으로 S3 버킷에 액세스한다.
- App 사용자는 이런 방식으로 AWS 리소스에 직접 액세스할 수 있으며, 이러한 패턴은 DynamoDB, Lambda 등에 적용될 수 있다.
- DAX를 사용하여 DynamoDB에서 읽기를 캐싱할 수 있다.
- API Gateway 수준에서 REST 요청을 캐싱할 수 있다.
- Cognito, STS를 통한 인증 및 권한을 부여하여 보안을 강화할 수 있다.

---

### Serverless hosted website: MyBlog.com

- 이 웹 사이트는 전 세계적으로 확장되어야 한다.
- 블로그는 쓰기 작업이 자주 발생하지 않지만, 읽기 작업은 빈번하게 발생한다.
- 웹 사이트 중 일부는 순수하게 정적인 파일이고, 나머지는 동적 REST API다.
- 캐싱이 가능한 경우 구현해야 한다.
- 가입한 모든 새로운 사용자는 환영 이메일을 수신해야 한다.
- 블로그에 업로드된 모든 사진의 썸네일이 생성되어야 한다.

#### CloudFront와 S3를 통해 정적 콘텐츠 제공

![blog-serving-static-content.png](images%2FServerlessArchitectures%2Fblog-serving-static-content.png)

#### OAC(Origin Access Control)을 통한 보안 강화

![blog-serving-static-content-securely.png](images%2FServerlessArchitectures%2Fblog-serving-static-content-securely.png)

#### API Gateway와 Lambda를 통해 Serverless REST API 제공

![blog-adding-public-serverless-rest.png](images%2FServerlessArchitectures%2Fblog-adding-public-serverless-rest.png)

#### DynamoDB 글로벌 테이블 활용

![blog-leveraging-dynamodb-global-tables.png](images%2FServerlessArchitectures%2Fblog-leveraging-dynamodb-global-tables.png)

#### Amazon SES(Simple Email Service)와 DynamoDB Stream을 통한 Welcome 이메일 전송

![blog-welcome-email-flow.png](images%2FServerlessArchitectures%2Fblog-welcome-email-flow.png)

#### S3 트리거와 Lambda를 활용하여 썸네일 이미지 생성

![blog-thumbnail-generation-flow.png](images%2FServerlessArchitectures%2Fblog-thumbnail-generation-flow.png)

#### 정리

- 정적 컨텐츠가 CloudFront와 S3를 통해 배포되는 것을 확인하였다.
- REST API는 서버가 없고 Public이므로 Cognito가 필요하지 않다.
- DynamoDB Global Table을 활용하여 데이터를 전 세계적으로 서비스한다.
  - 대신 Aurora Global Database를 사용할 수도 있다.
- DynamoDB 스트림이 Labmda Function을 트리거할 수 있도록 구현하였다.
- Lambda Function에는 SES를 사용할 수 있는 IAM 역할이 있다.
- Serverless 방식으로 이메일을 보내는 데 SES(Simple Email Service)를 사용하였다.
- S3가 SQS/SNS/Lambda를 트리거하여 이벤트를 전송하였다.

---

### Micro Service Architecture(MSA)

- REST API를 사용하여 많은 서비스가 직접 상호 작용한다.
- 각 마이크로서비스의 각 아키텍처는 형태가 다를 수 있다.
- 마이크로서비스 아키텍처를 원하므로 각 서비스에 대한 개발 라이프사이클을 간소화할 수 있다.

#### 마이크로서비스 환경

![microservice-environment.png](images%2FServerlessArchitectures%2Fmicroservice-environment.png)

- 각 마이크로서비스를 원하는 방식으로 자유롭게 설계할 수 있다.
- 동기(Synchronous) 패턴: API Gateway, Load Balancers
- 비동기(Asynchronous) 패턴: SQS, Kineses, SNS, Lambda Triggers(S3)
- 마이크로서비스의 당면 과제:
  - 각 마이크로서비스를 생성하기 위한 반복적인 오버헤드
  - 서버 밀도/유틸화 최적화 관련 문제
  - 여러 버전의 여러 마이크로서비스를 동시에 실행하는 복잡성
  - 여러 개별 서비스와 통합하기 위한 클라이언트 측 코드 요구 사항의 확산.
- Serverless 패턴을 통해 해결할 수 있는 과제는 아래와 같다.
  - API Gateway, Lambda 자동 확장 및 사용량당 비용을 지불한다.
  - API를 손쉽게 복제하고 환경을 재현할 수 있다.
  - API Gateway를 위한 Swagger 통합을 통한 클라이언트 SDK를 생성한다.

#### 소프트웨어 업데이트 offloading

- offloading: CPU가 TCP/IP 프로세싱을 하지 않도록 하여 성능을 향상시키는 방법이다.
- EC2에서 소프트웨어 업데이트를 한 번씩 배포하는 애플리케이션이 있다.
- 새로운 소프트웨어 업데이트가 나오면 많은 요청을 받고 콘텐츠가 네트워크를 통해 대량으로 배포되기 때문에 많은 비용이 발생한다.
- 애플리케이션을 변경하는 것이 아니라 비용과 CPU를 최적화하는 방법에 대해서 알아본다.

- 현재 우리의 서비스가 아래의 이미지와 같다고 가정해 본다.

![microservice-current-state.png](images%2FServerlessArchitectures%2Fmicroservice-current-state.png)

- Load Balancer 앞에 CloudFront를 위치하도록 설계를 변경하여 간편하게 문제를 해결할 수 있다.

![microservice-easy-way-to-fix.png](images%2FServerlessArchitectures%2Fmicroservice-easy-way-to-fix.png)

- CloudFront를 사용하는 이유는 아래와 같다.
  - 아키텍처의 변경이 없다.
  - Edge에서 소프트웨어 업데이트 파일을 캐시할 수 있다.
  - 소프트웨어 업데이트 파일은 동적이 아니며 정적인 파일이다.
  - 우리의 EC2 인스턴스는 Serverless가 아니다.
  - 그러나 CloudFront는 회사의 규모에 맞게 확장이 가능하다.
  - ASG는 그만큼 확장되지 않으며 EC2에서 엄청난 비용 절감 효과를 얻을 수 있다.
  - 가용성, 네트워크 대역폭 비용 등도 절감할 수 있다.
  - 기존 애플리케이션을 보다 확장성이 높고 저렴하게 만드는 간편한 방법이다.

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03