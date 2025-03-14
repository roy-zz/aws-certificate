# Networking (Pre-Requisites)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "네트워킹 사전 지식 중 Switching, Routing, DNS"에 대해서 자세하게 알아보도록 한다.

---

## Switching & Routing

![1-networking-pre-requisites.png](images%2F1-networking-pre-requisites.png)

### 기본 네트워크 라우팅 개념

#### 스위칭 (Switching)

![2-switching.png](images%2F2-switching.png)

- 스위치는 로컬 네트워크(LAN) 내에서 여러 장치를 연결하고 데이터를 효율적으로 전달하는 네트워크 장치다.
- 라우터와 달리 스위치는 동일한 네트워크 내에서 작동하며, MAC 주소를 기반으로 패킷을 전달한다.

#### 라우팅 (Routing)

![3-routing.png](images%2F3-routing.png)

- **라우팅 테이블**:
  - 라우팅 테이블은 네트워크 패킷이 어떤 경로를 통해 목적지 네트워크로 전달되어야 하는지를 결정하는 데 사용되는 데이터베이스다.
  - 각 시스템(컴퓨터, 라우터 등)은 자체 라우팅 테이블을 가지고 있다.
- **라우터**:
  - 라우터는 서로 다른 네트워크 간에 패킷을 전ㄷ라하는 네트워크 장비다.
  - 라우터는 여러 네트워크 인터페이스를 가지고 있으며, 라우팅 테이블을 사용하여 패킷을 최적의 경로로 전달한다.
- **네트워크 주소**:
  - 네트워크 주소는 네트워크를 식별하는 데 사용된다. 예를 들어, 1.0은 특정 네트워크를 나타낸다.
  - IP 주소와 서브넷 마스크를 사용하여 네트워크 주소를 정의한다.

### 특정 네트워크 접근을 위한 라우팅 테이블 설정

#### 시스템 C의 라우팅 테이블 설정

![4-gateway-1.png](images%2F4-gateway-1.png)

![5-gateway-2.png](images%2F5-gateway-2.png)

- 시스템 C가 `192.168.1.0` 네트워크에 접근해야 하는 경우, 시스템 C의 라우팅 테이블에 다음과 같은 항목을 추가한다.
  - 목적지 네트워크: `192.168.1.0`
  - 다음 홉(Next Hop): `192.168.2.1` (라우터의 IP 주소)
- 이 설정을 통해 시스템 C는 `192.168.1.0` 네트워크로 가는 모든 패킷을 `192.168.2.1` IP 주소를 가진 라우터로 전달한다.
- 라우터는 `192.168.1.0` 네트워크에 연결되어있고, 라우팅 테이블을 통해 `192.168.1.0` 네트워크로 패킷을 전달한다.

### 인터넷 접근을 위한 라우팅 테이블 설정

- **인터넷 연결**
  - 시스템들이 인터넷에 접근하기 위해서 라우터가 인터넷에 연결되어야 한다.
  - 라우터는 인터넷 서비스 제공자(ISP)를 통해 인터넷에 연결된다.

![6-gateway-3.png](images%2F6-gateway-3.png)

- **특정 인터넷 네트워크 접근**
  - 만약 시스템들이 `172.217.194.0` 네트워크 (예: Google)에 접근해야 하는 경우, 라우팅 테이블에 다음과 같은 항목을 추가해야 한다.
    - 목적지 네트워크: `172.217.194.0`
    - 다음 홉: `192.168.2.1` (라우터의 IP 주소)
  - 라우터는 인터넷 연결을 통해 `172.217.194.0` 네트워크로 패킷을 전달한다.

### 기본 게이트웨이와 라우팅 테이블

![7-default-gateway-1.png](images%2F7-default-gateway-1.png)

- **기본 게이트웨이의 역할**:
  - 기본 게이트웨이는 시스템이 라우팅 테이블에 정의되지 않은 외부 네트워크로 패킷을 전달할 때 사용하는 라우터의 IP 주소다.
  - 인터넷과 같이 다양한 네트워크에 접근해야 하는 경우, 모든 네트워크에 대한 라우팅 테이블 항목을 추가하는 대신 기본 게이트웨이를 설정한다.
- **`0.0.0.0`의 의미**:
  - 라우팅 테이블에서 `0.0.0.0`은 모든 IP 주소를 의미하며, 기본 경로(default route)를 나타낸다.
  - `0.0.0.0`을 목적지 네트워크로 설정하면 라우팅 테이블에 명시된 경로가 없는 모든 패킷이 해당 경로를 통해 전달된다.
- **게이트웨이가 필요 없는 경우**
  - 시스템이 동일한 로컬 네트워크 내의 다른 장치와 통신하는 경우, 게이트웨이가 필요하지 않다.
  - 예를 들어, `192.168.2.0` 네트워크 내의 시스템 C는 동일한 네트워크 내의 다른 장치와 통신할 때 게이트웨이 없이 직접 통신한다.

![8-default-gateway-2.png](images%2F8-default-gateway-2.png)

- **다중 라우터 환경**
  - 네트워크에 여러 라우터가 있는 경우(예: 인터넷 라우터, 내부 사설 네트워크 라우터), 각 네트워크에 대한 별도의 라우팅 테이블 항목을 설정해야 한다.
  - 내부 사설 네트워크에 대한 항목과 인터넷을 포함한 모든 외부 네트워크에 대한 기본 게이트웨이 항목을 설정한다.
- **인터넷 연결 문제 해결**
  - 인터넷 연결 문제가 발생하면 라우팅 테이블과 기본 게이트웨이 설정을 확인하는 것이 중요하다.

### 리눅스 호스트를 라우터로 구성

![9-default-gateway-3.png](images%2F9-default-gateway-3.png)

- **네트워크 구성**
  - 호스트 A, B, C가 있으며, A와 B는 `192.168.1.0` 네트워크에, B와 C는 `192.168.2.0` 네트워크에 연결되어 있다.
  - 호스트 B는 두 네트워크에 연결되어 있으며, 두 개의 네트워크 인터페이스를 가지고 있다.
  - A의 IP는 `192.168.1.5`, B의 IP는 `192.168.1.6`과 `192.168.2.5`다.
- **호스트 A와 C의 통신 설정**:
  - 호스트 A는 `192.168.2.0` 네트워크에 대한 경로를 모르기 때문에 호스트 C와 통신할 수 없다.
  - 호스트 A의 라우팅 테이블에 `192.168.2.0` 네트워크로 가는 경로를 추가하고 게이트웨이를 `192.168.1.6`(호스트 B의 IP)으로 설정한다.
  - 호스트 C도 마찬가지로 라우팅 테이블에 `192.168.1.0` 네트워크에 대한 경로를 모르기 때문에 호스트 A와 통신할 수 없다.
  - 호스트 C의 라우팅 테이블에 `192.168.1.0` 네트워크로 가는 경로를 추가하고, 게이트웨이를 `192.168.2.6`(호스트 B의 IP)으로 설정한다.

![10-default-gateway-4.png](images%2F10-default-gateway-4.png)

- **IP 포워딩 활성화**
  - 리눅스에서는 기본적으로 패킷 포워딩이 비활성화되어 있다. 즉, 호스트 B는 한 인터페이스에서 받은 패킷을 다른 인터페이스로 전달하지 않는다.
  - `/proc/sys/net/ipv4/ip_forward` 파일의 값을 1로 변경하여 IP 포워딩을 활성화해야 한다.
  - 재부팅 후에도 설정을 유지하려면 `/etc/sysctl.conf` 파일에서 `net.ipv4.ip_forward=1` 설정을 추가한다.

### 주요 리눅스 네트워크 명령어

![11-take-away.png](images%2F11-take-away.png)

- `ip link`: 네트워크 인터페이스 목록을 표시하고 수정한다.
- `ip addr`: 네트워크 인터페이스에 할당된 IP 주소를 표시한다.
- `ip addr add`: 네트워크 인터페이스에 IP 주소를 할당한다.
- `ip route` 또는 `route`: 라우팅 테이블을 표시한다.
- `ip route add`: 라우팅 테이블에 항목을 추가한다.
- `cat /proc/sys/net/ipv4/ip_forward`: IP 포워딩 활성화 여부를 확인한다.
- `/etc/network/interfaces`: 네트워크 인터페이스 설정을 영구적으로 저장하는 파일이다.
- `/etc/sysctl.conf`: 시스템 커널 파라미터 설정을 영구적으로 저장하는 파일이다.

---

## DNS

### etc/hosts 파일

![12-dns-1.png](images%2F12-dns-1.png)

![13-dns-2.png](images%2F13-dns-2.png)

![14-dns-3.png](images%2F14-dns-3.png)

- **역할**:
  - `/etc/hosts` 파일은 호스트 이름과 IP 주소를 매핑하는 로컬 파일이다.
  - 시스템은 호스트 이름 해석을 위해 먼저 이 파일을 참조한다.

![15-name-resolution-1.png](images%2F15-name-resolution-1.png)

![16-name-resolution-2.png](images%2F16-name-resolution-2.png)

![17-name-resolution-3.png](images%2F17-name-resolution-3.png)

- **사용 방법**:
  - IP 주소와 호스트 이름을 공백으로 구분하여 각 줄에 입력한다.
  - 예: `192.168.1.11 db`
- **장점**:
  - 간단하고 로컬에서 빠르게 호스트 이름을 해석할 수 있다.
- **단점**:
  - 규모가 큰 네트워크에서는 관리하기 어렵다.
  - IP 주소가 변경되면 모든 호스트의 파일을 수정해야 한다.
  - 파일에 적혀있는 것이 실제 호스트의 이름과 다를수도 있다.

### DNS (Domain Name System) 

![18-dns-1.png](images%2F18-dns-1.png)

- **역할**:
  - DNS 서버는 호스트 이름과 IP 주소 매핑을 중앙에서 관리한다.
  - 호스트는 DNS 서버에 쿼리하여 호스트 이름을 해석한다.

![19-dns-2.png](images%2F19-dns-2.png)

- **설정 방법**:
  - `/etc/resolv.conf` 파일에 DNS 서버의 IP 주소를 추가한다.
  - 예: `nameserver 192.168.1.100`
- **장점**:
  - 중앙 집중식 관리로 IP 주소 변경 시 DNS 서버만 수정하면 된다.
  - 규모가 큰 네트워크에서 효율적으로 호스트 이름을 관리할 수 있다.
- **단점**:
  - DNS 서버가 작동하지 않으면 호스트 이름 해석이 불가능하다.

### 호스트 이름 해석 순서

![20-dns-3.png](images%2F20-dns-3.png)

- `/etc/nsswitch.conf` 파일:
  - 이 파일은 호스트 이름 해석 순서를 정의한다.
  - `hosts:` 라인은 호스트 이름 해석에 사용되는 소스를 지정한다.
  - 기본적으로 `files dns` 순서로 설정되어 있다.
    - `files`: `/etc/hosts` 파일을 먼저 참조한다.
    - `dns`: DNS 서버를 참조한다.
- **순서 변경**:
  - `/etc/nsswitch.conf` 파일을 수정하여 호스트 이름 해석 순서를 변경할 수 있다.

### 외부 DNS 서버

![21-dns-4.png](images%2F21-dns-4.png)

- **공용 DNS 서버**:
  - 인터넷의 모든 웹사이트 정보를 알고있는 공용 DNS 서버를 사용 할 수 있다.
  - 예: Google의 8.8.8.8
- **/etc/resolv.conf 파일 수정**:
  - `/etc/resolv.conf` 파일에 공용 DNS 서버의 IP 주소를 추가한다.
- **DNS 서버 포워딩**:
  - 내부 DNS 서버에서 외부 호스트 이름 쿼리를 공용 DNS 서버로 포워딩하도록 설정할 수 있다.

### 호스트 이름 해석 과정

1. 호스트는 호스트 이름 해석을 요청한다.
2. `/etc/nsswitch.conf` 파일에 정의된 순서에 따라 호스트 이름 해석을 시도한다.
3. `/etc/hosts` 파일에 해당 호스트 이름이 있으면 IP 주소를 반환한다.
4. `/etc/hosts` 파일에 없으면 `/etc/resolv.conf` 파일에 설정된 DNS 서버에 쿼리한다.
5. DNS 서버가 IP 주소를 반환하면 호스트는 해당 IP 주소를 사용한다.
6. DNS 서버가 IP 주소를 찾을 수 없으면 오류를 반환한다.
7. 외부 사이트의 경우, `/etc/resolv.conf`에 외부 DNS 서버가 설정되어있거나, 내부 DNS 서버가 포워딩 설정을 가지고 있다면 외부 DNS 서버에 쿼리한다.

### 참고 사항

- `/etc/hosts` 파일과 DNS 서버에 동일한 호스트 이름에 대한 다른 IP 주소가 설정된 경우, `/etc/nsswitch.conf` 파일에 정의된 순서에 따라 IP 주소가 결정된다.
- `/etc/hosts` 파일은 로컬 테스트 환경에서 유용하게 사용될 수 있다.

### 도메인 이름 (Domain Name)

![22-domain-names-1.png](images%2F22-domain-names-1.png)

![23-domain-names-2.png](images%2F23-domain-names-2.png)

- **정의**:
  - 도메인 이름은 IP 주소를 사람이 기억하기 쉬운 이름으로 변환하는 시스템이다.
  - 인터넷에서 웹사이트나 서비스를 식별하는 데 사용된다.

![24-domain-names-3.png](images%2F24-domain-names-3.png)

- **구조**:
  - 도메인 이름은 점(.)으로 구분된 여러 부분으로 구성된다.
  - 예: www.google.com
  - **루트 도메인 (.)**: 모든 도메인의 시작점이다.
  - **최상위 도메인 (Top-Level Domain, TLD)**: 도메인의 목적을 나타낸다. (.com, .net, .org, .edu, 등)
  - **도메인 이름**: 특정 조직이나 개인에게 할당된 이름이다. (google, mycompany 등)
  - **서브 도메인**: 도메인 내부의 특정 서비스를 나타낸다. (www, mail, maps, drive 등)
- **계층 구조**:
  - 도메인 이름은 트리 구조를 형성하며, 각 레벨은 하위 레벨을 포함할 수 있다.

### DNS 서버 작동 방식

![25-domain-names-4.png](images%2F25-domain-names-4.png)

- **내부 DNS 서버**:
  - 조직 내부에서 사용하는 DNS 서버다.
  - 내부 도메인 이름과 IP 주소를 관리한다.
- **외부 DNS 서버 (인터넷)**:
  - 인터넷에서 도메인 이름과 IP 주소를 관리한다.
  - 루트 DNS 서버, TLD DNS 서버, 권한 있는 DNS 서버 등이 있다.
- **이름 해석 과정**:
  1. 사용자가 도메인 이름을 입력한다.
  2. 로컬 시스템은 `/etc/resolv.conf` 파일에 설정된 DNS 서버에 쿼리한다.
  3. 내부 DNS 서버는 쿼리를 처리하거나 외부 DNS 서버로 전달한다.
  4. 외부 DNS 서버는 루트 DNS 서버부터 시작하여 권한 있는 DNS 서버까지 쿼리를 전달한다.
  5. 권한 있는 DNS 서버는 도메인 이름에 해당하는 IP 주소를 반환한다.
  6. DNS 서버는 결과를 캐싱하여 다음 쿼리 속도를 향상시킨다.

### /etc/resolv.conf 파일 설정

![27-search-domain-1.png](images%2F27-search-domain-1.png)

- **search 옵션**:
  - 호스트 이름에 자동으로 추가할 도메인 이름을 지정한다.
  - 예: `search mycompany.com`
  - 호스트 이름만 입력해도 자동으로 도메인 이름을 추가하여 쿼리한다.
  - 여러 개의 도메인 이름을 지정할 수 있다.
- **nameserver 옵션**:
  - 사용할 DNS 서버의 IP 주소를 지정한다.

### DNS 레코드 유형

![28-record-types-1.png](images%2F28-record-types-1.png)

- **A 레코드**: 호스트 이름과 IPv4 주소를 매핑한다.
- **AAAA 레코드**: 호스트 이름과 IPv6 주소를 매핑한다.
- **CNAME 레코드**: 호스트 이름의 별칭을 지정한다. (다른 호스트 이름으로 매핑)

### DNS 테스트 도구

![29-nslookup-1.png](images%2F29-nslookup-1.png)

- **nslookup**: DNS 서버에 쿼리하여 호스트 이름 정보를 확인한다.
  - `/etc/hosts` 파일을 참조하지 않는다.

![30-dig-1.png](images%2F30-dig-1.png)

- **dig**: DNS 서버에 쿼리하여 자세한 DNS 레코드 정보를 확인한다.
  - `/etc/hosts` 파일을 참조하지 않는다.

### 내부 도메인 이름 설정

- 조직 내부에서 사용하는 도메인 이름을 설정하고 내부 DNS 서버에 등록한다.
- 내부 도메인 이름은 외부 도메인 이름과 유사한 구조를 가질 수 있다.
- 내부 도메인 이름을 사용하여 내부 서비스에 쉽게 접근할 수 있다.

---

## Core DNS

### DNS 서버의 필요성

- 대규모 환경에서 호스트 이름과 IP 주소 관리를 위해 DNS 서버가 필요하다.
- 실제로 호스트를 DNS 서버로 등록하는 방법을 알아본다.

### CoreDNS 소개

![31-core-dns-1.png](images%2F31-core-dns-1.png)

- CoreDNS는 유연하고 확장 가능한 DNS 서버 소프트웨어다.
- 다양한 플러그인을 통해 다양한 기능을 제공한다.
- Kubernetes 환경에서 DNS 서버로 널리 사용된다.

### CoreDNS 설치

![32-core-dns-2.png](images%2F32-core-dns-2.png)

- CoreDNS 바이너리는 GitHub 릴리즈 페이지에서 다운로드하거나 Docker 이미지로 사용할 수 있다.
- 전통적인 방법인 curl 또는 wget을 사용하여 바이너리를 다운로드하고 압축을 해제한다.
- 압축을 해제하면 `coredns` 실행 파일을 얻을 수 있다.

#### CoreDNS 실행

- `coredns` 실행 파일을 실행하여 DNS 서버를 시작한다.
- CoreDNS는 기본적으로 DNS 서버의 기본 포트인 53번 포트를 사용한다.

### 호스트 이름 - IP 주소 매핑 설정

![33-core-dns-3.png](images%2F33-core-dns-3.png)

- CoreDNS는 다양한 방법으로 호스트 이름-IP 주소 매핑을 설정할 수 있다.
- 가장 간단한 방법은 서버의 `/etc/hosts` 파일에 매핑 정보를 추가하는 것이다.
- `/etc/hosts` 파일에 IP 주소와 호스트 이름을 공백으로 구분하여 입력한다.

### Corefile 설정

![34-core-dns-4.png](images%2F34-core-dns-4.png)

- CoreDNS는 `Corefile`이라는 설정 파일에서 설정을 로드한다.
- `Corefile`은 CoreDNS의 동작을 정의하는 플러그인 설정을 포함한다.
- `/etc/hosts` 파일을 사용하여 매핑 정보를 로드하는 간단한 `Corefile` 예시

```json
.:53 {
    hosts /etc/hosts {
        fallthrough
    }
    forward . 8.8.8.8 8.8.4.4
    cache 30
    loop
    reload
    loadbalance
}
```

- 위의 `Corefile`은 다음과 같은 기능을 설정합니다.
  - 53번 포트에서 DNS 서버를 실행합니다.
  - `/etc/hosts` 파일에서 호스트 이름-IP 주소 매핑을 로드합니다.
  - 외부 DNS 서버(8.8.8.8, 8.8.4.4)로 쿼리를 전달합니다.
  - 30초 동안 결과를 캐싱합니다.
  - 루프 감지를 활성화합니다.
  - 설정 파일 변경 시 자동으로 다시 로드합니다.
  - 로드 밸런싱을 활성화합니다.
- `Corefile`을 수정하고 CoreDNS를 재시작하면 변경된 설정이 적용됩니다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)