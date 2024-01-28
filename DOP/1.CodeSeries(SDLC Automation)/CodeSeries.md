# Code Series

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "소프트웨어 개발 생명 주기(SDLC)와 관련된 AWS 코드 시리즈 서비스들"에 대해서 알아보도록 한다.

---

### CI/CD

- CI/CD를 사용하지 않는다면 아래와 같은 순서로 배포를 진행하게 된다.
  - AWS 기본 리소스를 수동으로 생성한다.
  - AWS CLI를 통해서 AWS와 프로그래밍적으로 상호 작용한다.
  - Elastic Beanstalk를 사용하여 AWS에 코드를 배포한다.
- 이렇게 수동으로 이루어지는 작업이 많은 경우 휴먼 에러가 발생할 가능성이 높다.
- 우리의 코드가 리포지토리에 있고 이 코드가 AWS에 배포되기를 원하며 아래와 같은 요구사항이 있다.
  - 자동으로 우리가 생각하는 올바른 방법으로
  - 배포하기 전에 테스트 완료 여부 확인
  - 다양한 단계(dev, test, stage, prod)로 진행 가능
  - 필요한 경우 수동 승인 가능
- 이러한 요구사항을 충족시키기 위해서 AWS CI/CD를 학습해야 한다.
- 이번 섹션에서는 안정성을 높이면서 지금까지 수행한 배포를 자동화한다.
- AWS 코드 시리즈에는 아래와 같은 서비스들이 있다.
  - AWS CodeCommit: 코드 저장소
  - AWS CodePipeline: 코드에서 Elastic Beanstalk로 파이프라인 자동화
  - AWS CodeBuild: 코드 구축 및 테스트
  - AWS CodeDeploy: EC2 인스턴스에 코드 배포(Elastic Beanstalk X)
  - AWS CodeStar: 소프트웨어 개발 활동을 한 곳에서 관리
  - AWS CodeArtifact: 소프트웨어 패키지 저장, 게시 및 공유
  - AWS CodeGuru: 머신 러닝을 이용한 자동화된 코드 리뷰

#### Continuous Integration (CI)

- 개발자들은 리포지토리로 코드를 자주 푸시한다. (예: GitHub, CodeCommit, Bitbucket 등..)
- 테스트/빌드 서버는 코드를 푸시하자마자 코드를 확인한다. (예: CodeBuild, Jenkins CI 등..)
- 개발자는 성공/실패에 테스트 및 검사에 대한 결과를 피드백 받는다.
- 이러한 과정을 통해 얻는 이점은 아래와 같다.
  - 버그를 일찍 찾아내고 버그를 고칠 수 있다.
  - 코드 테스트를 신속하게 마무리할 수 있다.
  - 배포를 자주할 수 있다.

![1-continuous-integration.png](images%2F1-continuous-integration.png)

#### Continuous Delivery (CD)

- 필요할 때마다 소프트웨어를 안정적으로 릴리즈할 수 있도록 보장한다.
- 배포가 자주 진행되고 신속하게 이루어진다.
- "3개월에 한 번 출시"하던 애플리케이션을 "하루에 5번 출시"할 수 있다.
- 이는 일반적으로 자동화된 배포(예: CodeDeploy, Jenkins CD, Spinnaker 등..)를 의미한다.

![2-continuous-delivery.png](images%2F2-continuous-delivery.png)

- v1 버전이 실행 중인 애플리케이션 서버가 있고 우리는 새로운 버전을 배포하려고 한다.
- 개발자는 리포지토리에 코드를 푸시하고 빌드 서버에서 코드를 가져가서 테스트하고 빌드한다.
- 테스트 및 빌드에 성공한 결과물은 배포 서버에 전달되고 새로운 v2 버전 애플리케이션이 배포된다.

#### Technology Stack for CI/CD

![3-technology-stack-cicd.png](images%2F3-technology-stack-cicd.png)

- AWS에서 CI/CD를 구축하기 위한 다양한 기술 스택이 있다.
- 개발된 코드는 CodeCommit에 저장된다.
  - GitHub, Bitbucket과 같은 3rd 파티 저장소를 사용할 수 있다.
- AWS CodeBuild를 통해서 코드를 테스트하고 빌드할 수 있다.
  - Jenkins CI와 같은 3rd 파티 서비스를 사용할 수 있다.
- AWS Elastic Beanstalk를 통해서 새로운 인프라를 프로비저닝하면서 애플리케이션을 배포할 수 있다.
- AWS CodeDeploy를 사용하여 EC2 인스턴스, AWS 람다, ECS 등에 애플리케이션을 배포할 수 있다.
- 이러한 모든 과정을 조율하여 CI/CD 진행에 필요한 설정을 정의하려면 AWS CodePipeline으로 모든 것을 오케스트레이션해야 한다.

---

### AWS CodeCommit

- 버전 관리는 시간이 지남에 따라 코드에 발생한 다양한 변경 사항을 이해하는 기능이다.
  - 필요한 경우 롤백을 지원한다.
- 이러한 모든 기능들은 Git과 같은 버전 관리 시스템을 사용하여 활성화된다.
- Git 리포지토리는 컴퓨터에서 동기화할 수 있지만 일반적으로 중앙 온라인 리포지토리에 업로드된다.
- 리포지토리를 사용하면 얻을 수 있는 이점은 아래와 같다.
  - 쉽게 다른 개발자와 협업할 수 있다.
  - 코드가 어딘가에 백업되어 있는지 확인할 수 있다.
  - 확인 및 감사가 가능하기 때문에 누가 언제 어떤 코드를 커밋했는지 확인하거나 되돌릴 수 있다.
- Git 저장소는 많은 비용이 발생할 수 있다.
- 일반적으로 GitHub, GitLab, Bitbucket과 같은 리포지토리 서비스들이 많이 사용된다.
- "AWS CodeCommit"은 아래와 같은 특징을 가지고 있다.
  - Private Git 저장소다.
  - 리포지토리의 크기에 제한이 없으며 심리스하게 확장된다.
  - 완벽하게 관리되고 고가용성이다.
  - AWS Cloud 계정에서만 코드를 생성할 수 있으므로 보안 및 컴플라이언스를 강화할 수 있다.
  - 암호화 및 접근 제어와 같은 보안을 제공한다.
  - Jenkins, AWS CodeBuild 및 기타 CI 툴과 통합될 수 있다.

![4-aws-codecommit.png](images%2F4-aws-codecommit.png)

#### Security

- Git을 사용하여 상호 작용을 수행한다.
- **Authentication**
  - SSH Keys: AWS 사용자는 IAM 콘솔에서 SSH 키를 구성할 수 있다.
  - HTTPS: AWS CLI 자격 증명 도우미 또는 IAM 사용자용 Git 자격 증명을 사용한다.
- **Authorization**
  - 리포지토리에 대한 사용자/역할 권한을 관리하는 IAM 정책을 사용한다.
- **Encryption**
  - 저장소는 AWS KMS를 사용하여 자동으로 암호화된다.
  - HTTPS 또는 SSH를 사용하여 전송 중 암호화된다.
- **Cross-account Access**
  - SSH 키 또는 AWS 자격 증명을 공유하지 않는다.
  - AWS 계정에서 IAM 역할을 사용하고 AWS STS(`AssumeRole` API)를 사용한다.

#### CodeCommit vs GitHub

- CodeCommit과 GitHub은 아래와 같은 차이가 있다.

![5-codecommit-github.png](images%2F5-codecommit-github.png)

#### EventBridge를 통한 모니터링

- EventBridge와 통합하여 거의 실시간으로 CodeCommit 이벤트를 모니터링할 수 있다.
- `pullRequestCreated`, `pullRequestStatusChanged`, `referenceCreated`, `commentOnCommitCreated` 등의 이벤트를 모니터링한다.

![6-monitoring-eventbridge.png](images%2F6-monitoring-eventbridge.png)

#### CodeCommit으로 마이그레이션

- 다른 Git 리포지토리(예: GitHub, GitLab 등..)에서 호스팅되는 프로젝트를 CodeCommit 리포지토리로 마이그레이션할 수 있다.

![7-migration.png](images%2F7-migration.png)

- CodeCommit 리포지토리를 생성하고 `git clone` 명령을 수행한다.
- `git clone` 명령을 실행하면 서버에서 Git 리포지토리에 모든 컨텐츠를 가져와 로컬 컴퓨터에 저장한다.
  - 프로젝트 파일뿐만 아니라 커밋을 포함한 모든 이력을 가져온다.
- 복제가 완료되면 해당 프로젝트를 CodeCommit 리포지토리로 푸시한다.

#### Cross-Region 복제

- 글로벌로 퍼져있는 개발자들의 대기 시간을 단축하거나 백업 등의 이유로 교차 리전 복제가 사용될 수 있다.

![8-cross-region-replication.png](images%2F8-cross-region-replication.png)

- `us-east-1`에 있는 저장소를 복사해 `eu-west-2`에 복제본을 생성한다.
- 기존 브랜치로 무언가를 푸시하거나 브랜치를 생성 또는 삭제하면 CodeCommit은 해당 이벤트를 EventBridge에게 전달한다.
  - `referenceCreated` 또는 `referenceUpdated`라는 이벤트로 로그가 생성된다.
- EventBridge는 이 이벤트를 받아 ECS 작업(Task)을 실행하고 ECS 작업은 CodeCommit 저장소에서 `git clone` 명령을 수행한다.
  - ECS 작업 대신 CodeBuild를 사용해도 된다.
- 리포지토리를 복제해서 `eu-west-2`에 있는 리포지토리에 저장하면 된다.

#### 브랜치 보안

- 기본적으로 CodeCommit 리포지토리에 대한 푸시 권한을 가진 사용자는 모든 브랜치에 기여할 수 있다.
- IAM 정책을 사용하여 사용자가 특정 브랜치에 코드를 푸시하거나 병합하도록 제한할 수 있다.
  - 예를 들어, 시니어 개발자만 "Production" 브랜치에 푸시/병합할 수 있도록 설정할 수 있다.
- 리소스 기반 정책은 지원되지 않는다.

![9-branch-security.png](images%2F9-branch-security.png)

- 아래와 같은 IAM 정책을 생성할 수 있다.

![10-branch-security.png](images%2F10-branch-security.png)

#### Pull Request(PR) 승인 규칙

- 코드를 병합하기 전에 사용자가 공개 PR을 승인하도록 함으로써 코드의 품질을 보장할 수 있다.
- 승인할 사용자 풀 및 PR을 승인해야 하는 사용자의 수를 지정할 수 있다.
- IAM Principal ARN(IAM 사용자, Federated 사용자, IAM Roles, IAM Groups)을 지정할 수 있다.
  - 특정 리포지토리의 PR에 자동으로 승인 규칙을 적용할 수 있다.
  - 예를 들어, "dev" 및 "prod" 브랜치에 대한 서로 다른 규칙을 정의할 수 있다.

![11-pull-request-approval-rules.png](images%2F11-pull-request-approval-rules.png)

---

### AWS CodePipeline

- CI/CD를 오케스트레이션하는 시각적 워크플로우를 제공한다.
- **Source**: CodeCommit, ECR< S3, BitBucket, GitHub
- **Build**: CodeBuild, Jenkins, CloudBees, TeamCity
- **Test**: CodeBuild, AWS Device Farm, 3rd 파티 툴 등..
- **Deploy**: CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, S3 등..
- **Invoke**: Lambda, Step Functions
- 각각의 단계로 구성된다.
  - 각 단계는 순차적으로 동작하거나 병렬 동작을 가질 수 있다.
  - 예를 들어, Build -> Test -> Deploy -> Load Testing ..과 같은 단계를 구성할 수 있다.
  - 수동 승인은 모든 단계에서 정의할 수 있다.

#### Artifact

- 각 파이프라인 단계는 아티팩트를 생성할 수 있다.
- S3 버킷에 저장되어 다음 단계로 아티팩트가 전달된다.

![12-codepipeline-artifact.png](images%2F12-codepipeline-artifact.png)

- CodePipeline을 통해서 Source, Build, Deploy 세 개의 단계가 관리되고 있다.
- 모든 단계들은 아티팩트를 생성할 수 있으며 모든 아티팩트는 파이프라인 외부에 생성된다.
- 생성된 아티팩트는 S3 버킷에 저장되어 다음 단계로 전달되는데 이를 통해 다음 단계에서 자신이 해야 할 일을 파악한다.
- 먼저 개발자가 CodeCommit으로 코드를 푸시하면 CodeCommit은 CodePipeline의 오케스트레이션 작업을 거쳐 모든 코드를 추출하여 아티팩트를 생성하고 S3에 저장한다.
- 다음으로 CodeBuild가 실행되면, 앞서 추출된 것과 동일한 아티팩트가 CodeBuild 서비스에 바로 입력된다.
  - 덕분에 CodeBuild는 CodeCommit에 직접 접근할 필요가 없으며 CodePipeline이 S3를 통해 CodeBuild로 코드를 전달해 준다.
- CodeBuild가 코드를 빌드하면 배포 관련 아티팩트가 생성되고 CodePipeline에 의해 S3 버킷에 저장된다.
- CodePipeline은 아티팩트를 CodeDeploy에 전달하고 CodeDeploy는 아티팩트를 받아 배포할 코드를 확인하고 배포 작업을 진행한다. 

#### Troubleshooting

- CodePipeline Pipeline/Action/Stage 실행 상태 변경의 경우
- CloudWatch Events를 사용한다.
  - 실패한 파이프라인에 대한 이벤트를 생성할 수 있다.
  - 취소든 스테이지에 대한 이벤트를 생성할 수 있다.
- CodePipeline 단계가 실패하면 파이프라인이 중지되고 콘솔에서 정보를 얻을 수 있다.
- 파이프라인에서 작업을 수행할 수 없는 경우 첨부된 "IAM Service Role"에 충분한 IAM 권한이 있는지 확인해야 한다.
- AWS CloudTrail을 사용하여 AWS API 호출을 감사할 수 있다.

#### Event vs Webhook vs Polling

- CodePipeline을 시작하는 방법에는 이벤트, 웹훅, 폴링이 있다.
- 이벤트는 기본값이며 가장 권장되고 선호되는 방식이다.

![13-event-webhook-polling.png](images%2F13-event-webhook-polling.png)

- 이벤트 방식의 경우 CodeCommit이 있다면 새로운 커밋이 있을 경우, EventBridge에서 이벤트가 발생한다.
  - 해당 이벤트와 관련된 EventBridge 규칙이 CodePipeline을 실행시킨다.
  - 선호도가 가장 높은 방법이며, CodeCommit뿐만 아니라 AWS의 다른 모든 이벤트를 활용할 수 있다.
  - AWS 서비스가 아닌 GitHub을 사용하려면 "CodeStarSourceConnection"을 사용해야 한다.
  - GitHub 애플리케이션의 하나로 GitHub을 AWS에 연결하여 CodePipeline을 실행할 수 있게 해준다.
- 웹훅 방식의 경우 CodePipeline을 HTTP 엔드포인트에 노출한 다음 엔드포인트를 스크립트로 실행해야 한다.
  - 스크립트가 웹훅에 있는 CodePipeline에 페이로드를 전송하면 CodePipeline이 동작을 시작한다.
- 폴링 방식의 경우 CodePipeline이 주기적으로 소스를 폴링하도록 할 수 있다.

#### Action Types 

![14-action-types.png](images%2F14-action-types.png)

- Owner
  - "AWS"라고 되어 있는 건 해당 작업이 AWS 서비스와 관련이 있다는 뜻이다.
  - "3rd 파티"로 되어 있는 건 작업이 타사 서비스와 관련되어 있다는 의미다.
  - 커스텀은 젠킨스와 같은 서비스와 관련되어 있다는 의미다.
- Action Type
  - Source: S3, ECR, GitHub
  - Build: CodeBuild, Jenkins
  - Test: CodeBuild, Device Farm, Jenkins
  - Approval: Manual
  - Invoke: Lambda, Step Functions
  - Deploy: S3, CloudFormation, CodeDeploy, Elastic Beanstalk, OpsWorks, ECS, Service Catalog 등..

#### Manual Approval Stage

![15-manual-approval-stage.png](images%2F15-manual-approval-stage.png)

- CodePipeline을 생성하고 수동 승인 단계를 추가하였다.
  - 여기서 중요한 점은 소유자(Onwer)가 AWS라는 점이며, AWS와 관련되어 있음을 의미한다.
- 수동 승인은 AWS 서비스가 제공하는 기능이고 작업 유형은 Manual이다.
- 수동 승인 단계에 도달하면 먼저 SNS 토픽을 실행시키고 사용자에게 이메일을 전송한다.
  - 여기서 사용자는 AWS의 IAM 사용자로 등록된 사용자다.
  - 메일을 받은 사용자는 이 단계를 승인해야 한다.
  - 단계를 승인하려면 "GetPipeline*", "PutApprovalResult" 두 개의 권한이 필요하다.

#### CloudFormation as a Target

- CloudFormation Deploy Action을 사용하여 AWS 리소스를 배포할 수 있다.
- 예를 들어, CDK 또는 SAM을 사용하여 람다 함수를 배포할 수 있다. (COdeDeploy 대신 사용)
- CloudFormation StackSets과 함께 여러 AWS 계정 및 AWS 리전에 걸쳐 구축할 수 있다.
- 다양한 설정에 대한 구성이 가능하다.
  - Stack name, Change Set name, template, parameters, IAM Role, Action Mode...

![16-cloudformation-target.png](images%2F16-cloudformation-target.png)

- CodeCommit에 코드가 있고 코드는 그 자체로 템플릿이 될 수 있다.
- CloudFormation에서 변경 세트를 생성하고 수동 승인 단계를 둬서 변경 세트가 제대로 생성되었는지 확인한다.
- 이후 변경 세트를 수동으로 승인하여 실행하거나 자동으로 실행할 수 있다.

- **Action Mode**
  - 변경 집합 생성 또는 교체, 변경 집합 실행
  - 스택 생성 또는 수정, 스택 삭제, 장애가 발생한 스택 교체
- **Template Parameter Overrides**
  - 매개변수 값을 재정의할 JSON 객체를 지정한다.
  - 코드 파이프라인 입력 아티팩트에서 매개변수 값을 가져온다.
  - 템플릿에 모든 매개변수 이름이 있어야 한다.
  - 정적: 템플릿 구성 파일 사용(권장)
  - 동적: 매개 변수 재지정 사용

#### CloudFormation Integration

- CREATE_UPDATE: 스택을 생성하거나 기존 스택을 업데이트한다.
- DELETE_ONLY: 스택이 존재하는 경우 삭제한다.

![17-cloudformation-integration.png](images%2F17-cloudformation-integration.png)

- CodePipeline에서 CodeBuild를 통해 앱을 빌드하고 있다.
- 이 과정에서 `template.yaml` 파일이 생성된다.
- CloudFormation을 사용해 인프라와 애플리케이션을 배포하고 "CREATE_UPDATE" 모드를 사용해 스택을 생성하거나 기존 스택을 업데이트한다.
  - 예시에는 스택이 ALB와 Auto Scaling Group, EC2 인스턴스로 구성되어 있다.
- 다음 단계에서는 CodeBuild에서 애플리케이션을 테스트한다.
  - 예를 들어, HTTP 프로토콜을 사용하여 ALB 테스트 등을 수행하여 애플리케이션이 정상적으로 동작하는지 확인할 수 있다.
  - 기능 테스트, 로드 테스트 등 원하는 테스트를 진행할 수 있다.
- 테스트 인프라는 CloudFormation의 "DELETE_ONLY" 작업을 사용하여 스택을 삭제할 수 있다.
- CloudFormation 작업을 사용하여 운영 인프라를 배포하는데 여기서도 CREATE_UPDATE 작업이 사용된다.
  - 운영 인프라는 업데이트를 위해 사용된다.

#### Best Practice

![18-codepipeline-best-practice.png](images%2F18-codepipeline-best-practice.png)

- 하나의 CodeDeploy가 있을 때, CodeDeploy에서 여러 배포 그룹으로 병렬 배포를 수행할 수 있다.
  - 한 번에 여러 환경에 배포하거나 여러 배포 그룹에 배포하고 싶을 때는 CodeDeploy 배포를 여러 개 생성하는 대신 하나만 생성해 여러 그룹에 배포해야 한다.
- CodePipeline의 속도를 향상시킬 수 있는 또 다른 방법은 "병렬 작업"을 사용하는 것이다.
  - 예를 들어, CodeCommit에서 두 가지 빌드 작업을 수행하거나 빌드 및 테스트 작업을 함께 수행하고 싶다면 차례대로 하나씩 실행하는 대신에 RunOrder 값을 동일하게 지정하여 병렬도 실행되게 할 수 있다.
  - 빌드 작업에도 매우 유용하며 여러 리전에 배포해야 하는 경우에도 활용할 수 있다.
- 운영 환경에 배포하기 전에 Pre-운영에 배포하고 수동 승인을 거쳐 운영에 배포되도록 할 수 있다.

#### CodePipeline & EventBridge

- EventBridge: 실행 상태의 변화(예: 특정 단계에서의 실패)를 감지하고 대응한다.

![19-codepipeline-eventbridge.png](images%2F19-codepipeline-eventbridge.png)

- CodePipeline에서 CodeBuild가 실패하면 EventBridge가 동작을 가로챌 수 있다.
- EventBridge는 람다 함수 같은 것을 실행하여 코드를 진단하거나 SNS 알림을 호출하여 이메일 등으로 사용자에게 알림을 전송할 수 있다.

#### Invoke Action

- Lambda: 파이프라인 내에서 람다 함수를 호출한다.
  - 원하는 코드를 무엇이든 실행할 수 있고 REST API도 호출이 가능하다.
  - CodePipeline에서 API를 바로 호출할 수는 없기 때문에 람다 함수 호출을 활용하면 CodePipeline 기능을 크게 확장할 수 있다.

![20-invoke-action-lambda.png](images%2F20-invoke-action-lambda.png)

- Step Function: 파이프라인 내에서 "State Machine"을 실행한다.
  - 예를 들어, DynamoDB 테이블에 아이템을 삽입하거나 ECS 작업(Task)을 실행해 그래프와 작업 흐름도로 모든 작업이 순차적으로 처리되는 것을 볼 수 있다.
  - CodeBuild가 끝나고 나면 CodeDeploy를 통해서 배포된다.

![21-invoke-action-step-functions.png](images%2F21-invoke-action-step-functions.png)

#### Multi Region

- 파이프라인의 작업은 서로 다른 리전에 있을 수 있다.
  - 예를 들어, CloudFormation을 통해 람다 함수를 여러 리전으로 배포할 수 있다.
- S3 아티팩트 저장소는 작업을 수행하는 각 리전에 정의되어야 한다.
  - CodePipeline은 모든 아티팩트 버킷에 읽기/쓰기 액세스 권한이 있어야 한다.
  - 콘솔을 사용해 CodePipeline을 정의하면 자동적으로 기본 아티팩트 버킷을 사용하도록 설정되낟.
  - CLI나 SDK를 사용해 CodePipeline을 만들거라면 이런 아티팩트 버킷을 미리 생성해 놓고 CodePipeline에 적절한 권한을 부여해야 한다.

![22-multi-region.png](images%2F22-multi-region.png)

- CodePipeline은 교차 리전 작업을 수행할 때 한 AWS 리전에서 다른 리전으로 입력 아티팩트를 복사처리한다.
  - 리전 간 작업에서는 입력 아티팩트의 이름만 참조한다.

#### CloudFormation Multi Region

![23-cloudformation-multi-region.png](images%2F23-cloudformation-multi-region.png)

- 람다 함수를 `eu-west-1`과 `us-east-2` 리전에 배포해야 한다.
- 가장 먼저 CodePipeline을 정의하고, 다중 리전을 지원하기 위해 여러 개의 아티팩트 스토어를 생성한다.
- 첫 번째 리전은 `eu-west-1`로 파이프라인이 있는 곳이다.
- 코드는 CodeCommit을 통해서 CodeBuild로 전달되어 빌드된다.
- yaml 파일이 생성되는데, 이 파일은 CloudFormation에서 람다 함수를 배포하는 데 사용된다.
  - 배포할 리전이 여러 곳이기 때문에 템플릿을 여러 개 만들어 배포할 리전마다 템플릿을 하나씩 둬야 한다.
- 템플릿 생성이 끝나면 파이프라인이 있는 리전에서 CloudFormation 작업을 실행한다.
  - 이때 입력값으로 이전 단계에 CodeBuild가 생성한 템플릿을 참조하고 그 결과 람다 함수가 배포된다.
- CodePipeline에서 여러 리전에 접근해야 하기 때문에 아티팩트 스토어를 `us-east-2`에도 생성해야 한다.
  - 아티팩트 스토어에 필요한 입력 아티팩트는 CodePipeline에 의해 자동으로 복사된다.
  - 예를 들어, `template-us-east-2.yaml`파일이 자동으로 복사된다.
- CloudFormation은 복사된 파일을 입력으로 받아들이고, 자동으로 올바른 아티팩트 스토어의 파일을 가져와 사용한다.

---

### AWS CodeBuild

- 완벽하게 관리되는 CI(Continuous Integration) 서비스다.
- 지속적으로 확장되어 관리 또는 프로비저닝할 서버가 없다.
- 소스 코드 컴파일, 테스트 실행, 소프트웨어 패키지 제작 등의 작업을 한다.
- Jenkins와 같은 다른 빌드 도구의 대안이다.
- 빌드를 완료하는 데 걸리는 시간동안 사용한 컴퓨팅 리소스에 대해 분당 비용을 청구한다.
- 재현 가능한 빌드를 위해 도커를 활용한다.
- 미리 패키지화된 도커 이미지를 사용하거나 사용자 지정 도커 이미지를 직접 만든다.
- 아래와 같은 보안 사항을 제공한다.
  - 빌드 아티팩트 암호화를 위한 KMS와의 통합
  - IAM CodeBuild 권한, VPC 네트워크 보안
  - AWS CloudTrail을 통한 API 호출 로그
- Source: CodeCommit, S3, Bitbucket, GitHub
- Build instruction: 코드 파일의 `buildspec.yml` 또는 콘솔에서 수동으로 삽입
- 출력 로그는 S3 또는 CloudWatch Log에 저장할 수 있다.
- CloudWatch 메트릭을 사용하여 빌드 통계를 모니터링할 수 있다.
- EventBridge를 사용하여 실패한 빌드 탐지 및 알림을 트리거할 수 있다.
- CloudWatch 알람을 사용하여 장애에 대한 "임계값"이 필요한지 여부를 알릴 수 있다.
- CodePipeline 또는 CodeBuild 내에서 Build Project를 정의할 수 있다.
- 여러 종류의 환경을 지원한다.
  - Java, Ruby, Python, Go, Node.js, Android, .NET Core, PHP, Docker

#### 작동 방식

![24-how-it-works.png](images%2F24-how-it-works.png)

- CodeCommit이 있고 소스 코드와 수많이 파일이 있고 여기서 가장 중요한 파일은 `buildspec.yml`이다.
- CodeBuild는 이 코드를 가져온 다음 자체적으로 컨테이너를 생성하여 빌드 환경을 구성한다.
- 컨테이너는 모든 소스 코드와 `buildspec.yml` 파일을 로드하고 `buildspec.yml` 파일에 포함된 모든 명령을 실행한다.
- CodeBuild는 이 컨테이너를 빌드하기 위해 도커 이미지를 가져오는데 AWS가 이런 환경에 맞춰 미리 패키징한 이미지를 사용해도 되고 별도의 도커 이미지를 사용해 필요한 코드릴 실행해도 된다.
- CodeBuild는 `buildspec.yml`의 모든 명령을 실행하고 명령이 긴 경우를 위해 CodeBuild에서는 S3 버킷의 파일을 캐시하는 기능을 제공한다.
  - 이 기능을 통해서 빌드에서 자주 쓰이는 파일을 캐시할 수 있다.
- 로그 옵션을 활성화하면 모든 로그가 CloudWatch Logs와 S3에 저장된다.
- CodeBuild에서 코드를 빌드하거나 테스트하는 작업이 끝나면 아티팩트가 생성된다.
  - 이 아티팩트는 컨테이너 외부로 추출되어 S3 버킷에 저장된다.

#### buildspec.yml

![25-buildspec.png](images%2F25-buildspec.png)

- `buildspec.yml` 파일은 코드의 루트에 있어야 한다.
- env: 환경 변수를 정의한다.
  - variables: 평문 변수
  - parameter-store: SSM Parameter Store
  - secrets-manager: Secrets Manager에 저장된 변수
- phases: 실행할 명령을 지정한다.
  - install: build에 필요한 종속성을 설치
  - pre_build: 빌드 전에 실행할 최종 명령
  - build: 실제 빌드 명령
  - post_build: 마무리 작업
- artifacts: S3에 업로드할 내용 (KMS로 암호화)
- cache: 향후 빌드 속도 향상을 위해 S3에 캐싱할 파일(보통 종속성)

#### Local Build

- 로그 이상의 심층적인 문제 해결이 필요한 경우에 사용된다.
- 도커 설치 후 데스크톱에서 CodeBuild를 로컬로 실행할 수 있다.
- 이를 위해 CodeBuild 에이전트를 사용한다.
- 자세한 내용은 [공식 홈페이지](https://docs.aws.amazon.com/codebuild/latest/userguide/use-codebuild-agent.html)에서 확인한다.

#### Inside VPC

- 기본적으로 CodeBuild 컨테이너는 VPC 외부에서 시작된다.
  - VPC의 리소스에 액세스할 수 없다.
- VPC 구성 메뉴에서 "VPC ID", "서브넷 ID", "보안 그룹 ID" 등을 설정할 수 있다.
- CodeBUild 컨테이너가 VPC내부에 있는 RDS, ElastiCache, EC2 인스턴스, ALB 등의 리소스에 접근할 수 있게 된다.
- 통합 테스트, 데이터 쿼리, 내부 로드 밸런서 등에 사용된다.

![26-inside-vpc.png](images%2F26-inside-vpc.png)

#### Environment Variable

- **Default Environment Variables**
  - AWS에서 정의 및 제공한다.
  - AWS_DEFAULT_REGION, CODEBUILD_BUILD_ARN, CODEBUILD_ID, CODEBUILD_IMAGE 등이 있다.
- **Custom Environment Variables**
  - Static: 빌드 시점에 정의된다. (`start-build` API 호출을 사용하여 재정의)
  - Dynamic: SSM Parameter Store와 Secrets Manager를 사용한다.

![27-environment-variables.png](images%2F27-environment-variables.png)

#### Security

- CodeBuild Service 역할을 통해 CodeBuild가 사용자를 대신하여 AWS 리소스에 액세스할 수 있다. (필요한 권한 할당)
- 아래와 같은 사용 사례가 있다.
  - CodeCommit 리포지토리에서 코드 다운로드
  - SSM Parameter Store에서 파라미터 가져오기
  - 빌드 아티팩트를 S3 버킷에 업로드
  - CloudWatch 로그에 로그 저장
  - CloudWatch 로그에 로그 저장
- 전송 중 및 미사용 데이터에 대한 암호화를 제공한다. (캐시, 로그 등..)
- 출력된 아티팩트 암호화 빌드를 제공한다. (KMS에 액세스 필요)

![28-security.png](images%2F28-security.png)

#### Build Badges

![29-build-badges.png](images%2F29-build-badges.png)

- 최신 빌드 상태를 표시하는 동적으로 생성된 배지다.
- CodeBuild 프로젝트의 공개 URL을 통해 액세스가 가능하다.
- CodeCommit, GitHub 및 BitBucket을 지원한다.
- 배지는 브랜치 수준에서 사용이 가능하다.

#### Triggers

![30-triggers.png](images%2F30-triggers.png)

- CodeCommit에서 EventBridge로 이벤트를 보내 해당 이벤트로 CodeBuild를 실행할 수도 있다.
- CodeCommit에서 EventBridge로 이벤트를 보낸 다음 람다 함수를 호출하여 조금 더 복잡한 작업을 수행할 수 있다.
  - 그런 다음 람다에서 API를 호출하여 CodeBuild를 실행한다.
- CodeBuild는 GitHub을 지원하므로 GitHub에서 이벤트를 발생시켜 웹훅을 전달해서 CodeBuild를 트리거할 수 있다.

#### Validate Pull Request

- PR에서 제안된 코드 변경 내용을 병합하기 전에 검증한다.
- 높은 수준의 코드 품질을 보장하고 코드 충돌을 방지한다.

![31-validate-pull-request.png](images%2F31-validate-pull-request.png)

- 브랜치가 몇 개 있는 CodeCommit 리포지토리에서 개발용 브랜치를 운영 브랜치로 병합하려고 하는 상황이다.
- 먼저 PR을 생성해야 한다.
- 리포지토리에 PR이 생성되거나 업데이트되면, 이에 대한 이벤트가 EventBridge로 전송된다.
- EventBridge에서는 먼저 람다 함수를 실행하는데 함수의 역할은 PR을 업데이트해 테스트 빌드가 시작되었다는 코멘트를 추가하는 것이다.
- EventBridge의 두 번째 목적지는 CodeBuild다.
  - CodeBuild는 실행되어 해당 PR을 처리한다.
  - 해당 요청의 코드를 확인하고 테스트하여 테스트 및 빌드에 대한 성공 여부를 판단한다.
  - 결과는 EventBridge로 전달되고 여기서는 다시 이벤트 처리 방법을 알고 있는 람다 함수에 전달한다.
  - 결과적으로 PR이 업데이트되어 빌드 결과 코멘트가 추가된다.
- 결과에 문제가 없다면 PR을 병합하면 된다.

![32-validate-pull-request.png](images%2F32-validate-pull-request.png)

- 좌측 이미지의 하단부에는 빌드가 시작되었다고 쓰여 있다.
  - 두번째 코멘트를 확인해보면 빌드가 실패했다는 사실을 파악할 수 있다.
  - 코멘트에는 "failing"이라는 빌드 배지도 추가되어 있다.
- 우측 이미지는 PR 요청이 통과한 경우다.
  - 코멘트도 "passing"이라는 배지가 붙어 있다.
  - 이를 통해 우리는 해당 PR을 마스터 브랜치에 병합해도 안전하다는 사실을 알 수 있다.

#### Test Report

![33-test-report.png](images%2F33-test-report.png)

- CodeBuild는 다양한 테스트를 수행하는데 CodeBuild UI에는 이런 테스트에 대한 보고서를 바로 확인할 수 있다.
- 메뉴 이름은 "Report Group"과 "Test Report"다.
- 예시를 보면 CodeBuild에서 수행한 테스트의 통과율은 75%다.
  - 보고서에서 빌드 과정 중 수행하는 테스트의 세부 정보를 확인할 수 있기 때문에 로그를 직접 확인할 필요가 없다.

- 빌드 중에 실행되는 테스트에 대한 세부 정보를 포함한다.
- 단위 테스트, 구성 테스트, 기능 테스트가 포함된다.
- 다음 형식으로 리포트를 만들 수 있는 테스트 프레임워크를 사용하여 테스트 케이스를 생성한다.
  - JUnit XML, NUnit XML, NUnit3 XML
  - Cucumber JSON, TestNG XML, VIsual Studio TRX
- 테스트 보고서를 만들고 테스트에 대한 정보와 함께 `buildspec.yml` 파일에 보고서 그룹 이름을 추가한다.

![34-test-report.png](images%2F34-test-report.png)

---

### AWS CodeDeploy

- 애플리케이션 배포를 자동화하는 배포 서비스다.
- EC2 인스턴스, 온프레미스 서버, 람다 함수, ECS 서비스에 새로운 버전의 애플리케이션을 배포한다.
- 배포 실패 시 자동 롤백 기능 또는 CloudWatch 경보를 트리거할 수 있다.
- 점진적 배포 제어를 제공한다.
- `appspec.yml`이라는 파일은 배포가 수행되는 방식을 정의한다.

![35-codedeploy.png](images%2F35-codedeploy.png)

#### EC2/On-premises Platform

- EC2 인스턴스 및 온프레미스 서버에 구축할 수 있다.
- 인플레이스(In-Place) 배포 또는 Blue/Green 배포를 수행한다.
- 대상 인스턴스에서 CodeDeploy 에이전트를 실행해야 한다.
- 배포되는 속도를 정의할 수 있다.
  - AllAtOnce: 모든 인스턴스를 한 번에 업데이트하는 대신 가동 중지 시간이 가장 길어지낟.
  - HalfAtATime: 가용 용량이 50%로 줄어든다.
  - OneAtATime: 한 번에 하나의 인스턴스만 배포하기 때문에 배포 시간이 가장 오래 걸린다.
  - Custom: 사용자가 직접 배포되는 비율을 정의할 수 있다.

#### In-Place Deployment

![36-inplace-deployment.png](images%2F36-inplace-deployment.png)

- 네 개의 인스턴스에서 v1을 실행하고 있다.
- HalfAtATime으로 설정되어 있기 때문에 업데이트를 위해 2개의 인스턴스가 가동을 멈춘다.
- 에이전트가 두 인스턴스의 애플리케이션을 멈추고 마찬가지로 에이전트가 애플리케이션을 v2로 업그레이드한다.
- 업그레이드가 끝나면 설치를 위해 나머지 절반의 가동이 중단되고 나머지 또한 v2로 업그레이드된다.

#### Blue-Green Deployment

![37-blue-green-deployment.png](images%2F37-blue-green-deployment.png)

- ALB가 v1 애플리케이션으로 트래픽을 라우팅하고 있다.
- 배포를 위해서 새로운 ASG를 생성하고 v2 애플리케이션을 배포한다.
- 새로운 애플리케이션의 실행이 완료되면 ALB가 트래픽을 v2 애플리케이션으로 전달한다.
- v2 애플리케이션이 정상이라고 판단되면 v1 애플리케이션이 실행되는 ASG를 종료한다.

#### CodeDeploy Agent

- 전제 조건으로 EC2 인스턴스에서 CodeDeploy 에이전트가 실행되고 있어야 한다.
- System Manager를 사용하는 경우 자동으로 설치 및 업데이트할 수 있다.
- 배포 번들을 가져오려면 EC2 인스턴스에 충분한 권한이 있어야 S3 버킷에 액세스할 수 있다.

![38-codedeploy-agent.png](images%2F38-codedeploy-agent.png)

#### Lambda Platform

- CodeDeploy를 사용하면 람다 별칭에 대한 트래픽 시프트를 자동화할 수 있다.
- 이러한 기능들은 SAM 프레임워크 내에 통합되어 있다.
- Linear
  - 100%까지 N분마다 트래픽이 증가한다.
  - `LambdaLinear10PercentEvery3Minutes`: 3분마다 트래픽이 10%씩 선형적으로 이동한다.
  - `LambdaLinear10PercentEvery10Minuites`: 10분마다 트래픽이 10%씩 선형적으로 이동한다.
- Canary
  - 소량의 트래픽만 전송하다 한 번에 트래픽을 전환한다.
  - `LambdaCanary10Percent5Minutes`: 10%의 트래픽으로 테스트를 하고 문제가 없는 경우 5분 후에 트래픽을 이동한다.
  - `LambdaCanary10Percent30Minutes`: 10%의 트래픽으로 테스트를 하고 문제가 없는 경우 30분 후에 트래픽을 이동한다.
- AllAtOnce
  - 즉시 트래픽을 이동한다.

![39-lambda-platform.png](images%2F39-lambda-platform.png)

#### ECS Platform

- CodeDeploy를 통해 새로운 ECS 작업 정의의 배포를 자동화할 수 있다.
- Blue/Green 배포만 가능하다.
- Linear
  - 100%까지 N분마다 트래픽이 증가한다.
  - `ECSLinear10PercentEvery3Minutes`: 3분마다 트래픽이 10%씩 선형적으로 이동한다.
  - `ECSLinear10PercentEvery10Minuites`: 10분마다 트래픽이 10%씩 선형적으로 이동한다.
- Canary
  - 소량의 트래픽만 전송하다 한 번에 트래픽을 전환한다.
  - `ECSCanary10Percent5Minutes`: 10%의 트래픽으로 테스트를 하고 문제가 없는 경우 5분 후에 트래픽을 이동한다.
  - `ECSCanary10Percent30Minutes`: 10%의 트래픽으로 테스트를 하고 문제가 없는 경우 30분 후에 트래픽을 이동한다.
- AllAtOnce
  - 즉시 트래픽을 이동한다.

![40-ecs-platform.png](images%2F40-ecs-platform.png)

#### In-place Deployment

- EC2 태그 또는 ASG를 사용하여 배포할 인스턴스를 식별한다.
- 로드 밸런서 사용: 인스턴스가 업데이트되기 전에 트래픽이 중지되고 인스턴스가 업데이트된 후에 다시 시작된다.

![41-inplace-deployment.png](images%2F41-inplace-deployment.png)

- "FinanceApp"이라는 이름의 프로젝트가 있는 인스턴스를 모두 업데이트해야 한다.
  - 해당하는 인스턴스에서 InPlace 배포가 일어난다.
- 인스턴스가 ASG에 포함된 경우 ASG만 식별하여 해당 그룹에 속한 모든 인스턴스를 업데이트할 수 있다.
  - ASG 방식을 사용하면 ASG에 새로운 인스턴스가 추가되었을때 CodeDeploy가 알아서 해당 EC2 인스턴스에 새로운 버전을 배포한다.

#### In-place Deployment Hooks

- Hooks: 각 EC2 인스턴스에서 CodeDeploy가 실행할 하나 이상의 스크립트를 의미한다.

![42-inplace-deployment-hook.png](images%2F42-inplace-deployment-hook.png)

- 로드 밸런서를 사용하여 In-place 배포를 한다면 먼저 트래픽을 차단해야 한다.
  - 차단 작업은 총 세 단계로 구성되어 있다.
  - "BeforeBlockTraffic" 단계에는 트래픽을 차단하기 전에 분석 작업을 수행하기 위해 분석 플랫폼으로 트래픽을 전송해 처리한 요청의 개수 같은 것을 확인할 수 있다.
  - "BlockTraffic" 단계는 회색으로 CodeDeploy가 담당하고 있으며 로드 밸런서에서 EC2 인스턴스로 가는 트래픽을 차단한다.
  - "AfterBlockTraffic"에서는 인스턴스가 더 이상 트래픽을 수신하지 않기 때문에 원하는 작업을 수행할 수 있다.
- 다음으로 "ApplicationStop" 단계로 넘어가서 애플리케이션을 중단하는 스크립트를 실행한다.
  - CodeDeploy는 다음 "DownloadBundle" 단계에서 S3 같은 곳에서 번들을 다운로드한다.
  - "BeforeInstall" 단계에서는 파일 권한 변경 등의 기타 작업을 모두 마쳐야 한다.
  - "Install" 단계에서는 CodeDeploy 에이전트에게 다운로드한 새로운 번들을 설치하려면 인스턴스에서 파일을 어떻게 이동해야 하는지 알려준다.
  - "AfterInstall" 단계에서는 CodeDeploy에게 애플리케이션을 어떻게 시작해야 하는지, 어떤 스크립트를 실행해야 하는지 알려준다.
  - "ApplicationStart" 단계가 지나서 "ValidateService" 단계에서는 서비스가 정상적으로 실행되었는지 검증하는 스크립트를 실행할 수 있다.
- "BeforeAllowTraffic" 단계는 트래픽을 수신하기 직전에 실행해야 하는 단계다.
  - "AllowTraffic" 단계에서는 로드 밸런서가 트래픽을 다시 수신한다.
  - "AfterAllowTraffic" 단계에서는 스크립트를 실행해 트래픽 수신 후에도 애플리케이션이 정상적으로 동작하는지를 확인할 수 있다.

#### EC2 Deployment Hooks

![43-ec2-deployment-hooks.png](images%2F43-ec2-deployment-hooks.png)

- CodeDeploy 에이전트가 배포하는 파일 중에는 우리가 정의한 스크립트가 포함되어 있다.
- `appspec.yml` 파일에는 후크 섹션이 정의되어 있고, 여기에는 스크립트를 실행하는 방법이 적혀 있다.
- 이 파일을 예로 들면, "BeforeInstall", "AfterInstall", "ApplicationStart", "ValidateService" 후크가 있고 실행할 스크립트의 위치가 정의되어 있다.
  - 다시 말해, CodeDeploy 에이전트는 배포 진행 중 일정 시간을 할애하여 스크립트를 실행한다.
- "AfterInstall" 단계에서는 `appspec.yml` 파일에 "RunResourceTests.sh" 스크립트를 실행하는 것으로 정의되어 있다.
- CodeDeploy 에이전트는 환경 변수를 가져와 스크립트의 일부로 주입할 수 있다.
  - 예를 들어, 배포 그룹 이름에 CodeDeploy에서 가져온 환경 변수를 넣을 수 있다.
  - 필요할 경우 스크립트가 실행되는 중에도 배포 그룹 이름을 참조할 수 있다.

#### Deployment Hooks 예시

- BeforeInstall: 파일 복호화 및 현재 버전의 백업 작성과 같은 사전 설치 작업에 사용된다.
- AfterInstall: 애플리케이션 구성 또는 파일 사용 권한 변경 등의 작업에 사용된다.
- ApplicationStart: "ApplicationStop" 중에 중지된 서비스를 시작하는데 사용된다.
- ValidateService: 배포가 성공적으로 완료되었는지 확인하는데 사용된다.
- "BeforeAllowTraffic": 로드 밸런서에 등록하기 전에 EC2 인스턴스에서 작업을 실행한다.
  - 예를 들어, 애플리케이션에 대해 상태 검사를 수행하고 상태 검사에 성공하지 못할 경우 배포에 실패한다.

#### Blue/Green Deployments

- 수동 모드: Blue/Green용 EC2 인스턴스 프로비저닝 및 태그로 식별한다.
- 자동 모드: CodeDeploy에 의해 새로운 ASG가 프로비저닝된다. (설정 복사됨)
- Blue/Green 배포를 위해서는 로드 밸런서가 필요하다.

![44-blue-green-deployment.png](images%2F44-blue-green-deployment.png)

- 수동 모드의 경우 두 인스턴스 모두 수동으로 미리 프로비저닝해야 한다.
  - v1을 가리키고 있는 로드 밸런서에서 트래픽을 한 번에 또는 점진적으로 v2로 이동시켜 v1 버전에 트래픽이 오지 않게 해야 한다.
  - 수동 모드에서는 v1, v2 인스턴스를 수동으로 프로비저닝해야 한다.
- 자동 모드에서는 ASG를 두고 이를 참조하기만 하면 된다.
  - 자동으로 새로운 ASG가 CodeDeploy에 의해 생성되고 동일한 설정이 복사되어 적용된다.
  - 새로운 ASG가 추가되면 ALB는 v1에서 v2로 트래픽을 리디렉션한다.

#### Blue/Green Instance Termination

- BlueInstanceTerminationOption: Blue/Green 배포 후 원래 EC2 인스턴스(Blue)를 종료할지 여부를 지정한다.
- Action Terminate: 대기 시간을 지정할 수 있고 기본 1시간이며 최대 2일을 지정할 수 있다.
- Action Keep Alive: 인스턴스는 계속 실행되지만 ELB 및 배포 그룹에서 등록 취소된다.
  - 종료를 원하는 경우 사용자가 직접 종료해야 한다.

![45-blue-green-instance-termination.png](images%2F45-blue-green-instance-termination.png)

#### Blue/Green Deployment Hook

![46-blue-green-deployment-hook.png](images%2F46-blue-green-deployment-hook.png)

- Blue/Green 배포의 경우 새로 배포되는 인스턴스에 들어오고 있는 트래픽이 없기 때문에 In-place 배포와 다르다.
- 이번에는 v2 애플리케이션이 실행되는 것이 우선이므로 "ApplicationStop", "BeforeInstall", "Install", "AfterInstall" 단계가 먼저 실행된다.
- v2 애플리케이션이 정상이라고 판단되면 ELB가 v2 인스턴스로 트래픽을 전송한다.
  - 트래픽이 점차 v1에서 v2로 이동하게 되면, v1 인스턴스가 ALB로부터 받는 트래픽이 점차 줄어들게 된다.
  - v1 인스턴스에서 실행되는 후크는 "BeforeBlockTraffic", "BlockTraffic", "AfterBlockTraffic"만 남는다.
  - 이후 인스턴스는 종료되기 때문에 "ApplicationStop"이나 나머지 후크닌 실행할 필요가 없게 된다.

#### Deployment Configuration

- 배포하는 동안 언제든지 사용 가능한 상태로 유지해야 하는 인스턴스의 수를 지정할 수 있다.
- 사전에 정의된 배포 구성을 사용할 수 있다.
  - `CodeDeployDefault.AllAtOnce`: 한 번에 가능한 많은 인스턴스를 배포한다.
  - `CodeDeployDefault.HalfAtATime`: 한 번에 최대 절반의 인스턴스로 배포한다.
  - `CodeDeployDefault.OneAtATime`: 한 번에 하나의 인스턴스에만 배포한다.
- 자체 사용자 정의 배포 구성을 생성할 수 있다.

#### Trigger

- 배포 또는 EC2 인스턴스 이벤트를 SNS 토픽에 게시할 수 있다.
- 예를 들어, 배포 성공, 배포 실패, 인스턴스 실패와 같은 이벤트가 있다.

![47-code-deploy-trigger.png](images%2F47-code-deploy-trigger.png)

#### Deployment to ECS

- ECS 서비스에 대한 새 ECS 태스크 정의 배포를 자동으로 처리한다.
- Blue/Green만 지원하며 서비스가 로드 밸런서에 연결되어 있어야 한다.
- ECS 태스크 정의 및 새 컨테이너 이미지가 이미 생성되어 있어야 한다.
- 새 ECS 태스크 개정(`TaskDefinition`) 및 로드 밸런서 정보(`LoadBalancerInfo`)에 대한 참조가 `appspec.yml` 파일에 지정되어 있다.
- CodeDeploy 에이전트가 필요하지 않다.

![48-deployment-to-ecs.png](images%2F48-deployment-to-ecs.png)

- 개발자는 새로운 컨테이너 이미지를 ECR에 업로드하고 새 ECS 태스크 개정 버전을 생성한다.
  - 개정 버전이 ECR에서 생성한 새 이미지를 참조하게 한다.
- `appspec.yml` 파일을 생성해 앞서 생성한 새 ECS 태스크 정의를 참조하도록 정의하고 필요할 경우 로드 밸런서 정보도 지정한다.
  - 이렇게 생성된 `appspec.yml` 파일은 반드시 S3 버킷에 위치해야 한다.
- 여기에서 CodeDeploy를 호출할 때 우리가 실행하고자 하는 `appspec.yml` 파일을 지정하면 CodeDeploy가 실행 방법을 인지하고 어떤 ECS 태스크 정의를 배포해야 하는지 판단한다.
- CodeDeploy가 새 ECS 태스크 세트를 ECS 클러스터에 배포하고 모든 작업이 끝나면 구 버전의 ECS 태스크를 삭제한다.

![49-deployment-to-ecs.png](images%2F49-deployment-to-ecs.png)

- 코드를 푸시하는 것에서부터 ECS 클러스터에 배포하는 것까지 자동화하고 싶다면 CodePipeline을 사용하면 된다.
- 푸시된 코드는 CodeCommit을 거쳐 CodeBuild 단계로 이동하여 총 세 가지 작업을 수행한다.
  - 도커 이미지를 빌드하고 ECR로 업로드한다.
  - ECS 서비스에서 새로운 ECS 태스크 정의 개정 버전을 생성해 등록한다.
  - `appspec.yml` 파일을 생성하고 `appspec.yml` 파일은 새 ECS 작업 정의를 참조한다.
  - `appspec.yml` 예시를 확인해보면 프로퍼티 밑에 `TaskDefinition`이 있고 내용에는 새로 만든 ECS 태스크 정의의 ARN이 있다.
  - 그 밑의 `LoadBalancerInfo`에는 컨테이너 이름과 컨테이너 포트가 있다.
- CodeBuild의 모든 작업이 완료되면 CodeDeploy 단계로 넘어간다.
  - 여기서는 `appspec.yml`을 입력 아티팩트로 참조한다.
  - 이를 통해 CodeDeploy는 ECS 클러스터로 트래픽을 배포하고 이동하는 방법을 파악해 새 ECS 작업을 등록하고 기존 작업은 제거한다.
  - 이 경우에도 우리가 직접 태스크 정의와 기타 파일 등을 생성해야 하지만 전반 과정은 CodePipeline이 자동화해 준다.

![50-deployment-to-ecs.png](images%2F50-deployment-to-ecs.png)

- "Canary", "Linear", "All At Once"를 사용하여 트래픽을 새 작업 세트로 전환할 수 있다.
  - `ECSLinear10PercentEvery1Minutes`: 1분이 지날 때마다 기존 그룹의 트래픽이 10% 감소한다.
  - `ECSLinear10PercentEvery3Minutes`: 3분이 지날 때마다 기존 그룹의 트래픽이 10% 감소한다.
  - `ECSCanary10Percent5Minutes`: 5분 동안 트래픽의 10%만 새 대상 그룹으로 보내다가 모든 게 정상적으로 동작하면 5분 뒤에 나머지 트래픽을 한꺼번에 새 대상 그룹으로 보낸다.
  - `ECSCanary10Percent15Minutes`: 15분 동안 트래픽의 10%만 새 대상 그룹으로 보내다가 모든 게 정상적으로 동작하면 15분 뒤에 나머지 트래픽을 한꺼번에 새 대상 그룹으로 보낸다.
  - `ECSAllAtOnce`: 트래픽 대상 그룹을 v1에서 v2로 일괄 변경한다.
- 트래픽이 재조정되기 전에 대체(Green) 버전을 테스트할 두 번째 ELB 테스트 리스너를 정의할 수 있다.
  - 메인 운영 리스너가 모든 트래픽을 리디렉션하기 전에 새 대상 그룹으로 테스트용 트래픽을 전송할 수 있다.

#### ECS Deployment Hooks

- Hooks: ECS에서는 후크로 스크립트를 사용할 수 없기 때문에 람다 함수를 사용한다.
- `AfterAllowTestTraffic`: ELB 리스너가 트래픽을 대체 ECS 태스크 세트로 서비스하는 테스트 후 람다 함수를 실행한다.
  - 예를 들어, 애플리케이션에서 상태 검사를 수행하고 상태 검사가 성공하지 못할 경우 롤백을 트리거한다.

![51-ecs-deployment-hooks.png](images%2F51-ecs-deployment-hooks.png)

- "BeforeInstall" 후크가 있고 그 다움에는 "Install"이 있다.
  - "Install"에서는 ECS 서비스가 새 ECS 태스크 정의를 바탕으로 새 ECS 작업을 생성한다.
  - "AfterInstall"에서는 "ValidateAfterInstall"과 같은 람다 함수를 실행한다.
  - 이러한 람다 함수 호출을 통해서 새 태스크 그룹이 정상적으로 동작하는지 등을 확인할 수 있다.
- 테스트 리스너를 정의했다면 새 대상 그룹으로 테스트용 트래픽을 보낼 수 있다.
  - ECS 태스크에서 모든 게 정상적으로 동작하는지 새로운 후크와 새 람다 함수로 검증한다.
  - 이런 식으로 람다 함수를 실행해 새 인스턴스가 테스트용 트래픽을 제대로 처리하는지 확인할 수 있다.
- "BeforeAllowTraffic" 단계는 트래픽을 허용하기 전 즉, 운영 트래픽을 옮기기 전 단계다.
  - "AllowTraffic" 단계에서는 운영 트래픽을 실제로 이동시킨다.
  - 마지막으로 트래픽을 옮긴 다음에는 "AfterAllowTraffic"이라는 또 다른 람다 함수를 호출해 새 ECS 작업이 샐제로 트래픽을 잘 처리하고 있는지 확인할 수 있다.

#### Deployment to Lambda

- 새로운 람다 함수 버전을 만들고 배포하려는 경우에 사용된다.

![52-deployment-to-lambda.png](images%2F52-deployment-to-lambda.png)

- 새로운 람다 버전을 생성한 다음 이를 별칭에 배포하는 방식이다.
- 여기서 v1을 가리키고 있는 별칭이 v2를 가리키도록 만들어야 한다.
- `appspec.yml` 파일에 새로운 버전에 대한 정보를 정의하고 S3 버킷에 저장한다.
  - `appspec.yml` 파일을 사용해 CodeDeploy 서비스를 호출한다.
  - CodeDeploy가 자동으로 람다 별칭을 업데이트하여 새 버전인 v2를 참조하게 된다.
  - 트래픽도 자동으로 v1에서 v2로 이동하여 모든 게 업데이트 된다.
- CodeDeploy 에이전트는 필요하지 않다.

#### appspec.yml

```yaml
version: 0.0

Resources:
  - myLambdaFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: myLambdaFunction
        Alias: myLambdaFunctionAlias
        CurrentVersion: 1
        TargetVersion: 2
```

- Name (필수): 배포할 람다 함수의 이름이다.
- Alias (필수): 람다 함수에 대한 별칭 이름이다.
- CurrentVersion (필수): 현재 람다 함수 트래픽의 버전이다.
- TargetVersion (필수): 람다 함수 트래픽이 시프트되어야 하는 새로운 버전이다.

![53-deployment-to-lambda.png](images%2F53-deployment-to-lambda.png)

- 람다를 자동화 방식으로 배포하기 위해 CodePipeline이 사용된다.
- CodeCommit에 코드를 푸시하면 CodeBuild가 코드를 빌드한 다음 새로운 람다 버전을 생성하고, `appspec.yml` 파일을 생성한다.
- CodeDeploy는 `appspec.yml` 파일을 입력 아티팩트로 받아 이를 참고해 트래픽의 목적지를 우리가 만든 람다 함수 별칭의 현재 버전에서 대상 버전으로 바꾼다.

![54-deployment-to-lambda.png](images%2F54-deployment-to-lambda.png)

- Blue/Green 배포만 지원된다.
- "Canary", "Linear", "All At Once"를 사용하여 트래픽을 새로운 람다 함수로 전환할 수 있다.
  - `LambdaLinear10PercentEvery1Minute`: 1분마다 새로운 버전으로 향하는 트래픽이 10% 증가한다.
  - `LambdaLinear10PercentEvery(2,3,10)Minutes`: 2,3,10분마다 새로운 버전으로 향하는 트래픽이 10% 증가한다.
  - `LambdaCanary10Percent(5,10,15,30)Minutes`: 5,10,15,30분동안 10%의 트래픽만 새로운 버전으로 보내다가 한 번에 전환한다.
  - `LambdaAllAtOnce`: 한 번에 모든 트래픽을 새로운 함수로 전달한다.

#### Lambda Deployment Hooks

- Hooks: 배포당 한 번씩 실행되는 람다 함수다.

![55-lambda-deployment-hooks.png](images%2F55-lambda-deployment-hooks.png)

- "BeforeAllowTraffic"과 "AfterAllowTraffic"에서만 후크를 실행할 수 있다.
- 예를 들어, "BeforeAllowTraffic"을 사용하면 "ValidateBeforeTraffic" 같은 람다 함수를 호출하여 이 단계에 필요한 상태 확인 등을 수행할 수 있다.
- "AfterAllowTraffic"에서 새 람다 버전이 배포된 뒤에 모든 게 정상적으로 동작하는지 확인할 수도 있다.

#### Redeploy & Rollbacks

- Rollback: 이전에 배포한 애플리케이션의 버전을 다시 배포한다.
- 배포를 롤백할 수 있다.
  - Automatically: 배포 실패 시 롤백 또는 CloudWatch Alarm 임계값을 충족할 때 롤백된다.
  - Manually
- 롤백이 수행되지 않도록 구성할 수 있다.
- **롤백이 발생하면 CodeDeploy는 마지막 정상 버전을 사용해 새로운 배포를 실행**한다. 
  - 복원된 버전을 배포하는 것이 아니다.

#### Troubleshooting

- 배포 에러: `InvalidSignatureException - Signature expired: [time] is now earlier than [time]`
  - CodeDeploy가 작업을 수행하려면 정확한 시간 참조가 필요하다.
  - EC2 인스턴스의 날짜와 시간이 올바르게 설정되지 않은 경우 배포 요청의 서명 날짜와 일치하지 않을 수 있으며, 이 날짜는 CodeDeploy에서 거부한다.
- 문제를 해결하기 위해서는 EC2 인스턴스의 시간과 AWS에서 제공하는 시간을 동기화하는 방법을 사용해야 한다.

![56-troubleshooting.png](images%2F56-troubleshooting.png)

- 배포 또는 모든 라이프사이클 이벤트를 건너뛰면(EC2/On-Premise) 다음의 오류 중 하나가 발생한다.
  - "The overall deployment failed because too many individual instances failed deployment"
  - "Too few healthy instances are available for deployment"
  - "Some instances in your deployment group are experiencing problems (Error code: HEALTH_CONSTRAINTS)"
- 원인
  - CodeDeploy 에이전트가 설치되어 있지 않거나 실행 중이거나 CodeDeploy에 연결할 수 없다.
  - CodeDeploy Service 역할 또는 IAM 인스턴스 프로파일에 필요한 사용 권한이 없을 수 있다.
  - HTTP Proxy를 사용하는 경우 `:proxy_uri:` 파라미터를 사용하여 CodeDeploy 에이전트를 구성한다.
  - CodeDeploy와 Agent 간의 날짜 및 시간 불일치가 발생한다.

![57-troubleshooting.png](images%2F57-troubleshooting.png)

- ASG로의 CodeDeploy 배포가 진행 중이고 스케일아웃 이벤트가 발생하면 새 인스턴스는 현재 배포 중인 애플리케이션 버전이 아닌 가장 최근에 배포된 애플리케이션으로 업데이트된다.
- ASG에는 다양한 버전의 애플리케이션을 호스팅하는 EC2 인스턴스가 있다.
  - v1 버전과 v2 버전이 혼재하게 된다.
- 기본적으로 CodeDeploy는 모든 오래된 EC2 인스턴스를 업데이트하기 위해 자동으로 후속 배포를 시작하여 혼재된 버전을 새로운 버전으로 통일시킨다.
- 일반적으로 두 버전을 동시에 실행하는 시간이 존재한다.

![58-troubleshooting.png](images%2F58-troubleshooting.png)

- Issue: 배포 로그에 오류가 보고되지 않은 Blue/Green 배포에 트래픽 라이프사이클 허용 이벤트가 실패한다.
  - 원인: ELB에서 상태 점검이 잘못 구성되었다.
  - 해결 방법: ELB 상태 점검 구성의 오류를 검토하고 수정한다.

![59-troubleshooting.png](images%2F59-troubleshooting.png)

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)