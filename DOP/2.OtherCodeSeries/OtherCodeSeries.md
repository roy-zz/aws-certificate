# Other Code Series

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "이전 장에서 다루지 않은 AWS 코드 시리즈"에 대해서 알아보도록 한다.

---

### AWS CodeArtifact

- 소프트웨어 패키지는 구축될 때 서로에게 종속되며(코드 종속성), 새로운 패키지가 생성된다.
- 이러한 종속성을 저장하고 검색하는 것을 "아티팩트 관리"라고 한다.
- CodeArtifact는 소프트웨어 개발을 위한 안전하고 확장 가능하며 비용 효율적인 아티팩트 관리다.
- Maven, Gradle, npm, tarn, twine, pip, NuGet과 같은 일반적인 종속성 관리 도구와 함께 작동한다.
- 개발자와 CodeBuild는 CodeArtifact에서 바로 종속성을 검색할 수 있다.

![1-codeartifact.png](images%2F1-codeartifact.png)

- CodeArtifact를 사용하면 모든 아티팩트가 AWS의 VPC 내에 상주하게 된다.
  - 다른 아티팩트 관리 시스템을 사용하는 경우라면 AWS 외부에 위치할 수도 있다.
  - 상황에 따라 자체적으로 아티팩트 관리 시스템을 구축해 EC2 인스턴스에 두는 경우도 있다.
- CodeArtifact에서는 도메인을 정의하게 되는데 각 도메인은 리포지토리 그룹을 의미한다.
  - 개발자 관점에서 보았을 때 `npm` 명령어를 수행해 자바스크립트 패키지에 필요한 의존성을 CodeArtifact로 가져올 수 있다.
  - CodeArtifact는 퍼블릭 아티팩트 리포지토리의 프록시이므로 자바스크립트 개발자가 직접 퍼블릭 아티팩트 리포지토리에 접근할 필요가 없다.
- 이렇게 시스템이 구축하는 첫 번째 이유는 네트워크 보안이다.
  - 자바스크립트 개발자는 CodeArtifact와만 상호 작용하고 CodeArtifact가 대신 퍼블릭 리포지토리로 요청을 전송한다.
- 두 번째 이유는 리포지토리에서 가져온 의존성을 캐시화하여 CodeArtifact 내부에 저장할 수 있다는 점이다.
  - 퍼블릭 아티팩트 리포지토리에서 해당 의존성이 사라져도 CodeArtifact 내부에는 복사본이 남는다.
- IT 리도 또는 개발자가 패키지를 게시 및 승인하여 CodeArtifact의 여러 리포지토리에 업로드할 수 있다.
  - 이는 모든 아티팩트가 VPC의 한 지점에 상주한다는 의미고 코드를 빌드하는데 필요한 모든 의존성이 CodeArtifact의 아티팩트 관리 시스템에 존재하게 된다는 의미다.
- CodeBuild의 경우에도 퍼블릭 리포지토리에서만이 아니라 CodeArtifact에서도 직접 이런 정보를 가져올 수 있다.

#### EventBridge Integration

![2-eventbridge-integration.png](images%2F2-eventbridge-integration.png)

- CodeArtifact는 패키지가 생성되거나 변경되거나 삭제됐을 때, EventBridge로 이벤트를 전송한다.
- EventBridge는 람다 함수, Step Function, SNS, SQS와 같은 서비스를 호출할 수 있다.
- CodeArtifact는 패키지 버전이 업데이트될 때, EventBridge를 통해서 CodePipeline도 호출할 수 있다.
  - CodePipeline에서 CodeCommit을 사용해 의존성이 업데이트 된 것을 감지했다.
  - 그러면 보안을 위해 CodeBuild가 실행되어 업데이트된 의존성으로 애플리케이션을 다시 빌드한다.
  - 이후 CodeDeploy를 사용하여 운영 환경에 새로운 애플리케이션을 배포한다.
  - 이러한 파이프라인 구축을 통하여 코드를 빌드할 때 항상 최신의 의존성이 반영되도록 할 수 있다.

#### Resource Policy

- 다른 계정이 CodeArtifact에 액세스할 수 있도록 권한을 부여하는데 사용할 수 있다.
- 지정된 주체가 리포지토리의 모든 패키지를 읽거나 모두 읽을 수 없도록 구성할 수 있다.
  - 특정 패키지에 대한 권한만 부여할 수는 없다.
  - 아래의 예시에서 IAM User "bob"은 권한을 부여받았기 때문에 리포지토리의 모든 패키지에 대한 접근 권한을 가지게 된다.

![3-resource-policy.png](images%2F3-resource-policy.png)

- 이러한 접근 권한 부여는 리소스 정책 없이는 불가능하다.

![4-resource-policy.png](images%2F4-resource-policy.png)

#### Upstream Repository

![5-upstream-repository.png](images%2F5-upstream-repository.png)

- CodeArtifact 리포지토리는 다른 코드 아티팩트 리포지토리를 업스트림 리포지토리로 가질 수 있다.
- 패키지 관리자 클라이언트가 단일 리포지토리 엔드포인트를 사용하여 둘 이상의 리포지토리에 포함된 패키지에 액세스할 수 있도록 허용한다.
- 최대 10개의 업스트림 리포지토리를 가질 수 있다.
- 리포지토리를 정의할 때, "External Connection"을 설정할 수 있는데 리포지토리당 하나만 설정할 수 있다.

#### External Connection

- 외부 연결은 CodeArtifact 리포지토리와 외부/퍼블릭 리포지토리(예: Maven, npm, PyPi, NuGet 등..) 사이의 연결이다.
- CodeArtifact 리포지토리에 아직 없는 패키지를 가져올 수 있다.
- 리포지토리에 최대 1개의 외부 연결을 생성할 수 있다.
- 여러 외부 연결을 위해서는 여러 개의 저장소를 생성해야 한다.

![6-external-connection.png](images%2F6-external-connection.png)

- `npmjs.com` 도메인에 연결하는 상황을 가정해본다.
  - 도메인에 하나의 CodeArtifact 리포지토리를 `npmjs.com`에 외부 연결하여 구성한다.
  - 업스트림을 사용하여 다른 모든 리포지토리를 구성한다.
  - `npmjs.com`에서 가져온 패키지는 각 리포지토리에서 가져와 저장하는 대신 업스트림 리포지토리에 캐시된다.

#### Retention

- 요청된 패키지 버전이 업스트림 리포지토리에서 발견된 경우 해당 패키지에 대한 참조가 유지되고 다운스트림 리포지토리에서 항상 사용할 수 있다.
- 보존된 패키지 버전은 업스트림 리포지토리의 변경 사항(예: 삭제 등..)에 영향을 받지 않는다.
- 중간 리포지토리가 패키지를 보관하지 않는다.

![7-retention.png](images%2F7-retention.png)

- `npmjs.com` 도메인에서 패키지를 가져오는 상황을 가정해본다.
  - Repository A에 연결된 Package Manager가 패키지 Lodash v4.17.20을 요청한다.
  - 패키지 버전이 세 개의 리포지토리에 없다.
  - 패키지 버전은 `npmjs.com`에서 가져온다.
  - Lodash 4.17.20 버전을 가져오면 아래의 리포지토리에 유지된다.
    - Repository A: 가장 다운스트림인 리포지토리
    - Repository C: `npmjs.com`에 대한 외부 연결이 있는 리포지토리
    - 패키지 버전이 중간 리포지토리이므로 Repository B에 유지되지 않는다.

#### Domains

- 리포지토리를 사용할 때는 도메인 개념을 도입할 수 있다.
- 도메인은 여러 계정과 계정에 속한 리포지토리들을 품을 수 있다.

![8-domain.png](images%2F8-domain.png)

- 중복제거된 스토리지: 자산은 여러 저장소에서 사용 가능한 경우에도 도메인에 한 번만 저장하면 된다. (스토리지 비용은 한 번만 지불)
- 빠른 복사: 업스트림 CodeArtifact 리포지토리의 패키지를 다운스트림으로 풀링하면 메타데이터 레코드만 업데이트된다.
- 리포지토리 및 팀 간의 간편한 공유: 도메인의 모든 자산 및 메타데이터가 단일 AWS KMS 키로 암호화된다.
- 여러 리포지토리에 걸쳐 정책 적용: 도메인 관리자는 다음과 같이 도메인에 걸쳐 "도메인 기반 정책"을 생성할 수 있다.
  - 도메인의 리포지토리에 액세스할 수 있는 계정 제한
  - 패키지 소스로 사용할 공용 리포지토리에 대한 연결을 구성할 수 있는 사용자

---

### Amazon CodeGuru

- 자동화된 코드 검토 및 애플리케이션 성능 권장을 위한 ML 기반 서비스다.
- 두 가지 기능을 제공한다.
  - CodeGuru Reviewer: 정적 코드 분석을 위한 자동 코드 리뷰 제공 (개발)
  - CodeGuru Profiler: 런타임 중 애플리케이션 성능에 대한 가시성/권장 사항 제공 (운영)

![9-codeguru.png](images%2F9-codeguru.png)

#### CodeGuru Reviewer

- 중요한 문제, 보안 취약성 및 찾기 어려운 버그를 식별한다.
- 예를 들어, 일반적인 코딩 모범 사례, 리소스 유출, 보안 탐지, 입력 유효성 검사 등에 사용된다.
- 머신러닝 및 자동화된 추론을 사용한다.
- 1,000개의 오픈 소스와 Amazon 리포지토리에 대한 수백만 건의 코드 리뷰를 통해 학습한 훈련을 제공한다.
- Java 및 Python을 지원한다.
- GitHub, BitBucket 및 AWS CodeCommit과 통합된다.
- 자세한 내용은 [공식 홈페이지](https://aws.amazon.com/codeguru/features/)를 확인한다.

![10-codeguru-reviewer.png](images%2F10-codeguru-reviewer.png)

- CodeGuru Reviewer Secrets Detector
  - 머신러닝을 사용하여 코드에 포함된 하드 코딩된 비밀(예: 비밀번호, API 키, 자격 증명, SSH 키 등..)을 식별한다.
  - 스캔 코드 외에도 구성 및 문서 파일을 스캔한다.
  - Secrets Manager를 통해 사용자의 비밀을 자동으로 보호할 수 있는 교정 조치를 제안한다.

![11-codeguru-reviewer-extras.png](images%2F11-codeguru-reviewer-extras.png)

- CodeGuru가 CodeCommit 리포지토리를 스캔한다.
- 스캔 중 `travis.yml` 파일에서 하드코딩된 보안 정보를 발견한다.
- "자격 증명 보호"를 제안하고 버튼을 클릭하면 자동으로 Secret Manager에 보안 암호를 생성하여 발견된 보안 정보를 저장한다.

#### CodeGuru Profiler

- 애플리케이션의 런타임 동작을 이해하는데 도움이 된다.
- 예를 들어, 로깅 루틴에서 애플리케이션이 과도한 CPU를 사용하고 있는지 확인할 수 있다.
- 아래와 같은 기능을 제공한다.
  - 코드 비효율성 식별 및 제거
  - 애플리케이션 성능 향상(예: CPU 사용률 감소)
  - 컴퓨팅 비용 절감
  - 힙 메모리 요약(메모리를 사용하는 객체 식별)
  - 이상 탐지
- AWS 또는 On-Premise에서 실행되는 애플리케이션을 지원한다.
- 애플리케이션의 모니터링에 필요한 오버헤드를 최소화할 수 있다.
- 자세한 내용은 [공식 홈페이지](https://aws.amazon.com/codeguru/features/)를 확인한다.

![12-codeguru-profiler.png](images%2F12-codeguru-profiler.png)

- 다음 중 하나를 사용하여 CodeGuru Profiler를 람다 함수에 통합하여 적용할 수 있다.
- Function Decorator `@with_lambda_profiler`
  - 람다 함수에 `.zip` 파일에 `codeguru_profiler_agent` 종속성을 추가하거나 람다 레이어를 사용한다.

![13-codeguru-profiler-extras.png](images%2F13-codeguru-profiler-extras.png)

- 람다 함수 구성에서 프로파일링을 사용할 수 있다.

---

### EC2 Image Builder

- 가상 머신 또는 컨테이너 이미지 생성 자동화에 사용된다.
- 예를 들어, EC2 AMI 생성 자동화, 유지보수, 검증 및 테스트와 같은 작업이 있다.
- 일정에 따라 실행할 수 있다. (매주, 패키지 업데이트 시 등..)
- 기본 리소스에 대한 비용만 지불하면 되는 무료 서비스다.
- 여러 리전 및 여러 계정에 AMI를 게시할 수 있다.

![14-ec2-image-builder.png](images%2F14-ec2-image-builder.png)

- 빌드 명령을 설정한 다음 빌더 EC2 인스턴스를 생성한다.
- 인스턴스는 아무것도 없는 상태로 생성되는데 이후 Image Builder 서비스가 앞에서 설정한 빌드 명령을 바탕으로 빌드 컴포넌트라는 것을 실행해 인스턴스의 소프트웨어를 구성한다.
- 빌드 명령 수행이 끝나면 결과물로 새로운 AMI가 생성되고 AMI에 대한 테스트가 진행된다.
- 새 AMI를 사용해 새로운 EC2 인스턴스가 생성되고 여기서 AMI에 관한 모든 테스트가 진행된다.
- 예를 들어, AMI가 제대로 동작하는지 안전한지에 대한 테스트를 할 수 있다.
- 이후 AMI는 이미지가 생성된 리전으로 배포되거나 다른 리전 또는 다른 계정으로 배포할 수 있다.

#### CI/CD Architecture

![15-cicd-architecture.png](images%2F15-cicd-architecture.png)

- 모든 것을 오케스트레이션하는 CodePipeline이 있다.
- CodePipeline 안에서는 맨 처음 CodeCommit이 소스 코드를 가져온다.
- CodeCommit에서 애플리케이션을 만들면 해당 소스 코드가 CodeBuild로 전달되고 코드가 컴파일되어 애플리케이션이 빌드되면 실행 파일이 생성된다.
- 애플리케이션을 배포할 준비가 끝나면 AMI를 빌드해야 한다.
  - 예시에서는 CloudFormatoin을 사용하고 이는 자동으로 EC2 Image Builder 서비스를 실행한다.
  - CodeBuild에서 마지막으로 빌드된 코드를 가져와 이를 바탕으로 AMI를 생성한다.
- 마지막 3단계 롤아웃에서도 CloudFormation이 사용된다.
  - ASG에서 이전 버전의 AMI를 사용 중인 EC2 인스턴스에 대해 롤링 업데이트를 수행한다.
  - CloudFormation에는 가끔씩 이미지를 업그레이드하는 로직이 있어서 시간이 지나면 모든 인스턴스의 AMI가 구버전에서 새 버전으로 업그레이드된다.

#### Sharing using RAM

- AWS RAM을 사용하여 AWS 계정 전체 또는 AWS Organization을 통해 이미지, 레시피 및 구성 요소를 공유한다.

![16-sharing-using-ram.png](images%2F16-sharing-using-ram.png)

- 두 개의 계정이 있고 EC2 Image Builder로 AMI 이미지를 빌드하고 있다.
- RAM을 사용하면 두 계정이 자동으로 이미지를 보내고 수락하도록 구성할 수 있다.
- B 계정에서 해당 AMI를 사용할 수 있게 되면 이로부터 EC2 인스턴스를 실행할 수 있다.

#### Tracking Latest AMIs

![17-tracking-latest-ami.png](images%2F17-tracking-latest-ami.png)

- 우리는 언제나 EC2 Image Builder로 빌드한 가장 최신 버전의 AMI ID를 SSM Parameter Store에 저장해야 한다.
  - 보안 수정 사항을 적용해 주간 빌드를 한다고 했을 때, 조직의 모든 사람이 해당 이미지의 최신 AMI ID를 알게 하기 위해서다.
- 이미지가 빌드되면 SNS에 알림이 생성되고 여기서 람다를 호출한다.
  - 람다를 사용하면 이 AMI ID를 SSM Parameter Store에 저장할 수 있다.
  - 사용자들은 Parameter Store에 접근하여 최신 AMI ID를 확인할 수 있다.
- 또는 CloudFormation을 사용해 EC2 인스턴스를 생성한 경우라면 SSM Parameter Store에서 동적으로 AMI ID를 참조하도록 할 수 있다.

---

### AWS Amplify

- 확장 가능한 전체 스택 웹 및 모바일 애플리케이션을 개발하고 배포하는데 도움이 되는 도구 및 서비스 세트다.
- 인증, 스토리지, API(REST, GraphQL), CI/CD, PubSub, Analytics, AI/ML, 예측, 모니터링 등을 제공한다.
- GitHub, AWS CodeCommit, Bitbucket, GitLab에서 소스 코드를 연결하거나 직접 업로드할 수 있다.

![18-aws-amplify.png](images%2F18-aws-amplify.png)

- 사용자는 Amplify CLI를 사용해 Amplify 백엔드를 생성한다.
  - 백엔드는 내부적으로 우리가 아는 다양한 AWS 리소스를 사용한다.
  - 데이터 스토리지에는 S3, 자격 증명에는 Cognito, API에는 AppSync, API Gateway 등이 있다.
- Amplify 프론트엔드 라이버리를 추가해 이 Amplify 백엔드에 연결한다.
  - 웹 애플리케이션이나 모바일 애플리케이션 또는 기타 프레임워크를 위한 프론트엔드 라이브러리다.
- Amplify 콘솔을 사용해 Amplify 자체 및 CloudFront에 배포하면 된다.
- 웹 및 모바일 앱을 위한 Elastic Beastalk라고 볼 수 있다.

#### Continuous Deployment

- CodeCommit에 연결하여 브랜치당 하나씩 배포한다.
- 애플리케이션을 사용자 지정 도메인(예: Route53)에 연결한다.

![19-continuous-deployment.png](images%2F19-continuous-deployment.png)

- 개발 브랜치와 운영 브랜치가 있고 양쪽 모두에 코드를 푸시하고 있다면 둘 다 Amplify에 연결할 수 있다.
- 예를 들어, 개발용 브랜치를 연결하면 Amplify가 이에 대한 배포를 생성할 것이고 우리는 이를 사용자 지정 도메인인 `dev.example.com` 같은 데 연결할 수 있다.
  - 사용자는 `dev.example.com`을 통해 개발 브랜치 상태에 액세스할 수 있게 된다.
- 운영 브랜치도 Amplify에 연결하여 별도의 사용자 지정 도메인 `example.com`에 배포한다.
  - 사용자는 `example.com`을 통해 운영 도메인에 액세스할 수 있게 된다.

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)