# Serverless

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "서버리스 서비스"에 대해서 알아보도록 한다.

---

### Elastic Beanstalk

![1-typical-architecture.png](images%2F1-typical-architecture.png)

- 대부분의 웹/앱은 동일한 아키텍처(ALB+ASG)를 갖는다.
- 대부분의 서비스가 동일한 아키텍처를 따른다면 매번 새롭게 생성하는 것은 비효율적이다.

- "Elastic Beanstalk"는 애플리케이션 배포에 대한 개발자 중심의 관점이다.
- 위에서 살펴본 이미지의 구성요소(EC2, ASG, ELB, RDS 등)를 사용한다.
- 관리형 서비스다.
    - 용량 프로비저닝, 로드 밸런식, 확장, 애플리케이션 자동 처리 상태 모니터링, 인스턴스 구성 등
    - 애플리케이션 코드만 개발자의 책임이다.
- 완전 관리되는 서비스지만 개발자가 구성을 완전히 제어할 수 있다.
- "Beanstalk"에 대한 비용은 무료이며 생성된 서비스들에 대한 비용만 지불하면 된다.

#### Component

- Application: "Elastic Beanstalk"의 구성 요소 모음이다.(Environment, Version, Configuration 등)
- Application Version: 애플리케이션 코드의 반복이다.
- Environment:
    - 애플리케이션 버전을 실행하는 AWS 리소스 모음이다.(한 번에 하나의 애플리케이션 버전만 가질 수 있다.)
    - Tiers: 웹 서버 환경 계층 및 작업자 환경 계층이다.
    - 여러 환경(Dev, Test, Prod 등)을 생성할 수 있다.
- 새로운 버전을 업로드하고 환경을 시작하고, 이후 라이프사이클을 관리한다.
- 반복하고 싶다면 새로운 버전을 업로드해서 버전을 업데이트하고 새로운 환경을 다시 배포하여 애플리케이션 스택을 업데이트할 수 있다.

![2-beanstalk-component.png](images%2F2-beanstalk-component.png)

#### Supported Platform

- 여러 플랫폼을 지원하며 원하는 플랫폼이 없는 경우 사용자 정의 플랫폼을 작성할 수 있다.
    - Go
    - Java SE
    - Java with Tomcat
    - .NET Core on Linux
    - .NET on Windows Server
    - Node.js
    - PHP
    - Python
    - Ruby
    - Packer Builder
    - Single Container Docker
    - Multi-Container Docker
    - Preconfigured Docker

#### Web Server Tier vs Worker Tier

![3-web-server-tier.png](images%2F3-web-server-tier.png)

- 웹 계층은 전통적인 아키텍처로 부하 응답이 있다.
- 웹 서버가 될 여러 EC2 인스턴스를 가진 "Auto Scaling Group"으로 트래픽을 전송한다.

![4-worker-tier.png](images%2F4-worker-tier.png)

- Beanstalk 아키텍처는 Worker 환경을 중심으로 한다.
- EC2 인스턴스에 직접 액세스하는 클라이언트가 없다.
- SQS를 이용하여 EC2 인스턴스로 메시지를 전송하여 처리하도록 한다.
- Worker 환경의 경우 SQS의 메시지가 많을수록 EC2 인스턴스의 수도 늘릴 것이다.

#### Deployment Mode

![5-deployment-mode.png](images%2F5-deployment-mode.png)

- Beanstalk에는 두 가지 배포 모드가 있다.
- 첫 번째는 단일 인스턴스를 사용한다.
    - 단일 EC2 인스턴스, 단일 RDS를 사용하여 개발 용도로 적합하다.
- 두 번째는 확장이 가능한 고가용성 모드다.
    - 다중 EC2 인스턴스, Load Balancer, 다중 AZ RDS를 통해서 운영 환경에서의 가용성을 높일 수 있다.

#### Deployment Mode for Update

- All at once: 한번에 모두 배포한다.
  - 가장 빠르지만 인스턴스가 자밋 동안 트래픽을 처리할 수 없는 다운타임이 발생한다.
- Rolling: 한 번에 몇 개의 인스턴스(버킷)를 업데이트한 후 첫 번재 버킷이 정상 상태가 되면 다음 버킷으로 이동한다.
- Rolling with additional batches: Rolling과 비슷하지만 배치를 이동하기 위해 새 인스턴스를 가동한다.
  - 이전 애플리케이션을 계속 사용할 수 있다.
- Immutable: 새 ASG에서 새로운 인스턴스를 가동하고 이러한 인스턴스에 버전을 배포한 다음 모든 것이 정상일 때 모든 인스턴스를 교체한다.
- Blue Green: 새로운 환경을 만들고 준비가 되면 전환한다.
- Traffic Splitting: Canary 테스트를 통해 적은 비율의 트래픽을 새로운 배포로 전달한다.

#### All at once

- 가장 빠른 배포지만 다운타임이 발생한다.
- 개발 환경에서 빠른 반복에 적합하다.
- 추가 비용이 발생하지 않는다.

![6-all-at-once.png](images%2F6-all-at-once.png)

- v1 버전의 EC2 인스턴스가 4개 실행되고 있다.
- Elastic Beanstalk가 먼저 모든 EC2 인스턴스에서 애플리케이션을 중단한다.
  - 회색으로 표시된 부분은 아무것도 실행되지 않는 부분이다.
- 그런 다음 v2 버전의 애플리케이션을 배포한다.

#### Rolling

- 애플리케이션이 용량이 부족한 상태로 실행된다.
- 배포되는 크기인 버킷의 크기를 지정할 수 있다.
- 동시에 두 버전의 애플리케이션이 실행되는 시점이 있다.
- 추가 비용이 발생하지 않지만 인스턴스의 수가 아주 많은 경우 배포하는데 많은 시간이 소요될 수 있다.

![7-rolling.png](images%2F7-rolling.png)

- v1 버전의 EC2 인스턴스가 4개 실행되고 있다.
- 예시에서는 버킷의 크기가 2로 설정되어 있다.
- 첫 2개의 인스턴스의 애플리케이션이 중단된다.
  - 남은 2개의 인스턴스에서는 v1 버전의 애플리케이션이 여전히 실행 중이다.
- 첫 2개의 인스턴스가 업데이트되어 v2 버전을 실행한다.
- 남은 2개의 인스턴스가 종료되고 v2 버전의 애플리케이션이 새롭게 실행된다.

#### Rolling with additional batches

- 애플리케이션이 용량이 부족한 상태로 실행된다.
- 배포되는 크기인 버킷의 크기를 지정할 수 있다.
- 동시에 두 버전의 애플리케이션이 실행되는 시점이 있다.
- 소액의 추가비용이 발생하고 배포되는데 소요되는 시간이 오래걸린다.
- 배포가 종료되면 추가 배치가 제거된다.
- 운영 환경에 적합하다.

![8-rolling-additional-batches.png](images%2F8-rolling-additional-batches.png)

- v1 버전의 EC2 인스턴스가 4개 있다.
- v2 버전의 새로운 애플리케이션을 2개의 EC2 인스턴스를 추가하여 실행한다.
- 처음 v1 애플리케이션 2개를 중단하고 v2로 업데이트한다.
- 남은 2개의 v1 애플리케이션을 v2로 업데이트한다.
- 최초에 여분으로 생성한 2개의 인스턴스를 종료한다.

#### Immutable

- 다운타임이 발생하지 않는다.
- 새로운 코드는 임시 ASG의 새로운 인스턴스에 배포된다.
- 높은 비용이 발생하며 배포를 위해 두 배의 용량을 사용한다.
- 가장 긴 배포 시간이 소요된다.
- 장애가 발생한 경우 가장 빠른 롤백을 지원한다.
- 운영 환경에 적합하다.

![9-immutable.png](images%2F9-immutable.png)

- ASG에는 3개의 인스턴스에서 v1 애플리케이션을 실행 중이다.
- 배포를 위해 새로운 임시 ASG를 생성한다.
- 먼저 Beanstalk가 하나의 인스턴스만 시작해 정상적으로 작동하는지 확인한다.
  - 성공적으로 상태를 확인하면 나머지 인스턴스를 전부 시작한다.
- 준비가 되면 기존 ASG에 임시 ASG를 병합한다.
  - 임시 ASG의 모든 인스턴스를 현재 ASG로 옮긴다.
- 옮기는 작업이 완료되면 v1 애플리케이션을 모두 종료한다.

#### Blue/Green

- Elastic Beanstalk에서 직접적으로 제공하는 기능이 아니다.
- 다운타임이 없고 릴리즈 기능에 유용해 더 많은 테스트를 할 수 있다는 장점이 있다.
- 새로운 "Stage" 환경을 생성하고 새로운 환경에 새로운 버전을 배포한다.
- 새로운 환경(Green)을 독립적으로 검증하고 문제가 있는 경우 롤백할 수 있다.
- Route53은 가중치 정책을 사용하여 약간의 트래픽을 스테이지 환경으로 리디렉션하도록 설정할 수 있다.
- Beanstal를 사용하여 환경 테스트 관려되면 "swap URLs"를 사용할 수 있다.

![10-blue-green.png](images%2F10-blue-green.png)

- Elastic Beanstalk의 Blue 환경에서 v1을 전부 실행하고 있고 Green 환경에서 v2를 배포한다.
- 동시에 실행되지만 Route53으로 가중치를 설정해 Blue 환경에 트래픽의 90%를 전송하여 안전이 보장된 인스턴스에 대부분의 트래픽을 보낸다.
  - 트래픽의 10% 정도만 Green 환경으로 보내 정상적으로 작동하는지 확인할 수 있다.
- 테스트가 완료되면 v1 환경으로 전달되던 트래픽을 v2 환경으로 이동시킨다.

#### Traffic Splitting

- **Canary 테스트에 사용**된다.
- 새로운 애플리케이션 버전이 동일한 용량의 임시 ASG로 배포된다.
- 구성 가능한 시간 동안 소량의 트래픽이 임시 ASG로 전송된다.
- 배포 상태가 모니터링된다.
- 배포에 실패하면 자동으로 롤백이 시작되며 매우 빠르게 작동한다.
- 애플리케이션 다운타임이 발생하지 않는다.
- 새 인스턴스가 임시 ASG에서 원본 ASG로 마이그레이션된다.
- 이전 애플리케이션 버전은 종료된다.

![11-traffic-splitting.png](images%2F11-traffic-splitting.png)

- 새로운 애플리케이션 버전을 동일한 용량의 임시 ASG로 배포한다.
  - 전체 용량은 두 배가 된다.
- 이렇게 구성된 상태에서 약간의 트래픽을 정해진 시간 동안 임시 ASG로 전송한다.
  - ALB를 사용해 트래픽의 90%를 메인 ASG로 전송하고 10%만 임시 ASG로 전송한다.
  - 이러한 작업들은 전부 자동으로 진행된다.
- 새로운 버전에 문제가 없다고 판단되면 모든 인스턴스는 메인 ASG로 마이그레이션된다.

#### Elastic Beanstalk Deployment Summary

- 아래의 표를 통해 배포 방식의 차이를 확인할 수 있고 자세한 내용은 [공식 홈페이지](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html)를 참고하도록 한다.

![12-deployment-summary.png](images%2F12-deployment-summary.png)

#### Web Server vs Worker Environment

- 애플리케이션에서 완료해야 할 작업을 수행하는 경우 이러한 작업을 전용 작업자 환경으로 오프로드한다.
- 애플리케이션을 두 계층으로 분리하는 것은 일반적으로 사용되는 방식이다.
- 예를 들어, 동영상 처리, zip 파일 생성등의 작업에 사용될 수 있다.
- `cron.yaml` 파일에서 주기적으로 실행되는 태스크를 정의할 수 있다.

![13-web-server-worker-environment.png](images%2F13-web-server-worker-environment.png)

- Web 계층에는 ELB와 EC2가 구성되어 있으며 고객의 요청을 받는 역할을 한다.
- 자동 확장을 위하여 Auto Scaling Group을 사용하고 있다.
- Worker 계층에는 SQS와 EC2가 구성되어 있으며 Web 계층에서 SQS 큐에 넣은 데이터를 처리한다.
- SQS 큐의 크기에 맞추어 EC2 인스턴스가 Auto Scaling된다.

#### Notification

- EventBridge에서 규칙을 생성하여 다음 이벤트에 대처할 수 있다.
  - Environment Operations Status: 생성, 수정, 종료 (시작, 성공, 실패)
  - Other Resources Status: ASG, ELB, EC2 인스턴스 (생성, 삭제)
  - Managed Updates Status: 시작, 실패
  - Environment Health Status

![14-notifications.png](images%2F14-notifications.png)

---

### AWS SAM - Serverless Application Model

- 서버리스 애플리케이션 개발 및 구축을 위한 프레임워크다.
- 모든 구성은 YAML 기반의 코드로 작성된다.
  - 람다 함수(`AWS::Serverless::Function`)
  - DynamoDB 테이블(`AWS::Serverless::SimpleTable`)
  - API 게이트웨이(`AWS::Serverless:API`)
  - Step Function - State Machine (`AWS::Serverless:StateMachine`)
- SAM은 람다, API 게이트웨이, DynamoDB를 로컬로 실행하는 데 도움을 줄 수 있다.
- SAM은 CodeDeploy를 사용하여 람다 함수(트래픽 시프트)를 구현할 수 있다.
- 백엔드에서 CloudFormation을 활용할 수 있다.

#### Recipe

- 전송 헤더는 SAM 템플릿임을 나타낸다.
  - Transform: `AWS::Serverless-2016-10-31`
- 코드 작성
  - `AWS::Serverless::Function`
  - `AWS::Serverless:Api`
  - `AWS::Serverless:SimpleTable`
- 패키지 및 배포
  - `aws cloudformation package / sam package`
  - `aws cloudformation deploy / sam deploy`

#### SAM Deployment

![15-sam-deployment.png](images%2F15-sam-deployment.png)

- YAML로 작성된 SAM 템플릿과 애플리케잇녀 코드가 있다.
- `sam build`를 수행하면 로컬에서 애플리케이션을 빌드한다.
- 변환된 CloudFormation 템플릿과 애플리케이션 코드가 생성된다.
- `sam package` 또는 `aws cloudformation package` 명령을 실행해 함수를 패키징하고 S3 버킷에 업로드한다.
  - CloudFormation 템플릿과 애플리케이션 코드가 모두 업로드된다.
- `sam deploy` 명령을 통해 S3 버킷에서 애플리케이션을 배포한다.
  - ChangeSet을 생성하고 CloudFormation에 실행해 애플리케이션 배포를 실행한다.
  - CloudFormation이 람다 함수, API Gateway, DynamoDB를 배포한다.

#### CLI Debugging

- AWS SAM 템플릿을 사용하여 정의된 서버리스 애플리케이션을 로컬에서 구축, 테스트 및 디버깅할 수 있다.
- 로컬에서 람다 함수 실행 환경을 제공한다.
- SAM CLI + AWS Toolkits를 통해서 코드 단계별 실행 및 디버깅을 할 수 있다.
- AWS Cloud9, VS Code, JetBrains, PyCharm, IntelliJ 등을 지원한다.
- AWS Toolkits: AWS SAM을 사용하여 구축된 람다 함수를 구축, 테스트, 디버깅, 배포 및 호출할 수 있는 IDE 플러그인이다.

![16-cli-debugging.png](images%2F16-cli-debugging.png)

#### CodeDeploy 연동

- SAM 프레임워크는 기본적으로 CodeDeploy를 사용하여 람다 함수를 업데이트한다.
- 트래픽 시프트를 지원한다.
- 배포를 검증하기 위한 사전 및 사후 트래픽 후크를 사용할 수 있다.
  - 트래픽 이동 시작 전과 종료 후에 검증한다.
- CloudWatch 경보를 사용한 쉽고 자동화된 롤백을 지원한다.

![17-sam-codedeploy.png](images%2F17-sam-codedeploy.png)

- 람다 별칭에 v1 람다 함수가 있다.
- CodeDeploy 배포를 트리거해 별칭을 v2로 업데이트해야 한다.
- CodeDeploy에서 선택 사항인 사전 트래픽 후크 테스트로 다른 람다 함수를 사용한다.
- 별칭으로 트래픽 시프트를 수행한다.
- 선택적으로 CloudWatch 알람을 모니터링해 배포 과정에 문제가 없는지 확인한다.
- 선택적으로 배포가 완료되면 사후 트래픽 후크 람다 함수를 실행할 수 있다.
- 배포가 완료되면 별칭에서 v1 람다 함수는 제거된다.

#### YAML

![18-sam-codedeploy.png](images%2F18-sam-codedeploy.png)

- **AutoPublishAlias**
  - 새 코드가 배포되는 시기를 감지한다.
  - 최신 코드로 해당 함수의 업데이트된 버전을 생성하고 게시한다.
  - 별칭이 업데이트된 버전의 람다 함수를 가리킨다.
- **DeploymentPreference**
  - Canary, Linear, AllAtOnce 중에 선택할 수 있다.
- **Alarms**
  - 롤백을 실행할 수 있는 경보다.
- **Hooks**
  - 배포 테스트를 위한 사전 및 사후 트래픽 시프트 람다 함수를 지정한다.

---

### AWS Cloud Development Kit (CDK)

- 친숙한 언어를 사용하여 클라우드 인프라를 정의한다.
  - Javascript, Typescript, Python, Java, .NET
- 코드는 CloudFormation 템플릿(JSON/YAML)으로 컴파일된다.
- 인프라와 애플리케이션 런타임 코드를 함께 배포할 수 있다.
  - 람다 함수에 적합하다.
  - ECS/EKS의 도커 컨테이너에 적합하다.

![19-cloud-development-kit.png](images%2F19-cloud-development-kit.png)

#### Example

![20-cdk-example.png](images%2F20-cdk-example.png)

- 예시에서는 Javascript가 사용되고 있다.
- 정의된 VPC와 ECS 클러스터와 Fargate 서비스와 함께 ALB가 정의되어 있다.
- 세 가지 서비스는 CDK CLI에 의해 컴파일되어 CloudFormation 템플릿에 사용되고 업로드 및 배포할 수 있다.

#### CDK vs SAM

- SAM
  - 서버리스 중심이다.
  - JSON 또는 YAML로 템플릿을 선언적으로 작성한다.
  - 람다를 빠르게 시작하는데 적합하다.
  - CloudFormation을 사용한다.
- CDK
  - 모든 AWS 서비스를 지원한다.
  - 프로그래밍 언어 JavaScript/TypeScript, Python, Java 및 .NET으로 인프라를 구축한다.
  - CloudFormation을 활용할 수 있다.

#### CDK + SAM

- SAM CLI를 사용하여 CDK 앱을 로컬에서 테스트할 수 있다.
- 먼저 `cdk synth`를 실행해야 한다.

![21-cdk-sam.png](images%2F21-cdk-sam.png)

- CDK 애플리케이션의 람다 함수에 `cdk synth` 명령어를 실행하면 CDK에 통합된 CloudFormation 템플릿이 생성된다.
- CloudFormation 템플릿으로 SAM 프레임워크를 사용해 SAM을 로컬에서 호출해 CloudFormation 템플릿으로 함수를 호출할 수 있다.

#### 실습

![22-cdk-hands-on.png](images%2F22-cdk-hands-on.png)

- CDK로 S3 버킷을 생성하고 람다 함수로 Rekognition을 호출하고 DynamoDB 테이블에 작업의 결과를 저장하는데 전부 CDK를 사용할 수 있다.
- CloudTrail을 준비하여 CDK에서 `step.sh`를 실행하여 명령어를 실행할 준비를 한다.

- `step.sh` 파일은 아래와 같다.

```shell
# 1. install the CDK
sudo npm install -g aws-cdk
# directory name must be cdk-app/ to go with the rest of the tutorial, changing it will cause an error
mkdir cdk-app
cd cdk-app/
# initialize the application
cdk init --language javascript
# verify it works correctly
cdk ls
# install the necessary packages
npm install @aws-cdk/aws-s3 @aws-cdk/aws-iam @aws-cdk/aws-lambda @aws-cdk/aws-lambda-event-sources @aws-cdk/aws-dynamodb
# 2. copy the content of cdk-app-stack.js into lib/cdk-app-stack.js
# 3. setup the Lambda function
mkdir lambda && touch index.py
# 4. bootstrap the CDK application
cdk bootstrap
# 5. (optional) synthesize as a CloudFormation template
cdk synth
# 6. deploy the CDK stack
cdk deploy
# 7. empty the s3 bucket
# 8. destroy the stack
cdk destroy
```

- `sudo npm install -g aws-cdk` 명령을 통해서 `aws-cdk-lib`를 설치한다.
- `mkdir cdk-app` 명령을 통해서 `cdk-app` 디렉토리를 생성할 수 있다.
- javascript를 사용하기 위해 `cdk init --language javascript` 명령을 실행한다.
- 정상적으로 초기화가 완료되었다면 `cdk ls` 명령어가 CdkAppStack을 반환해야 한다.

- `cdk-app-stack.js` 파일은 아래와 같다.

```
const cdk = require("@aws-cdk/core");
const s3 = require("@aws-cdk/aws-s3");
const iam = require("@aws-cdk/aws-iam");
const lambda = require("@aws-cdk/aws-lambda");
const lambdaEventSource = require("@aws-cdk/aws-lambda-event-sources");
const dynamodb = require("@aws-cdk/aws-dynamodb");

const imageBucket = "cdk-rekn-imagebucket";

class CdkAppStack extends cdk.Stack {
    /**
     *
     * @param {cdk.Construct} scope
     * @param {string} id
     * @param {cdk.StackProps=} props
     */
    constructor(scope, id, props) {
        super(scope, id, props);

        // The code that defines your stack goes here

        // ========================================
        // Bucket for storing images
        // ========================================
        const bucket = new s3.Bucket(this, imageBucket, {
            removalPolicy: cdk.RemovalPolicy.DESTROY,
        });
        new cdk.CfnOutput(this, "Bucket", { value: bucket.bucketName });

        // ========================================
        // Role for AWS Lambda
        // ========================================
        const role = new iam.Role(this, "cdk-rekn-lambdarole", {
            assumedBy: new iam.ServicePrincipal("lambda.amazonaws.com"),
        });
        role.addToPolicy(
            new iam.PolicyStatement({
                effect: iam.Effect.ALLOW,
                actions: [
                    "rekognition:*",
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents",
                ],
                resources: ["*"],
            })
        );

        // ========================================
        // DynamoDB table for storing image labels
        // ========================================
        const table = new dynamodb.Table(this, "cdk-rekn-imagetable", {
            partitionKey: { name: "Image", type: dynamodb.AttributeType.STRING },
            removalPolicy: cdk.RemovalPolicy.DESTROY,
        });
        new cdk.CfnOutput(this, "Table", { value: table.tableName });

        // ========================================
        // AWS Lambda function
        // ========================================
        const lambdaFn = new lambda.Function(this, "cdk-rekn-function", {
            code: lambda.AssetCode.fromAsset("lambda"),
            runtime: lambda.Runtime.PYTHON_3_8,
            handler: "index.handler",
            role: role,
            environment: {
                TABLE: table.tableName,
                BUCKET: bucket.bucketName,
            },
        });
        lambdaFn.addEventSource(
            new lambdaEventSource.S3EventSource(bucket, {
                events: [s3.EventType.OBJECT_CREATED],
            })
        );

        bucket.grantReadWrite(lambdaFn);
        table.grantFullAccess(lambdaFn);
    }
}

module.exports = { CdkAppStack };
```

- S3 버킷, 람다를 위한 역할, DynamoDB, 람다 함수를 생성하고 있다.
- 아래는 `index.py` 파일이다.

```python
#
# Lambda function detect labels in image using Amazon Rekognition
#

from __future__ import print_function
import boto3
import json
import os
from boto3.dynamodb.conditions import Key, Attr

minCofidence = 60


def handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']

    rekFunction(bucket, key)


def rekFunction(bucket, key):
    print("Detected the following image in S3")
    print("Bucket: " + bucket + " key name: " + key)

    client = boto3.client("rekognition")

    response = client.detect_labels(Image={"S3Object": {"Bucket": bucket, "Name": key}},
                                    MaxLabels=10, MinConfidence=minCofidence)

    # Get the service resource
    dynamodb = boto3.resource("dynamodb")

    # Instantiate a table resource object
    imageLabelsTable = os.environ["TABLE"]
    table = dynamodb.Table(imageLabelsTable)

    # Put item into table
    table.put_item(
        Item={"Image": key}
    )

    objectsDetected = []

    for label in response["Labels"]:
        newItem = label["Name"]
        objectsDetected.append(newItem)
        objectNum = len(objectsDetected)
        itemAtt = f"object{objectNum}"
        response = table.update_item(
            Key={"Image": key},
            UpdateExpression=f"set {itemAtt} = :r",
            ExpressionAttributeValues={":r": f"{newItem}"},
            ReturnValues="UPDATED_NEW"
        )

```

---

### AWS Step Functions

- 워크플로를 "State Machine"으로 모델링한다.
  - 워크플로 하나당 "State Machine"을 생성해야 한다.
  - 주문 이행, 데이터 처리 등에 사용된다.
  - 웹 애플리케이션 등 모든 워크플로를 지원한다.
- JSON으로 작성된다.
- 워크플로 시각화 및 워크플로 실행, 이력을 제공한다.
- SDK 호출, API 게이트웨이, EventBridge(CloudWatch 이벤트)로 워크플로를 시작할 수 있다.

![23-step-function.png](images%2F23-step-function.png)
  
#### Task States

- "State Machine"에서 일부 작업을 수행한다.

![24-task-states.png](images%2F24-task-states.png)

- 하나의 AWS 서비스를 호출한다.
  - 람다 함수를 호출할 수 있다.
  - AWS Batch 작업을 실행할 수 있다.
  - ECS 태스크를 실행하고 완료될 때까지 기다린다.
  - DynamoDB에서 항목에 삽입할 수 있다.
  - SNS, SQS에 메시지를 게시한다.
  - 다른 Step Function의 워크플로를 실행한다.
- 하나의 활동을 실행한다.
  - EC2, ECS, 온프레미스
  - 활동은 작업에 대한 Step Function 기능을 폴링한다.
  - 활동은 결과를 다시 Step Function으로 보낸다.

- 아래는 람다 함수를 호출하는 JSON 파일의 예시다.

```json
"Invoke Lambda function": {
  "Type": "Task",
  "Resource": "arn:aws:states:::lambda:invoke",
  "Parameters": {
    "FunctionName": "arn:aws:lambda:REGION:ACCOUNT_ID:function:FUNCTION_NAME",
    "Payload": {
      "Input.$": "$"    
    }
  },
  "Next": "NEXT_STATE",
  "TimeoutSeconds": 300
}
```

#### States

- Choice State: 브랜치로 보낼 조건을 테스트한다.
- Fail or Succeed State: 실패 또는 성공 시 실행을 중지한다.
- Pass State: 작업을 수행하지 않고 입력을 출력으로 전달하거나 일부 고정 데이터를 삽입하기만 하면 된다.
- Wait State: 특정 시간 동안 또는 지정된 시간/날짜까지 지연을 제공한다.
- Map State: 단계를 동적으로 반복한다.
- Parallel State: 실행의 병렬 분기를 시작한다.

#### Visual Workflow

![25-visual-workflow.png](images%2F25-visual-workflow.png)

- 작업을 시작하면 X초 동안 대기하고 작업 상태를 가져와 작업이 완료되었는지 확인한다.
- 완료되지 않았다면 다시 기다리고 반복된다.
- 작업이 완료되면 작업이 성공적으로 끝나 작업 완료 상태를 가져오거나 작업이 실패했다면 실패를 처리해야 할 수도 있다.

- 실행이 시작되면 작업을 제출하고 X초 동안 대기한다.
- 파란색이 진행 중인 단계다.
- 여러 번 반복하고 작업 완료 상태가 반환되면 마지막 단계로 넘어간다.

#### 예시

- 아래는 "State Machine" 예시다.

```json
{
  "Comment": "A Hello World example of the Amazon States Language using Pass states",
  "StartAt": "Lambda Invoke",
  "States": {
    "Lambda Invoke": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "OutputPath": "$.Payload",
      "Parameters": {
        "Payload.$": "$",
        "FunctionName": "<ENTER FUNCTION NAME HERE>"
      },
      "Retry": [
        {
          "ErrorEquals": [
            "Lambda.ServiceException",
            "Lambda.AWSLambdaException",
            "Lambda.SdkClientException",
            "Lambda.TooManyRequestsException"
          ],
          "IntervalSeconds": 1,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }
      ],
      "Next": "Choice State"
    },
    "Choice State": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$",
          "StringMatches": "*Stephane*",
          "Next": "Is Teacher"
        }
      ],
      "Default": "Not Teacher"
    },
    "Is Teacher": {
      "Type": "Pass",
      "Result": "Woohoo!",
      "End": true
    },
    "Not Teacher": {
      "Type": "Fail",
      "Error": "ErrorCode",
      "Cause": "Stephane the teacher wasn't found in the output of the Lambda Function"
    }
  }
}
```

- 아래는 에러 핸들링을 위한 예시다.

```json
{
  "Comment": "A Retry and Catch example of the Amazon States Language using an AWS Lambda Function",
  "StartAt": "InvokeMyFunction",
  "States": {
    "InvokeMyFunction": {
      "Type": "Task",
      "Resource": "<enter resource ARN here>",
      "Retry": [
        {
          "ErrorEquals": [
            "CustomError"
          ],
          "IntervalSeconds": 1,
          "MaxAttempts": 2,
          "BackoffRate": 2
        },
        {
          "ErrorEquals": [
            "States.TaskFailed"
          ],
          "IntervalSeconds": 30,
          "MaxAttempts": 2,
          "BackoffRate": 2
        },
        {
          "ErrorEquals": [
            "States.ALL"
          ],
          "IntervalSeconds": 5,
          "MaxAttempts": 5,
          "BackoffRate": 2
        }
      ],
      "Catch": [
        {
          "ErrorEquals": [
            "CustomError"
          ],
          "Next": "CustomErrorFallback"
        },
        {
          "ErrorEquals": [
            "States.TaskFailed"
          ],
          "Next": "ReservedTypeFallback"
        },
        {
          "ErrorEquals": [
            "States.ALL"
          ],
          "Next": "CatchAllFallback"
        }
      ],
      "End": true
    },
    "CustomErrorFallback": {
      "Type": "Pass",
      "Result": "This is a fallback from a custom lambda function exception",
      "End": true
    },
    "ReservedTypeFallback": {
      "Type": "Pass",
      "Result": "This is a fallback from a reserved error code",
      "End": true
    },
    "CatchAllFallback": {
      "Type": "Pass",
      "Result": "This is a fallback from a reserved error code",
      "End": true
    }
  }
}
```

---

### AWS AppConfig

- 애플리케이션에 동적 구성을 설정하고, 검증하고 배포할 수 있다.
- 코드 배포와 관계없이 애플리케이션에 동적으로 구성 변경 사항을 배포한다.
  - 애플리케이션을 다시 시작할 필요가 없다.
- 기능 플래그, 애플리케이션 조정, 허용/차단 목록 등을 제공한다.
- EC2 인스턴스, 람다, ECS, EKS의 앱과 함께 사용된다.
- 구성 변경 사항을 점진적으로 배포하고 문제가 발생하면 롤백한다.
- 배포하기 전에 다음을 사용하여 구성 변경 사항을 검증한다.
  - JSON 스키마의 문법을 검사할 수 있다.
  - 람다 함수의 코드를 실행하여 검증을 수행한다.

![26-appconfig.png](images%2F26-appconfig.png)

- AppConfig의 구성 소스에는 Parameter Store나 SSM Document, S3 버킷 등이 있을 수 있다.
- EC2 인스턴스나 실행되는 애플리케이션이 정기적으로 구성 변경 사항을 풀링한다.
- 예를 들어, 구성을 변경하면 자동으로 CloudWatch에서 문제가 있는지 모니터링하고 문제가 발생하면 경고를 트리거한다.
  - 경고를 통해 롤백이 트리거된다.

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)