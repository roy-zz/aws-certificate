# CloudFormation

- 이번 장에서는 **DevOps Engineer Professional (DOP)** 을 준비하며 "IaC 서비스인 CloudFormation"에 대해서 알아보도록 한다.

---

### AWS CloudFormation

#### Infrastructure as Code

- 많은 사용자들이 많은 작업을 수동으로 하고 있다.
- 수동으로 이루어진 작업은 다른 리전이나 다른 AWS 계정에서 재현하기 어렵다.
  - 또한 동일한 리전이더라도 인프라가 삭제된 경우에 재현이 어렵다.
- 이번 장에서는 코드를 통해서 인프라를 생성/수정/삭제하는 방법에 대해서 알아본다.

#### AWS CloudFormation

- CloudFormation은 모든 리소스(대부분 지원)에 대해 AWS Infrastructure의 개요를 설명하는 선언적인 방법이다.
- 예를 들어, CloudFormation 템플릿 내에서 다음과 같이 수행할 수 있다.
  - 보안 그룹을 원한다.
  - 이 보안 그룹을 사용하는 EC2 인스턴스 두 대를 원한다.
  - 이 EC2 인스턴스를 위한 Elastic IP 두 개를 원한다.
  - S3 버킷을 원한다.
  - 인스턴스 앞에 ELB를 원한다.
- 그런 다음 CloudFormation은 사용자가 지정한 정확한 구성을 사용하여 올바른 순서로 생성한다.

#### AWS CloudFormation 이점

- Infrastructure as code
  - 수동으로 생성되는 리소스가 없으므로 제어에 탁월하다.
  - 코드는 git을 사용하여 버전 관리가 가능하다.
  - 코드를 통해 인프라의 변경사항을 검토한다.
- Cost
  - 스택 내의 각 리소스에는 식별자가 스테이징되어 있으므로 스택 비용이 얼마인지 쉽게 확인할 수 있다.
  - CloudFormation 템플릿을 사용하여 리소스 비용을 추정할 수 있다.
  - Savings Strategy: 개발에서 템플릿 삭제를 오후 5시에 자동화하고 오전 8시에 재생성하여 안전하게 실행할 수 있다.
- Productivity
  - 클라우드의 인프라를 즉시 파괴하고 재구축할 수 있는 기능을 제공한다.
  - 템플릿에 대한 다이어그램을 자동으로 생성한다.
  - 선언적 프로그래밍을 지원한다. (주문 및 조정을 파악할 필요가 없음)
- Separation of concern
  - 많은 앱과 많은 레이어에 대해 원하는 대로 스택을 가질 수 있다.
  - 모든 네트워크와 서브넷을 생성하는 VPC CloudFormation 스택을 갖는 것이 보편적이다.
  - 앱 스택에 있어서 사용자가 배포할 모든 앱에 대해 앱 CloudFormation tmxordl dlTek.
- 이미 있는 것들을 최대한 재사용하는 전략을 사용한다.
  - 웹에는 수많은 CloudFormation 템플릿이 있고 그것을 활용할 수 있다.

#### 작동 방식

- 템플릿은 S3에서 업로드한 후 CloudFormation에서 참조해야 한다.
- 템플릿 업데이트를 위해서 이전 템플릿을 편집할 수 없다.
  - 새로운 버전의 탬플릿을 AWS에 다시 업로드해야 한다.
- 스택은 이름으로 식별된다.
- 스택을 삭제하면 CloudFormation에서 생성된 모든 아티팩트가 삭제된다.

#### 템플릿 배포

- Manual way
  - CloudFormation Designer에서 템플릿을 편집한다.
  - 콘솔을 사용하여 파라미터를 입력할 수 있다.
- Automated way
  - YAML 파일에서 템플릿을 편집한다.
  - AWS CLI를 사용하여 템플릿을 배포한다.
  - 플로우를 완전히 자동화하려는 경우 권장되는 방법이다.

#### Building Blocks

- Templates Components (각각 하나의 과정 섹션)
  - Resources: 템플릿에 선언된 AWS 리소스 (필수)
  - Parameters: 템플릿에 대한 동적 입력
  - Mappings: 템플릿의 정적 변수
  - Outputs: 생성된 것에 대한 참조
  - Conditionals: 리소스 생성을 수행할 조건 목록
  - 메타데이터
- Templates Helpers
  - References: 템플릿 안의 내용들을 연결할 수 있다.
  - Functions: 템플릿 안의 데이터를 변환할 수 있다.

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

### CloudFormation Drift

- "CloudFormation"을 사용하면 인프라를 생성할 수 있다.
- 하지만 수동으로 구성을 변경되는 경우 사용자를 보호하지 않는다.
- **"CloudFormation Drift"를 사용하여 자원이 수동으로 변경되었는지 확인할 수 있다.**
- 일부의 리소스는 지원하지 않으며 지원하지 않는 목록은 [여기](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift-resource-list.html)에서 확인할 수 있다.

---

### Stack Policy

- CloudFormation Stack 업데이트 중에는 모든 리소스에 대해 모든 업데이트 작업이 허용된다. (기본값)
- 스택 정책은 스택 업데이트 동안 특정 리소스에서 허용되는 업데이트 작업을 정의하는 JSON 문서다.
- 의도하지 않은 업데이트로부터 리소스를 보호한다.
- 스택 정책을 설정하면 스택의 모든 리소스가 기본적으로 보호된다.
- 업데이트를 허용할 리소스에 대한 명시적 허용을 지정한다.

![15-stack-policy.png](images%2F15-stack-policy.png)

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```

- "ProductionDatabase"를 제외한 모든 리소스에 대한 업데이트를 허용한다.

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

![16-change-sets.png](images%2F16-change-sets.png)

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

### EC2 User Data

- 대부분의 CloudFormation 템플릿은 AWS Cloud의 컴퓨팅 리소스 프로비저닝에 관한 것이다.
- 이러한 리소스는 EC2 인스턴스, ASG 등이 될 수 있다.
- 일반적으로 수행해야 하는 작업을 수행할 수 있도록 인스턴스를 자체 구성하기를 원한다.
- CloudFormation Init을 통해 EC2 플릿 상태를 완벽하게 자동화할 수 있다.

![17-user-data.png](images%2F17-user-data.png)

- AWS 콘솔을 사용하여 EC2 인스턴스를 생성한다.
- 이 EC2 인스턴스에 PHP와 MariaDB가 설치되어 있는지 확인해야 한다.
- 이를 위해 사용자 데이터 스크립트를 작성한다.
- 이 스크립트는 인스턴스의 첫 번재 부팅 시에 실행된다.

```yaml
Parameters:
  KeyName:
    Description: Name of an existing EC2 key pair for SSH access to the EC2 instance.
    Type: AWS::EC2::KeyPair::KeyName

  SSHLocation:
    Description: The IP address range that can be used to SSH to the EC2 instances
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.

  ImageId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref ImageId
      InstanceType: t2.micro
      KeyName: !Ref KeyName
      SecurityGroups:
        - !Ref WebServerSecurityGroup
      UserData:
        Fn::Base64: |
           #!/bin/bash
           yum update -y
           amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
           yum install -y httpd mariadb-server
           systemctl start httpd
           systemctl enable httpd
           usermod -a -G apache ec2-user
           chown -R ec2-user:apache /var/www
           chmod 2775 /var/www
           find /var/www -type d -exec sudo chmod 2775 {} \;
           find /var/www -type f -exec sudo chmod 0664 {} \;
           echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php

  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Enable HTTP access via port 80 + SSH access"
      SecurityGroupIngress:
      - CidrIp: 0.0.0.0/0
        FromPort: 80
        IpProtocol: tcp
        ToPort: 80
      - CidrIp: !Ref SSHLocation
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22
```

- CloudFormation 템플릿에 동일한 EC2 사용자 데이터 스크립트를 작성하는 방법에 대해서 알아본다.
- 전달해야 할 중요한 것은 `Fn::Base64` 함수를 통한 전체 스크립트다.
- 사용자 데이터 스크립트 로그는 `/var/log/cloud-init-output.log`에 있다.

#### EC2 User Data의 문제

- 매우 큰 인스턴스 구성을 사용하려고 할 때 문제가 발생할 수 있다.
- EC2 인스턴스를 종료하고 새 인스턴스를 생성하지 않고 EC2 인스턴스의 상태를 발전시킬 수 없다.
- EC2 사용자 데이터 스크립트를 읽기 쉽게 만들는 것이 쉽지 않다.
- EC2 사용자 데이터 스크립트가 성공적으로 완료되었음을 사용자에게 알리기 쉽지 않다.

---

### CFN init

- Amazon Linux 2 AMI에 직접 제공되는 4개의 Python 스크립트가 있거나 Amazon 이외의 Linux AMI에는 yum을 사용하여 설치할 수 있다.
  - `cfn-init`: 리소스 메타데이터 검색 및 해석, 패키지 설치, 파일 생성 및 서비스 실행에 사용된다.
  - `cfn-signal`: CreationPolicy 또는 WaitCondition을 사용하여 신호를 보내는 간단한 래퍼로, 스택의 다른 리소스를 준비 중인 애플리케이션과 동기화할 수 있다.
  - `cfn-get-metadata`: 리소스 또는 리소스 메타데이터의 특정 키 또는 서브트리로 의 경로에 대해 정의된 모든 메타데이터를 쉽게 검색할 수 있는 래퍼 스크립트다.
  - `cfn-hup`: 메타데이터에 대한 업데이트를 확인하고 변경사항이 감지되면 사용자 지정 후크를 실행하는 데몬이다.
- 일반적으로 `cfn-init`과 `cfn-signal`이 사용되며 선택적으로 `cfn-hup`가 사용된다.

#### AWS::CloudFormation::Init

- 구성은 다음의 항목을 포함하며 해당 순서로 실행된다.
  - Packages: Linux/Windows에서 미리 패키지화된 앱 및 구성 요소를 다운로드하여 설치하는데 사용된다. (예: MySQL, PHP 등..)
  - Groups: 사용자 그룹을 정의한다.
  - Users: 사용자를 정의하고 사용자가 속한 그룹을 정의한다.
  - Sources: 파일 및 아카이브를 다운로드하여 EC2 인스턴스에 저장할 수 있다.
  - Files: 인라인을 사용하거나 URL에서 가져올 수 있는 EC2 인스턴스에 파일을 만든다.
  - Commands: 일련의 명령을 실행한다.
  - Services: sysvinit 목록을 시작한다.

![18-cloudformation-init.png](images%2F18-cloudformation-init.png)

- `AWS::CloudFormation::Init`에서 여러 구성을 사용할 수 있다.
- 여러 구성을 사용하여 configSet을 생성한다.
- EC2 사용자 데이터에서 configSet을 호출한다.

#### cfn-init

- cfn-init 스크립트를 사용하면 복잡한 EC2 구성을 읽기 쉽게 만들 수 있다.
- EC2 인스턴스가 CloudFormation 서비스에 쿼리하여 데이터를 가져온다.
- 로그는 `/var/log/cfn-init.log`로 이동한다.

![19-cfn-init.png](images%2F19-cfn-init.png)

#### cfn-signal & wait condition

- cfn-init 실행 후에 EC2 인스턴스가 제대로 구성되었음을 CloudFormation에 알리는 방법이 필요하다.
- 이를 위해 cfn-signal을 사용할 수 있다.
  - cfn-init 직후 cfn-signal을 실행한다.
  - CloudFormation 서비스에 리소스 생성 성공/실패 지속 또는 실패를 알려준다.
- `WaitCondition`을 정의해야 한다.
  - 템플릿이 cfn-signal로부터 신호를 받을 때까지 차단된다.
  - `CreationPolicy`를 첨부한다. (EC2, ASG에서도 작동)
  - Count > 1을 정의할 수 있다. (신호가 1개 이상 필요한 경우)

```yaml
CreationPolicy:
  ResourceSignal:
    Timeout: PT15M
    Count: 1
```

![20-cfn-signal-wait-condition.png](images%2F20-cfn-signal-wait-condition.png)

```yaml
  # 상단 생략 ..
  UserData:
    Fn::Base64:
      !Sub |
        #!/bin/bash -xe
        
        # Get the latest CloudFormation helper scripts
        yum install -y aws-cfn-bootstrap
        
        # Start cfn-init
        /opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource WebServerHost --region ${AWS::Region}
        
        # cfn-init completed so signal success or not
        /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource WebServerHost --region ${AWS::Region}
  # 하단 생략 ..
```

- `yum install -y aws-cfn-bootstrap`을 통해 최신 CloudFormation 헬버 스크립트를 다운로드 한다.
- 이후 cfn-init를 실행한다 cfn-init의 전체 경로를 참조한다.
- cfn-signal은 -e를 반환한다.
  - `$`표시와 `?`표시가 있는데 명령이 성공하면 0을 반환하고 실패하면 오류 코드를 반환한다.
  - 

#### cfn-hup

- EC2 인스턴스에 15분마다 메타데이터 변경 사항을 찾고 메타데이터 구성을 다시 적용하도록 지시하는데 사용할 수 있다.
- 매우 강력하지만 작동 방식을 이해하려면 실제로 시도해 볼 필요가 있다.
- cfn-hup는 2개의 설정 파일에 의존하며 `/etc/cfn/cfn-hup.conf` 및 `/etc/cfn/hooks.d/cfn-auto-reloader.conf`를 참조해야 한다.

![21-cfn-hup.png](images%2F21-cfn-hup.png)

- CloudFormation은 실행하고 init 데이터를 받은 다음에 신호를 받는다.
- cfn-hup 서비스 덕분에 15분마다 메타데이터를 확인한다.
- CloudFormation 템플릿 안에서 메타데이터가 변경되면 설정이 다시 실행되고 cfn-init가 다시 실행된다.

```yaml
# 상단 생략..
services:
  sysvinit:
    httpd:
      enabled: 'true'
      ensureRunning: 'true'
    postfix:
      enabled: 'false'
      ensureRunning: 'false'
    cfn-hup:
      enable: 'true'
      ensureRunning: 'true'
      files:
        - "/etc/cfn/cfn-hup.conf"
        - "/etc/cfn/hooks.d/cfn-auto-reloader.conf"
# 하단 생략..
```

- cfn-hub enabled: 'true', ensureRunning: 'true'로 설정되어 있다.
- `/etc/cfn/cfn-hup.conf`, `/etc/cfn/hooks.d/cfn-auto-reloader.conf` 두 파일을 모니터링한다.
- 두 파일이 변경된 경우엔 cfn-hup 서비스가 다시 시작된다.
- 전체 yaml 파일은 아래에 있다.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: AWS CloudFormation Sample Template for CFN Init
Parameters:
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instances
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.

  SSHLocation:
    Description: The IP address range that can be used to SSH to the EC2 instances
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.

  MyS3BucketName:
    Description: Name of an existing bucket to download files from
    Type: String

  ImageId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP access via port 80 and SSH access via port 22
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: !Ref SSHLocation

  WebServerHost:
    Type: AWS::EC2::Instance
    Metadata:
      Comment: Install a simple PHP application
      AWS::CloudFormation::Init:
        config:
          packages:
            yum:
              httpd: []
              php: []
          groups:
            apache: {}
          users:
            "apache":
              groups:
                - "apache"
          sources:
            "/home/ec2-user/aws-cli": "https://github.com/aws/aws-cli/tarball/master"
          files:
            "/tmp/cwlogs/apacheaccess.conf":
              content: !Sub |
                [general]
                state_file= /var/awslogs/agent-state
                [/var/log/httpd/access_log]
                file = /var/log/httpd/access_log
                log_group_name = ${AWS::StackName}
                log_stream_name = {instance_id}/apache.log
                datetime_format = %d/%b/%Y:%H:%M:%S
              mode: '000400'
              owner: apache
              group: apache
            "/var/www/html/index.php":
              content: !Sub |
                <?php
                echo '<h1>AWS CloudFormation sample PHP application for ${AWS::StackName}</h1>';
                ?>
              mode: '000644'
              owner: apache
              group: apache
            "/etc/cfn/cfn-hup.conf":
              content: !Sub |
                [main]
                stack=${AWS::StackId}
                region=${AWS::Region}
              mode: "000400"
              owner: "root"
              group: "root"
            "/etc/cfn/hooks.d/cfn-auto-reloader.conf":
              content: !Sub |
                [cfn-auto-reloader-hook]
                triggers=post.update
                path=Resources.WebServerHost.Metadata.AWS::CloudFormation::Init
                action=/opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource WebServerHost --region ${AWS::Region}
              mode: "000400"
              owner: "root"
              group: "root"
            # Fetch a webpage from a private S3 bucket
            "/var/www/html/webpage.html":
              source: !Sub "https://${MyS3BucketName}.s3.${AWS::Region}.amazonaws.com/webpage.html"
              mode: '000644'
              owner: apache
              group: apache
              authentication: S3AccessCreds
          commands:
            test:
              command: "echo \"$MAGIC\" > test.txt"
              env:
                MAGIC: "I come from the environment!"
              cwd: "~"
          services:
            sysvinit:
              httpd:
                enabled: 'true'
                ensureRunning: 'true'
              postfix:
                enabled: 'false'
                ensureRunning: 'false'
              cfn-hup:
                enable: 'true'
                ensureRunning: 'true'
                files:
                  - "/etc/cfn/cfn-hup.conf"
                  - "/etc/cfn/hooks.d/cfn-auto-reloader.conf"
      AWS::CloudFormation::Authentication:
        # Define S3 access credentials
        S3AccessCreds:
          type: S3
          buckets:
            - !Sub ${MyS3BucketName}
          roleName: !Ref InstanceRole
    CreationPolicy:
      ResourceSignal:
        Timeout: PT15M
        
    Properties:
      ImageId: !Ref ImageId
      KeyName: !Ref KeyName
      InstanceType: t2.micro
      IamInstanceProfile: !Ref InstanceProfile # Reference Instance Profile
      SecurityGroups:
      - !Ref WebServerSecurityGroup
      UserData:
        Fn::Base64:
          !Sub |
            #!/bin/bash -xe
            
            # Get the latest CloudFormation helper scripts
            yum install -y aws-cfn-bootstrap
            
            # Start cfn-init
            /opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource WebServerHost --region ${AWS::Region}
            
            # cfn-init completed so signal success or not
            /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource WebServerHost --region ${AWS::Region}
          
  InstanceRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Action: 'sts:AssumeRole'
            Principal:
              Service: ec2.amazonaws.com
            Effect: Allow
            Sid: ''
      Policies:
        - PolicyName: AuthenticatedS3GetObjects
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Action:
                  - 's3:GetObject'
                Resource: !Sub 'arn:aws:s3:::${MyS3BucketName}/*'
                Effect: Allow

  InstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Path: /
      Roles:
        - !Ref InstanceRole

Outputs:
  InstanceId:
    Description: The instance ID of the web server
    Value:
      !Ref WebServerHost
  WebsiteURL:
    Value:
      !Sub 'http://${WebServerHost.PublicDnsName}'
    Description: URL for newly created LAMP stack
  PublicIP:
    Description: Public IP address of the web server
    Value:
      !GetAtt WebServerHost.PublicIp
```

---

### Continue Rolling Back an Update

- CloudFormation이 업데이트 중에 모든 변경 사항을 롤백할 수 없는 경우 스택이 "UPDATE_ROLLBACK_FAILED" 상태가 된다.
- 리소스를 원래 상태로 되돌릴 수 없으므로 롤백이 실패한다.
- 예를 들어, CloudFormation 외부에서 삭제된 이전 데이터베이스 인스턴스로 롤백한다.

![22-continue-rolling-back-update.png](images%2F22-continue-rolling-back-update.png)

- 스택 생성이 완료되고 RDS 데이터베이스 인스턴스가 스택의 일부로 생성되었다.
- 스택을 업데이트하면 새로운 RDS 인스턴스가 생성되야하지만 업데이트 롤백이 실패하게 된다.
  - 실패의 사유를 확인해보면 한 사용자가 이전에 RDS 데이터베이스를 삭제한 것으로 확인된다.
- 아래와 같은 해결 방법이 있다.
  - CloudFormation 외부에서 수동으로 오류를 수정한 다음 스택을 계속해서 롤백한다.
  - 롤백할 수 없는 리소스는 건너뛴다. (CloudFormation은 실패한 리소스를 "UPDATE_COMPLETE"로 표시)
- 이 상태에서는 스택을 업데이트할 수 없다.
- 중첩된 스택의 경우 상위 스택을 롤백하면 하위 스택도 모두 롤백하려고 한다.

---

### Custom Resources

- 스택을 생성, 업데이트, 삭제할 때마다 AWS CloudFormation이 실행하는 템플릿에 사용자 지정 프로비저닝 로직을 작성할 수 있도록 지원한다.
- `AWS::CloudFormation::CustomResource` 또는 `Custom::MyCustomResourceTypeName`(권장)을 사용하여 템플릿에 정의된다.
- 두 가지 유형을 지원한다.
  - Amazon SNS 지원 사용자 지정 리소스
  - AWS Lambda 지원 사용자 지정 리소스
- 아래와 같은 사례에 사용된다.
  - AWS 리소스가 아직 적용되지 않은 신규 서비스의 경우
  - 온프레미스의 리소스인 경우
  - 버킷을 삭제하기 전에 S3 버킷을 비워는 람다 함수를 실행해야 하는 경우
  - AMI Id를 조회해야 하는 경우
  - 사용자가 필요로하는 모든 경우

- 서비스 토큰은 CloudFormation이 람다 ARN 또는 SNS ARN과 같은 요청을 보낼 위치를 지정한다. 
  - 필수이며 동일한 리전에 위치해야 한다.
- 선택적으로 입력 데이터 파라미터를 사용하 ㄹ수 있다.

```yaml
Resources:
  LogicalResourceName:
    Type: Custom::MyCustomResourceTypeName
    Properties:
      ServiceToken: service_token
```

```yaml
Resources:
  MyFrontEndTest:
    Type: Custom::PingTester
    Properties:
      ServiceToken: 'arn:aws:sns:us-east-1:123456789012:CRTest'
      key1: string
      key2: 
        - list
      key3: map

Outputs:
  CustomResourceAttribute1:
    Value: !GetAtt
      - MyFrontEndTest
      - responseKey1
  CustomResourceAttribute2:
    Value: !GetAtt
      - MyFrontEndTest
      - responseKey2  
```

#### 작동 방식

![23-custom-resources-how-work.png](images%2F23-custom-resources-how-work.png)

- 템플릿 개발자가 있고 커스텀 리소스를 사용해서 템플릿을 생성한다.
  - 생성된 템플릿을 CloudFormation에 업로드한다.
- CloudFormation이 생성, 업데이트 혹은 삭제를 할 때마다 람다 함수나 SNS 토픽을 전송하고 리소스 제공자를 호출한다.
  - CloudFormation은 S3 사전 서명된 URL이 담긴 요청을 전송해서 응답을 받는다.
  - CloudFormation이 실제로 S3 버킷을 관찰하고 S3 버킷이 CloudFormation 커스텀 리소스 생성에 어떻게 반응할지를 알려주기 때문이다.
- 람다 함수는 API 호출을 하거나 리소스를 피로비저닝하거나 SNS 토픽을 전송할 수도 있다.
- 응답은 반드시 CloudFormation이 제공한 사전 서명된 URL을 이용해서 JSON 형식으로 S3에 업로드되어야 한다.
  - CloudFormation은 응답이 S3에 업로드된 걸 확인하고 성공인지 실패인지에 대한 결론을 내릴 수 있다.

#### Request & Response

- Request
  - RequestType은 Create, Update, Delete다.
  - ResponseURL에는 응답을 위한 사전 서명된 URL이 있다.
  - StackId, RequestId, ResourceType, LogicalResourceId 그리고 CloudFormation으로부터 직접 요청에 전달할 수 있는 입력 데이터인 ResourceProperties가 있다.

```yaml
{
  "RequestType": "Create",
  "ResponseURL": "http://pre-signed-s3-url-for-response",
  "StackId": "arn:aws:cloudformation:us-east-1:123456789012:stack/stack-name/guid",
  "RequestId": "unique id for this create request",
  "ResourceType": "Custom::CustomResource",
  "LogicalResourceId": "MyCustomResource",
  "ResourceProperties": {
    "Name": "Value",
    "List": [ "1", "2", "3" ]
  }
}
```

- Response
  - Status는 "SUCCESS"나 "FAULURE"가 된다.
  - PhysicalResourceId와 StackId, RequestId는 요청과 일치한다.
  - 사용자가 출력을 생성할 수 있는 Data 블럭에는 키-값 쌍으로 정의되어 있다.

```yaml
{
  "Status": "SUCCESS",
  "PhysicalResourceId": "CustomResource1",
  "StackId": "arn:aws:cloudformation:us-east-1:123456789012:stack/stack-name/guid",
  "RequestId": "unique id for this create request",
  "LogicalResourceId": "MyCustomResource",
  "Data": {
    "OutputName1": "Value1",
    "OutputName2": "Value2"
  }
}
```

---

### Lambda-backed Custom Resource

- 비어 있지 않은 S3 버킷을 삭제할 수 없다.
- 비어 있지 않은 S3 버킷을 삭제하려면 먼저 그 안에 있는 모든 객체를 삭제해야 한다.
- 삭제하기 전에 S3 버킷을 비우는데 사용되는 AWS 람다를 사용하여 사용자 지정 리소스를 생성한다.

![24-lambda-backed-custom-resource.png](images%2F24-lambda-backed-custom-resource.png)

- 사용자는 CloudFormation 템플릿에 스택 삭제를 실행한다.
- 스택에는 람다 함수로 지원되는 커스텀 리소스가 있을 수 있다. 
  - 람다 함수에는 S3 버킷을 지우기 위한 스크립트가 내장되어 있다.
- 버킷 비우기를 마친 다음에 CloudFormation은 S3 버킷 자체를 삭제할 수 있다.

```yaml
Parameters:
  S3BucketName:
    Type: String
    Description: "S3 bucket to create"
    AllowedPattern: "[a-zA-Z][a-zA-Z0-9_-]*"

Resources:
  SampleS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref S3BucketName
    DeletionPolicy: Delete

  S3CustomResource:
    Type: Custom::S3CustomResource
    Properties:
      ServiceToken: !GetAtt AWSLambdaFunction.Arn
      bucket_name: !Ref SampleS3Bucket    ## Additional parameter here

  AWSLambdaFunction:
     Type: AWS::Lambda::Function
     Properties:
       Description: "Empty an S3 bucket!"
       FunctionName: !Sub '${AWS::StackName}-${AWS::Region}-lambda'
       Handler: index.handler
       Role: !GetAtt AWSLambdaExecutionRole.Arn
       Timeout: 360
       Runtime: python3.8
       Code:
         ZipFile: |
          import boto3
          import cfnresponse
          ### cfnresponse module help in sending responses to CloudFormation
          ### instead of writing your own code

          def handler(event, context):
              # Get request type
              the_event = event['RequestType']        
              print("The event is: ", str(the_event))

              response_data = {}
              s3 = boto3.client('s3')

              # Retrieve parameters (bucket name)
              bucket_name = event['ResourceProperties']['bucket_name']
              
              try:
                  if the_event == 'Delete':
                      print("Deleting S3 content...")
                      b_operator = boto3.resource('s3')
                      b_operator.Bucket(str(bucket_name)).objects.all().delete()

                  # Everything OK... send the signal back
                  print("Execution succesfull!")
                  cfnresponse.send(event, context, cfnresponse.SUCCESS, response_data)
              except Exception as e:
                  print("Execution failed...")
                  print(str(e))
                  response_data['Data'] = str(e)
                  cfnresponse.send(event, context, cfnresponse.FAILED, response_data)

  AWSLambdaExecutionRole:
     Type: AWS::IAM::Role
     Properties:
       AssumeRolePolicyDocument:
         Statement:
         - Action:
           - sts:AssumeRole
           Effect: Allow
           Principal:
             Service:
             - lambda.amazonaws.com
         Version: '2012-10-17'
       Path: "/"
       Policies:
       - PolicyDocument:
           Statement:
           - Action:
             - logs:CreateLogGroup
             - logs:CreateLogStream
             - logs:PutLogEvents
             Effect: Allow
             Resource: arn:aws:logs:*:*:*
           Version: '2012-10-17'
         PolicyName: !Sub ${AWS::StackName}-${AWS::Region}-AWSLambda-CW
       - PolicyDocument:
           Statement:
           - Action:
             - s3:PutObject
             - s3:DeleteObject
             - s3:List*
             Effect: Allow
             Resource:
             - !Sub arn:aws:s3:::${SampleS3Bucket}
             - !Sub arn:aws:s3:::${SampleS3Bucket}/*
           Version: '2012-10-17'
         PolicyName: !Sub ${AWS::StackName}-${AWS::Region}-AWSLambda-S3
       RoleName: !Sub ${AWS::StackName}-${AWS::Region}-AWSLambdaExecutionRole
```

- S3 버킷을 생성하고 삭제 정책(`DeletionPolicy: Delete`)를 통해서 템플릿이 삭제될 때 버킷도 삭제되도록 설정하였다.
- "S3CustomerResource"에서 커스텀 리소스를 생성한다.
  - 서비스 토큰에는 "AWSLambdaFunction"의 ARN이 입력된다.
- "AWSLambdaFunction"의 Description을 살펴보면 S3 버킷을 지운다고 명시되어 있다.
  - 함수 이름은 `!Sub` 함수 내에서 생성된다.
  - 역할은 CloudFormation에서 생성할 역할(`!GetAtt AWSLambdaExecutionRole.Arn`)이 된다.
  - Code 블럭에는 Python으로 작성된 버킷의 객체를 생성하는 코드가 있다.
- "AWSLambdaExecutionRole"에는 람다 함수에 할당되어야 하는 역할을 정의하고 있다.

---

### Service Role

- CloudFormation에서 스택 리소스를 생성/수정/삭제할 수 있도록 지원하는 역할이다.
- 기본적으로 CloudFormation은 사용자 자격 증명에서 생성하는 임시 세션을 사용한다.
- 아례와 같은 사용 사례가 있다.
  - 최소 권한 원칙을 달성하려는 경우
  - 그러나 스택 리소스를 생성하는데 필요한 모든 권한을 사용자에게 부여하는 것은 원하지 않는 경우
- 사용자가 스택의 리소스를 사용할 수 있는 권한이 없더라도 스택 리소스를 생성/수정/삭제할 수 있는 기능을 제공한다.

![25-service-role.png](images%2F25-service-role.png)

- 사용자가 EC2 인스턴스를 생성할 수 있으면 CloudFormation을 통해 EC2 인스턴스를 생성할 수 있다.
  - 하지만 종종 사용자는 타깃 리소스 생성에 액세스할 수 없는 경우가 있다.
  - 대신 사용자에 제한된 권한 세트만 제공한다. 예를 들어, `cloudformation:*`이나 `iam:PassRole`이 될 수 있다.
- `s3:CreateBucket` 권한 같은 걸 가진 서비스 역할을 사용해서 CloudFormation으로부터의 S3 버킷 생성을 허용할 수 있다.
  - 그러면 `iam:PassRole`을 사용함으로써 우린 서비스 역할을 사용할 수 있다.
  - CloudFormation은 서비스 역할을 이용해서 S3 버킷을 생성할 수 있다.

```yaml
Parameters:
  ImageId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: !Ref ImageId
```

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:*",
                "s3:*",
                "iam:PassRole"
            ],
            "Resource": "*"
        }
    ]
}
```

- t2.micro 타입의 EC2 인스턴스를 생성한다.
- ImageId는 우리가 Parameter Store에서 파라미터로 받는 최신 Amazon Linux 2를 참조한다.
- 우리가 생성할 사용자는 `cloudformation:*`, `s3:*` 그리고 `iam:PassRole`에 대한 액세스 권한만 있다.

---

### SSM Parameter Type

- System Manager Parameter Store의 참조 파라미터다.
- SSM 파라미터 키를 값으로 지정한다.
- CloudFormation은 항상 최신 값을 가져온다.
  - 파라미터의 버전을 지정할 수 없다.
- CloudFormation이 Secure String 값을 저장하지 않으므로 CloudFormation과 호환되어야 한다.
- SSM 파라미터 키에 대해 유효성 검사를 수행할 수 있지만 리턴된 값을 검증하 ㄹ수는 없다.
- 아래와 같은 SSM 파라미터 유형을 지원한다.
  - `AWS::SSM::Parameter::Name`
  - `AWS::SSM::Parameter::Value<String>`
  - `AWS::SSM::Parameter::Value<List<String>>` or `AWS::SSM::Parameter::Value<CommaDelimitedList>`
  - `AWS::SSM::Parameter::Value<AWS-Specific Parameter>`
  - `AWS::SSM::Parameter::Value<List<AWS-Specific Parameter>>`

![26-ssm-parameter-type.png](images%2F26-ssm-parameter-type.png)

- `/dev/ec2/instanceType`이 있다.
- 우리가 스택을 생성하면 CloudFormation은 SSM Parameter Store에 `/dev/ec2/instanceType`이라는 키를 쿼리한다.
  - t2.micro라는 값이 반환된다.

#### Fetch Latest AMI IDs

- SSM Parameter Store에서 최신 AMI ID를 조회한다. (AWS Public Parameters)

![27-fetch-latest-ami-ids.png](images%2F27-fetch-latest-ami-ids.png)

- Amazon Linux 2를 사용하려 한다면 우리는 AMI ID가 리전마다 다르다는 것을 알고 있다.
  - 리전마다 다르기 때문에 템플릿에서 기본값을 사용할 수 없다.
  - 최신 Amazon Linux 2는 주기적으로 AWS에 의해 업데이트된다.
- AWS가 제공하는 Parameter Store 안에 파라미터가 있고 항상 최신 AMI ID를 반환한다.
- 템플릿을 만들 때 파라미터를 전달하고 스택을 생성한다.
  - CloudFormation은 SSM Parameter Store에서 최신 AMI ID를 가져온다.

```yaml
Parameters:
  InstanceType:
    Description: WebServer EC2 instance type
    Type: AWS::SSM::Parameter::Value<String>
    Default: /dev/ec2/instanceType
    
  ImageId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: !Ref ImageId
```

- InstanceType은 유형은 `AWS::SSM::Parameter::Value<String>`이며 기본값은`/dev/ec2/instanceType`의 파라미터 값을 참조한다.
- ImageId은 유형은 `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>`이며 기본값은 `/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2`의 파라미터 값을 참조한다.

---

### Dynamic References

- CloudFormation 템플릿 내 SSM Parameter Store 및 AWS Secrets Manager에 저장된 외부 값을 참조한다.
- CloudFormation은 스택 및 변경 집합 작업 중에 지정된 참조 값을 검색한다.
- 예를 들어, AWS Secret Manager에서 RDS DB 인스턴스 마스터 암호를 검색할 수 있다.
- 아래의 항목들을 지원한다.
  - ssm: SSM Parameter Store에 저장된 평문 문자열
  - ssm-secure: SSM Parameter Store에 저장된 보안 문자열
  - secretsmanager: AWS Secrets Manager에 저장된 비밀 문자열
- 템플릿에서 최대 60개의 동적 참조를 지원한다.

![28-dynamic-references.png](images%2F28-dynamic-references.png)

#### ssm

- String 및 String 배열 유형의 SSM Parameter Store에 저장된 참조 값을 조회한다.
- 버전이 지정되지 않은 경우 CloudFormation에서 최신 버전을 사용한다.
- 공용 SSM Parameters를 지원하지 않는다. (예: Amazon Linux 2 AMI)
- `{{resolve:ssm:parameter-name:version}}`과 같은 형태로 사용된다.

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      AccessControl: '{{resolve:ssm:S3AccessControl:2}}'
```

#### ssm-secure

- 보안 문자열 유형의 SSM Parameter Store에 저장된 참조 값을 조회한다.
- 예를 들어, 비밀번호, 라이선스 키 등을 조회할 때 사용된다.
- CloudFormation은 실제 파라미터 값을 저장하지 않는다.
- 버전이 지정되지 않은 경우 CloudFormation은 최신 버전을 사용한다.
- `{{resolve:ssm-secure:parameter-name:version}}`과 같은 형태로 사용된다.
- 지원되는 리소스와 함께 사용해야 하며 지원되는 리소스는 [공식 홈페이지](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/dynamic-references.html#template-parameters-dynamic-patterns-resources)를 참고하도록 한다.

```yaml
Resources:
  MyIAMUser:
    Type: AWS::IAM::User
    Properties:
      UserName: 'MyUserName'
      LoginProfile:
        Password: '{{resolve::ssm-secure:IAMUserPassword:10}}'
```

#### secretsmanager

- AWS Secrets Manager에 저장된 전체 비밀 또는 비밀 값을 조회한다.
- 예를 들어, 데이터베이스 자격 증명, 암호, 타사 API 키 등을 조회할 수 있다.
- 비밀을 업데이트하려면 비밀 관리자 동적 참조(리소스 속성 중 하나)가 포함된 리소스를 업데이트해야 한다.
- `{{resolve:secretsmanager:secret-id:secret-string:json-key:version-stage:version-id}}`와 같은 형태로 사용된다.

```yaml
Resources:
  MyRDSInstance:
    Type: AWS::RDS:DBInstance
    Properties:
      DBName: MyRDSInstance
      AllocatedStorage: 20
      DBInstanceClass: db.t2.micro
      Engine: mysql
      MasterUsername: '{{resolve:secretsmanager:MyRDSSecret:SecretString:username}}'
      MasterUserPassword: '{{resolve:secretsmanager:MyRDSSecret:SecretString:password}}'
```

#### 예시

```yaml
Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instances
  ImageId: # Public SSM Parameter
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref ImageId
      KeyName: !Ref KeyName
      # ssm dynamic reference
      InstanceType: '{{resolve:ssm:/ec2/instanceType:1}}'

  MyIAMUser:
    Type: AWS::IAM::User
    Properties:
      UserName: 'sample-user'
      LoginProfile:
        # ssm-secure dynamic reference (latest version)
        Password: '{{resolve:ssm-secure:/iam/userPassword}}'

  MyDBInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: db.t2.micro
      Engine: mysql
      AllocatedStorage: "20"
      VPCSecurityGroups:
      - !GetAtt [DBEC2SecurityGroup, GroupId]
      # secretsmanager dynamic reference
      MasterUsername: '{{resolve:secretsmanager:MyRDSSecret:SecretString:username}}'
      MasterUserPassword: '{{resolve:secretsmanager:MyRDSSecret:SecretString:password}}'
  
  DBEC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Allow access from anywhwere to database'
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 3306
        ToPort: 3306
        CidrIp: "0.0.0.0/0"
```

---

### StackSets

- 단일 작업/템플릿으로 여러 계정 및 영역에 걸쳐 스택을 생성, 업데이트 또는 삭제할 수 있다.
- StackSets를 생성하기 위한 관리자 계정이 필요하다.
- StackSets에서 스택 인스턴스를 생성, 업데이트, 삭제할 대상 계정이 있다.
- StackSets를 업데이트하면 모든 계정 및 리전에서 관련된 모든 스택 인스턴스가 업데이트된다.

![29-stacksets.png](images%2F29-stacksets.png)

- 리전 수준의 서비스다.
- AWS Organizations의 모든 계정에 적용이 가능하다.

#### Operations

- Create StackSet
  - 대상 계정과 리전에 템플릿을 제공한다.
- Update StackSet
  - 업데이트는 항상 모든 스택에 영향을 미친다.
  - 일부 스택만 업데이트하는 것은 불가능하다.
- Delete Stacks
  - 대상 계정/리전에서 스택 및 해당 리소스를 삭제한다.
  - StackSet에서 스택을 삭제한다. 단일 스택을 StackSet에서 분리한다는 의미이므로 독립적으로 실행될 수 있다.
  - StackSet에서 모든 스택 삭제한다. StackSet을 삭제하기 위해 직접 준비하는 과정이다.
- Delete StackSet
  - StackSet 내의 모든 스택 인스턴스를 삭제해야 삭제할 수 있다.

#### Deployment Options

- Deployment Order
  - 스택이 배포되는 리전의 순서를 제공한다.
  - 한 번에 리전당 몇 번의 작업이 수행되는지 지정할 수 있다.
- Maximum Concurrent Accounts
  - 한 번에 스택을 배포할 수 있는 리전별 최대 대상 계정 수/퍼센트를 지정할 수 있다.
- Failure Tolerance
  - CloudFormation이 모든 리전에서 작동을 중지하기 전에 발생하 ㄹ수 있는 스택 작업 실패의 최대 수/퍼센트를 지정할 수 있다.
- Region Concurrency
  - StackSet을 리전에 순차적으로 배포할지 또는 병렬로 배포할지 여부를 지정한다.
- Retain Stacks
  - StackSet을 삭제할 때 StackSet에서 제거할 때 스택 및 해당 리소스를 계속 실행하기 위해 사용된다.

#### Permission Models

- Self-managed Permissions
  - 관리자 및 대상 계정 모두에서 신뢰할 수 있는 관계가 설정된 IAM 역할을 생성한다.
  - IAM 역할 생성 권한이 있는 대상 계정에 배포한다.
- Service-managed Permissions
  - AWS Organization에서 관리하는 계정에 배포할 수 있다.
  - StackSet은 사용자를 대신해서 조직안의 IAM 역할을 생성한다.
  - AWS Organization에서 모든 기능을 사용하도록 설정해야 한다.
  - 자동 배포(Automatic Deployments)를 사용해서 사용자 조직에 향후 추가되는 모든 계정에 배포할 수 있다.

![30-permission-model-stackset.png](images%2F30-permission-model-stackset.png)

#### AWS Organizations 통합

- 조직의 새 계정에 스택 인스턴스를 자동으로 배포하는 기능을 제공한다.
- StackSets 관리를 AWS Organization의 멤버 계정에 위임할 수 있다.
- 위임된 관리자가 조직에서 관리하는 계정에 배포하려면 먼저 AWS 조직과 신뢰할 수 있는 액세스를 사용하도록 설정해야 한다.

![31-aws-organizations.png](images%2F31-aws-organizations.png)

- 다양한 OU에 다양한 멤버 계정이 있는 조직이 있다.
- 그 중에 한명을 관리자 계정으로 지정할 수 있다.
- StackSet을 생성할 때 스택 인스턴스는 모든 멤버 계정에 배포되고 관리된다.
  - 새로운 계정이 생성되면 자동으로 새로운 스택 인스턴스가 그 계정에 생성된다.

#### 예시

- StackSets를 사용하여 클릭 한 번으로 AWS 리전 전체에서 AWS Config를 활성화한다.

![32-handson-stacksets.png](images%2F32-handson-stacksets.png)

- 시간이 지남에 따라 모든 리소스 설정을 추적할 수 있게 된다.
- `StackSetAdministrationRole`을 생성하여 관리자 계정에 배포해야 한다.
  - CloudFormation이 이 IAM 역할을 맡게 된다.
- 정책 자체에서는 이 역할이 `AWSCloudFormationStackSetExecutionRole`이라는 역할을 맡은 권한이 있다고 지정한다.
  - 즉, 우리는 대상 계정에서 역할을 맡을 권한을 관리자에게 부여하고 있다.

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: Configure the AWSCloudFormationStackSetAdministrationRole to enable use of AWS CloudFormation StackSets.

Resources:
  AdministrationRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: AWSCloudFormationStackSetAdministrationRole
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              Service: cloudformation.amazonaws.com
            Action:
              - sts:AssumeRole
      Path: /
      Policies:
        - PolicyName: AssumeRole-AWSCloudFormationStackSetExecutionRole
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action:
                  - sts:AssumeRole
                Resource:
                  - "arn:*:iam::*:role/AWSCloudFormationStackSetExecutionRole"
```

- `ExecutionRole`을 생성한다.
  - CloudFormation 템플릿이 사용자의 StackSet 기능을 사용하려는 모든 대상 계정에 적용해야 한다.
  - 만약 AWS에 다른 대상 계정이 있다면 모든 대상 계정에 이 템플릿을 배포해야 한다.
- 템플릿에는 `Parameters`가 있고 우리가 지정해야 하는 `AdministratorAccountId`라는 파라미터다.
- `AssumeRolePolicyDocument`를 통해서 누가 이 역할을 맡을 수 있는지 지정할 수 있다.

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: Configure the AWSCloudFormationStackSetExecutionRole to enable use of your account as a target account in AWS CloudFormation StackSets.

Parameters:
  AdministratorAccountId:
    Type: String
    Description: AWS Account Id of the administrator account (the account in which StackSets will be created).
    MaxLength: 12
    MinLength: 12

Resources:
  ExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: AWSCloudFormationStackSetExecutionRole
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              AWS:
                - !Ref AdministratorAccountId
            Action:
              - sts:AssumeRole
      Path: /
      ManagedPolicyArns:
        - !Sub arn:${AWS::Partition}:iam::aws:policy/AdministratorAccess
```

- `enable-aws-config.yaml` 파일을 생성해본다.
- Metadata 안에 `CloudFormation::Interface`가 있다.
  - 템플릿 안에서 파라미터를 그룹화한다.
- Parameters에는 String이 있고 허용되는 값들이 True 또는 False로 지정되어 있다.
- IncludeGlobalResourceType도 있고 True 또는 False로 지정되어 있다.

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: Enable AWS Config

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: Recorder Configuration
        Parameters:
          - AllSupported
          - IncludeGlobalResourceTypes
          - ResourceTypes
      - Label:
          default: Delivery Channel Configuration
        Parameters:
          - DeliveryChannelName
          - Frequency
      - Label:
          default: Delivery Notifications
        Parameters:
          - TopicArn
          - NotificationEmail
    ParameterLabels:
      AllSupported:
        default: Support all resource types
      IncludeGlobalResourceTypes:
        default: Include global resource types
      ResourceTypes:
        default: List of resource types if not all supported
      DeliveryChannelName:
        default: Configuration delivery channel name
      Frequency:
        default: Snapshot delivery frequency
      TopicArn:
        default: SNS topic name
      NotificationEmail:
        default: Notification Email (optional)

Parameters:
  AllSupported:
    Type: String
    Default: True
    Description: Indicates whether to record all supported resource types.
    AllowedValues:
      - True
      - False

  IncludeGlobalResourceTypes:
    Type: String
    Default: True
    Description: Indicates whether AWS Config records all supported global resource types.
    AllowedValues:
      - True
      - False

  ResourceTypes:
    Type: List<String>
    Description: A list of valid AWS resource types to include in this recording group, such as AWS::EC2::Instance or AWS::CloudTrail::Trail.
    Default: <All>

  DeliveryChannelName:
    Type: String
    Default: <Generated>
    Description: The name of the delivery channel.

  Frequency:
    Type: String
    Default: 24hours
    Description: The frequency with which AWS Config delivers configuration snapshots.
    AllowedValues:
      - 1hour
      - 3hours
      - 6hours
      - 12hours
      - 24hours

  TopicArn:
    Type: String
    Default: <New Topic>
    Description: The Amazon Resource Name (ARN) of the Amazon Simple Notification Service (Amazon SNS) topic that AWS Config delivers notifications to.

  NotificationEmail:
    Type: String
    Default: <None>
    Description: Email address for AWS Config notifications (for new topics).

Conditions:
  IsAllSupported: !Equals
    - !Ref AllSupported
    - True
  IsGeneratedDeliveryChannelName: !Equals
    - !Ref DeliveryChannelName
    - <Generated>
  CreateTopic: !Equals
    - !Ref TopicArn
    - <New Topic>
  CreateSubscription: !And
    - !Condition CreateTopic
    - !Not
      - !Equals
        - !Ref NotificationEmail
        - <None>

Mappings:
  Settings:
    FrequencyMap:
      1hour   : One_Hour
      3hours  : Three_Hours
      6hours  : Six_Hours
      12hours : Twelve_Hours
      24hours : TwentyFour_Hours

Resources:

  ConfigBucket:
    DeletionPolicy: Retain
    Type: AWS::S3::Bucket

  ConfigBucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref ConfigBucket
      PolicyDocument:
        Version: 2012-10-17
        Statement:
          - Sid: AWSConfigBucketPermissionsCheck
            Effect: Allow
            Principal:
              Service:
                - config.amazonaws.com
            Action: s3:GetBucketAcl
            Resource:
              - !Sub "arn:${AWS::Partition}:s3:::${ConfigBucket}"
          - Sid: AWSConfigBucketDelivery
            Effect: Allow
            Principal:
              Service:
                - config.amazonaws.com
            Action: s3:PutObject
            Resource:
              - !Sub "arn:${AWS::Partition}:s3:::${ConfigBucket}/AWSLogs/${AWS::AccountId}/*"

  ConfigTopic:
    Condition: CreateTopic
    Type: AWS::SNS::Topic
    Properties:
      TopicName: !Sub "config-topic-${AWS::AccountId}"
      DisplayName: AWS Config Notification Topic

  ConfigTopicPolicy:
    Condition: CreateTopic
    Type: AWS::SNS::TopicPolicy
    Properties:
      Topics:
        - !Ref ConfigTopic
      PolicyDocument:
        Statement:
          - Sid: AWSConfigSNSPolicy
            Action:
              - sns:Publish
            Effect: Allow
            Resource: !Ref ConfigTopic
            Principal:
              Service:
                - config.amazonaws.com

  EmailNotification:
    Condition: CreateSubscription
    Type: AWS::SNS::Subscription
    Properties:
      Endpoint: !Ref NotificationEmail
      Protocol: email
      TopicArn: !Ref ConfigTopic

  ConfigRecorderRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - config.amazonaws.com
            Action:
              - sts:AssumeRole
      Path: /
      ManagedPolicyArns:
        - !Sub "arn:${AWS::Partition}:iam::aws:policy/service-role/AWS_ConfigRole"

  ConfigRecorder:
    Type: AWS::Config::ConfigurationRecorder
    DependsOn:
      - ConfigBucketPolicy
    Properties:
      RoleARN: !GetAtt ConfigRecorderRole.Arn
      RecordingGroup:
        AllSupported: !Ref AllSupported
        IncludeGlobalResourceTypes: !Ref IncludeGlobalResourceTypes
        ResourceTypes: !If
          - IsAllSupported
          - !Ref AWS::NoValue
          - !Ref ResourceTypes

  ConfigDeliveryChannel:
    Type: AWS::Config::DeliveryChannel
    DependsOn:
      - ConfigBucketPolicy
    Properties:
      Name: !If
        - IsGeneratedDeliveryChannelName
        - !Ref AWS::NoValue
        - !Ref DeliveryChannelName
      ConfigSnapshotDeliveryProperties:
        DeliveryFrequency: !FindInMap
          - Settings
          - FrequencyMap
          - !Ref Frequency
      S3BucketName: !Ref ConfigBucket
      SnsTopicARN: !If
        - CreateTopic
        - !Ref ConfigTopic
        - !Ref TopicArn
```

---

### StackSet Drift 탐지

- StackSet의 각 스택 인스턴스와 연결된 스택에 대해 드리프트 감지를 수행한다.
- 스택에 있는 리소스의 현재 상태가 예상 상태와 다른 경우
  - 드리프트된 것으로 간주되는 스택
  - 스택과 연관된 스택 인스턴스가 드리프트된 것으로 간주된다.
  - StackSet은 드리프트된 것으로 간주된다.
- 드리프트 탐지를 통해 관리되지 않는 변경사항을 식별할 수 있다.
- CloudFormation을 통해 직접 스택으로 변경된 사항은 드리프트된 것으로 간주되지 않는다.
- StackSet에서 드리프트 탐지를 중지할 수 있다.

![33-stackset-drift-detection.png](images%2F33-stackset-drift-detection.png)

---

### AWS Service Catalog

- AWS를 처음 접하는 사용자는 옵션이 너무 많기 때문에 규정을 준수하지 않는 스택을 생성할 수 있는 가능성이 있다.
- 관리자에 의해 미리 정의된 승인된 서비스만 실행할 수 있는 "셀프 서비스 포털"(self-service portal)을 제공할 수 있다.
- VM, 데이터베이스, 스토리지 옵션 등이 포함된다.

#### Diagram

![34-service-catalog-diagram.png](images%2F34-service-catalog-diagram.png)

- 관리자 태스크와 사용자 태스크가 있다.
- 관리자는 파워 유저이기 때문에 CloudFormation 템플릿을 작성할 수 있다.
- 모든 제품을 합쳐서 각각의 포트폴리오를 생성하고 포트폴리오에는 IAM 역할이 할당되어 컨트롤할 수 있다.
- 사용자는 Service Catalog를 통해 제품 목록을 확인할 수 있다.
- IAM에서 승인된 제품들만 확인할 수 있다.

#### 요약

- AWS에서 승인된 IT 서비스 카탈로그를 생성 및 관리한다.
- "제품"(Product)은 CLoudFormation 템플릿이다.
- 예를 들어, VM 이미지, 서버, 소프트웨어, 데이터베이스, 리전, IP 주소 범위 등을 포함할 수 있다.
- CloudFormation을 통해 관리자가 일관성을 확보하고 표준화할 수 있다.
- 이러한 템플릿들은 포트폴리오(팀)에 할당된다.
- 팀에게는 제품을 출시할 수 있는 셀프 서비스 포털이 제공된다.
- 구축된 모든 제품은 중앙에서 관리되는 구축된 서비스다.
- 거버넌스, 컴플라이언스 및 일관성을 지원한다.
- 깊은 AWS 지식 없이 제품 출시에 사용자가 액세스할 수 있다.
- "ServiceNow"와 같은 "셀프 서비스 포털"과의 통합을 지원한다.

#### Stack Set Constraints

- CloudFormation StackSets를 사용하여 제품 배포 옵션을 구성할 수 있다.
- Accounts: 제품을 생성할 AWS 계정을 식별한다.
- Regions: 주문과 함께 구축하려는 AWS 리전을 식별한다.
- Permissions: 대상 AWS 계정을 관리하기 위한 IAM StackSet 관리자 역할을 지정한다.

![35-stack-set-constraints.png](images%2F35-stack-set-constraints.png)

#### Launch Constraints

- 사용자가 최소한의 IAM 권한으로 제품을 시작, 업데이트 또는 종료할 수 있도록 제품에 할당된 IAM 역할이다.
- 예를 들어, 최종 사용자는 Service Catalog에만 액세스할 수 있으며 필요한 다른 모든 권한은 제약 조건 IAM 역할 실행에 연결된다.
- IAM 역할에는 다음 권한이 있어야 한다.
  - CloudFormation Full Access
  - CloudFormation 템플릿의 AWS 서비스
  - CloudFormation 템플릿이 포함된 S3 버킷 (읽기 액세스)

![36-launch-constraints.png](images%2F36-launch-constraints.png)

#### Continuous Delivery Pipeline

![37-continuous-delivery-pipeline.png](images%2F37-continuous-delivery-pipeline.png)

- CodeCommit 코드 리포지토리의 모든 것을 Service Catalog에 동기화하는 상황을 가정한다.
- 새 제폼을 CodeCommit에 푸시할 때 `mapping.yml`이라는 파일을 관리해 어떤 제품이 어떤 템플릿에 해당하는지 파악한다면 템플릿의 버전을 관리할 수 있다.
- Service Catalog에 적용하기만 하면 된다.
- 람다 함수를 사용하면 되고 CodeCommit에 커밋하거나 푸시할 때마다 람다 함수가 호출된다.
  - 람다 함수는 YAML 템플릿과 `mapping.yml`과 같은 데이터를 포함해 템플릿의 모든 정보를 읽는다.
  - 그런 다음 람다 함수의 로직에 따라 서비스 카탈로그에 제품을 업데이트하거나 생성한다.

---

### 참고한 강의

- [DevOps Engineer Professional](https://www.udemy.com/course/aws-certified-devops-engineer-professional-korean)
- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)