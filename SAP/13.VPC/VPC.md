# VPC

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "VPC"에 대해서 알아보도록 한다.

---

### VPC

- CIDR: IP 주소 블록이다.
  - 예를 들어, 192.168.0.0/26은 192.168.0.0 ~ 192.168.0.63 총 64개의 IP를 의미한다.
  - 보안 그룹, 라우트 테이블, VPC, 서브넷 등에 사용된다.
- Private IP
  - 10.0.0.0 ~ 10.255.255.255 (10.0.0.0/8) <- 일반적으로 대규모 네트워크에 사용된다.
  - 172.16.0.0 ~ 172.31.255.255 (172.31.0.0/12) <- 일반적으로 회사 네트워크에 사용된다.
  - 192.168.0.0 ~ 192.168.255.255 (192.168.0.0/16) <- 일반적으로 홈 네트워크에 사용된다.
- Public IP
  - Private IP를 제외한 모든 IP는 Public IP다.
- VPC
  - VPC에는 변경할 수 없는 CIDR 블록의 정의된 목록이 있어야 한다.
  - VPC 내의 각 CIDR는 최소 `/28`, 최대 `/16/` 크기를 지정할 수 있다.
  - VPC는 비공개이므로 Private IP CIDR 범위만 허용된다.
- Subnet
  - VPC 내에서 VPC CIDR의 하위 집합인 CIDR로 정의된다.
  - 서브넷 내의 모든 인스턴스가 Private IP를 가진다.
  - 모든 서브넷에서 처음 4개의 IP와 마지막 1개의 IP는 AWS에 의해 예약되어 있으므로 사용자는 사용할 수 없다.
- Route Table
  - 네트워크 트래픽이 향하는 위치를 제어하는데 사용된다.
  - 특정 서브넷과 연결할 수 있다.
  - 여러 규칙이 있는 경우 "가장 구체적인" 라우팅 규칙을 따른다.
- Internet Gateway (IGW)
  - VPC가 인터넷에 연결할 수 있도록 도와주며 고가용성이고 수평적으로 확장된다.
  - Public IPv4 또는 Public IPv6가 있는 인스턴스에 대해 NAT 역할을 수행한다.
- Public Subnet
  - IGW에 0.0.0.0/0을 전송하는 라우트 테이블이 있다.
  - 인스턴스에 Public IPv4가 있어야 인터넷과 통신할 수 있다.
- Private Subnet
  - Public 서브넷에서 NAT 인스턴스 또는 NAT 게이트웨이를 설정하여 인터넷에 액세스할 수 있다.
  - 0.0.0.0/0이 NAT로 트래픽을 라우팅하도록 경로를 설정해야 한다.

#### NAT Instance

- Public 서브넷에 배포하는 EC2 인스턴스다.
- Private 서브넷의 경로를 편집하여 NAT 인스턴스로 0.0.0.0/0 트래픽을 라우팅한다.
- 장애에 탄력적이지 않으며, 인스턴스 유형에 따라 제한된 대역폭을 제공하지만 비용이 저렴하다.
- 페일오버를 사용자가 직접 관리해야 한다.
- Source/Target 확인을 사용하지 않도록 설정해야 한다. (EC2 설정)

![1-nat-instance.png](images%2F1-nat-instance.png)

#### NAT Gateway

- 관리형 NAT 솔루션으로, 대역폭이 자동으로 확장된다.
- 단일 AZ 내에서 장애에 대한 탄력성을 향상시킨다.
- HA를 위해 여러 AZ에 여러 NAT 게이트웨이를 구축해야 한다.
- Elastic IP가 있는 외부 서비스는 NAT 게이트웨이의 IP를 소스로 본다.

![2-nat-gateway.png](images%2F2-nat-gateway.png)

#### Network ACL (NACL)

- 서브넷 수준에서 정의된 상태 비저장 방화벽이며 내부에 있는 모든 인스턴스에 적용된다.
- 허용 또는 거부 규칙을 지원한다.
- 상태 비저장(Stateless)는 반환되는 트래픽은 규칙에 의해 명시적으로 허용되어야 한다.
- 특정 IP 주소를 빠르고 저렴하게 차단하는데 도움이 된다.

#### Security Group

- 인스턴스 수준에서 적용되며 허용 규칙만 지원되고 거부 규칙은 없다.
- 상태 저장(Stateful)은 반환되는 트래픽은 규칙에 관계없이 자동으로 허용된다.
- 동일한 리전에 있는 다른 보안 그룹을 참조할 수 있다. (피어링된 VPC, 교차 계정)

#### VPC Flow Logs

- VPC를 통해 인터넷 트래픽을 기록한다.
- VPC 수준, 서브넷 수준 또는 ENI 수준에서 정의할 수 있다.
- "거부된 인터넷 트래픽"을 캡처하는데 유용하게 사용된다.
- CloudWatch Logs 및 Amazon S3로 전송할 수 있다.

#### Bastion Host

- Public EC2 인스턴스(bastion host)를 통해 Private EC2 인스턴스로 SSH 접속할 수 있다.
- 인스턴스를 직접 관리하여 페일오버나 재해 복구를 대비해야 한다. 
- SSM Session Manager는 SSH를 사용하지 않고 원격으로 제어할 수 있는 보다 안전한 방법이다.

#### IPv6

- 모든 IPv6은 공용으로 사용되며 총 3.4 * 10^38 개의 주소가 있다.
- 예를 들어, 2600:lfl8:80c:a900::/56과 같은 CIDR를 생성할 수 있다.
- 주소가 "무작위"이고 너무 많기 때문에 온라인에서 스캔할 수 없다.
- IPv6 지원을 사용하여 인스턴스를 생성할 수 있다.
- Public Subnet
  - IPv6 지원을 사용하여 인스턴스를 생성할 수 있다.
  - IGW에 대한 ::/0(모든 IPv6)에 대한 라우트 테이블 항목을 생성한다.
- Private Subnet 
  - VPC에서 "Egress-Only Internet Gateway"(송신 전용)를 생성할 수 있다.
  - Private 서브넷의 라우트 테이블 항목을 ::/0에서 Egress-Only IGW로 추가한다.

---

### VPC Peering

- AWS의 네트워크를 사용하여 두 대의 VPC를 Private하게 연결할 수 있다.
- 동일한 네트워크에 있는 것처럼 행동하도록 설정할 수 있다.
- 두 VPC의 CIDR이 중복되어서는 안된다.
- VPC 연결은 전파되지 않는다.
  - 만약 서로 통신해야 하는 VPC가 있다면 새롭게 연결을 생성해야 한다.
- 다른 AWS 계정으로 VPC 피어링을 수행할 수 있다.
- **인스턴스가 통신할 수 있도록 각 VPC의 서브넷에서 라우트 테이블을 업데이트해야 한다.**

![3-vpc-peering.png](images%2F3-vpc-peering.png)

- VPC A에 있는 인스턴스는 VPC B와 통신할 수 있고 그 반대도 동일하다.
- VPC A와 VPC C 사이에도 VPC 피어링을 생성한다.
- 만약 VPC B와 VPC C 사이에 피어링이 생성되지 않았다면 B와 C는 서로 통신할 수 없다.

#### Good to Know

- VPC 피어링은 리전 간, 상호 계정 간에 작동할 수 있다.
- 피어링된 VPC의 보안 그룹을 참조할 수 있다.

![4-vpc-peering-good-to-know.png](images%2F4-vpc-peering-good-to-know.png)

#### 가장 긴 접두사 일차

- VPC는 가장 긴 접두사 일치를 사용하여 가장 구체적인 경로를 선택한다.

![5-longest-prefix-match.png](images%2F5-longest-prefix-match.png)

- VPC A가 있고 VPC B와 VPC C로 확대된다.
- VPC B와 VPC C는 같은 CIDR 블록을 가지고 있기 때문에 서로 연결될 수 없다.
- 하지만 VPC A & VPC B, VPC A & VPC C는 겹치는 CIDR 블록이 없기 때문에 연결될 수 있다.
- 라우트 테이블을 확인해보면 VPC A에서 "172.16.0.0/16" 범위의 IP는 로컬이 된다.
- "10.0.0.77/32"는 가장 구체적인 경로가 되고 요청은 VPC B로 전달된다.
- 예시에서 "10.0.0.77"의 가장 긴 접두사는 10.0.0.77/32가 된다.
- 다른 표현으로는 "가장 구체적인 경로"가 있다.
- 더 자세한 내용은 [공식 홈페이지](https://docs.aws.amazon.com/vpc/latest/peering/peering-configurations-partial-access.html#one-to-two-vpcs-lpm)에서 확인한다.

#### 불가능한 구성

- VPC의 CIDR이 중첩되는 경우 피어링을 생성할 수 없다.

![6-overlapping-cidr-ipv4.png](images%2F6-overlapping-cidr-ipv4.png)

- VPC 피어링은 전이되지 않기 때문에 연결이 필요한 각각의 VPC들은 전부 피어링을 생성해 주어야 한다.

![7-no-transitive-vpc-peering.png](images%2F7-no-transitive-vpc-peering.png)

- 온프레미스와 연결된 VPC와 피어링을 생성한다고 해서 온프레미스에 접근할 수는 없다.
- 인터넷 게이트웨이가 있는 VPC와 피어링을 생성한다고 해서 인터넷에 접근할 수는 없다.

![8-no-edge-edge-routing.png](images%2F8-no-edge-edge-routing.png)

- VPC 피어링은 NAT 디바이스에 대한 Edge-to-Edge 라우팅을 지원하지 않는다.

![9-no-edge-edge-routing.png](images%2F9-no-edge-edge-routing.png)

---

### Transit Gateway

![10-network-topology-complicated.png](images%2F10-network-topology-complicated.png)

- 전형적인 아키텍처 또는 네트워크 토폴로지를 보면 매우 복잡해지는 것을 확인할 수 있다.
- 수천 개의 VPC와 온프레미스 허브-앤-스포크(Star) 연결 간에 전이되는 피어링이 가능하다.

![11-transit-gateway.png](images%2F11-transit-gateway.png)

- 리전 별 리소스, 리전 간에 작업이 가능하다.
- RAM(Resource Access Manager)을 사용하여 상호 계정간 공유할 수 있다.
- 여러 리전에서 Transit 게이트웨이를 피어링할 수 있다.
- 라우트 테이블을 보면 VPC가 다른 VPC와 통신하는 것을 제한할 수 있다.
- IP 멀티캐스트를 지원한다. (다른 AWS 서비스에서는 지원되지 않음)
- VPC의 인스턴스는 AWS Transit 게이트웨이에 연결된 다른 VPC의 NAT 게이트웨이, NLB, PrivateLink 및 EFS에 액세스할 수 있다.

#### Central NAT Gateway

![12-central-nat-gateway.png](images%2F12-central-nat-gateway.png)

- 왼쪽에 Egress VPC가 있고 그 안에는 NAT 게이트웨이가 있다.
- NAT 게이트웨이를 2개의 AZ로 만든다.
- 즉, AZ1과 AZ2가 아키텍처 내의 다른 VPC로부터의 인터넷 트래픽을 가져온다.
- NAT 게이트웨이는 Egress-VPC에서 공유된다.
- Private App VPC는 TGW를 통해 인터넷에 액세스할 수 있다.
- 이 예시에서 App VPC가 TGW 라우트 테이블을 기반으로 서로 통신할 수 없다.
- 자세한 내용은 [공식 홈페이지](https://aws.amazon.com/blogs/networking-and-content-delivery/creating-a-single-internet-exit-point-from-multiple-vpcs-using-aws-transit-gateway/)에서 확인할 수 있다.

#### RAM을 사용한 공유

- AWS RAM을 사용하여 계정 간 또는 AWS 조직 간에 VPC 첨부에 대한 트랜짓 게이트웨이를 공유할 수 있다.

![13-sharing-through-ram.png](images%2F13-sharing-through-ram.png)

- 계정 A에 트랜짓 게이트웨이가 설정되어 있다.
- 계정 A에서 먼저 트랜짓 게이트웨이를 공유하면 계정 B에서 요청을 수락하고 사용한다.

#### 다중 계정 & 다중 VPC

![14-prevent-vpc-communicating.png](images%2F14-prevent-vpc-communicating.png)

- 트랜짓 게이트웨이를 이용하면 여러 계정과 여러 VPC에 연결할 수 있다.
- 리소스 액세스 관리자를 이용해 일종의 네트워크 토폴로지로 갈 수 있다.
- 서로 다른 계정의 서로 다른 VPC들이 동일한 트랜짓 게이트웨이에 연결된다.
- 물론, Prod 계정과 Staging 계정은 서로 통신하지 않는다.
  - Dev 계정과 Pre-Prod 계정도 서로 통신하지 않는다.
- 라우트 테이블을 생성하여 트랜짓 게이트웨이 어태치먼트를 다음 홉으로 바라보도록 설정할 수 있다.

#### Direct Connect Gateway

![15-direct-connect-gateway.png](images%2F15-direct-connect-gateway.png)

- 온프레미스 데이터 센터나 "Direct Connect Location", "Direct Connect Gateway"를 가지고 있다면 여러 리전으로 연결할 수 있다.
- `us-east-1` 리전의 트랜짓 게이트웨이와 `us-west-2` 리전의 트랜짓 게이트웨이가 서로 피어링되도록 구성할 수 있다.
- 트랜짓 게이트웨이가 피어링된다면 서로 통신할 때 AWS 외부로 나가지 않고 AWS 내부에서 통신이 가능하다.

#### Inter & Intra Region Peering

![16-inter-intra-region-peering.png](images%2F16-inter-intra-region-peering.png)

- 각 피어링 Attachement에 대해 시간당 비용이 청구되고 데이터 처리에 대한 비용은 없다.
- Attachement를 통해 데이터가 리전을 넘나드는 경우에 표준 비용이 발생한다.
- 트랜짓 게이트웨이 ID를 지정해야 한다.

---

### VPC Endpoint

- VPC 엔드포인트를 사용하면 Public(WWW) 네트워크를 대신 Private 네트워크를 사용하여 AWS 서비스들에 연결할 수 있다.
- 수평적으로 확장된다.
- IGW, NAT 등을 사용하지 않고 AWS 서비스에 연결할 수 있다.
- VPC 엔드포인트 게이트웨이는 S3 또는 DynamoDB에 연결할 수 있다.
- VPC 엔드포인트 인터페이스는 DynamoDB를 제외한 모든 AWS 서비스에 연결할 수 있다.
- 연결에 문제가 발생한 경우에는 "VPC에서 DNS 설정" 또는 "라우트 테이블"을 확인해야 한다.

![17-vpc-endpoint.png](images%2F17-vpc-endpoint.png)

#### VPC Endpoint Gateway

- S3 및 DynamoDB에서만 작동하며 VPC 당 하나의 게이트웨이를 생성해야 한다.
- 라우트 테이블의 항목을 업데이트해야 한다.
- 게이트웨이가 VPC 수준에서 정의된다.

![18-vpc-endpoint-gateway.png](images%2F18-vpc-endpoint-gateway.png)

- S3에 접근하고자 하는 EC2 인스턴스가 있다면 라우트 테이블을 수정해야한다.
- VPC에서 DNS 확인을 사용하도록 설정해야 한다.
- S3에 대해 동일한 공개 호스트 이름을 사용할 수 있다.
- 게이트웨이 엔드포인트를 VPC (VPN, DX, TGW, 피어링) 밖으로 확장할 수 없다.

#### VPC Endpoint Interface

- 전용 엔드포인트 인터페이스 호스트 이름을 가지는 ENI를 프로비저닝해야 한다.
- 보안을 위해 보안 그룹을 사용한다.
- Private DNS를 설정해야 하며 엔드포인트를 생성할 때 설정할 수 있다.
  - 서비스의 공용 호스트 이름이 Private 엔드포인트 인터페이스 호스트 이름으로 확인된다.
  - VPC 설정에서 "Enable DNS hostnames"와 "Enable DNS Support"를 "True"로 설정해야 한다.
  - Athena를 예시로 확인해본다.
    - `vpce-0b7d2995e9dfe5418-mwrths3x.athena.us-east-1.vpce.amazonaws.com`
    - `vpce-0b7d2995e9dfe5418-mwrths3x-us-east-1a.athena.us-east-1.vpce.amazonaws.com`
    - `vpce-0b7d2995e9dfe5418-mwrths3x-us-east-1b.athena.us-east-1.vpce.amazonaws.com`
    - `athena.us-east-1.amazonaws.com` (private DNS)
- Direct Connect 및 Site-to-Site VPN에서 인터페이스에 액세스할 수 있다.

---

### VPC Endpoint Policy

- 엔드포인트 정책은 서비스에 대한 액세스를 제어하기 위한 JSON 문서다.
- IAM 사용자 정책 또는 서비스별 정책(예: S3 버킷 정책)을 재정의하거나 대체하지 않는다.

```json
{
  "Statement": [{
    "Action": ["sqs:SendMessage"],
    "Effect": "Allow",
    "Resource": "arn:aws:sqs:us-east-2:123456789012:MyQueue",
    "Principal": {
      "AWS": "arn:aws:iam:123456789012:user/MyUser"
    }
  }]
}
```

- IAM 사용자는 VPC 엔드포인트 외부에서 다른 SQS API를 계속 사용할 수 있다.
- SQS 대기열 정책을 추가하여 VPC 엔드포인트를 통해 수행되지 않은 작업을 거부할 수 있다.

#### VPC Endpoint Policy & S3 Bucket Policy

- 버킷 `my_secure_bucket`에 대한 액세스를 제한하는 VPC 엔드포인트 정책을 생성할 수 있다.

```json
{
  "Statement": [
    {
      "Sid": "Access-to-specific-bucket-only",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::my_secure_bucket",
                   "arn:aws:s3:::my_secure_bucket/*"]   
    }
  ]
}
```

- Amazon Linux 2 리포지토리에 대한 액세스를 허용하는 VPC 엔드포인트 정책을 생성할 수 있다.

```json
{
  "Statement": [
    {
      "Sid": "AmazonLinux2AMIRepositoryAccess",
      "Principal": "*",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::amazonlinux.*.amazonaws.com/*"
      ]
    }
  ]
}
```

- S3 버킷 정책은 다음과 같다.
  - `Condition: "aws:sourceVpce":"vpce-1a2b3c4d"`에서 특정 VPC 엔드포인트에서 전송되지 않는 트래픽을 거부한다.
  - `Condition: "aws:sourceVpc": "vpc-111bbb22"`는 특정 VPC를 위해 사용된다.
- `aws:sourceVpc` 조건은 여러 개의 엔드포인트가 있고 모든 엔드포인트에 대해 S3 버킷에 대한 액세스를 관리하려는 경우 VPC 엔드포인트에서만 작동한다.
- S3 버킷 정책은 특정 Public IP 주소 또는 EIP 주소에서만 액세스를 제한할 수 있다.
  - Private IP를 기준으로 액세스를 제한할 수는 없다.
- `aws:SourceIp` 조건이 VPC 엔드포인트에는 적용되지 않는다.

- 아래는 단 하나의 특정 VPC 엔드포인트를 통해 S3 버킷에 대한 접근을 제한한다.
  - vpce에 정의된 VPC를 제외한 모든 엔드포인트를 제한한다.

```json
{
  "Version": "2012-10-17",
  "Id": "Policy1415115909152",
  "Statement": [
    {
      "Sid": "Access-to-specific-VPCE-only",
      "Principal": "*",
      "Action": "s3:*",
      "Effect": "Deny",
      "Resource": ["arn:aws:s3:::my_secure_bucket",
                   "arn:aws:s3:::my_secure_bucket/*"],
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpce": "vpce-1a2b3c4d"     
        }
      }
    }
  ]
}
```
- 특정 소스 VPC가 아닌 트래픽은 전부 거부하는 정책을 생성할 수 있다.

```json
{
  "Version": "2012-10-17",
  "Id": "Policy1415115909152",
  "Statement": [
    {
      "Sid": "Access-to-specific-VPC-only",
      "Principal": "*",
      "Action": "S3:*",
      "Effect": "Deny",
      "Resource": ["arn:aws:s3:::my_secure_bucket",
                   "arn:aws:s3:::my_secure_bucket/*"],
      "Condition": {
        "StringNotEquals": {
          "aws:sourceVpc": "vpc-111bbb22"
        }
      }
    }
  ]
}
```

#### Troubleshooting

![19-troubleshooting.png](images%2F19-troubleshooting.png)

- Private 서브넷의 EC2 인스턴스를 S3 버킷에 연결할 때, 잘못될 수 있는 많은 가능성이 있다.
- 처음으로는 EC2 인스턴스의 보안 그룹을 확인해야 한다.
  - 아웃바운드 규칙으로 인해서 트래픽이 나가지 못한다면 S3에 접근할 수 없다.
- VPC 엔드포인트 게이트웨이를 생성하고 S3 버킷에 접근해야 한다.
  - VPC 엔드포인트 정책을 생성하여 인스턴스가 S3 버킷에 액세스할 수 있도록 한다.
- 라우트 테이블도 업데이트해서 S3를 위한 모든 요청이 VPC 엔드포인트 게이트웨이를 통하도록 해야 한다.
- EC2 인스턴스 역할이 S3 버킷에 접근할 수 있는 권한이 있는지 확인해야 한다.

---

### AWS PrivateLink (VPC Endpoint Service)

- 서비스를 자신 또는 다른 계정의 1,000개의 VPC에 노출시키는 가장 안전하고 확장가능한 방법이다.
- VPC 피어링, 인터넷 게이트웨이, NAT, 라우트 테이블 등이 필요하지 않다.
- 서비스 VPC에는 NLB가 필요하고 고객 VPC에는 ENI가 필요하다.
- NLB가 여러 AZ에 있고 ENI가 여러 AZ에 있는 경우 솔루션은 내결함성을 가진다.

![20-aws-privatelink.png](images%2F20-aws-privatelink.png)

- "서비스 VPC"와 "애플리케이션 서비스"가 있고 다른 계정의 고객에게 액세스 권한을 제공해야 한다.
- "고객 VPC"와 "고객 애플리케이션"은 우리의 애플리케이션 서비스에 액세스해야 한다.
- 애플리케이션을 Public으로 공개하는 경우 특정 계정 뿐만 아니라 모든 사용자들이 액세스할 수 있으므로 보안상 문제가 발생할 수 있다.
- "서비스 VPC"에는 NLB를 생성하고 "고객 VPC"에는 ENI를 생성하여 PrivateLink를 통하도록 구성할 수 있다.
  - NLB와 ENI는 Private하게 통신한다.

#### S3 with Direct Connect

![21-privatelink-s3-direct-connect.png](images%2F21-privatelink-s3-direct-connect.png)

- 회사의 데이터 센터가 Direct Connect를 통해 AWS와 연결되어 있고 S3 버킷에 액세스해야 한다.
- 첫 번째 옵션으로 Public VIF를 사용하여 S3의 공용 URL을 통해 버킷에 액세스할 수 있다.
  - 이러한 방법을 사용하는 경우 버킷은 Public 액세스를 허용해야 한다.
- 두 번째 옵션으로는 PrivateLink와 엔드포인트 인터페이스를 생성하여 Private VIF를 통해 통신하도록 할 수 있다.
  - 게이트웨이 엔드포인트는 VPC 내에서만 작동하기 때문에 사용할 수 없다.
  - 버킷이 Public으로 공개되는 것을 방지할 수 있다.

#### PrivateLink and VPC Peering

![22-privatelink-vpc-peering.png](images%2F22-privatelink-vpc-peering.png)

- VPC 피어링을 통해 PrivateLink에 액세스하는 방법이 있다.
- VPC 엔드포인트를 하나의 리전에서 생성했고 다른 리전에서 액세스하는 상황이다.
- S3 버킷에 개인 액세스 권한을 갖도록 한다.
- `us-east-1`의 VPC와 EC2 인스턴스를 사용하고 `eu-west-1`의 VPC와 피어링하면 인스턴스의 인터페이스 엔드포인트는 VPC 엔드포인트의 URL을 이용해 다른 리전의 Private S3 버킷에 액세스할 수 있다.

---

### Site to Site VPN

- On-Premise
  - 소프트웨어 또는 하드웨어 VPN 어플리아언스를 온프레미스 네트워크에 설정한다.
  - Public IP를 사용하여 온프레미스 VPN에 액세스할 수 있어야 한다.
- AWS-Side
  - VGW(Virtual Private Gateway)를 설정하고 VPC에 연결한다.
  - 온프레미스 VPN 어플라이언스를 가리키도록 고객 게이트웨이를 설정한다.

![23-site-to-site-vpn.png](images%2F23-site-to-site-vpn.png)

- IPSec를 사용하여 암호화된 이중화를 위해 두 개의 VPN 연결(터널)이 생성된다.
- 선택적으로 "Global Accelerator"를 사용하여 가속화할 수 있다.

#### Route Propagation

![24-route-propagation.png](images%2F24-route-propagation.png)

- 회사 데이터 센터와 VPC가 있고 둘 다 중첩되지 않은 CIDR와 Site-to-Site VPN 연결을 가지고 있다.
- VPC 쪽에는 VGW가 있고 기업 데이터 센터 쪽에는 CGW가 있다.
- Private 서브넷에 있는 인스턴스가 VGW를 통해 통신할 수 있도록 해야 한다.
- 서브넷 수준에서 인스턴스가 VGW로 가는 길을 찾을 수 있도록 VPC 쪽에서 라우트 테이블을 설정해야 한다.
- CIDR과 회사 데이터 센터에 상응하고 회사 데이터 센터에 상응하도록 온프레미스 라우터를 업데이트해야 한다.
- Private 서브넷에 해당하는 모든 IP는 CGW를 통해 라우팅 되도록 설정해야 한다.
- 라우팅을 설정하는 두 가지 방법이 있다.
- **Static Routing**
  - CGW를 통해 기업 데이터 센터에서 10.0.0.1/24용 정적 경로를 생성할 수 있다.
  - VGW를 통해 AWS에서 10.3.0.0/20용 정적 경로를 생성할 수 있다.
- **Dynamic Routing (BGP)**
  - BGP(Border Gateway Protocol)를 사용하여 경로를 자동으로 공유할 수 있다. (eBGP는 인터넷을 위해 사용)
  - 라우팅 테이블을 업데이트할 필요가 없으며 동적으로 수행된다.
  - CGW 및 VGW의 ASN(Autonomous System Number)을 지정하기만 하면 된다.

#### Internet Access

![25-internet-access-not-okay.png](images%2F25-internet-access-not-okay.png)

- VPC와 Public 서브넷이 있고 NAT 게이트웨이도 있지만 인터넷 게이트웨이와 연결되어 있어야 `google.com`을 요청할 수 있다.
- 회사 데이터 센터에는 고객 게이트웨이에 연결된 서버가 있다.
- VGW에 대한 Site-to-Site VPN도 설정했고 Direct Connect로 대체할 수 있다.
- **이 방법을 통해서 회사 데이터 센터는 인터넷에 연결될 수 없다.**
  - NAT 게이트웨이는 Site-to-Site VPN이나 Direct Connect 오는 모든 소스 네트워크는 NAT 게이트웨이를 통해 인터넷 게이트웨이나 Public 인터넷으로 갈 수 없다.

![26-internet-access-okay.png](images%2F26-internet-access-okay.png)

- 설정은 완전 동일하지만 NAT 게이트웨이 대신 NAT 인스턴스를 사용하는 경우 온프레미스 서버는 `google.com`을 요청할 수 있다.
- NAT 인스턴스의 경우 소프트웨어를 더 세분화하여 제어할 수 있으므로 NAT 게이트웨이와는 다르다.

![27-internet-access-okay.png](images%2F27-internet-access-okay.png)

- 이번에는 반대로 VPC의 인스턴스가 회사 데이터 센터의 NAT를 통해서 `google.com`에 요청하는 상황이다.
- Site-to-Site VPN 또는 Direct Connect를 통해서 연결되어 있으며 인스턴스는 인터넷에 연결될 수 있다.

#### AWS VPN CloudHub

- 각 VGW(Virtual Private Gateway)에 대해 최대 10개의 고객 게이트웨이 연결이 가능하다.
- 리전 간 1차 또는 2차 네트워크 연결을 위한 저비용 Hub-And-Spoke 모델을 제공한다.
- 여러 VPN 연결이 있는 경우 사이트 간 보안 통신을 제공한다.
- VPN 연결이기 때문에 Public 인터넷을 통해 연결된다.
- 온프레미스 위치 간 페일오버 연결이 될 수 있다.

![28-vpn-cloudhub.png](images%2F28-vpn-cloudhub.png)

#### Multiple VPC

- VPN 기반 고객의 경우 고객 VPC별로 별도의 VPN 연결을 생성하는 것이 AWS의 권장 사항이다.
- Direct Connect 게이트웨이가 있으므로 Direct Connect가 권장된다.

![29-vpn-multiple-vpc.png](images%2F29-vpn-multiple-vpc.png)

#### Shared Service

- 온프레미스 및 "공유 서비스 VPC" 간의 VPN 피어링을 생성한다.
- 온프레미스와 "공유 서비스 VPC" 간에 서비스, 애플리케이션, 데이터베이스 복제 또는 "공유 서비스 VPC" 내 프록시를 구축한다.
- VPC와 "공유 서비스 VPC" 간에 VPC 피어링을 생성한다.
- VPC는 "공유 서비스 VPC" 서비스에 직접 액세스할 수 있으며 온프레미스에 VPN 연결이 필요하지 않다.

![30-shared-service.png](images%2F30-shared-service.png)

---

### AWS Client VPN

- OpenVPN을 사용하여 컴퓨터에서 AWS 및 온프레미스의 Private 네트워크에 연결한다.

![31-client-vpn.png](images%2F31-client-vpn.png)

- 컴퓨터에 소프트웨어를 설치하고 그 연결을 공용 인터넷을 통해 VPC로 직접 사용하게 되고 Private IP를 사용해 EC2 인스턴스에 액세스할 수 있다.
- Site-to-Site VPN을 활성화했다면 회사 데이터 센터에 액세스할 수 있다.
- 따라서 회사 데이터 센터 내의 내부 리소스에 컴퓨터가 액세스할 수 있다.

#### Peered VPC

- 클라이언트 VPN이 VPC 피어링과 호환된다.

![32-peered-vpc.png](images%2F32-peered-vpc.png)

- VPC 중 하나(VPC-A)에 클라이언트 VPN을 연결하면 VPC 피어링을 통해 다른 VPC(VPC-B)의 리소스에도 접근할 수 있다.

#### Access to On-Premise

- 클라이언트 VPN을 사용하여 AWS를 통해 온프레미스 리소스에 액세스할 수 있다.

![33-access-to-onpremise.png](images%2F33-access-to-onpremise.png)

- VPC와 온프레미스 서버가 Site-to-Site VPN을 통해 연결되어 있다.
- 클라이언트 VPN 엔드포인트를 통해서 클라이언트와 VPC-A가 연결되어 있다.
- 클라이언트는 VPC-A를 통해서 온프레미스 데이터 센터의 리소스에 접근할 수 있으며 접근은 VPC-A를 통해서만 가능하다.

#### Internet Access

![34-clientvpn-internet-access.png](images%2F34-clientvpn-internet-access.png)

- Public 서브넷과 인터넷 게이트웨이가 활성화되어 있다.
- 클라이언트 VPN 엔드포인트를 만들면 엔드포인트는 두 Public 서브넷에 연결되다.
- 덕분에 인터넷 게이트웨이를 통해 인터넷으로 가는 경로가 생성된다.
- 사용자가 VPN 연결로 VPC 리소스에 액세스할 수도 있지만 VPC를 통해 인터넷 게이트웨이로 인터넷에 액세스할 수도 있다.
- Private 서브넷을 사용하는 경우에도 NAT 게이트웨이가 추가되었을 뿐 동일하게 작동한다.

#### Transit Gateway

![35-transit-gateway.png](images%2F35-transit-gateway.png)

- VPC-A가 사용자와 클라이언트 VPN을 통해서 연결되어 있다.
- VPC-A는 VPC-B, VPC-C와 트랜짓 게이트웨이를 통해 연결되어 있고 사용자는 VPC-B, VPC-C의 리소스에도 액세스할 수 있다.
- VPC-A는 트랜짓 게이트웨이를 통해 Direct Connect 또는 VPN 연결을 통해 온프레미스 데이터 센터와 연결되어 있다.
  - 사용자는 클라이언트 VPN을 통해서 온프레미스 데이터 센터에 연결될 수 있다.

---

### Direct Connect

- 원격 네트워크에서 VPC로 전용 프라이빗 연결을 제공한다.
- 데이터 센터와 AWS Direct Connect 로케이션 간에 전용 연결을 설정해야 한다.
- VPN 솔루션을 실행하는 것보다 더 많은 비용이 발생한다.
- VIF를 통해 AWS 서비스에 대한 Private 액세스가 가능하다.
- ISP를 우회하여 네트워크 비용을 절감하고 대역폭 및 안정성을 향상시킨다.
- 기본적으로 고가용성을 제공하지 않기 때문에 페일오버를 위해서 DX 또는 VPN을 설정해야 한다.

#### Virtual Interface (VIF)

- Public VIF: Public VPC 엔드포인트(S3 버킷, EC2 서비스 등..)에 연결한다.
- Private VIF: VPC의 리소스(EC2 인스턴스, ALB 등..)에 연결한다.
- Trasit Virtual Interface: 트랜짓 게이트웨이를 사용하여 VPC의 리소스에 연결한다.
- Private VIF를 통해 VPC 인터페이스 엔드포인트에 액세스할 수 있다.

#### Diagram

![36-direct-connect-diagram.png](images%2F36-direct-connect-diagram.png)

- 회사 데이터 센터에 연결하고 싶은 리전이 있다.
- 회사의 데이터 센터에 라우터와 방화벽을 설치한다.
- 회사의 데이터 센터에서 Virtual Private Gateway에 액세스할 때는 Private VIF를 사용해서 접근한다.
- 만약 공공 AWS 서비스에 액세스하는 경우 Public VIF를 사용해서 접근한다.

#### Connection Types

- **Dedicated Connection**
  - 1Gbps, 10Gbps, 100Gbps 용량을 지원한다.
  - 고객 전용 물리적 이더넷 포트를 제공한다.
  - AWS에 먼저 요청한 후 AWS Direct Connect 파트너에 의해 완료된다.
- **Hosted Connection**
  - 50Mbps, 500Mbps, 최대 10Gbps 용량을 지원한다.
  - 연결 요청은 AWS Direct Connect 파트너를 통해 이루어진다.
  - 일바 파트너에서 1, 2, 5, 10Gbps 용량 선택이 가능하다.
- 설치하는데 일반적으로 1개월 이상의 시간이 소요된다.

#### Encryption

- 전송 중인 데이터는 암호화되지 않는다.
- AWS Direct Connect + VPN은 IPsec으로 암호화된 전용 연결을 제공한다.
- Public VIF를 사용하는 VPN과 Direct Connect 연결을 사용할 수 있다.
- 보안 수준을 높이는데는 적합하지만 기능 구현이 더 복잡해진다.

![37-direct-connect-encryption.png](images%2F37-direct-connect-encryption.png)

#### Link Aggregation Groups (LAG)

- 기존 DX 연결을 논리적 연결로 합산하여 속도 및 페일오버를 향상한다.
- 최대 4개의 연결을 집계할 수 있다. (active-active 모드)
- 시간 경과에 따라 LAG에 연결을 추가할 수 있다.
- LAG의 모든 연결은 아래와 같은 특징이 있다.
  - 전용 연결이어야 한다.
  - 대역폭이 동일해야 한다.
  - 동일한 AWS Direct Connect 엔드포인트에서 종료해야 한다.
- LAG가 작동할 수 있는 최소 연결 수를 설정할 수 있다.

![38-link-aggregation-groups.png](images%2F38-link-aggregation-groups.png)

#### Direct Connect Gateway

- 여러 다른 리전(동일/교차 계정)에서 하나 이상의 VPC에 Direct Connect를 설정하려면 Direct Connect Gateway를 사용해야 한다.

![39-direct-connect-gateway.png](images%2F39-direct-connect-gateway.png)

- 두 개의 영역이 있고 고객 게이트웨이가 있따.
- 특정 지역에 Direct Connect를 설정하고 Private VIF를 통해 Direct Connect 게이트웨이와 연결된다.

#### Direct Connect Gateway + Transit Gateway

![40-direct-connect-gateway-transit-gateway.png](images%2F40-direct-connect-gateway-transit-gateway.png)

- Direct Connect 게이트웨이를 두 개씩 활용할 수도 있다.
- 트랜짓 게이트웨이는 많은 VPC와 VPN 직접 연결이 가능하다.
- 맨 위의 Direct Connect 게이트웨이를 연결하려면 Direct Connect 게이트웨이를 통과해야 한다.

---

### 온프레미스 중복 연결

![41-site-to-site-active-active-connection.png](images%2F41-site-to-site-active-active-connection.png)

- 회사 데이터 센터 1과 2가 있고 둘 다 전용 VPN 연결을 통해 AWS로 연결되어 있다.
- 회사 데이터 센터 1과 2는 서로 내부 연결을 통해 연결되어 있다.
- 예를 들어, 첫 번째 VPN 연결이 끊어지면 네트워크는 회사 데이터 센터 1에서 회사 데이터 센터 2로 연결될 수 있고 반대도 가능하다.

#### Direct Connect - High Availability

![42-direct-connect-high-availability.png](images%2F42-direct-connect-high-availability.png)

- Direct Connect도 S2S VPN과 동일하게 고가용성을 유지할 수 있다.
- 두 개의 데이터 센터에 두 개의 Direct Connect를 설정하고 데이터 센터는 서로 내부 연결되어 있다.
- 하나의 Direct Connect가 다운되더라도 다른 Direct Connect를 통해서 AWS에 액세스할 수 있다.

![43-direct-connect-hybrid.png](images%2F43-direct-connect-hybrid.png)

- S2S VPN과 Direct Connect를 같이 사용할 수 있다.
- 기본적으로 Direct Connect를 사용하고 Direct Connect에 문제가 발생한 경우 백업용으로 설정한 S2S VPN을 사용해서 통신한다.

#### Direct Connect Gateway - SiteLink

- Direct Connect 로케이션에서 다른 로케이션으로 데이터를 전송할 수 있다.
- AWS 리전을 우회한다.
- 온프레미스 데이터 센터를 Direct Connect 로케이션에 연결하여 사설 네트워크 연결을 생성한다.
- Direct Connect 로케이션 간에 가장 빠른 경로를 통해 데이터를 전송한다.

![44-direct-connect-gateway-sitelink.png](images%2F44-direct-connect-gateway-sitelink.png)

- 회사의 데이터 센터가 2개가 있고 라우터가 있지만 서로 다른 지역에 있다.
- 각 데이터 센터는 서로 다른 Direct Connect Location에 연결되어 있다.
- AWS에는 Direct Connect Gateway가 설치되어 있고 SiteLink를 활성화하면 데이터 센터간 트래픽이 연결될 수 있다.

---

### VPC Flow Logs

- 인터페이스로 들어가는 IP 트래픽에 대한 정보를 캡처한다.
  - VPC Flow Logs
  - Subnet Flow Logs
  - ENI(Elastic Network Interface) Flow Logs
- 연결 문제를 모니터링하고 문제를 해결하는데 도움이 된다.
- 플로우 로그 데이터는 S3, CloudWatch Logs 및 Kinesis Data Firehose로 이동할 수 있다.
- AWS 관리 인터페이스에서도 네트워크 정보를 캡처할 수 있다.
  - ELB, RDS, ElastiCache, Redshift, WorkSpaces, NAT 게이트웨이, 트랜짓 게이트웨이 등..

#### Syntax

![45-syntax.png](images%2F45-syntax.png)

- `srcaddr` & `dstaddr`: 문제가 있는 IP를 식별하는데 도움이 된다.
- `srcport` & `dstport`: ID에 문제가 있는 포트를 지원한다.
- `action`: 보안 그룹 또는 NACL로 인한 요청의 성공 또는 실패를 나타낸다.
- 사용 패턴이나 악의적인 행동에 대한 분석에 사용할 수 있다.
- S3의 Athena 또는 CloudWatch Logs Insights를 사용하여 VPC Flow Logs를 쿼리할 수 있다. 
- VPC Flow Logs의 자세한 예시는 [공식 홈페이지](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-records-examples.html)에서 확인할 수 있다.

#### Troubleshoot SG & NACL

- 보안 그룹의 경우 Stateful이기 때문에 인바운드가 허용되는 경우 아웃바운드도 허용된다.
- NACL의 경우 Steteless이기 때문에 인바운드가 허용된다고 아웃바운드가 허용되지 않는다.
  - 아웃바운드도 반드시 허용해주어야 한다.

![46-incoming-request.png](images%2F46-incoming-request.png)

![47-outgoing-request.png](images%2F47-outgoing-request.png)

#### VPC Flow Logs & NAT Gateway

- VPC Flow Logs에는 Public IP 주소에서 들어오는 인바운드 트래픽에 대한 액션이 ACCEPT로 표시된다.
  - 그러나 NAT 게이트웨이에 대해 알고 있는 바로는 인터넷에서 들어오는 트래픽을 받지 않는 것으로 확인된다.
  - NAT 게이트웨이가 인터넷에서 들어오는 트래픽을 받아들이는지 확인이 필요하다.
- 인바운드 트래픽이 보안 그룹 또는 NACL에 의해 허용된다.
  - NAT 게이트웨이는 트래픽을 허용하지 않고 트래픽이 감소한다.
  - CloudWatch 로그 그룹에서 쿼리를 실행하여 확인할 수 있다.
- `xxx.xxx`을 VPC CIDR의 첫 두 옥텟으로 설정한다.
- Public IP를 로그에 표시된 IP로 변경한다.
- NAT 게이트웨이의 Private IP에 트래픽이 표시되지만 다른 곳에서는 트래픽이 요청되지 않고 삭제되었다.

![48-vpc-flow-logs-nat-gateway.png](images%2F48-vpc-flow-logs-nat-gateway.png)

---

### AWS Network Firewall

#### Network Protection

- AWS에서 네트워크를 보호하기 위해 아래의 기능들을 사용할 수 있따.
  - Network Access Control List(NACL)
  - Amazon VPC Security Group
  - AWS WAF (악의적인 요청으로부터 보호)
  - AWS Shield & AWS Shield Advanced
  - AWS Firewall Manager (계정 간 관리)

#### AWS Network Firewall

- 전체 Amazon VPC를 보호한다.
- Layer 3 ~ Layer 7까지 보호한다.
- 방향과 상관없이 검사할 수 있다.
  - VPC to VPC 트래픽
  - 인터넷으로 아웃바운드 트래픽
  - 인터넷에서 인바운드 트래픽
  - Direct Connect / Site-to-Site VPN In/Out 트래픽
- 내부적으로 AWS Network Firewall은 AWS 게이트웨이 로드밸런서를 사용한다.
- AWS Firewall Manager를 통해 여러 VPC에 적용할 수 있도록 상호 계정 간에 규칙을 중앙에서 관리할 수 있다.

![49-aws-network-firewall.png](images%2F49-aws-network-firewall.png)

#### Fine Grained Control

- 1,000개의 규칙을 지원한다.
  - IP & Port: 예를 들어, 10,000개의 IP를 필터링할 수 있다.
  - 프로토콜: 예를 들어, 아웃바운드 통신을 위한 SMB 프로토콜을 차단할 수 있다.
  - Stateful 도메인 리스트 규칙 그룹: `*.mycorp.com` 또는 타사 소프트웨어 레포의 아웃바운드 트래픽만 허용할 수 있다.
  - 정규식을 이용한 일반 패턴을 매칭할 수 있다.
- 트래픽 필터링: 규칙과 일치하는 트래픽에 대한 허용, 삭제 또는 경고를 지원한다.
- 침입 방지 기능을 통해 네트워크 위협으로부터 보호하기 위한 능동적인 흐름 검사를 제공한다.
  - 게이트웨이 로드밸런서와 같이 AWS에서 모두 관리된다.
- 규칙 일치 로그를 S3, CloudWatch Logs, Kinesis Data Firehose로 전송할 수 있다.

#### Architecture

![50-network-firewall-architecture.png](images%2F50-network-firewall-architecture.png)

- VPC와 인스턴스가 있고 모든 트래픽이 Inspection VPC를 통과해야 한다.
  - 여기서 모든 트래픽이 분석된다.
- 인터넷에 접속하기 위한 요구사항도 있다.
- 게이트웨이 내부에서 Egress VPC를 생성한다.
  - 인터넷 게이트웨이는 인터넷에 접속할 수 있게 해준다.
- 또한 온프레미스 데이터 센터나 기업 데이터 센터도 있을 수 있으므로 VPN 연결이나 Direct Connect 연결이 필요하다.
- 모든 트래픽이 Inspection VPC를 통과해야 하고 VPC 연결이 복잡해지기 때문에 트랜짓 게이트웨이를 중앙에 추가할 수 있다.

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)