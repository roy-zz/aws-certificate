# Virtual Private Cloud

이번 장에서는 SAA를 준비하며 **Virtual Private Cloud의 약자인 VPC**에 대해서 알아보도록 한다.

---

### VPC Components Diagrams

- 아래는 VPC 컴포넌트들의 다이어그램이다.

![vpc-components-diagram.png](images%2FVPC%2Fvpc-components-diagram.png)

#### IPv4 CIDR 이해

- 클래스가 없는(Classless) 도메인 간의 라우팅 - IP 주소 할당 방법이다.
- Security Groups 규칙 및 AWS 네트워킹에서 일반적으로 사용된다.

![cidr-ipv4.png](images%2FVPC%2Fcidr-ipv4.png)

- IP 주소 범위를 정의하는 데 도움이 된다.
  - WW.XX.YY.ZZ/32 => 하나의 IP다.
  - 0.0.0.0/0 => 모든 IP다.
  - 192.168.0.0/26 => 192.168.0.0 ~ 192.168.0.63 총 64개의 IP 주소를 가진다.

- CIDR는 두 가지 구성 요소로 이루어져 있다.
- Base IP
  - 범위(XX.XX.XX.XX)에 포함된 IP를 나타낸다.
  - 예를 들어, 10.0.0.0, 192.168.0.0 등..
- Subnet Mask
  - IP에서 변경할 수 있는 비트 수를 정의한다.
  - 예를 들어, /0, /24, /32
  - 두 가지 형태를 취할 수 있다.
    - /8 <-> 255.0.0.0
    - /16 <-> 255.255.0.0
    - /24 <-> 255.255.255.0
    - /32 <-> 255.255.25.255

#### Subnet Maks의 이해

- 서브넷 마스크는 기본적으로 기본 IP의 일부가 기본 IP로부터 추가적인 다음 값을 가져올 수 있도록 한다.
- 아래는 여러가지 예시이다.

![subnet-mask.png](images%2FVPC%2Fsubnet-mask.png)

- 192.168.0.0/24 = 192.168.0.0 ~ 192.168.0.255 (256개의 IP)
- 192.168.0.0/16 = 192.168.0.0 ~ 192.168.255.255 (65,536개의 IP)
- 134.56.78.123/32 = 132.56.78.123 (1개의 IP)
- 0.0.0.0/0 = 모든 IP

#### Public vs Private IP (IPv4)

- The IANA(Internet Assgined Numbers Authority)는 개인(LAN) 및 공용(Internet) 주소의 사용을 위해 IPv4 주소의 특정 블록을 설정하였다.
- 개인(Private) IP는 특정 값만 허용할 수 있다.
  - 10.0.0.0 ~ 10.255.255.255 (10.0.0.0/8): 거대한 네트워크에 사용된다.
  - 172.16.0.0 ~ 172.31.255.255 (172.16.0.0/12): AWS의 기본 VPC 범위다.
  - 192.168.0.0 ~ 192.168.255.255 (192.168.0.0/16): 홈 네트워크에서 사용된다.
- 위에서 나열한 IP 주소를 제외하고는 모든 주소가 Public으로 공개된다.

#### 기본 VPC

- 모든 새로운 AWS 계정에는 기본 VPC가 있다.
- 서브넷이 지정되지 않은 경우, 새로운 EC2 인스턴스가 기본 VPC로 시작된다.
- 기본 VPC에는 인터넷 연결이 있으며 내부의 모든 EC2 인스턴스에는 공용(Public) IPv4 주소가 있다.
- Public 및 Private IPv4 DNS 이름도 있다.

---

### VPC in AWS (IPv4)

- VPC는 Virtual Private Cloud의 약자다.
- AWS Region에 여러 개의 VPC를 보유할 수 있으며 최대 5개까지 사용할 수 있다. 필요한 경우 요청을 통해 증설할 수 있다.
- VPC당 최대 CIDR은 각 CIDR에 대해서 5다.
  - 최소 크기는 /28로 총 16개의 IP를 가진다.
  - 최대 크기는 /16으로 총 65,536개의 IP를 가진다.
- VPC는 비공개이므로 Private IPv4 범위만 허용된다.
  - 10.0.0.0 ~ 10.255.255.255 (10.0.0.0/8)
  - 172.16.0.0 ~ 172.31.255.255 (172.16.0.0/12)
  - 192.168.0.0 ~ 192.168.255.255 (192.168.0.0/16)
- VPC CIDR는 다른 네트워크(예: 기업)와 중복되지 않아야 한다.
  
- 기본적으로 새로운 AWS 계정을 생성하면 아래와 같이 Region에 기본 VPC가 생성된다.

![default-vpc.png](images%2FVPC%2Fdefault-vpc.png)

- 기본 VPC에 필요에 따라 Public 서브넷과 Private 서브넷을 추가할 수 있다.

![default-vpc-adding-subnet.png](images%2FVPC%2Fdefault-vpc-adding-subnet.png)

#### Subnet (IPv4)

- AWS는 각 서브넷에 5개의 IP 주소(처음 4개 & 마지막 1개)를 예약한다.
- 5개의 IP주소는 사용할 수 없으며 EC2 인스턴스에 할당할 수 없다.
- 예를 들어, CIDR 블록이 10.0.0.0/24인 경우, 예약된 IP 주소는 아래와 같다.
  - 10.0.0.0 - Network Address
  - 10.0.0.1 - VPC 라우터 용으로 AWS에 예약
  - 10.0.0.2 - AWS에서 제공한 DNS에 매핑하기 위해 AWS에서 예약
  - 10.0.0.3 - 추후 사용을 위해 AWS에서 예약
  - 10.0.0.255 - Network Broadcast Address. AWS는 VPC에서 브로드캐스트를 지원하지 않으므로 해당 주소는 예약되어 있다.
- 만약 EC2 인스턴스에 대해 29개의 IP 주소가 필요한 경우:
  - /27 크기의 서브넷을 선택할 수 없다. (32개의 IP 주소, 32 - 5 = 27 < 29)
  - /26 크기의 서브넷을 선택해야 한다. (64개의 IP 주소, 64 - 5 = 59 > 29)

#### Internet Gateway (IGW)

- VPC의 리소스(예: EC2 인스턴스)가 인터넷에 연결되도록 허용한다.
- 수평적으로 확장되며 가용성이 높고 중복성이 있다.
- VPC와 별도로 생성해야 한다.
- 하나의 VPC는 하나의 IGW에만 연결할 수 있고 그 반대도 가능하다.
- "Internet Gateway" 자체에서 인터넷 액세스를 허용하지 않는다.
- 경로 테이블(Route Table)도 편집해야 한다.
- "Internet Gateway"가 추가된 형태는 아래와 같다.

![default-vpc-adding-internet-gateway.png](images%2FVPC%2Fdefault-vpc-adding-internet-gateway.png)

- 위에서 살펴본 것처럼 "Internet Gateway" 자체로는 인터넷 액세스를 허용하지 않는다.
- 인터넷 액세스를 할 수 있도록 경로 테이블(Route Table)을 추가한 형태는 아래와 같다.

![default-vpc-adding-route-table.png](images%2FVPC%2Fdefault-vpc-adding-route-table.png)

#### Bastion Hosts

- "Bastion Host"를 사용하여 개인 EC2 인스턴스에 SSH를 적용할 수 있다.
- "Bastion"은 다른 모든 Private 서브넷에 연결된 Public 서브넷에 있다.
- "Bastion Host Security Group"은 포트 22의 인터넷에서 제한된 CIDR(예: 회사의 Public CIDR)에서 인바운드를 허용해야 한다.
- EC2 인스턴스의 Security Group은 "Bastion Host"의 Security Group 또는 Bastion Host의 Private IP를 허용해야 한다.

![bastion-hosts.png](images%2FVPC%2Fbastion-hosts.png)

---

### NAT Instance 

- 이전 버전의 서비스지만 아직 시험에 출시되고 있다.
- "NAT"는 "Network Address Translation"의 약자다.
- Private 서브넷의 EC2 인스턴스가 인터넷에 연결되도록 허용한다.
- Public 서브넷에서 실행해야 한다.
- EC2 인스턴스 설정을 사용하지 않는다. "Source/Destination"
- 반드시 Elastic IP가 연결되어 있어야 한다.
- Private 서브넷에서 NAT 인스턴스로 트래픽을 라우팅하려면 Route Table을 구성해야 한다.

![nat-instance.png](images%2FVPC%2Fnat-instance.png)

- NAT 인스턴스가 추가된 형태는 아래와 같다.

![default-vpc-adding-nat-instance.png](images%2FVPC%2Fdefault-vpc-adding-nat-instance.png)

- 미리 구성된 Amazon Linux API를 사용할 수 있다.
  - 2020년 12월31일에 Standard 지원을 종료하였다.
- 기본적으로 고가용성/복원성이 지원되지 않는다.
  - 다중 AZ + 복원력이 있는 사용자 데이터 스크립트에서 ASG를 작성해야 한다.
- 인터넷 트래픽 대역폭은 EC2 인스턴스 유형에 따라 다르다.
- 반드시 Security Group과 Rule을 관리해야 한다.
  - Inbound
    - 개인 서브넷에서 오는 HTTP/HTTPS 트래픽을 허용해야 한다.
    - 홈 네트워크에서 SSH를 허용한다.(Internet Gateway를 통해 액세스 제공)
  - Outbound
    - 인터넷에 HTTP/HTTPS 트래픽을 허용한다.

---

### NAT Gateway

- AWS에서 관리하는 NAT, 높은 대역폭, 고가용성(HA)을 지원하며 관리할 필요가 없다.
- 사용량 및 대역폭에 대해 시간당 지불한다.
- NATGW는 특정 AZ에 생성되며, Elastic IP를 사용한다.
- 동일한 서브넷의 EC2 인스턴스에서 사용할 수 없으며, 다른 서브넷에서만 사용할 수 있다.
- IGW가 필요하다. (Private Subnet => NATGW => IGW)
- 5Gbps 대역폭을 제공하며 최대 45Gbps까지 자동 확장 가능하다.
- 관리할 Security Group이 없다.
  
- 기본 VPC에 "NAT Instance" 대신 "NAT Gateway"가 추가된 모습은 아래와 같다.

![default-vpc-adding-nat-gateway.png](images%2FVPC%2Fdefault-vpc-adding-nat-gateway.png)

- 단일 AZ 영역 내에서 탄력적인 "NAT Gateway"를 설정할 수 있다.
- 내결함성(fault-tolerance)을 위해 여러 AZ에 여러 "NAT Gateway"를 만들 수 있다.
- AZ가 다운되면 NAT가 필요 없기 때문에 AZ 간 failover가 필요하지 않다.

![nat-gateway-high-availability.png](images%2FVPC%2Fnat-gateway-high-availability.png)

#### NAT Gateway vs NAT Instance

- "NAT Gateway"와 "NAT Instance"의 차이는 아래의 이미지와 같다.

![nat-gateway-nat-instance.png](images%2FVPC%2Fnat-gateway-nat-instance.png)

#### Network Access Control List (NACL)

- NACL은 서브넷을 오가는 트래픽을 제어하는 방화벽과 같다.
- 서브넷당 하나의 NACL이 할당되며, 새로운 서브넷에는 기본 NACL이 할당된다.
- NACL 규칙은 아래와 같이 정의한다.
  - 규칙에는 숫자(1~32766)가 있으며, 우선 순위가 높고 숫자가 작다.
  - 첫 번째 규칙 일치가 결정을 주도한다.
  - 예를 들어, "100 ALLOW 10.0.0.10/32"과 "200 DENY 10.0.0.10/32"라는 설정이 있을 때, 100은 200보다 우선되므로 IP 주소가 허용된다.
  - 마지막 규칙은 별표(`*`)이며 규칙이 일치하지 않는 경우 요청을 거부한다.
  - AWS에서는 규칙을 100개씩 증분하여 추가할 것을 권장한다.
- 새로 생성된 NACL은 모든 것을 거부한다.
- NACL은 서브넷 수준에서 특정 IP 주소를 차단하는 훌륭한 방법이다.

- Security Group과 비교하여 살펴보았을 때, Security Group 이전에 NACL에서 먼저 필터링되는 것을 확인할 수 있다.

![security-group-nacls.png](images%2FVPC%2Fsecurity-group-nacls.png)

- Security Group과 NACL의 정확한 차이점은 아래의 표를 참고하면 된다.

![security-group-vs-nacl.png](images%2FVPC%2Fsecurity-group-vs-nacl.png)

- NACL이 추가된 형태는 아래와 같다.

![default-vpc-adding-nacls.png](images%2FVPC%2Fdefault-vpc-adding-nacls.png)

- 기본 NACL의 경우 연관된 서브넷으로 인바운드/아웃바운드되는 모든 것을 허용한다.
- 기본 NACL을 수정하지 않고, 새로운 NACL을 생성한다.

![default-nacl.png](images%2FVPC%2Fdefault-nacl.png)

#### 임시(Ephemeral) Ports

- 두 엔드포인트가 연결을 설정하려면 포트를 사용해야 한다.
- 클라이언트가 정의된 포트에 연결하면 사용 후 임시 포트에서 응답이 필요하다.
- 서로 다른 운영체제는 서로 다른 포트 범위를 사용한다.
  - IANA & MS Windows 10 -> 49152 ~ 65535
  - Many Linux Kernels -> 32768 ~ 60999

![ephemeral-ports.png](images%2FVPC%2Fephemeral-ports.png)

- NACL에 임시(Ephemeral) 포트가 적용된 형태는 아래와 같다.

![nacl-ephemeral-ports.png](images%2FVPC%2Fnacl-ephemeral-ports.png)

- 아래의 이미지와 같이 각 대상 서브넷의 CIDR에 대한 NACL 규칙을 생성할 수 있다.

![nacl-rules-each-target-subnet-cidr.png](images%2FVPC%2Fnacl-rules-each-target-subnet-cidr.png)

---

### VPC Peering

- AWS의 네트워크를 사용하여 두 개의 VPC를 Privately 연결할 수 있다.
- 동일한 네트워크에 있는 것처럼 행동하게 한다.
- 단, 두 VPC의 CIDR 영역이 중복되면 안된다.
- VPC Peering은 다른 연결로 전파되지 않는다. 서로 통신해야 하는 각 VPC에 대해 설정해야 한다.
- EC2 인스턴스가 서로 통신할 수 있도록 하려면 각 VPC의 서브넷에서 "Route Table"을 업데이트해야 한다.

![vpc-peering.png](images%2FVPC%2Fvpc-peering.png)

- 서로 다른 AWS 계정/지역의 VPC간에 VPC 피어링 연결을 생성할 수 있다.
- Peering될 VPC의 "Security Group"을 참조할 수 있다. (서로 다른 계정 - 동일 지역)

![vpc-peering-good-to-know.png](images%2FVPC%2Fvpc-peering-good-to-know.png)

- VPC Peering이 추가된 형태는 아래와 같다.

![default-vpc-adding-vpc-peering.png](images%2FVPC%2Fdefault-vpc-adding-vpc-peering.png)

---

### VPC Endpoints (AWS PrivateLink에 의해 제공)

- 모든 AWS 서비스는 공개되어 있다.(Public URL)
- "VPC Endpoint"를 사용하면 Public 인터넷을 사용하지 않고 전용 네트워크를 사용하여 AWS 서비스에 연결할 수 있다.
- 충분하게 수평적으로 확장이 가능하다.
- "VPC Endpoint"를 통해 IGW, NATGW를 통해 AWS 서비스에 액세스할 필요가 없어진다.
- 아래와 같은 문제가 발생할 수 있다.
  - VPC에서 DNS 설정을 결정한다.
  - Route Table을 확인해야 한다.

![vpc-endpoint.png](images%2FVPC%2Fvpc-endpoint.png)

- **Interface Endpoints (powered by PrivateLink)**
  - ENI(private IP address)를 진입점으로 브로비저닝한다. (보안 그룹을 연결해야 한다.)
  - 대부분의 AWS 서비스를 지원한다.
  - 시간당 요금 + 처리된 GB당 요금을 지불한다.
- **Gateway Endpoints**:
  - 게이트웨이를 프로비저닝하고 "Route Table"에서 대상으로 사용해야 한다. (보안 그룹을 사용하지 않음)
  - S3 및 DynamoDB를 모두 지원한다.
  - 무료로 사용할 수 있다.

![vpc-endpoint-type.png](images%2FVPC%2Fvpc-endpoint-type.png)

- 시험에서 게이트웨이 유형을 선호할 가능성이 높다.
- 게이트웨이 유형의 경우 비용이 무료이며, 인터페이스 유형의 경우 비용이 발생한다.
- 인터페이스 Endpoint는 On-Premise(Site to Site VPN 또는 Direct Connect), 다른 VPC 또는 다른 Region에서 액세스할 때 사용하는 것이 좋다.

![gateway-vs-interface.png](images%2FVPC%2Fgateway-vs-interface.png)

- DynamoDB는 AWS Public 서비스다.
- Option1: Public Internet을 통해서 액세스한다.
  - "Lambda"는 VPC에 있으므로 Public 서브넷 및 "Internet Gateway"에 "NAT Gateway"가 필요하다.
- Option2 (권장): Private VPC 네트워크에서 액세스
  - DynamoDB용 VPC Gateway Endpoint를 배포한다.
  - Route Table을 변경한다.

![lambda-vpc-accessing-dynamodb.png](images%2FVPC%2Flambda-vpc-accessing-dynamodb.png)

- VPC Endpoint를 추가한 형태는 아래와 같다.

![default-vpc-adding-vpc-endpoint.png](images%2FVPC%2Fdefault-vpc-adding-vpc-endpoint.png)

---

### VPC Flow Logs

- 인터페이스로 들어가는 IP 트래픽에 대한 정보를 캡처한다.
  - VPC Flow Logs
  - Subnet Flow Logs
  - ENI(Elastic Network Interface) Flow Logs
- 연결 문제를 모니터링하고 해결하는 데 도움이 된다.
- Flow Logs 데이터는 S3, CloudWatch Logs 및 Kinesis Data Firehose로 이동할 수 있다.
- ELB, RDS, ElastiCache, Redshift, WorkSpaces, NATGW, Transit Gateway등 AWS 관리 인터페이스어세도 네트워크 정보를 캡처한다.
  
- VPC Flow Flogs가 추가된 형태는 아래와 같다.

![default-vpc-adding-vpc-flow-logs.png](images%2FVPC%2Fdefault-vpc-adding-vpc-flow-logs.png)

#### VPC Flow Logs Syntax

![vpc-flow-logs-syntax.png](images%2FVPC%2Fvpc-flow-logs-syntax.png)

- **srcaddr * dstaddr**: 문제가 있는 IP를 식별하는 데 도움이 된다.
- **srcport * dstport**: 문제가 있는 포트를 식별하는 데 도움이 된다.
- **Action**: Security Group / NACL로 인한 요청 성공 또는 실패
- 사용 패턴이나 악의적인 행동에 대한 분석으로 사용될 수 있다.
- S3의 "Athena" 또는 "CloudWatch Log Insights"를 사용하여 "VPC Flow Logs"를 쿼리할 수 있다.
- "VPC Flow Logs"에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs- records-examples.html)를 참고한다.

- Security Group / NACL에 의해 발생한 오류를 해결하는 방법을 살펴본다.
- 아래의 이미지에서 Inbound는 NACL 또는 Security Group에 의해 차단되며, Outbound는 NACL에 의해 차단된다.

![flow-logs-troubleshoot-incoming.png](images%2FVPC%2Fflow-logs-troubleshoot-incoming.png)

- 아래의 이미지에서 Outbound는 NACL 또는 Security Group에 의해 차단되며, Inbound는 NACL에 의해 차단된다.

![flow-logs-troubleshoot-outbound.png](images%2FVPC%2Fflow-logs-troubleshoot-outbound.png)

- 아래는 많이 사용되는 "VPC Flow Logs" 아키텍처다.

![vpc-flow-logs-architectures.png](images%2FVPC%2Fvpc-flow-logs-architectures.png)

---

### AWS Site-to-Site VPN

- **Virtual Private Gateway(VGW)**
  - VPN 집중 장치이며, AWS에서 VPN 연결을 위해 사용된다.
  - VGW가 생성되어 사이트 간 VPN 연결을 생성할 VPC에 연결된다.
  - Autonomous System Number(ASN) 사용자 지정 가능성이 있다.
- **Customer Gateway(CGW)**
  - VPN 연결의 고객 측에 있는 소프트웨어 응용프로그램 또는 물리적 장치다.
  - 자세한 내용은 [여기](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html#DevicesTested)를 참고한다.

#### Site-to-Site VPN Connections

- Customer Gateway Device (On-premises)
  - Customer Gateway 장치의 공용 인터넷 라우팅 가능 IP 주소
  - NAT 순회(NAT-T)를 사용하도록 설정된 NAT 장치 뒤에 있는 경우, NAT 장치의 Public IP를 사용한다.
- 서브넷과 연결된 "Route Table"에서 가상 개인 게이트웨이에 대한 경로 전파에 사용된다.
- On-premises에서 EC2 인스턴스를 ping해야 하는 경우 Security Group의 인바운드에 ICMP 프로토콜을 추가해야 한다.

![site-to-site-vpn-connection.png](images%2FVPC%2Fsite-to-site-vpn-connection.png)

#### AWS VPN CloudHub

- 여러 VPN 연결이 있는 경우 여러 사이트 간에 안전한 통신을 제공한다.
- 서로 다른 위치(Region) 간의 기본 또는 보조 네트워크 연결을 위한 저비용 `hub-and-spoke` 모델을 제공한다.(VPN만 해당)
- VPN 연결이라 공용 인터넷을 사용해야 한다.
- 설정하기 위해서 동일한 VGW에서 여러 VPN 연결을 연결하고 동적 라우팅을 설정하고 "Route Table"을 구성해야 한다.

![vpn-cloudhub.png](images%2FVPC%2Fvpn-cloudhub.png)

---

### Direct Connect(DX)

- 원격 네트워크에서 VPC로 전용 프라이빗 연결을 제공한다.
- DC 및 AWS Direct Connect 위치 간에 전용 연결을 설정해야 한다.
- VPC에 가상 개인 게이트웨이를 설정해야 한다.
- 동일한 연결에서 Public 리소스(S3) 및 Private(EC2) 액세스를 할 수 있다.
- 아래와 같은 사용 사례가 있다.
  - 대역폭 처리량 향상 - 대용량 데이터 세트와 함께 작동하여 비용을 절감할 수 있다.
  - 보다 일관된 네트워크 환경 - 실시간 데이터 피드를 사용하는 애플리케이션
  - 하이브리드 환경 - On-premise + Cloud 환경
- IPv4와 IPv6 모두 지원한다.

![direct-connect-diagram.png](images%2FVPC%2Fdirect-connect-diagram.png)


#### Direct Connect Gateway

- 여러 Region(동일한 계정)에서 하나 이상의 VPC에 직접 연결을 설정하려면 "Direct Connect Gateway"를 사용해야 한다.

![direct-connect-gateway.png](images%2FVPC%2Fdirect-connect-gateway.png)

#### Connection 유형

- **전용(Dedicated) 연결**: 1Gbps, 10Gbps 및 100Gbps 용량을 가지고 있다.
  - 고객 전용 물리적 이더넷 포트를 제공한다.
  - 먼저 AWS에 요청한 후 "AWS Direct Connect Partners"에서 완료한다.
- **호스트(Hosted) 연결**: 50Mbps, 500Mbps, to 10Gbps까지 지원할 수 있다.
  - 연결 요청은 "AWS Direct Connect Partners"를 통해 이루어진다.
  - 필요에 따라 용량을 추가하거나 제거할 수 있다.
  - 일부 "AWS Direct Connect Partners"에서 1, 2, 5, 10Gbps가 사용이 가능하다.
- 새로운 연결을 설정하는 데 리드 타임이 1개월 이상 걸리는 경우가 많다.

#### 암호화

- 전송 중인 데이터는 암호화되지 않았지만 Private 통신이다.
- AWS Direct Connect + VPN을 통해 IPsec 암호화된 Private 연결을 제공한다.
- 보안 수준을 높이기에는 좋지만 구현은 조금 더 복잡해진다.

![direct-connect-encryption.png](images%2FVPC%2Fdirect-connect-encryption.png)

#### 복원력(Resiliency)

![direct-connect-high-resiliency.png](images%2FVPC%2Fdirect-connect-high-resiliency.png)

- 여러 Location에서 하나의 연결을 사용한다.

![direct-connect-maximum-resiliency.png](images%2FVPC%2Fdirect-connect-maximum-resiliency.png)

- 하나 이상의 Location에 있는 개별 장치에서 종료되는 별도의 연결을 통해 복원력을 극대화할 수 있다.
  
- "Direct Connect"에 실패할 경우 백업 "Direct Connect"(비용이 많이 발생) 또는 "Site-to-Site" VPN 연결을 설정할 수 있다.

![site-to-site-connection-backup.png](images%2FVPC%2Fsite-to-site-connection-backup.png)

- 네트워크 토폴로지가 아래의 이미지와 같이 복잡해질 수 있다.

![network-topologies-complicated.png](images%2FVPC%2Fnetwork-topologies-complicated.png)

---

### Transit Gateway

- 수천 대의 VPC와 On-Premise, "hub-and-spoke"(star) 연결 간의 과도한 피어링 기능을 제공한다.
- 지역별 리소스, 지역 간 작업이 가능하다.
- RAM(Resource Access Manager)을 사용하여 계정 간 공유가 가능하다.
- 여러 Region에 걸쳐 "Transit Gateway"를 피어(peer)할 수 있다.
- Route Table: VPC가 다른 VPC와 대화할 수 있는 제한
- "Direct Connect Gateway", VPN 연결과 함께 작동한다.
- IP Multicast를 지원한다.(다른 AWS 서비스에서는 지원되지 않음)

![transit-gateway.png](images%2FVPC%2Ftransit-gateway.png)

#### Site-to-Site VPN ECMP

- ECMP: "Equal-cost multi-path" routing
- 여러 최적 경로를 통해 패킷을 포워딩할 수 있도록 하는 라우팅 전략이다.
- 여러 개의 사이트 간 VPN 연결을 생성하여 AWS에 대한 연결 대역폭을 확장한다.

![site-to-site-vpn-ecmp.png](images%2FVPC%2Fsite-to-site-vpn-ecmp.png)

- ECMP를 사용하여 늘어난 처리량을 확인하면 아래와 같다.

![transit-gateway-throughput-ecmp.png](images%2FVPC%2Ftransit-gateway-throughput-ecmp.png)

- 여러 계정 간 직접 연결을 아래의 이미지와 같이 공유할 수 있다.

![transit-gateway-share-direct-connect-multiple-accounts.png](images%2FVPC%2Ftransit-gateway-share-direct-connect-multiple-accounts.png)

- "AWS Resource Access Manager"를 사용하여 "Transit Gateway"를 다른 계정과 공유할 수 있다.

---

### VPC Traffic Mirroring

- VPC에서 네트워크 트래픽을 캡처하고 검사할 수 있다.
- 관리하는 보안 어플라이언스로 트래픽을 라우팅할 수 있다.
- ENI로 부터 트래픽을 캡쳐하고, ENI 또는 네트워크 로드 밸런서로 보낸다.
- 모든 패킷 캡처 또는 관심 있는 패킷을 캡처한다.
- Source와 Target은 동일한 VPC에 있을 수도, 다른 VPC에 있을 수도 있다.
- 컨텐츠 검사, 위협 모니터링, 문제 해결 등에 사용된다.

![vpc-traffic-mirroring.png](images%2FVPC%2Fvpc-traffic-mirroring.png)

---

### IPv6

- 43억 개의 주소를 제공하도록 설계된 IPv4는 곧 모두 고갈될 예정이다.
- IPv6는 IPv4의 후속 제품이다.
- IPv6는 고유 IP 주소인 3.4 * 10^38 개를 제공하도록 설계되었다.
- 모든 IPv6 주소는 Public 및 인터넷 라우팅이 가능하며, Private을 위한 범위는 없다.
- `x.x.x.x.x.x.x.x` 형태를 가지며 x는 16진수이고 범위는 0000~ffff이다.
- 아래는 IPv6 예시이다.
  - 2001:db8:3333:4444:5555:6666:7777:8888
  - 2001:db8:3333:4444:cccc:dddd:eeee:ffff
  - :: -> 모든 8개의 세그먼트가 0이다.
  - 2001:db8:: -> 마지막 6개의 세그먼트가 0이다.
  - ::1234:5678 -> 처음 6개의 세그먼트가 0이다.
  - 2001:db8::1234:5678 -> 중간 4개의 세그먼트가 0이다.

- VPC 및 서브넷에 대해 IPv4를 사용하지 않도록 설정할 수 없다.
- IPv6(Public IP)를 이중 스택 모드로 작동하도록 설정할 수 있다.
- EC2 인스턴스에는 최소 Private Internal IPv4와 Public IPv6가 할당된다.
- Internet Gateway를 통해 IPv4 또는 IPv6를 사용하여 인터넷으로 통신할 수 있다.

![Ipv6-in-vpc.png](images%2FVPC%2FIpv6-in-vpc.png)

- VPC 및 서브넷에 대해 IPv4를 사용하지 않도록 설정할 수 없으므로 서브넷에서 EC2 인스턴스를 시작할 수 없는 경우가 있다.
  - IPv6를 획득할 수 없어서 발생하는 문제가 아니다.
  - 서브넷에서 사용 가능한 IPv4가 없기 때문이다.
- 이러한 문제를 해결하기 위해 서브넷에 새로운 IPv4 CIDR을 생성할 수 있다.

![Ipv6-troubleshooting.png](images%2FVPC%2FIpv6-troubleshooting.png)

#### 송신 전용(egress-only) Internet Gateway

- IPv6에만 사용된다. (NAT Gateway와 유사하지만 IPv6를 위한 것)
- 인터넷이 인스턴스에 IPv6 연결을 시작하지 못하도록 하면서 IPv6을 통한 VPC 아웃바운드 연결의 인스턴스를 허용한다.
- Route Table을 반드시 업데이트해야 한다.

![egress-only-internet-gateway.png](images%2FVPC%2Fegress-only-internet-gateway.png)

- IPv6의 라우팅 방식은 아래의 이미지와 같다.

![Ipv6-routing.png](images%2FVPC%2FIpv6-routing.png)

---

### VPC 요약

- **CIDR**: IP 범위
- **VPC(Virtual Private Cloud)**: IPv4 & IPv6 CIDR 목록을 정의한다.
- **Subnets**: AZ에 연결되어 CIDR을 정의한다.
- **Internet Gateway**: VPC 수준에서 IPv4 및 IPv6 인터넷 액세스를 제공한다.
- **Route Tables**: 서브넷에서 IGW, VPC Peering 연결, VPC Endpoints로 경로를 추가하려면 편집이 필요하다.
- **Bastion Host**: Private 서브넷의 EC2 인스턴스에 SSH 접속이 필요할 때, Public 서브넷에 설치하고 경유하도록 설정한다.
- **NAT Instances**: Private 서브넷의 EC2 인스턴스에 인터넷 액세스를 허용한다. Public 서브넷에 설정해야 하며, Source/Destination 확인 플래그를 비활성화한다.
- **NAT Gateway**: AWS에서 관리하는 NAT Gateway를 통해 Private EC2 인스턴스(IPv4만 해당)에 대한 확장 가능한 인터넷 액세스를 제공한다.
- **Private DNS + Route 53**: DNS확인 + DNS 호스트 이름(VPC) 활성화
- **NACL**: Stateless, 서브넷의 인바운드, 아웃바운드 규칙, 임시(Ephemeral) 포트
- **Security Group**: Stateful, EC2 인스턴스 레벨에서 작동한다.
- **Reachability Analyzer**: AWS 리소스간 네트워크 연결 테스트를 수행한다.
- **VPC Peering**: 중복되지 않는 CIDR, 비일시적인 CIDR로 두 대의 VPC를 연결한다.
- **VPC Endpoint**: VPC내에서 AWS 서비스(S3, DynamoDB, Cloud Formation, SSM)에 대한 Private 액세스를 제공한다.
- **VPC Flow Logs**: VPC/서브넷/ENI 레벨에서 트래픽을 수락 및 거부할 수 있도록 설정할 수 있으며 공격을 식별하고 Athena 또는 CloudWatch Log Insight를 사용하여 분석할 수 있다.
- **Site-to-Site VPN**: 고객 데이터센터의 Gateway, VPC에 가상 사설 게이트웨이, Public 인터넷을 통한 사이트 간 VPN 설정이다.
- **AWS VPN CloudHub**: 사이트를 연결하는 "hub-and-spoke" VPN 모델이다.
- **Direct Connect**: VPC에 Virtual Private Gateway를 설정하고 AWS Direct Connect Location에 직접 개인 연결을 설정한다.
- **Direct Connect Gateway**: 다양한 AWS 지역에 있는 많은 VPC에 Direct Connect를 설정한다.
- **AWS PrivateLink/VPC Endpoint**:
  - 서비스 VPC에서 고객 VPC로 서비스를 Private하게 연결한다.
  - VPC Peering, Public Internet, NAT Gateway, Route Table이 필요없다.
  - Network Load Balancer & ENI와 함께 사용해야 한다.
- **ClassicLink**: EC2와 Classic EC2 인스턴스를 VPC에 Privately 연결한다.
- **Transit Gateway**: VPC, VPN 및 DX를 위한 Peering 연결
- **Traffic Mirroring**: 추가 분석을 위해 ENI에서 네트워크 트래픽을 복사한다.
- **Egress-only Internet Gateway**: NAT Gateway와 유사하지만 IPv6와 함께 작동한다.

---

### 네트워크 비용

- GB당 AWS의 네트워킹 비용이다.

![network-costs-aws.png](images%2FVPC%2Fnetwork-costs-aws.png)

- 비용 절감과 네트워크 성능 향상을 위해서 Public IP 대신 Private IP를 사용하는 것이 좋다.
- 고가용성(HA) 비용으로 동일한 AZ를 사용하여 비용을 절감할 수 있다.
  
- 송신(egress) 트래픽: 아웃바운드 트래픽이다.(AWS에서 외부로)
- 수신(ingress) 트래픽: 인바운드 트래픽이다. (외부에서 AWS로, 일반적으로 무료)
- 비용을 최소화하기 위해 AWS 내에서 인터넷 트래픽을 최대한 유지해야 한다.
- 동일한 AWS Region에 위치한 Direct Connect Location을 사용하면 송신 네트워크에 대한 비용이 절감된다.

![minimizing-egress-network-cost.png](images%2FVPC%2Fminimizing-egress-network-cost.png)

#### S3 데이터 전송 가격(미국 기준)

- S3 Ingress: 무료
- S3 to Internet: GB당 $0.09
- S3 Transfer Acceleration:
  - 전송 시간 단축(50 ~ 500% 향상)
  - 데이터 전송 가격에 추가 비용: GB당 +0.04~0.08
- S3 to CloudFront: GB당 $0.00
- CloudFront to Internet: GB당 $0.085
  - 캐싱 기능(낮은 대기 시간)
  - S3 Request 관련 비용 절감(CloudFront의 경우 7배 저렴)
- S3 Cross Region Replication: GB당 $0.02

![s3-data-transfer-pricing.png](images%2FVPC%2Fs3-data-transfer-pricing.png)

#### NAT Gateway vs Gateway VPC Endpoint

- NAT Gateway와 Gateway VPC Endpoint의 가격차이는 아래의 이미지와 같다.

![nat-gateway-gateway-vpc.png](images%2FVPC%2Fnat-gateway-gateway-vpc.png)

---

### AWS Network Firewall

- Amazon VPC 전체를 보호한다.
- Layer 3에서 Layer 7으로 보호한다.
- 어느 방향이든 검사할 수 있다.
  - VPC to VPC 트래픽
  - 인터넷으로 아웃바운드 트래픽
  - 인터넷에서 인바운드 트래픽
  - Direct Connect와 Site-to-Site VPN 양방향
- 직접 연결 및 사이트간 VPN 연결/발신
- 내부적으로 AWS Network Firewall은 "AWS Gateway Load Balancer"를 사용한다.
- 규칙은 "AWS Firewall Manager"가 여러 VPC에 적용하기 위해 여러 계정에 걸쳐 중앙 집중식으로 관리할 수 있다.

![network-firewall.png](images%2FVPC%2Fnetwork-firewall.png)

- 세밀한 제어 장치를 지원한다.
- 1,000개의 규칙을 지원한다.
  - IP & Port: IP 10,000개 필터링
  - 프로토콜: 예를 들어, 아웃바운드 통신을 위한 SMB 프로토콜을 차단한다.
  - Stateful 도메인 목록 규칙 그룹: `*.mycorp.com` 또는 타사 소프트웨어 레포(repo)에 대한 아웃바운드 트래픽만 허용한다.
  - 정규식을 이용한 일반 패턴을 매칭한다.
- 트래픽 필터링: 규칙과 일치하는 트래픽에 대해서 허용하거나 삭제 또는 경고할 수 있다.
- 침입 방지 기능을 통해 네트워크 위협으로부터 보호하는 능동적 흐름을 검사한다.
- 규칙 일치 로그를 Amazon S3, CloudWatch 로그, Kinesis Data Firehose로 전송한다.

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03