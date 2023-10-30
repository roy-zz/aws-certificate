# AWS CloudFormation

이번 장에서는 **SysOps Administrator**를 준비하며 **CloudFormation**에 대해서 알아보도록 한다.

---

### Infrastructure as a Code (IaaS)

- 현재 우리는 대부분의 작업을 수동 작업으로 하고 있다.
- 이 모든 수동 작업은 재현하기가 매우 어렵다.
  - 다른 지역에서 실행하는 경우
  - 다른 AWS 계정에서 실행하는 경우
  - 모든 항목이 삭제된 동일한 지역에서 실행하는 경우
- 모든 인프라가 코드로 이루어져 관리될 수 있다면 많은 부분을 자동화할 수 있다.
- 해당 코드가 배포되어 인프라를 생성/업데이트/삭제할 수 있다.

---

### CloudFormation

- "CloudFormation"은 모든 리소스에 대해 AWS 인프라의 개요를 설명하는 선언적 방법이며, 대부분의 리소스를 지원한다.
- 예를 들어, "CloudFormation" 템플릿에서 아래와 같이 항목을 요청할 수 있다.
  - 보안 그룹
  - 보안 그룹을 사용하고 있는 EC2 인스턴스
  - 두 개의 "Elastic IP"와 EC2 인스턴스
  - S3 버킷
  - EC2 인스턴스 앞에서 작동하는 ELB
- 그런 다음 "CLoudFormation"은 사용자가 지정한 정확한 구성을 사용하여 올바른 순서로 해당 항목을 생성한다.

#### Benefit

- Infrastructure as a code
  - 수동으로 생성되는 리소스가 없으므로 제어가 뛰어나다
  - 예를 들어, "git"을 사용하여 코드의 버전을 제어할 수 있다.
  - 인프라 변경 사항은 코드를 통해 검토된다.
- Cost
  - 스택 내의 각 리소스에는 식별자가 지정되어 있으므로 스택 비용이 많이 발생하는 부분을 쉽게 확인할 수 있다.
  - "CloudFormation" 템플릿을 사용하여 리소스 비용을 추정할 수 있다.
  - 오후 5시에 템플릿을 자동으로 삭제하고 오전 8시에 다시 생성하는 방식으로 업무 시간에만 인프라가 구성되도록 하여 비용을 절감할 수 있다.
- Productivity
  - 클라우드의 인프라를 즉시 파괴하고 재구축할 수 있는 능력을 제공한다.
  - 템플릿에 대한 다이어그램을 자동으로 생성하여 PPT와 같은 문서에 첨부할 수 있다.
  - 선언적 프로그래밍을 통해 순서 및 조정을 파악할 필요가 없다.
- 관심사 분리: 많은 앱과 레이어에 대해 많은 스택을 생성한다.
  - VPC 스택
  - Network 스택
  - App 스택
- 재발명할 필요가 없다.
  - 웹에 존재하는 기존의 템플릿을 활용할 수 있다.
  - 문서를 활용할 수 있다.

#### 작동 원리

- 템플릿은 S3에 업로드한 다음 "CloudFormation"에서 참조해야 한다.
- 템플릿을 업데이트하기 위해 이전 템플릿을 편집할 수 없다. 
  - "CloudFormation"이 이전 버전에서 새로운 버전으로 업데이트할 때 어떤 작업이 필요한지 알기 위해서 이전 버전은 수정할 수 없다.
- 스택의 이름은 고유하며 아주 길어질 수 있다.
- 스택이 삭제되는 경우 "CloudFormation"에 의해 기존의 모든 단일 아티팩트가 삭제된다.

#### 배포

- 수동 방식
  - "CloudFormation Designer"에서 템플릿을 편집한다.
  - 콘솔을 사용하여 매개변수를 입력한다.
- 자동 방식
  - YAML 파일에서 템플릿을 편집한다.
  - AWS CLI를 사용하여 템플릿을 배포한다.
  - 흐름을 완전히 자동화하려는 경우 권장되는 방법이다.

#### Building Block

- 템플릿 구성요소
  - Resources: 템플릿에 선언된 AWS 리소스(필수)
  - Parameters: 템플릿의 동적 입력
  - Mappings: 템플릿의 정적 변수
  - Outputs: 생성된 내용에 대한 참조
  - Conditionals: 리소스 생성을 수행하기 위한 조건 목록
  - 메타데이터
- Template helpers
  - References
  - Functions

---

### Introductory (입문)

- 간단한 EC2 인스턴스를 생성하고 "Elastic IP"를 추가하는 예시를 살펴본다.
- 여기에 두 개의 보안 그룹을 추가한다.
- 코드 구문을 자세히 살펴보지 않고 파일의 구조 또한 추후에 살펴보도록 한다.

- 아래는 단순히 EC2 인스턴스를 생성하는 예시이다.

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-a4c7edb2
      InstanceType: t2.micro
```

- 아래는 EC2 인스턴스, Elastic IP, 보안 그룹을 생성하는 예시이다.

```yaml
Parameters:
  SecurityGroupDescription:
    Description: Security Group Description
    Type: String

Resources:
  MyInstance:
  Type: AWS::EC2::Instance
  Properties:
    AvailabilityZone: us-east-1a
    ImageId: ami-a4c7edb2
    InstanceType: t2.micro
    SecurityGroups: 
      - !Ref SSHSecurityGroup
      - !Ref ServerSecurityGroup
  
  MyEIP:
    Type: AWS::EC2::EIP
    Properties:
      InstanceId: !Ref MyInstance
  
  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - CidrIp: 0.0.0.0/0
          FromPort: 22
          IpProtocol: tcp
          ToPort: 22
  
  ServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: !Ref SecurityGroupDescription
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 192.168.1.1/32
```

---

### YAML

- "YAML"과 "JSON"은 "CloudFormation"에서 사용할 수 있는 언어다.
- "YAML"은 여러 방면에서 훌륭하지만, "JSON"은 적합하지 않다.
- 시험에서는 대부분 "YAML" 기반의 문제가 출시된다.
- "YAML"은 아래와 같은 문법을 가지고 있다.
  - Key-Value 형식으로 이루어져 있다.
  - 중첩된 객체를 가질 수 있다.
  - 배열을 지원한다. (`-`)
  - 멀티라인 문자열을 지원한다. (`|`)
  - 주석을 추가할 수 있다. (`#`)

![1-yaml-crash-course.png](images%2F1-yaml-crash-course.png)

---

### Resource

- "Resources"는 "CloudFormation" 템플릿의 핵심요소다.(필수)
- 생성 및 구성될 다양한 AWS 구성 요소를 나타낸다.
- 리소스가 선언되고 서로 참조할 수 있다.
- AWS는 사용자를 위해 리소스 생성, 업데이트 및 삭제를 파악한다.
- 224가지 이상의 리소스 유형이 있다.
- 리소스 유형 식별자의 형식은 다음과 같다.
  - `AWS::aws-product-name::data-type-name`
  
- 모든 리소스에 대한 정보는 [여기](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)에서 찾을 수 있다.
- EC2 인스턴스에 대한 정보는 [여기](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html)에서 찾을 수 있다.
- 문서에는 JSON과 YAML 문법을 통해 리소스를 선언할 수 있는 방법을 설명하고 있다.
- 문서를 확인해 보면 리소스가 업데이트될 때, 중단이 발생하는지 또는 교체가 필요한지 확인할 수 있다.

- 동적인 양의 리소스를 생성할 수 없다.
  - "CloudFormation" 템플릿의 모든 내용을 선언해야 한다.
  - 코드 생성을 수행할 수 없다.
- 모든 AWS 서비스를 지원하지 않는다.
  - 대부분의 서비스를 지원하지만 모든 서비스를 지원하지 않는다.
  - 지원하지 않는 서비스에 대해서는 "Lambda Customer Resources"를 사용해서 해결해야 한다.

---

### Parameters

- "Parameters"는 "CloudFormation" 템플릿에 입력을 제공하는 방법이다.
- 아래의 항목을 기억해야 한다.
  - 회사 전체에서 템플릿을 재사용하고 싶은 경우에 사용할 수 있다.
  - 일부 입력 사항을 미리 결정할 수 없다.
- "Parameter"는 매우 강력하고 제어되며 유형 덕분에 템플릿에서 오류가 발생하는 것을 방지할 수 있다.
- "CloudFormation" 리소스 구성이 향후 변경될 가능성이 있다면 "Parameter"로 만드는 것이 좋다.
- 콘텐츠를 변경하기 위해 템플릿을 다시 업로드할 필요가 없다.

![2-parameters.png](images%2F2-parameters.png)

- "Parameter"에서 지원하는 여러 설정이 있다.
- Type
  - String
  - Number
  - CommaDelimitedList
  - List<Type>
  - AWS Parameter
- Description
- Constraints
- ConstraintDescription (String)
- Min/MaxLength
- Min/MaxValue
- Defaults
- AllowedValues (array)
- AllowedPattern (regexp)
- NoEcho (boolean)

#### 참조

- `Fn::Ref` 함수를 활용하여 매개변수를 참조할 수 있다.
- 매개변수는 템플릿의 어느 위치에서나 사용할 수 있다.
- YAML에서 약자는 `!Ref`다.
- 함수는 템플릿 내의 다른 요소를 참조할 수도 있다.

![3-reference-parameter.png](images%2F3-reference-parameter.png)

#### Pseudo Parameters

- AWS는 모든 "CloudFormation" 템플릿에서 의사 매개변수를 제공한다.
- 언제든지 사용할 수 있으며 기본적으로 활성화되어 있다.
- 예를 들어, 활성화되어 있는 계정의 ID를 알고 싶다면 `AWS::AccountID`와 같은 방법으로 참조할 수 있다.

![4-pseudo-parameters.png](images%2F4-pseudo-parameters.png)

---

### Mappings

- "Mapping"은 "CloudFormation" 템플릿 내의 고정 값이다.
- 다양한 환경(dev vs prod), 지역(AWS Region), AMI 유형 등을 구별하는데 편리하게 사용된다.
- 모든 값은 템플릿 내에 하드코딩된다.

![5-mappings.png](images%2F5-mappings.png)

#### Mappings vs Parameters

- "Mapping"은 취할 수 있는 모든 값을 미리 알고 있고 그 값이 아래와 같은 변수에서 추론될 수 있을 때 유용하다.
  - Region
  - Availability Zone
  - AWS Account
  - Environment(dev, prod)
- 템플릿을 더욱 안전하게 제어할 수 있다.
- 값이 실제로 사용자별로 특정하거나, 예측할 수 없는 경우 "Parameter"를 사용한다.

- `Fn::FindInMap`을 사용하여 특정 키에서 명명된 값을 반환한다.
- `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`

![6-accessing-mapping-value.png](images%2F6-accessing-mapping-value.png)

---

### Outputs

- "Output" 섹션은 다른 스택으로 가져올 수 있는 선택적 출력 값을 선언한다.(먼저 Export하는 경우)
- AWS 콘솔이나 AWS CLI를 사용하여 출력을 볼 수도 있다.
- 예를 들어, 네트워크 "CloudFormation"을 정의하고 VPC ID 및 Subnet ID와 같은 변수를 출력하는 경우 매우 유용하다.
- 전문가들이 VPC와 서브넷을 담당할 수 있도록 크로스 스택 협업을 가능하게 하고, 앱 개발자로서 이런 값들을 참조할 수 있다.
- "Output"이 다른 "CloudFormation" 스택에서 참조되는 경우 "CloudFormation" 스택을 삭제할 수 없다.

- 하나의 템플릿의 일부로 SSH 보안 그룹을 생성한다.
- 해당 보안 그룹을 참조하는 출력을 생성한다.
- 아래의 예시에서 "SSHSecurityGroupID"는 "SSHSecurityGroup"이라는 이름으로 내보내진다.

![7-outputs-example.png](images%2F7-outputs-example.png)

- 위의 보안 그룹을 활용하는 두 번째 템플릿을 생성한다.
- 값을 참조하기 위해 `Fn::ImportValue` 함수를 사용한다.
- 모든 참조가 삭제될 때까지 기본 스택을 삭제할 수 없다.

![8-cross-stack-reference.png](images%2F8-cross-stack-reference.png)

---

### Conditions

- "Condition"은 조건에 따라 리소스 또는 출력 생성을 제어하는 데 사용된다.
- "Condition"은 원하는 대로 지정할 수 있지만 일반적인 아래와 같은 항목들이 사용된다.
  - Environment (dev/test/prod)
  - AWS Region
  - Any parameter value
- 각 조건은 다른 "Condition", "Parameter" 또는 "Mapping"을 참조할 수 있다.

![9-condition-define.png](images%2F9-condition-define.png)

- 위의 예시를 살펴보면 "EnvType"이 `prod`인 경우에만 리소스를 생성한다.
- 논리적 ID(예시에서 `CreateProdResources`)는 사용자가 선택하는 것이다.
- 내장 기능(논리적)은 아래 중 하나일 수 있다.
  - `Fn::And`
  - `Fn::Equals`
  - `Fn::If`
  - `Fn:Not`
  - `Fn::Or`
- "Condition"은 "Resources", "Outputs" 등 여러 곳에 적용될 수 있다.

![10-using-a-condition.png](images%2F10-using-a-condition.png)

---

### Intrisic(내장) 함수

- "CloudFormation" 템플릿에는 내장된 여러가지 함수가 있다.
  - `Ref`
  - `Fn::GetAtt`
  - `Fn::FindInMap`
  - `Fn::ImportValue`
  - `Fn::Join`
  - `Fn::Sub`
  - `Condition Functions` (`Fn::If`, `Fn::Not`, `Fn::Equals` 등)

#### Fn::Ref

- `Fn::Ref` 함수를 활용하여 "Parameter"를 참조할 수 있다.
  - "Parameters": 매개 변수의 값을 반환한다.
  - "Resources": 기본 리소스의 물리적 ID를 반환한다. (예: EC2 ID)
- YAML에서의 약어는 `!Ref`다.

![11-fn-ref.png](images%2F11-fn-ref.png)

#### Fn::GetAtt

- "Attributes"은 사용자가 생성하는 모든 리소스에 연결된다.
- 리소스가 어떤 속성을 가지고 있는지 공식 문서를 사용해서 확인하는 것이 가장 좋다.
- 아래의 예시는 EC2 인스턴스의 가용 지역 속성을 참조하는 방법이다.

![12-fn-getatt.png](images%2F12-fn-getatt.png)

#### Fn::FindInMap

- `Fn::FindInMap`을 사용하여 특정 키에서 명명된 값을 반환한다.
- `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`와 같은 방식으로 사용한다.

![13-fn-findinmap.png](images%2F13-fn-findinmap.png)

#### Fn::ImportValue

- 다른 템플릿으로 내보낸 값(Exported)을 가져올 때 사용한다.
- `Fn::ImportValue` 함수를 사용한다.

![14-fn-import-value.png](images%2F14-fn-import-value.png)

#### Fn::Join

- 특정 구분 기호로 값을 결합할 수 있다.
  - `!Join [ delimeter, [ comma-delimited list of values ] ]`
- 아래의 구문은 "a:b:c"라는 문자를 생성한다.
  - `!Join [ ":", [ a, b, c ] ]`

#### Fn::Sub

- `Fn::Sub` 또는 약자로 `!Sub`는 텍스트의 변수를 대체하는 데 사용된다.
  이는 템플릿을 완전히 사용자 정의할 수 있는 매우 편리한 기능이다.
- 예를 들어, `Fn::Sub`를 "Reference" 또는 "AWS Pseudo Variable"와 결합할 수 있다.
- 문자열은 `${VariableName}`을 포함해야 하며 이를 대체한다.

```yaml
!Sub
  - String
  - { Var1Name: Var1Value, Var2Name: Var2Value }
```

```yaml
!Sub String
```

#### Conditions

- 논리적 ID(예시에서는 `CreateProdResources`)는 사용자가 선택하는 것이다.
- 내장 기능(논리적)은 아래의 항목 중 하나일 수 있다.
  - `Fn::And`, `!And`
  - `Fn::Equals`, `!Equals`
  - `Fn::If`, `!If`
  - `Fn::Not`, `!Not`
  - `Fn::Or`, `!Or`

---

### EC2 User Data

- 콘솔을 통해 EC2 인스턴스 시작 시 사용자 데이터(User Data)를 얻을 수 있다.
- "CloudFormation"에 포함될 수도 있다.
- **전달해야 할 중요한 것은 `Fn::Base64` 함수를 통해 스크립트 전체를 전달하는 것이다.
- 사용자 데이터 스크립트 로그는 `/var/log/cloud-init-output.log`에 있다.

```yaml
Parameters:
  SSHKey:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-009d6802948d06e52
      InstanceType: t2.micro
      KeyName: !Ref SSHKey
      SecurityGroups:
        - !Ref SSHSecurityGroup

      UserData: 
        Fn::Base64: |
          #!/bin/bash -xe
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          echo "Hello World from user data" > /var/www/html/index.html

  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: SSH and HTTP
      SecurityGroupIngress:
      - CidrIp: 0.0.0.0/0
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22
      - CidrIp: 0.0.0.0/0
        FromPort: 80
        IpProtocol: tcp
        ToPort: 80
```

---

### cfn-init

- "User Data"는 단순하기 때문에 복잡한 프로세스를 추가하기에는 무리가 있다.
- 복잡하고 구조화된 스크립트를 원할 때 `cfn-init`을 쓴다.
- "AWS::CloudFormation::Init"는 반드시 리소스의 메타데이터에 포함되어야 한다.
- "cfn-init" 스크립트를 사용하면 복잡한 EC2 구성을 읽을 수 있게 만드는 데 도움이 된다.
- EC2 인스턴스는 "CloudFormation" 서비스에 쿼리하여 초기 데이터를 가져온다.
- 로그는 `/var/log/cfn-ini.log`로 이동한다.

![16-cfn-init.png](images%2F16-cfn-init.png)

- 아래의 YAML 파일을 확인해보면 User Data를 사용할 때와는 다른 부분이 있다.
- 동일하게 Base64를 사용하지만 "!Sub" 함수를 통과하고 있다.
- 먼저 선행되어야 하는 작업은 `aws-cfn-bootstrap` 스크립트를 업데이트하여 최신 버전을 다운로드 하는 것이다.
- `/opt/os/bin/cfn-init` 경로의 `cfn-init` 스크립트를 실행한다.
- `StackId` 변수를 전달한다.
- "Metadata"에 "AWS::CloudFormation::Init"를 사용하는 경우 "User Data"를 사용하는 것보다 가독성이 높다.

```yaml
Parameters:
  SSHKey:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-009d6802948d06e52
      InstanceType: t2.micro
      KeyName: !Ref SSHKey
      SecurityGroups:
        - !Ref SSHSecurityGroup
      # we install our web server with user data
      UserData: 
        Fn::Base64:
          !Sub |
            #!/bin/bash -xe
            # Get the latest CloudFormation package
            yum update -y aws-cfn-bootstrap
            # Start cfn-init
            /opt/aws/bin/cfn-init -s ${AWS::StackId} -r MyInstance --region ${AWS::Region} || error_exit 'Failed to run cfn-init'
    Metadata:
      Comment: Install a simple Apache HTTP page
      AWS::CloudFormation::Init:
        config:
          packages:
            yum:
              httpd: []
          files:
            "/var/www/html/index.html":
              content: |
                <h1>Hello World from EC2 instance!</h1>
                <p>This was created using cfn-init</p>
              mode: '000644'
          commands:
            hello:
              command: "echo 'hello world'"
          services:
            sysvinit:
              httpd:
                enabled: 'true'
                ensureRunning: 'true'

  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: SSH and HTTP
      SecurityGroupIngress:
      - CidrIp: 0.0.0.0/0
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22
      - CidrIp: 0.0.0.0/0
        FromPort: 80
        IpProtocol: tcp
        ToPort: 80
```

---

### cfj-signal & wait conditions

- 우리는 "cfn-init" 이후에 EC2 인스턴스가 올바르게 구성되었음을 "CloudFormation"에 알리는 방법을 아직 모른다.
- "CloudFormation"에게 알리기 위해 **cfn-signal**을 사용할 수 있다.
  - "cfn-init" 직후에 "cfn-signal"을 실행한다.
  - "CloudFormation" 서비스에 계속 진행하지 않으면 실패하도록 지시한다.
- "WaitCondition"을 정의해야 한다.
  - "cfn-signal"에서 신호를 받을 때까지 템플릿을 차단한다.
  - "CreationPolicy"를 첨부한다. (EC2, ASG에서도 작동)

![17-cfn-signal-wait-conditions.png](images%2F17-cfn-signal-wait-conditions.png)

- 아래의 YAML 파일을 보면 `/opt/aws/bin/cfn-init` 스크립트 실행 이후에 `/opt/aws/bin/cfn-signal` 스크립트를 실행하는 것을 확인할 수 있다.
- "SampleWaitCondition"이 추가된 것을 확인할 수 있다.
  - 하나의 인스턴스가 실행되기를 기다린다.
  - 최대 2분까지 시그널을 기다린다.

```yaml
Parameters:
  SSHKey:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-009d6802948d06e52
      InstanceType: t2.micro
      KeyName: !Ref SSHKey
      SecurityGroups:
        - !Ref SSHSecurityGroup
      UserData: 
        Fn::Base64:
          !Sub |
            #!/bin/bash -xe
            # Get the latest CloudFormation package
            yum update -y aws-cfn-bootstrap
            # Start cfn-init
            /opt/aws/bin/cfn-init -s ${AWS::StackId} -r MyInstance --region ${AWS::Region}
            # Start cfn-signal to the wait condition
            /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource SampleWaitCondition --region ${AWS::Region}
    Metadata:
      Comment: Install a simple Apache HTTP page
      AWS::CloudFormation::Init:
        config:
          packages:
            yum:
              httpd: []
          files:
            "/var/www/html/index.html":
              content: |
                <h1>Hello World from EC2 instance!</h1>
                <p>This was created using cfn-init</p>
              mode: '000644'
          commands:
            hello:
              command: "echo 'hello world'"
          services:
            sysvinit:
              httpd:
                enabled: 'true'
                ensureRunning: 'true'

  SampleWaitCondition:
    CreationPolicy:
      ResourceSignal:
        Timeout: PT2M
        Count: 1
    Type: AWS::CloudFormation::WaitCondition

  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: SSH and HTTP
      SecurityGroupIngress:
      - CidrIp: 0.0.0.0/0
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22
      - CidrIp: 0.0.0.0/0
        FromPort: 80
        IpProtocol: tcp
        ToPort: 80
```

---

### cfn-signal Troubleshooting

- **"Wait Condition"이 EC2 인스턴스로부터 필요한 수의 신호를 수신하지 못하는 경우 우리는 원인을 파악할 수 있어야 한다.**
- 사용 중인 AMI에 "CloudFormation Helper Script"가 설치되어 있는지 확인해야 한다.
  AMI에 Helper Script가 포함되어 있지 않은 경우 해당 스크립트를 인스턴스에서 다운로드할 수도 있다.
- `cfn-init` & `cfn-signal` 명령이 인스턴스에서 성공적으로 실행되었는지 확인한다.
  `/var/log/cloud-init.log` 또는 `/var/log/cfn-init.log`와 같은 로그를 보면 인스턴스 시작을 디버그하는 데 도움이 된다.
- 인스턴스에 로그인하여 로그를 검색할 수 잇지만 실패 시 롤백을 비활성화해야 한다.
  그렇지 않으면 스택 생성에 실패한 후 "CloudFormation"이 인스턴스를 삭제한다.
- 인스턴스가 인터넷에 연결되어 있는지 확인한다. 인스턴스가 VPC에 있는 경우, 인스턴스는 프라이빗 서브넷에 있으면 NAT 장치를 통해서 인터넷과 통신한다.
  퍼블릭 서브넷에 있는 경우 인터넷 게이트웨이를 통해 인터넷에 연결할 수 있어야 한다.
  만약 그렇지 않다면 "CloudFormation"과 통신할 수 없기 때문에 성공 신호를 보낼 수 없다.

```yaml
Parameters:
  SSHKey:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-009d6802948d06e52
      InstanceType: t2.micro
      KeyName: !Ref SSHKey
      SecurityGroups:
        - !Ref SSHSecurityGroup
      UserData: 
        Fn::Base64:
          !Sub |
            #!/bin/bash -xe
            # Get the latest CloudFormation package
            yum update -y aws-cfn-bootstrap
            # Start cfn-init
            /opt/aws/bin/cfn-init -s ${AWS::StackId} -r MyInstance --region ${AWS::Region}
            # Start cfn-signal to the wait condition
            /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource SampleWaitCondition --region ${AWS::Region}
    Metadata:
      Comment: Install a simple Apache HTTP page
      AWS::CloudFormation::Init:
        config:
          packages:
            yum:
              httpd: []
          files:
            "/var/www/html/index.html":
              content: |
                <h1>Hello World from EC2 instance!</h1>
                <p>This was created using cfn-init</p>
              mode: '000644'
          commands:
            hello:
              command: "echo 'boom' && exit 1"
          services:
            sysvinit:
              httpd:
                enabled: 'true'
                ensureRunning: 'true'

  SampleWaitCondition:
    CreationPolicy:
      ResourceSignal:
        Timeout: PT1M
        Count: 1
    Type: AWS::CloudFormation::WaitCondition

  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: SSH and HTTP
      SecurityGroupIngress:
      - CidrIp: 0.0.0.0/0
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22
      - CidrIp: 0.0.0.0/0
        FromPort: 80
        IpProtocol: tcp
        ToPort: 80
```

- 위의 예시를 살펴보면 `exit 1` 커맨드가 추가되어 있는 것을 확인할 수 있다.
- 정상적으로 "AWS::CloudFormation::Init"이 실행된다면 스크립트가 실행되면 의도된 스크립트 오류(`exit 1`)에 의해 실행이 실패하게 될 것이다.

---

### CloudFormation Rollback

- Stack Creation Fails:
  - 기본값으로 모든 항목이 롤백되어 삭제된다. 로그를 확인할 수 있다.
  - 롤백을 비활성화하고 발생한 문제를 해결할 수 있다.
- Stack Update Fails:
  - 스택은 이전에 알려진 작업 상태로 자동으로 롤백된다.
  - 발생한 일과 오류 메시지를 로그에서 볼 수 있는 기능을 제공한다.

#### Stack 생성 실패

- "CloudFormation" 스택 생성에 실패하면 `ROLLBACK_COMPLETE` 상태가 된다.
- 이는 아래의 항목들을 의미한다.
  - "CloudFormation"이 일부 리소스를 생성하려고 시도했다.
  - 하나의 리소스 생성에 실패했다.
  - "CloudFormation"이 리소스를 롤백했다. (`ROLLBACK`, `DO_NOTHING`)
  - 스택 생성 실패 `ROLLBACK_COMPLETE`상태다.
- 오류를 해결하는 방법은 한 가지뿐이다. 실패한 스택을 생성하고 새로운 스택을 생성한다.
- 실패한 스택 생성에서는 업데이트, 검증 또는 변경 설정을 수행할 수 없다.

#### Stack 실패 옵션

- **Roll back all stack resource**
  - 스택이 실패한 경우 스택의 모든 자원과 변경 사항을 롤백하여 스택을 이전 성공한 상태로 복원한다.
  - 스택이 이전 상태로 복원되므로 안전한 롤백이 가능하다.
  - 프로덕션 환경에서 권장되는 옵션이다.
- **Preserve successfully provisioned resources**
  - 스택이 실패한 경우에도 성공적으로 프로비젼된 리소스는 유지한 채로 실패한 자원만 롤백된다.
  - 실패한 리소스만 롤백되고 나머지 리소스는 그대로 유지된다.
  - 일부 리소스를 재사용하거나 중요한 데이터를 보존해야 하는 경우 유용하게 사용할 수 있다.

---

### Nested stacks

- 중첩 스택은 다른 스택의 일부인 스택이다.
- 반복되는 패턴/공통 구성 요소를 별도의 스택으로 분리하고 다른 스택에서 호출할 수 있다.
- 예를 들어, 재사용되는 로드 밸런서 설정, 재사용되는 보안 그룹등에 사용된다.
- **중첩 스택은 모범 사례로 간주된다.**
- 중첩 스택을 업데이트하려면 항상 상위(루트 스택)을 업데이트한다.

```yaml
Parameters:
  SSHKey:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance
  
Resources:
  myStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/cloudformation-templates-us-east-1/LAMP_Single_Instance.template
      Parameters:
        KeyName: !Ref SSHKey
        DBName: "mydb"
        DBUser: "user"
        DBPassword: "pass"
        DBRootPassword: "passroot"
        InstanceType: t2.micro
        SSHLocation: "0.0.0.0/0"

Outputs:
  StackRef:
    Value: !Ref myStack
  OutputFromNestedStack:
    Value: !GetAtt myStack.Outputs.WebsiteURL
```

- 스크립트를 확인해보면 "TemplateURL"의 값으로 `https://s3.amazonaws.com/cloudformation-templates-us-east-1/LAMP_Single_Instance.template` 주소가 입력되어 있다.
- 주소를 입력해보면 JSON으로 구성된 파일을 확인할 수 있고 여러 입력값을 요구하고 있다.
- 이렇게 우리는 미리 구성된 스크립트를 가지고 원하는 파라미터만 입력하여 간편하게 서비스를 구축할 수 있다.

---

### ChangeSets

- 스택을 업데이트할 때, 더 큰 신뢰를 얻으려면 변경 사항이 발생하기 전에 무엇을 변경해야 하는지 알아야 한다.
- "ChangeSets"는 업데이트 성공 여부를 알려주지 않는다.
- 어떤 리소스가 바뀌고, 어떤 리소스는 삭제되며, 어떤 리소스가 생성되는지 가르쳐준다.

![18-change-sets.png](images%2F18-change-sets.png)

---

### CloudFormation Drift

- "CloudFormation"을 사용하면 인프라를 생성할 수 있다.
- 하지만 수동으로 구성을 변경되는 경우 사용자를 보호하지 않는다.
- **"CloudFormation Drift"를 사용하여 자원이 수동으로 변경되었는지 확인할 수 있다.**
- 일부의 리소스는 지원하지 않으며 지원하지 않는 목록은 [여기](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift-resource-list.html)에서 확인할 수 있다.

---

### Retaining Data on Deletes

- 모든 리소스에 "DeletionPolicy"를 배치하여 "CloudFormation" 템플릿이 삭제될 때 발생하는 상황을 제어할 수 있다.
- **DeletionPolicy=Retain**
  - "CloudFormation" 삭제 시 보존/백업할 리소스를 지정한다.
  - 리소스를 유지하려면 "Retain"을 지정한다. (모든 리소스/중첩 스택에 작동)
- **DeletionPolicy=Snapshot**
  - EBS 볼륨, ElastiCache Cluster, ElastiCache ReplicationGroup, RDS DBInstance, RDS DBCluster, Redshift Cluster에만 해당된다.
  - 인스턴스가 삭제되기 직전에 스냅샷을 생성하도록 하여 데이터를 보관할 수 있다.
- **DeletionPolicy=Delete (기본 동작)**
  - `AWS::RDS::DBCluster` 리소스의 경우 기본 정책은 스냅샷이다.
  - S3 버킷을 삭제하려면 먼저 버킷의 모든 콘텐츠를 비워야 한다.

#### Termination Protection

- "CloudFormation" 템플릿이 실수로 삭제되는 것을 방지하기 위해 "TerminationProtection"을 사용할 수 있다.
- EC2 인스턴스의 삭제 방지 기능와 유사하게 작동한다.

---

### CloudFormation StackSets

- 단일 작업으로 여러 계정 및 지역에 걸쳐 스택 생성, 업데이트 또는 삭제한다.
- "StackSet"을 생성하기 위한 관리자 계정
- "StackSets"에서 스택 인스턴스를 생성, 업데이트, 삭제하는 신뢰할 수 있는 계정이 필요하다.
- "Stack Sets"을 업데이트하면 연결된 모든 스택 인스턴스가 모든 계정 및 리전에서 업데이트된다.
- 대상에 대한 최대 동시 작업 설정 기능(`#` 또는 `%`)을 지원한다.
- 실패 허용 범위 설정 기능(`#` 또는 `%`)을 지원한다.

---

### Continue Rolling Back an Update

- "CloudFormation"이 업데이트 중에 모든 변경 사항을 롤백할 수 없으면 스택이 `UPDATE_ROLLBACK_FAILED` 상태로 전환된다.
- 리소스가 원래 상태로 돌아갈 수 없어 롤백이 실패한다.
- 예를 들어, "CloudFormation" 외부에서 삭제된 이전 데이터베이스 인스턴스로 롤백하는 경우가 있다.
- 해결 방법은 아래와 같다.
  - "CloudFormation" 외부에서 수동으로 오류를 수정한 후 계속해서 스택을 롤백한다.
  - 성공적으로 롤백할 수 없는 리소스를 건너뛴다. 이러한 경우 "CloudFormation"은 실패한 리소스를 `UPDATE_COMPLETE`로 표시한다.
- 이 상태에서는 스택을 업데이트할 수 없다.
- 중첩 스택의 경우 상위 스택을 롤백하면 모든 하위 스택도 롤백하려고 시도한다.

![19-rolling-back-update.png](images%2F19-rolling-back-update.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [CloudFormation 리소스 목록](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [CloudFormation EC2 인스턴스](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html)
- [CloudFormation 보안 그룹](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html)
- [CloudFormation Elastic IP](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-eip.html)
- [LAMP_Single_Instance](https://s3.amazonaws.com/cloudformation-templates-us-east-1/LAMP_Single_Instance.template)