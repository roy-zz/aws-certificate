# Amazon VPC

이번 장에서는 **SysOps Administrator**를 준비하며 **Virtual Private Cloud의 약자인 VPC**에 대해서 알아보도록 한다.

---

### CIDR

#### IPv4

- **CIDR(Classless Inter-Domain Routing)**: IP 주소 할당 방법이다.
- 일반적으로 보안 그룹 규칙 및 AWS 네트워킹에 사용된다.

![1-cidr-ipv4.png](images%2F1-cidr-ipv4.png)

- IP 주소 범위를 정의하는 데 도움이 된다.
  - `WW.XX.YY.ZZ/32` => 하나의 IP가 할당된다.
  - `0.0.0.0/0` => 모든 IP가 할당된다.
  - `192.168.0.0` => `192.168.0.0` ~ `192.168.0.63` 범위의 총 64개의 IP가 할당된다.

- CIDR은 두 가지 구성요소로 구성된다.
- **Base IP**
  - 범위(`XX.XX.XX.XX`)에 포함된 IP를 나타낸다.
  - 예를 들어, `10.0.0.0`, `192.168.0.0` 등이 있다.
- **Subnet Mask**
  - IP에서 변경할 수 있는 비트 수를 정의한다.
  - 예를 들어, `/0`, `/24`, `/32` 등이 있다.
  - 두 가지 형식을 가질 수 있다.
    - `/8` = `255.0.0.0`
    - `/16` = `255.255.0.0`
    - `/24` = `255.255.255.0`
    - `/32` = `255.255.255.255`

#### Subnet Mask

- 서브넷 마스크는 기본적으로 기본 IP의 일부가 기본 IP에서 추가 다음 값을 얻을 수 있도록 허용한다.
  - `192.168.0.0/32` => `192.168.0.0` 1개의 IP를 허용한다.
  - `192.168.0.0/31` => `192.168.0.0` ~ `192.168.0.1` 2개의 IP를 허용한다.
  - `192.168.0.0/30` => `192.168.0.0` ~ `192.168.0.3` 4개의 IP를 허용한다.
  - `192.168.0.0/29` => `192.168.0.0` ~ `192.168.0.7` 8개의 IP를 허용한다.
  - `192.168.0.0/28` => `192.168.0.0` ~ `192.168.0.15` 16개의 IP를 허용한다.
  - `192.168.0.0/27` => `192.168.0.0` ~ `192.168.0.31` 32개의 IP를 허용한다.
  - `192.168.0.9/0` => `0.0.0.0` ~ `255.255.255.255` 모든 IP를 허용한다.

![2-cidr-subnet-mask.png](images%2F2-cidr-subnet-mask.png)

- [CIDR 연습 사이트](https://www.ipaddressguide.com/cidr)를 통해 CIDR를 손쉽게 변환할 수 있다.

#### Public IP vs Private IP (IPv4)

- IANA(Internet Assigned Numbers Authority)는 개인(LAN) 및 공용(인터넷) 주소 사용을 위해 특정 IPv4 주소 블록을 설정했다.
- Private IP는 특정 값만 사용할 수 있다.
  - `10.0.0.0` ~ `10.255.255.255` (`10.0.0.0/8`): 대규모 사설 네트워크에 사용한다.
  - `172.16.0.0` ~ `172.31.255.255` (`172.16.0.0/12`): AWS 기본 VPC의 영역에 사용된다.
  - `192.168.0.0` ~ `192.168.255.255` (`192.168.0.0/16`): 홈 네트워크에 사용한다.
- 위에서 나열한 주소 이외에는 전부 인터넷에서 공용으로 사용된다.

---

### Default VPC

- 모든 새 AWS 계정에는 기본 VPC가 있다.
- 서브넷이 지정되지 않은 경우 새 EC2 인스턴스가 기본 VPC로 시작된다.
- 기본 VPC에는 인터넷 연결이 있고 내부의 모든 EC2 인스턴스에는 Public IPv4 주소가 있다.
- Public 및 Private IPv4 DNS 이름도 얻는다.

---

### VPC

- VPC는 Virtual Private Cloud의 약자다.
- AWS 리전에는 여러 개의 VPC가 있을 수 있다. (리전당 최대 5개 - 소프트 제한)
- 각 CIDR에 대해 VPC당 CIDR는 5개다.
  - 최소 크기는 `/28`로 16개의 IP 주소를 가진다.
  - 최대 크기는 `/16`으로 65,536개의 IP 주소를 가진다.
- VPC는 Private이므로 Private IPv4 범위만 허용된다.
  - `10.0.0.0` ~ `10.255.255.255` (`10.0.0.0/8`)
  - `172.16.0.0` ~ `172.31.255.255` (`172.16.0.0/12`)
  - `192.168.0.0` ~ `192.168.255.255` (`192.168.0.0/16`)
- VPC CIDR는 다른 네트워크(예. 기업)와 겹쳐서는 안된다.

- 리전에 VPC가 생성된 모습은 아래와 같다.

![3-state-of-hands-on.png](images%2F3-state-of-hands-on.png)

#### Subnet (IPv4)

- AWS는 각 서브넷에 5개의 IP 주소(처음 4개, 마지막 1개)를 예약한다.
- 이 5개의 IP 주소는 사용할 수 없으며 EC2 인스턴스에 할당할 수 없다.
- 예를 들어, CIDR 블록이 `10.0.0.0/24`인 경우 예약된 IP 주소는 아래와 같다.
  - `10.0.0.0`: 네트워크 주소
  - `10.0.0.1`: AWS에서 VPC 라우터 용으로 예약한다.
  - `10.0.0.2`: Amazon 제공 DNS에 매핑하기 위해 AWS에서 예약한다.
  - `10.0.0.3`: 향후 사용을 위해 AWS에서 예약한다.
  - `10.0.0.255`: 네트워크 브로드캐스트 주소다. AWS는 VPC에서 브로드캐스트를 지원하지 않으므로 주소가 예약되어 있다.
- **만약 EC2 인스턴스에 29개의 IP 주소가 필요한 경우**
  - 크기가 `/27`인 서브넷은 선택할 수 없다. (IP 주소 32개, 32 - 5 = 27 < 29)
  - 크기가 `/26`인 서브넷을 선택해야 한다. (IP 주소 64개, 64 - 5 = 59 > 29)

- VPC에 Public 서브넷과 Private 서브넷이 추가된 모습은 아래와 같다.

![4-adding-subnets.png](images%2F4-adding-subnets.png)

#### Internet Gateway (IGW)

- VPC의 리소스(예. EC2 인스턴스)가 인터넷에 연결되도록 허용한다.
- 수평으로 확장되며 가용성이 높고 중복된다.
- VPC와 별도로 생성되어야 한다.
- 하나의 VPC는 하나의 IGW에만 연결할 수 있으며, 그 반대의 경우도 마찬가지다.

![5-adding-internet-gateway.png](images%2F5-adding-internet-gateway.png)

- Internet Gateway 자체로는 인터넷 액세스를 허용하지 않으며, 아래의 이미지와 같이 경로 테이블(Route Table)도 편집해야 한다.

![6-editing-route-table.png](images%2F6-editing-route-table.png)

#### Bastion Host

- Bastion Host를 사용하여 Private EC2 인스턴스에 SSH로 접속할 수 있다.
- Bastion은 Public 서비넷에 있으며 다른 모든 Private 서브넷에 연결된다.
- Bastion Host 보안 그룹은 제한된 CIDR(예. 회사의 Public CIDR)에서 포트 22를 통한 인터넷으로부터의 인바운드를 허용해야 한다.
- EC2 인스턴스의 보안 그룹은 Bastion Host의 보안 그룹 또는 Bastion 호스트의 Private IP를 허용해야 한다.

![7-bastion-host.png](images%2F7-bastion-host.png)

#### NAT Instance

- NAT는 "Network Address Translation"의 약자다.
- Private 서브넷의 EC2 인스턴스가 인터넷에 연결되도록 허용한다.
- Public 서브넷에서 시작되어야 한다.
- EC2 설정을 비활성화해야 한다. Source / Destination 확인
- 탄력적 IP가 연결되어 있어야 한다.
- Route Table은 Private 서브넷에서 NAT 인스턴스로 트래픽을 라우팅하도록 구성되어야 한다.

![8-nat-instance.png](images%2F8-nat-instance.png)

- NAT 인스턴스가 추가된 모습은 아래와 같다.

![9-adding-nat-instance.png](images%2F9-adding-nat-instance.png)

- NAT 인스턴스를 사용하기 위해 사전 구성된(Pre-configured) Amazon Linux AMI를 사용할 수 있다.
  - 2020년 12월 31일에 표준 지원이 종료되었다.
- 가용성이 높지 않고 기본적으로 복원력이 뛰어난 설정이 아니다.
  - 다중 AZ와 탄력성을 위해서 user-data 스크립트를 통해 ASG를 생성해야 한다.
- 인터넷 트래픽 대역폭은 EC2 인스턴스 유형에 따라 다르다.
- 보안 그룹 및 규칙을 관리해야 한다.
  - Inbound
    - Private 서브넷에서 들어오는 HTTP/HTTPS 트래픽을 허용해야 한다.
    - 홈 네트워크에서 SSH를 허용해야 한다. (Internet Gateway를 통해 액세스가 제공)
  - Outbound
    - 인터넷에 대한 HTTP/HTTPS 트래픽을 허용해야 한다.

#### NAT Gateway

- AWS를 통해 관리되는 NAT로, 더 높은 대역폭, 고가용성을 제공하며 사용자가 관리할 필요가 없다.
- 사용량 및 대역폭에 따라 시간당 비용을 지불하면 된다.
- NATGW는 특정 가용 영역에 생성되며 Elastic IP(EIP)를 사용한다.
- 동일한 서브넷의 EC2 인스턴스에서는 사용할 수 없으며, 다른 서브넷에서 사용해야 한다.
- IGW를 필요로 한다. (Private Subnet -> NATGW -> IGW)
- 5Gbps의 대역폭을 제공하며 최대 100Gbps까지 자동으로 확장할 수 있다.
- 관리할 보안 그룹이 없거나 필요하지 않다.

![10-adding-nat-gateway.png](images%2F10-adding-nat-gateway.png)

- NAT Gateway는 단일 가용 영역 내에서 복원력이 뛰어나다.
- 내결함성을 위해 여러 AZ에 여러 NAT Gateway를 생성해야 한다.
- 아래와 같이 다중 NAT Gateway를 설치한 경우, AZ가 중단되더라도 NAT가 필요하지 않으므로 교차 AZ 장애 조치가 필요하지 않다.

![11-nat-gateway-high-availability.png](images%2F11-nat-gateway-high-availability.png)

- NAT Gateway와 NAT Instance는 아래와 같은 차이를 가지고 있다.

![12-nat-gateway-vs-nat-instance.png](images%2F12-nat-gateway-vs-nat-instance.png)

---

### DNS Resolution

- **DNS Resolution** (`enableDnsSupport`)
  - Route 53 Resolver 서버의 DNS Resolver가 VPC에 대해 지원되는지 결정한다.
  - True (기본값): `169.254.169.253` 또는 VPC IPv4 네트워크 범위 + 2의 기본에 있는 예약된 IP 주소에서 Amazon Resolver DNS 서버를 쿼리한다.

![13-dns-resolution-in-vpc.png](images%2F13-dns-resolution-in-vpc.png)

- **DNS Hostnames** (`enableDnsHostname`)
  - 기본으로 생성되는 VPC에는 기본적으로 활성화되어 있고, 새롭게 생성하는 VPC들에는 기본적으로 비활성화되어 있다.
  - `enableDnsSupport=true`가 아닌 이상 아무 작업도 수행되지 않는다.
  - True인 경우 Public IPv4가 있는 경우, EC2 인스턴스에 Public 호스트 이름을 할당한다.

![14-dns-resolution-in-vpc-hostname.png](images%2F14-dns-resolution-in-vpc-hostname.png)

- Route 53의 Private 호스팅 영역에서 사용자 지정 DNS 도메인 이름을 사용하는 경우, `enableDnsSupport`와 `enableDnsHostname` 모두 True 설정해야 한다.

![15-dns-resolution-in-vpc-private-hosted-zone.png](images%2F15-dns-resolution-in-vpc-private-hosted-zone.png)

---

### NACL

- NACL은 "Network Access Control List"의 약자다.
- NACL은 서브넷과 주고받는 트래픽을 제어하는 방화벽과 같다.
- 서브넷당 하나의 NACL을 가지고 있으며, 새로운 서브넷에는 기본 NACL이 할당된다.
- NACL 규칙을 정의한다.
  - 규칙에는 숫자(1 ~ 32766)가 있으며 숫자가 낮을수록 우선순위가 높다.
  - 첫 번째 규칙이 일치로 결정이 내려진다.
  - 예를 들어, `#100 ALLOW 10.0.0.10/32`와 `#200 ALLOW 10.0.0.10/32`를 정의하면 100이 200보다 우선순위가 높으므로 해당 IP 주소가 허용된다.
  - 마지막 규칙은 별표(`*`)이며 일치하는 규칙이 없는 경우 모든 요청을 거부한다.
  - AWS에서는 우선순위를 100 단위로 추가할 것을 권장한다.
- 새로 생성된 NACL은 모든 것을 거부한다.
- NACL은 서브넷 수준에서 특정 IP 주소를 차단하는 효과적인 방법이다.

![16-security-group-nacl.png](images%2F16-security-group-nacl.png)

- 보안 그룹은 상태를 저장하기 때문에(Stateful) Inbound만 허용되어 있더라도 요청에 대한 응답이 가능하다.
- NACL은 상태를 저장하지 않기 때문에(Stateless) Inbound와 Outbound 모두 허용되어야 요청에 대한 응답이 가능하다.
- 서브넷 단위로 NACL이 추가된 모습은 아래와 같다.

![17-adding-nacl.png](images%2F17-adding-nacl.png)

#### Default NACL

- 기본으로 추가된 NACL은 연결된 서브넷과의 모든 Inbound/Outbound를 허용한다.
- 기본 NACL을 수정하는 대신, 사용자 정의 NACL을 생성하는 것이 권장된다.

![17-adding-nacl.png](images%2F17-adding-nacl.png)

#### Ephemeral Port

- 두 엔드포인트가 연결을 설정하려면 포트를 사용해야 한다.
- 클라이언트는 정의된 포트에 연결하고 임시 포트(Ephemeral Port)에서 응답을 기대한다.
- 운영 체제마다 서로 다른 포트 범위를 사용한다.
  - IANA & MS Windows 10 -> 49152 ~ 65535
  - 대부분의 리눅스 커널 -> 32768 ~ 60999

![18-ephemeral-port.png](images%2F18-ephemeral-port.png)

#### NACL & Ephemeral Port 통합

![18-ephemeral-port.png](images%2F18-ephemeral-port.png)

- Private 서브넷과 Public 서브넷이 있다면 각 서브넷에 NACL이 하나씩 연결된다.
- 첫 번째 NACL을 고려하면 데이터베이스 서브넷 CIDR 포트 3306에서 아웃바운드 3306 TCP를 허용해야 한다.
- 데이터베이스 관점에서 DB NACL은 Web CIDR에서 인바운드 3306 TCP를 허용해야 한다.
- 데이터베이스가 클라이언트에게 요청을 다시 보낼 때, 클라이언트에 Ephemeral 포트가 있다.
- DB NACL은 포트에서 아웃바운드 TCP를 1024 ~ 65535까지의 범위를 허용해야 한다.
- Web NACL은 DB 서브넷 CIDR의 범위 내의 포트에서 인바운드 TCP를 허용해야 한다.

![19-nacl-rule-each-target-subnet-cidr.png](images%2F19-nacl-rule-each-target-subnet-cidr.png)

- 여러 개의 NACL과 서브넷이 있을 경우 각각의 NACL 조합이 NACL 내에서 허용돼야 한다.
- CIDR을 사용하니 각 서브넷마다 고유의 CIDR이 있고 NACL에 각각 서브넷의 CIDR을 추가해야 한다.

#### Security Group vs NACL

- Security Group과 NACL의 차이점은 아래의 표와 같다.

![20-security-group-vs-nacls.png](images%2F20-security-group-vs-nacls.png)

---

### Reachability Analyzer

- VPC에 있는 두 엔드포인트 간의 네트워크 연결 문제를 해결하는 네트워크 진단 도구다.
- 네트워크 구성 모델을 구축한 후 이러한 구성을 기반으로 연결 가능성을 확인한다. (패킷을 보내지 않음)
- 목적지가 아래와 같은 경우
  - 연결 가능(Reachable): 가상 네트워크 경로의 홉별 세부 정보를 생성한다.
  - 연결 불가능(Not Reachable): 차단 구성 요소(예. SG, NACL, Route Table 등)를 식별한다.
- 사용 사례: 연결 문제 해결, 의도한대로 네트워크 구성이 되었는지 확인.

![21-reachability-analyzer.png](images%2F21-reachability-analyzer.png)

---

### VPC Peering

- AWS 네트워크를 사용하여 두 개의 VPC를 비공개로 연결한다.
- 동일한 네트워크에 있는 것처럼 동작하도록 만든다.
- CIDR이 중복되어서는 안된다.
- VPC Peering은 연결이 전파되지 않는다. 서로 통신해야 하는 VPC들이 있다면 전부 연결을 따로 해주어야 한다.
- **EC2 인스턴스가 서로 통신할 수 있도록 각 VPC 서브넷의 라우팅 테이블에 업데이트**를 해야 한다.

![22-vpc-peering.png](images%2F22-vpc-peering.png)

- 서로 다른 AWS 계정/리전의 VPC 간에 VPC 피어링 연결을 생성할 수 있다.
- 피어링된 VPC에서 보안 그룹을 참조할 수 있다. (교차 계쩡 작동 - 동일한 리전)

![23-vpc-peering-good-to-know.png](images%2F23-vpc-peering-good-to-know.png)

- VPC Peering 연결이 추가된 모습은 아래와 같다.

![24-adding-vpc-peering.png](images%2F24-adding-vpc-peering.png)

---

### VPC Endpoint (AWS PrivateLink)

- 모든 AWS 서비스는 공개적으로 노출된다. (Public URL)
- VPC 엔드포인트 (AWS PrivateLink 제공)를 사용하면 Public 인터넷 대신 Private 네트워크를 사용하여 AWS 서비스에 연결할 수 있다.
- 중복되고 수평으로 확장된다.
- AWS 서비스에 액세스하기 위해 IGW, NATGW 등이 필요하지 않다.
- 문제가 있는 경우 아래의 항목을 확인할 필요가 있다.
  - VPC에서 DNS 설정 확인
  - Route Table 확인

![25-vpc-endpoint-aws-privatelink.png](images%2F25-vpc-endpoint-aws-privatelink.png)

#### Endpoint 유형

- **Interface Endpoint (PrivateLink 제공)**
  - ENI(Private IP)를 진입점으로 프로비저닝한다. (보안 그룹을 연결해야 함)
  - 대부분의 AWS 서비스를 지원한다.
  - 시간당 비용과 처리된 데이터 GB당 비용을 지불해야 한다.
- **Gateway Endpoint**
  - 게이트웨이를 피로비저닝하고 라우팅 테이블에서 대상으로 사용해야 한다. (보안 그룹을 사용하지 않음)
  - S3와 DynamoDB 모두 지원한다.
  - 무료로 사용할 수 있다.

![26-type-of-endpoint.png](images%2F26-type-of-endpoint.png)

- Gateway는 시험에서 항상 선호될 가능성이 높다.
- Gateway의 경우 무료로 사용할 수 있지만, Interface의 경우 비용을 지불해야 한다.
- Interface 엔드포인트는 온프레미스(Site-to-Site VPN or Direct Connect), 다른 VPC 또는 다른 리전에서 액세스가 필요한 기본 액세스다.

![27-gateway-interface-endpoint-s3.png](images%2F27-gateway-interface-endpoint-s3.png)

- DynamoDB는 AWS의 공용 서비스다.
- 옵션1: 공용 인터넷에서 액세스
  - Lambda는 VPC에 있기 때문에 Public 서브넷의 NAT 게이트웨이와 인터넷 게이트웨이가 필요하다.
- 옵션2: 비공개 VPC 네트워크에서 액세스 - 권장되며 무료
  - DynamoDB용 VPC Gateway 엔드포인트 배포
  - 경로 테이블 변경

![28-lambda-vpc-accessing-dynamodb.png](images%2F28-lambda-vpc-accessing-dynamodb.png)

---

### VPC Flow Logs

- 인터페이스로 들어오는 IP 트래픽에 대한 정보를 캡처한다.
  - VPC Flow Logs
  - Subnet Flow Logs
  - Elastic Network Interface (ENI) Flow Logs
- 연결 문제를 모니터링하고 해결하는 데 도움이 된다.
- Flow Logs 데이터는 S3, CloudWatch Logs 및 Kinesis Data Firehose로 이동할 수 있다.
- ELB, RDS, ElastiCache, Redshift, WorkSpaces, NATGW, Transit Gateway 등 AWS 관리형 인터페이스에서도 네트워크 정보를 캡처한다.
- VPC에 VPC Flow Logs가 추가된 모습은 아래와 같다.

![29-vpc-flow-logs.png](images%2F29-vpc-flow-logs.png)

#### Syntax

![30-vpc-flow-logs-syntax.png](images%2F30-vpc-flow-logs-syntax.png)

- `srcaddr` & `dstaddr`: 문제가 있는 IP 식별에 도움이 된다.
- `scrport` & `dstport`: 문제가 있는 포트를 식별하는 데 도움이 된다.
- Action: 보안 그룹/NACL로 인한 요청의 성공 또는 실패
- 사용 패턴이나 악의적인 행위에 대한 분석에 사용될 수 있다.
- S3의 Athena 또는 CloudWatch Logs Insights를 사용하여 VPC Flow Logs를 쿼리할 수 있따.

#### Troubleshoot

- Incoming Request
  - 인바운드 거부: NACL 또는 SG
  - 인바운드 승인, 아웃바운드 거부: NACL

![31-vpc-flow-logs-incoming.png](images%2F31-vpc-flow-logs-incoming.png)

- Outgoing Request
  - 아웃바운드 거부: NACL 또는 SG
  - 인바운드 승인, 아웃바운드 거부: NACL

![32-vpc-flow-logs-outgoing.png](images%2F32-vpc-flow-logs-outgoing.png)

- VPC Flow Logs는 아래와 같이 여러 아키텍처로 사용될 수 있다.

![33-vpc-flow-logs-architectures.png](images%2F33-vpc-flow-logs-architectures.png)

---

### AWS Site-to-Site VPN

- **Virtual Private Gateway (VGW)**
  - VPN 연결의 AWS 측에 있는 VPN 집중 장치다.
  - VGW가 생성되어 Site-to-Site VPN 연결을 생성하려는 VPC에 연결된다.
  - ASN(자율 시스템 번호)을 사용자 정의할 수 있는 가능성이 있다.
- **Customer Gateway (CGW)**
  - VPN 연결을 통해 고객 측의 소프트웨어 애플리케이션 또는 물리적 장치다.

![34-site-to-site-vpn-connection.png](images%2F34-site-to-site-vpn-connection.png)

- **Customer Gateway Device (On-premises)**
  - 사용할 IP 주소는 무엇인가
    - 고객 게이트웨이 디바이스에 대한 Public 인터넷 라우팅 가능 IP 주소
    - NAT 통과(NAT-T)가 활성화된 NAT 장치 뒤에 있는 경우 NAT 장치의 Public IP를 사용한다.
- **중요 단계**: 서브넷과 연결된 라우팅 테이블에서 가상 프라이빗 게이트웨이에 대한 경로 전파를 활성화한다.
- 온프레미스에서 EC2 인스턴스를 `ping`해야 하는 경우 보안 그룹의 인바운드에 ICMP 프로토콜을 추가해야 한다.

- VPC에 Site-to-Site VPN이 추가된 모습은 아래와 같다.

![35-adding-site-to-site-vpn.png](images%2F35-adding-site-to-site-vpn.png)

---

### AWS VPN CloudHub

- VPN 연결이 여러 개인 경우 여러 사이트 간 보안 통신을 제공하낟.
- 서로 다른 위치 간의 기본 또는 보조 네트워크 연결을 위한 저렴한 hub-and-spoke 모델을 제공한다. (VPN만 해당)
- VPN 연결이므로 Public 인터넷을 사용한다.
- 동일한 VGW에서 여러 VPN 연결을 설정하고 연결하고 동적 라우팅을 설정하고 라우팅 테이블을 구성한다.

![36-aws-vpn-cloud-hub.png](images%2F36-aws-vpn-cloud-hub.png)

---

### Direct Connect Gateway

- 여러 다른 리전(동일 계정)에 있는 하나 이상의 VPC에 Direct Connect를 설정하려면 "Direct Connect Gateway"를 사용해야 한다.

![37-direct-connect-gateway.png](images%2F37-direct-connect-gateway.png)

#### Connection 유형

- Dedicated Connection: 1Gbps, 10Gbps, 100Gbps 용량
  - 고객 전용 물리적 이더넷 포트
  - 먼저 AWS에 요청한 후 AWS Direct Connect 파트너가 완료한다.
- Hosted Connection: 50Mbps, 500Mbps, 10Gbps
  - 연결 요청은 AWS Direct Connect 파트너를 통해 이루어진다.
  - 필요에 따라 용량을 추가하거나 제거할 수 있다.
  - 일부 AWS Direct Connect 파트너에서 1, 2, 5, 10Gbps 사용 가능하다.
- 새로운 연결을 설정하는 데 리드 타임이 1개월 이상 걸리는 경우가 많다.

#### Encryption

- 전송 중인 데이터는 암호화되지 않지만 비공개다.
- AWS Direct Connect + VPN은 IPsec으로 암호화된 Private 연결을 제공한다.
  - 추가 보안 수준에 적합하지만 적용하기가 약간 더 복잡하다.

![38-direct-connect-encryption.png](images%2F38-direct-connect-encryption.png)

#### Resiliency

- 다중 위치에 하나의 연결을 생성하여 높은 탄력성을 유지할 수 있다.

![39-direct-connect-high-resiliency.png](images%2F39-direct-connect-high-resiliency.png)

- 다중 위치에 다중 연결을 생성하여 탄력성을 최대로 높일 수 있다.

![40-direct-connect-maximum-resiliency.png](images%2F40-direct-connect-maximum-resiliency.png)

- Direct Connect 연결에 실패하는 경우 백업용으로 Direct Connect를 사용할 수 있지만, Site-to-Site VPN 연결을 사용할 수도 있다.

![41-site-to-site-vpn-as-backup.png](images%2F41-site-to-site-vpn-as-backup.png)

#### VPC 간 서비스 노출

- VPC 간 서비스를 노출하는 방법은 두 가지가 있다.
- Option 1: Public으로 설정한다.
  - Public World Wide Web(WWW)를 통과한다.
  - 접근 관리가 어렵다.

![42-expose-service-public.png](images%2F42-expose-service-public.png)

- Option 2: VPC 피어링을 생성한다.
  - 많은 피어링 관계를 생성해야 한다.
  - 전체 네트워크를 개방한다.

![43-expose-service-vpc-peering.png](images%2F43-expose-service-vpc-peering.png)

---

### AWS PrivateLink (VPC Endpoint Service)

- 수천 개의 VPC(자체 또는 다른 계정)에 서비스를 노출하는 가장 안전하고 확장 가능한 방법이다.
- VPC 피어링, 인터넷 게이트웨이, NAT, 라우팅 테이블이 필요하지 않다.
- 네트워크 로드 밸런서(서비스 VPC)와 ENI(고객 VPC) 또는 GWLB가 필요하다.
- NLB가 여러 AZ에 있고 ENI가 여러 AZ에 있는 경우 솔루션은 내결함성이 있다.

![44-vpc-privatelink.png](images%2F44-vpc-privatelink.png)

#### PrivateLink & ECS

![45-privatelink-ecs.png](images%2F45-privatelink-ecs.png)

- ECS와 함께 PrivateLink를 사용하는 예시이다.
- 응용 프로그램을 노출하기 위해서 Network Load Balancer가 필요하다.
- ALB는 NLB의 타깃이 되고 다른 VPC에게는 PrivateLink에 직접 ENI가 될 수 있다.

---

### AWS ClassicLink

- EC2-Classic: 인스턴스는 다른 고객과 공유되는 단일 네트워크에서 실행된다.
- Amazon VPC: 인스턴스는 AWS 계정과 논리적으로 격리되어 실행된다.
- ClassicLink를 사용하면 EC2-Classic 인스턴스를 계정의 VPC에 연결할 수 있다.
  - 보안 그룹을 연결해야 한다.
  - Private IPv4 주소를 사용하여 통신 가능하다.
  - Public IPv4 주소 또는 탄력적 IP 주소를 사용할 필요가 없다.

---

### Transit Gateway

- **수천 개의 VPC와 온프레미스 간의 전이적 피어링을 위한 hub-and-spoke(Star) 연결**이다.
- 지역 리소스, 지역 간 작업이 가능하다.
- RAM(Resource Access Manager)를 사용하여 교차 계정 공유를 사용할 수 있다.
- 여러 지역에 걸쳐 Transit Gateway를 피어링할 수 있다.
- 라우팅 테이블: VPC가 다른 VPC와 통신할 수 있는 제한이 있다.
- Direct Connect Gateway, VPN 연결과 함께 작동한다.
- IP Multicast를 지원하지만, 다른 AWS 서비스에서는 지원되지 않는다.

![46-transit-gateway.png](images%2F46-transit-gateway.png)

#### Site-to-Site VPN ECMP

- ECMP는 "Equal-cost multi-path routing"의 약자로 "동일 비용 다중 경로 라우팅"을 의미한다.
- 여러 최상의 경로를 통해 패킷을 전달할 수 있는 라우팅 전략이다.
- 사용 사례: **여러 Site-to-Site VPN 연결을 생성하여 AWS 연결 대역폭을 늘린다.**

![48-transit-gateway-throughput-ecmp.png](images%2F48-transit-gateway-throughput-ecmp.png)

- VPN에서 가상 사설 게이트웨이로 가면 하나의 터널이 생성된다. VPN 연결은 두 개의 터널로 이루어져 있다.
- 하나의 VPC에 하나의 연결이며 최대 처리량은 1.25Gbps다.
- VPN을 Transit Gateway로 사용한다면 하나의 Site-to-Site VPN을 여러 VPC로 연결해야 한다.
- 하나의 Site-to-Site VPN 연결은 2개의 터널을 사용하고 ECMP 덕분에 5Gbps가 절약된다.
- 또한 2~3개의 Transit Gateway에 Site-to-Site VPN 연결을 추가하여, ECMP를 통해 처리량을 2~3배로 늘릴 수 있따.
- 설정할 때는 Transit Gateway를 통과하는 데이터당 GB 비용을 지불해야 한다. 성능 최적화에 따른 비용 추가다.

#### 계정 간 공유

![49-transit-gateway-share-multiple-account.png](images%2F49-transit-gateway-share-multiple-account.png)

- AWS Resource Access Manager(ACM)를 사용하여 Transit Gateway를 다른 계정과 공유할 수 있다.
- 회사의 데이터 센터와 Direct Connect 연결 지점을 직접 연결한다.
- Transit Gateway를 설치해 두 VPC 계정에 각각 연결한다.

---

### VPC Traffic Mirroring

- VPC의 네트워크 트래픽을 캡처하고 검사할 수 있다.
- 관리하는 보안 어플라이언스로 트래픽을 라우팅한다.
- 트래픽을 캡처한다.
  - Source: ENI
  - Target: ENI 또는 Network Load Balancer
- 모든 패킷을 캡처하거나 관심 있는 패킷을 캡처한다. 선택적으로 패킷을 자를 수 있다.
- 소스와 대상은 동일한 VPC에 있을 수도 있고 다른 VPC에 있을 수도 있다. (VPC Peering)
- 사용 사례: 콘텐츠 검사, 위협 모니터링, 문제 해결 등..

![50-vpc-traffic-mirroring.png](images%2F50-vpc-traffic-mirroring.png)

---

### IPv6

- VPC 및 서브넷에 대하 IPv4를 비활성화할 수 없다.
- IPv6(Public IP 주소)를 활성화하여 듀얼 스택 모드에서 작동할 수 있다.
- EC2 인스턴스에는 최소한 Private 내부 IPv4와 Public IPv6가 제공된다.
- Internet Gateway를 통해 IPv4 또는 IPv6를 사용하여 인터넷과 통신할 수 있다.

![51-Ipv6-in-vpc.png](images%2F51-Ipv6-in-vpc.png)

#### Troubleshooting

- **VPC 및 서브넷에 대해 IPv4를 비활성화할 수 없다**.
- 서브넷에서 EC2 인스턴스를 시작할 수 없는 경우 아래의 항목을 확인해야 한다.
  - IPv6를 획득하지 못해서는 아니다. (사용가능한 범위가 넓다)
  - 서브넷에 사용 가능한 IPv4가 없을 가능성이 있다.
- 문제 해결을 위하여 서브넷에 IPv4 CIDR 블록을 추가한다.

![52-ipv6-troubleshooting.png](images%2F52-ipv6-troubleshooting.png)

---

### Egress-only Internet Gateway

- IPv6에만 사용된다. NAT Gateway와 유사하며 IPv6 전용으로 사용된다.
- 인터넷이 인스턴스에 대한 IPv6 연결을 시작하는 것을 방지하면서 IPv6를 통한 VPC 아웃바운드 연결을 허용한다.
- 라우팅 테이블을 업데이트해야 한다.

![53-egress-only-internet-gateway.png](images%2F53-egress-only-internet-gateway.png)

#### IPv6 Routing

![54-ipv6-routing.png](images%2F54-ipv6-routing.png)

- Public 서브넷과 Private 서브넷이 있고 둘 다 IPv4와 IPv6가 있다.
- 인터넷에 액세스하는 웹 서버를 고려할 때, IPv4와 인터넷 게이트웨이를 통할 수 있다.
- Public 서브넷의 라우트 테이블은 로컬 IPv4와 로컬 IPv6 트래픽을 위한 로컬 라인이 된다.
- Private 서브넷의 경우에도 IPv4와 IPv6를 갖고 있다.
- Private 서브넷에서 IPv4가 인터넷에 연결되기 위해서는 NAT 게이트웨이를 사용해야 한다.
- 반면, IPv6가 인터넷에 연결되기 위해서는 Egress-only Internet Gateway를 사용할 수 있다.

---

### VPC Summary

- **CIDR**: IP 범위
- **VPC**: Virtual Private Cloud => IPv4 및 IPv6 CIDR 목록을 정의한다.
- **Subnet**: AZ에 연결되어 CIDR을 정의한다.
- **Internet Gateway**: VPC 수준에서 IPv4 및 IPv6 인터넷 액세스를 제공한다.
- **Route Table**: 서브넷에서 IGW, VPC 피어링 연결, VPC 엔드포인트로의 경로를 추가하려면 편집해야 한다.
- **Bastion Host**: SSH 연결에 대한 Public EC2 인스턴스로, Private 서브넷의 EC2 인스턴스에 대한 SSH 연결이 있다.
- **NAT Instance**: Private 서브넷의 EC2 인스턴스에 대한 인터넷 액세스를 제공한다. 
  이전 버전은 Public 서브넷에 설정해야 하며 소스/대상 확인 플래그를 비활성화해야 한다.
- **NAT Gateway**: AWS에서 관리하며 Private EC2 인스턴스, IPv4에만 확장 가능한 인터넷 액세스를 제공한다.
- **Private DNS + Route 53**: DNS 확인 + DNS 호스트 이름(VPC) 활성화
- **NACL**: 인바운드 및 아웃바운드에 대한 Stateless, 서브넷 규칙, 임시 포트를 제공한다.
- **Security Group**: Stateful, EC2 인스턴스 수준에서 작동한다.
- **Reachability Analyzer**: AWS 리소스 간의 네트워크 연결 테스트를 수행한다.
- **VPC Peering**: 겹치지 않는 CIDR, 비전이성을 사용하여 두 개의 VPC를 연결한다.
- **VPC Endpoint**: VPC 내의 AWS 서비스(S3, DynamoDB, CloudFormation, SSM)에 대한 비공개 액세스를 제공한다.
- **VPC Flow Logs**: ACCEPT 및 REJECT 트래픽에 대해 VPC/서브넷/ENI 수준에서 설정할 수 있으며 공격 식별을 돕고 Athena 또는 CloudWatch Logs Insights를 사용하여 분석한다.
- **Site-to-Site VPN**: 데이터 센터에 Customer Gateway, VPC에 Virtual Private Gateway를 설치하여, Public 인터넷을 통해 Site-to-Site VPN을 설정한다.
- **AWS VPN CloudHub**: 사이트 연결을 위한 "hub-and-spoke" VPN 모델이다.
- **Direct Connect**: VPC에 Virtual Private Gateway를 설정하고 AWS Direct Connect 위치에 대한 Direct Private 연결을 설정한다.
- **Direct Connect Gateway**: 다양한 AWS 지역의 여러 VPC에 Direct Connect를 설정한다.
- **AWS PrivateLink / VPC Endpoint Service**:
  - 서비스 VPC에서 고객 VPC로 비공개로 서비스를 연결한다.
  - VPC 피어링, Public 인터넷, NAT 게이트웨이, 라우팅 테이블이 필요하지 않다.
  - Network Load Balancer 및 ENI와 함께 사용해야 한다.
- **ClassicLink**: EC2-Classic, EC2 인스턴스를 VPC에 비공개로 연결한다.
- **Transit Gateway**: VPC, VPN 및 DX를 위한 전이적 피어링을 연결한다.
- **Traffic Mirroring**: 추가 분석을 위해 ENI에서 네트워크 트래픽을 복사한다.
- **Egress-only Internet Gateway**: NAT 게이트웨이와 비슷하지만 IPv6 전용이다.

---

### Networking Cost

![55-networking-cost-simplified.png](images%2F55-networking-cost-simplified.png)

- 비용 절감 및 네트워크 성능 향상을 위해 Public IP 대신 Private IP를 사용한다.
- 최대 비용 절감을 위해 동일한 AZ를 사용한다. (고가용성 비용)

#### 송신 네트워크 비용 최소화

- Egress Traffic: 아웃바운드 트래픽(AWS에서 외부로)
- Ingress Traffic: 인바운드 트래픽(외부에서 AWS로, 일반적으로 무료)
- 비용을 최소화하기 위해 AWS 내에서 인터넷 트래픽을 최대한 유지하려고 노력해야 한다.
- 동일한 AWS 리전에 같은 위치에 있는 Direct Connect 위치로 송신 네트워크 비용이 절감된다.

![56-minimizing-egress-traffic-network-cost.png](images%2F56-minimizing-egress-traffic-network-cost.png)

#### S3 데이터 전송 비용 (미국 기준)

- S3 Ingress: 무료
- S3 to Internet: GB당 $0.09
- S3 Transfer Acceleration
  - 더 빠른 전송 시간(50% ~ 500% 향상)
  - 데이터 전송 추가 비용: GB당 $0.08
- S3 to CloudFront: GB당 $0.00
- CloudFront to Internet: GB당 $0.085 (S3보다 약간 저렴)
  - 캐싱 기능(낮은 지연 시간)
  - S3 요청 요금과 관련된 비용 절감(CloudFront를 사용하면 7배 더 저렴)
- S3 교차 지역 복제: GB당 $0.02

![57-s3-data-transfer-pricing.png](images%2F57-s3-data-transfer-pricing.png)

#### NAT Gateway & Gateway VPC Endpoint 가격 비교

![58-nat-gateway-gateway-vpc-endpoint.png](images%2F58-nat-gateway-gateway-vpc-endpoint.png)

- NAT Gateway 대신 VPC Endpoint를 사용하면 비용을 많이 낮출 수 있다.

---

### Network Firewall

- AWS에서 네트워크를 보호하기 위한 많은 서비스가 존재한다.
  - Network Access Control List (NACL)
  - Amazon VPC Security Group
  - AWS WAF (악의적인 요청으로부터 보호)
  - AWS Shield & AWS Shield Advanced
  - AWS Firewall Manager (계정 전체에서 관리)

- **AWS Network Firewall을 사용하면 전체 VPC를 정교한 방식으로 보호**할 수 있다.

![59-aws-network-firewall.png](images%2F59-aws-network-firewall.png)

- 전체 Amazon VPC를 보호한다.
- Layer 3에서 Layer 7까지 보호가 가능하다.
- 어떤 방향에서도 검사할 수 있다.
  - VPC to VPC 트래픽
  - 인터넷으로 Outbound 트래픽
  - 인터넷에서 Inbound 트래픽
  - Direct Connect & Site-to-Site VPN 양방향
- 내부적으로 AWS 네트워크 방화벽은 AWS 게이트웨이 로드 밸런서를 사용한다.
- AWS Firewall Manager를 통해 여러 계정에 걸쳐 규칙을 중앙에서 관리하여 여러 VPC에 적용할 수 있다.

#### Fine Grained Control

- 1000가지의 규칙을 지원한다.
  - IP 및 Port: 예를 들어, 10,000개의 IP를 필터링한다.
  - 프로토콜: 예를 들어, 아웃바운드 통신을 위한 SMB 프로토콜을 차단한다.
  - Stateful 도메인 목록 규칙 그룹: `*.mycorp.com` 또는 타사 소프트웨어 저장소에 대한 아웃바운드 트래픽만 허용한다.
  - 정규식을 사용한 일반 패턴 일치를 지원한다.
- **트래픽 필터링: 규칙과 일치하는 트래픽을 허용, 삭제 또는 경고**한다.
- 침입 방지 기능(Gateway Load Balancer와 같지만 모두 AWS에서 관리)을 통해 네트워크 위협으로부터 보호하기 위한 활성 흐름을 검사하낟.
- 규칙 일치 로그를 Amazon S3, CloudWatch Logs, Kinesis Data Firehose로 보낸다.

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [CIDR 연습 사이트](https://www.ipaddressguide.com/cidr)
- [NAT Gateway vs NAT Instance](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html)
- [NACL Ephemeral Port](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#nacl-ephemeral-ports)
- [Security Group vs NACL](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-records-examples.html)
- [Customer Gateway Device Test](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html#DevicesTested)