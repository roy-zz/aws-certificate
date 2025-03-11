# Networking (Pre-Requisites)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "네트워킹 사전 지식 중 Namespace, Docker Networking, CNI"에 대해서 자세하게 알아보도록 한다.

---

## Namespace

### 네임스페이스(Namespaces)의 기본 개념

![1-namespace.png](images%2F1-namespace.png)

- **격리(Isolation)**: 네임스페이스는 리눅스 시스템에서 프로세스, 네트워크, 마운트 지점 등과 같은 시스템 자원을 격리시키는 기술이다.
- **컨테이너 격리**: 컨테이너는 네임스페이스를 사용하여 호스트 시스템과 다른 컨테이너로부터 격리된다.
- **비유**: 호스트 시스템을 집으로 비유하고, 네임스페이스를 집 안의 방으로 비유한다. 각 방(네임스페이스)은 독립적인 공간을 제공하며, 부모(호스트)는 모든 방을 볼 수 있지만 자녀(컨테이너)는 자신의 방만 볼 수 있다.

![2-process-namespace-1.png](images%2F2-process-namespace-1.png)

- **프로세스 격리**: 컨테이너 내부에서는 컨테이너의 프로세스만 보이며, 호스트 시스템의 다른 프로세스는 보이지 않는다. 
  - 호스트 시스템에서는 모든 프로세스를 볼 수 있다.
  - 프로세스 ID는 네임스페이스에 따라 다르게 표시된다.

### 네트워크 네임스페이스 (Network Namespace)

![3-network-namespace-1.png](images%2F3-network-namespace-1.png)

- **네트워크 격리**: 네트워크 네임스페이스는 네트워크 인터페이스, 라우팅 테이블, ARP 테이블 등을 격리한다.
- **컨테이너 네트워크 격리**: 컨테이너는 네트워크 네임스페이스를 사용하여 호스트 시스템의 네트워크 설정과 분리된 독립적인 네트워크 환경을 갖는다.
- **가상 인터페이스**: 네트워크 네임스페이스는 자체 가상 네트워크 인터페이스를 가질 수 있다.
- **독립적인 네트워크 설정**: 각 네트워크 네임스페이스는 자체 라우팅 테이블, ARP 테이블 등을 가진다.

### 네트워크 네임스페이스 생성 및 관리

![4-create-network-ns.png](images%2F4-create-network-ns.png)

- **생성**: `ip netns add <namespace-name>` 명령을 사용하여 새로운 네트워크 네임스페이스를 생성한다.
  - 예: `ip netns add red`, `ip netns add blue`
- **목록 확인**: `ip netns` 명령을 사용하여 생성된 네트워크 네임스페이스 목록을 확인한다.

![5-exec-in-network-ns-1.png](images%2F5-exec-in-network-ns-1.png)

- **네임스페이스 내부 명령어 실행**
  - `ip netns exec <namespace-name> <command>` 명령을 사용하여 네트워크 네임스페이스 내부에서 명령어를 실행한다.
    - 예: `ip netns exec red ip link`
  - `ip <command> -n <namespace-name>` 명령을 사용하여 특정 네트워크 네임스페이스 내부에서 IP 명령어를 실행한다.
    - 예: `ip link -n red`
- **인터페이스 확인**: `ip link` 명령을 사용하여 네트워크 인터페이스 목록을 확인한다. 네트워크 네임스페이스 내부에서는 호스트 시스템의 인터페이스가 보이지 않는다.

![6-exec-in-network-ns-2.png](images%2F6-exec-in-network-ns-2.png)

- **ARP 테이블 확인**: `arp` 명령을 사용하여 ARP 테이블을 확인한다. 네트워크 네임스페이스 내부에서는 호스트 시스템의 ARP 테이블이 보이지 않는다.

![7-exec-in-network-ns-3.png](images%2F7-exec-in-network-ns-3.png)

- **라우팅 테이블 확인**: `ip route` 명령을 사용하여 라우팅 테이블을 확인한다. 네트워크 네임스페이스 내부에서는 호스트 시스템의 라우팅 테이블이 보이지 않는다.

### 네트워크 네임스페이스 간 연결 설정

![8-exec-in-network-ns-4.png](images%2F8-exec-in-network-ns-4.png)

- **가상 이더넷 쌍(Virtual Ethernet Pair, veth)**: `ip link add` 명령을 사용하여 가상 이더넷 쌍을 생성한다.
  - 예: `ip link add veth-red type veth peer name veth-blue`

![9-exec-in-network-ns-5.png](images%2F9-exec-in-network-ns-5.png)

- **인터페이스 연결**: `ip link set <interface-name> netns <namespace-name>` 명령을 사용하여 인터페이스를 네트워크 네임스페이스에 연결한다.
  - 예: `ip link set veth-red netns red`, `ip link set veth-blue netns blue`

![10-exec-in-network-ns-6.png](images%2F10-exec-in-network-ns-6.png)

- **IP 주소 할당**: `ip addr add <ip-address>/<subnet-mask> dev <interface-name>` 명령을 사용하여 IP 주소를 할당한다.
  - 예: `ip addr add 192.168.15.1/24 dev veth-red -n red`, `ip addr add 192.168.15.2/24 dev veth-blue -n blue`

![11-exec-in-network-ns-7.png](images%2F11-exec-in-network-ns-7.png)

- **인터페이스 활성화**: `ip link set up dev <interface-name>` 명령을 사용하여 각 네트워크 네임스페이스의 인터페이스를 활성화한다.
  - 예: `ip link set up dev veth-red -n red`, `ip link set up dev veth-blue -n blue`

![12-exec-in-network-ns-8.png](images%2F12-exec-in-network-ns-8.png)

- **연결 확인**: `ping <ip-address>` 명령을 사용하여 네트워크 네임스페이스 간 연결을 확인한다.
  - 예: `ip link set up dev veth-red -n red`, `ip link set up dev veth-blue -n blue`

![13-exec-in-network-ns-9.png](images%2F13-exec-in-network-ns-9.png)

- **ARP 테이블 확인 (네임스페이스 내부)**: 각 네임스페이스 내부의 ARP 테이블에서 연결된 상대방의 MAC 주소를 확인할 수 있다.
- **ARP 테이블 확인 (호스트)**: 호스트 시스템의 ARP 테이블에는 네트워크 네임스페이스의 정보가 나타나지 않는다.

### 여러 네트워크 네임스페이스 간 통신 설정

![14-linux-bridge-1.png](images%2F14-linux-bridge-1.png)

- **가상 스위치(Linux Bridge)의 필요성**: 두 개 이상의 네트워크 네임스페이스 간 통신을 위해 가상 스위치가 필요하다.

![15-linux-bridge-2.png](images%2F15-linux-bridge-2.png)

- **Linux Bridge 생성**:
  - `ip link add <bridge-name> type bridge` 명령을 사용하여 Linux Bridge를 생성한다.
    - 예: `ip link add vnet0 type bridge`
  - `ip link set up dev <bridge-name>` 명령을 사용하여 브리지를 활성화한다.

#### 가상 이더넷 쌍(veth) 연결:

![16-linux-bridge-3.png](images%2F16-linux-bridge-3.png)

- 각 네트워크 네임스페이스와 브리지 사이에 가상 이더넷 쌍을 생성한다.

![17-linux-bridge-4.png](images%2F17-linux-bridge-4.png)

- `ip link add <veth-name> type veth peer name <veth-peer-name>` 명령을 사용하여 가상 이더넷 쌍을 생성한다.
  - 예: `ip link add veth-red type veth peer name veth-red-br`

![18-linux-bridge-5.png](images%2F18-linux-bridge-5.png)

![20-linux-bridge-7.png](images%2F20-linux-bridge-7.png)

- `ip link set <veth-name> netns <namespace-name>` 명령을 사용하여 veth 인터페이스를 네임스페이스에 연결한다.
  - 예: `ip link set veth-red netns red`, `ip link set veth-blue netns blue`

![19-linux-bridge-6.png](images%2F19-linux-bridge-6.png)

![21-linux-bridge-8.png](images%2F21-linux-bridge-8.png)

- `ip link set <veth-name> master <bridge-name>` 명령을 사용하여 veth 인터페이스를 브리지에 연결한다.
  - 예: `ip link set veth-red-br master v-net-0`, `ip link set veth-blue-br master v-net-0`

### 호스트와 네트워크 네임스페이스 간 통신 설정

![22-linux-bridge-9.png](images%2F22-linux-bridge-9.png)

- **브리지 인터페이스에 IP 주소 할당**: 브리지 인터페이스는 호스트의 네트워크 인터페이스 역할도 하므로, 호스트에서 네임스페이스로 접근하기 위해 브리지 인터페이스에 IP 주소를 할당한다.
  - `ip addr add <ip-address>/<subnet-mask> dev <bridge-name>` 명령을 사용하여 IP 주소를 할당한다.

![23-linux-bridge-10.png](images%2F23-linux-bridge-10.png)

- **호스트에서 네임스페이스로 통신 확인**: `ping` 명령을 사용하여 호스트에서 네임스페이스로 통신을 확인한다.

### 네트워크 네임스페이스에서 외부 네트워크 연결 설정

![24-linux-bridge-11.png](images%2F24-linux-bridge-11.png)

![25-linux-bridge-12.png](images%2F25-linux-bridge-12.png)

![26-linux-bridge-13.png](images%2F26-linux-bridge-13.png)

- **라우팅 테이블 설정**: 네임스페이스에서 외부 네트워크로 접근하기 위해 라우팅 테이블에 게이트웨이 정보를 추가한다.
  - `ip route add <external-network-address>/<subnet-mask> via <gateway-address> n <namespace-name>` 명령을 사용하여 라우팅 테이블을 설정한다.
  - 게이트웨이 IP 주소는 호스트의 브리지 인터페이스 IP 주소다.

![27-linux-bridge-14.png](images%2F27-linux-bridge-14.png)

- **NAT(Network Address Translation) 설정**: 호스트에서 NAT를 활성화하여 네임스페이스의 트래픽을 외부 네트워크로 전달한다.
  - `iptables -t nat -A POSTROUTING -s <namespace-network-address>/<subnet-mask> -j MASQUERADE` 명령을 사용하여 NAT를 설정한다.

![28-linux-bridge-15.png](images%2F28-linux-bridge-15.png)

- **외부 네트워크로 통신 확인**: `ping` 명령을 사용하여 네임스페이스에서 외부 네트워크로 통신을 확인한다.

![29-linux-bridge-16.png](images%2F29-linux-bridge-16.png)

- **인터넷 연결 설정**: 네임스페이스에서 인터넷에 접근하기 위해 기본 게이트웨이를 설정한다.
  - `ip route add default via <gateway-address> n <namespace-name>` 명령을 사용하여 기본 게이트웨이를 설정한다.
  - 게이트웨이 IP 주소는 호스트의 브리지 인터페이스 IP 주소다.
- **인터넷 통신 확인**: `ping` 명령을 사용하여 네임스페이스에서 인터넷으로 통신을 확인한다.

### 외부 네트워크에서 네트워크 네임스페이스로 접근 설정

- **포트 포워딩(Port Forwarding) 설정**: 외부 네트워크에서 네임스페이스의 서비스에 접근하기 위해 포트 포워딩을 설정한다.
  - `iptables -t nat -A PREROUTING -i <host-interface> -p tcp --dport <host-port> -j DNAT --to-destination <namespace-ip>:<namespace-port>` 명령을 사용하여 포트 포워딩을 설정한다.
- **외부 네트워크에서 네임스페이스 서비스 접근 확인**: 외부 네트워크에서 호스트의 포트를 통해 네임스페이스의 서비스에 접근한다.
- **라우팅 테이블 설정(선택 사항)**: 외부 네트워크의 호스트에서 네임스페이스 네트워크로 접근하기 위해 라우팅 테이블을 설정할 수 있다. 하지만 포트 포워딩이 더 일반적이다.
  - `ip route add <namespace-network-address>/<subnet-mask> via <host-ip>` 명령을 사용하여 라우팅 테이블을 설정한다.

### 핵심 개념

- **Linux Bridge**: 가상 스위치 역할을 하여 네트워크 네임스페이스 간 통신을 가능하게 한다.
- **veth 쌍**: 네트워크 네임스페이스와 브리지를 연결하는 가상 이더넷 인터페이스다.
- **NAT**: 네트워크 주소 변환을 통해 네임스페이스의 트래픽을 외부 네트워크로 전달한다.
- **포트 포워딩**: 외부 네트워크에서 특정 포트로 들어오는 트래픽을 네임스페이스의 특정 포트로 전달한다.
- **라우팅 테이블**: 네트워크 트래픽의 경로를 결정한다.

---

## Docker Networking

### Docker 네트워크 옵션

![30-none-1.png](images%2F30-none-1.png)

- **none 네트워크**:
  - 컨테이너가 어떤 네트워크에도 연결되지 않는다.
  - 컨테이너는 외부와 통신할 수 없으며, 외부에서도 컨테이너에 접근할 수 없다.
  - 컨테이너 간에도 통신할 수 없다.

![31-host-1.png](images%2F31-host-1.png)

- **host 네트워크**:
  - 컨테이너가 호스트의 네트워크를 공유한다.
  - 컨테이너는 호스트의 모든 네트워크 인터페이스와 포트를 사용할 수 있다.
  - 네트워크 격리가 없으므로 보안에 취약할 수 있다.
  - 같은 포트를 사용하는 컨테이너를 여러개 실행할 수 있다.

![32-bridge-1.png](images%2F32-bridge-1.png)

- **bridge 네트워크**:
  - Docker 호스트와 컨테이너가 연결되는 내부 사설 네트워크를 생성한다.
  - 기본적으로 `172.17.0.0/16` 네트워크를 사용한다.
  - 각 컨테이너는 이 네트워크에서 고유한 IP 주소를 할당받는다.

### Docker Bridge 네트워크 작동 방식

#### Docker0 인터페이스

![33-bridge-2.png](images%2F33-bridge-2.png)

- Docker 설치 시 기본적으로 생성되는 Linux Bridge 인터페이스다.
- Docker 네트워크 목록에서는 "bridge"로 표시된다.

![34-bridge-3.png](images%2F34-bridge-3.png)

- 호스트의 네트워크 인터페이스 역할을 하며, 컨테이너의 가상 스위치 역할을 한다.
- `172.17.0.1` IP 주소를 할당받는다.

#### 네트워크 네임스페이스 생성

![35-bridge-4.png](images%2F35-bridge-4.png)

- Docker는 컨테이너를 실행할 때마다 네트워크 네임스페이스를 생성한다.
- `ip netns` 명령어를 통해 확인이 가능하다. 단, Docker에서 생성한 namespace를 보기위해서는 약간의 작업이 필요하다.
- `docker inspect` 명령어를 사용하여 각 컨테이너와 연결된 네트워크 네임스페이스를 확인할 수 있다.

#### 가상 이더넷 쌍(veth) 생성

![36-bridge-5.png](images%2F36-bridge-5.png)

![37-bridge-6.png](images%2F37-bridge-6.png)

![38-bridge-7.png](images%2F38-bridge-7.png)

- Docker는 각 컨테이너의 네트워크 네임스페이스와 Docker0 브리지 사이에 가상 이더넷 쌍을 생성한다.
- veth 인터페이스의 한쪽 끝은 컨테이너의 네트워크 네임스페이스에 연결되고, 다른 쪽 끝은 Docker0 브리지에 연결된다.
- `ip link` 명령어를 사용하여 호스트와 컨테이너 내부의 veth 인터페이스를 확인할 수 있다.
- 각 컨테이너는 veth 인터페이스를 통해 Docker0 브리지와 통신한다.

#### IP 주소 할당

![39-bridge-8.png](images%2F39-bridge-8.png)

- Docker는 컨테이너의 네트워크 네임스페이스에 IP 주소를 할당한다.
- `ip addr` 명령어를 컨테이너의 네트워크 네임스페이스 내부에서 실행하여 컨테이너의 IP 주소를 확인할 수 있다.
- `docker exec` 명령어를 사용하여 컨테이너에 접속하여 `ip addr` 명령어를 실행하여 ip 주소를 확인할 수도 있다.

#### 컨테이너 간 통신

![40-bridge-9.png](images%2F40-bridge-9.png)

- Docker0 브리지를 통해 컨테이너 간에 통신할 수 있다.

### 포트 매핑(Port Mapping)

![41-bridge-10.png](images%2F41-bridge-10.png)

![42-bridge-11.png](images%2F42-bridge-11.png)

- 기본적으로 호스트에서 `172.17.0.3:80` 주소로 접근이 불가능하다.

#### 외부 접근 허용

![43-bridge-12.png](images%2F43-bridge-12.png)

- Docker 호스트 외부에서 컨테이너의 서비스에 접근하기 위해 포트 매핑을 사용한다.
- `docker run -p <host-port>:<container-port> <image>` 명령어를 사용하여 호스트 포트와 컨테이너 포트를 매핑한다.
- 호스트의 특정 포트로 들어오는 트래픽을 컨테이너의 특정 포트로 전달한다.

#### iptables 규칙

![44-bridge-13.png](images%2F44-bridge-13.png)

![45-bridge-14.png](images%2F45-bridge-14.png)

- Docker는 iptables NAT 규칙을 사용하여 포트 매핑을 구현한다.
- PREROUTING 체인에 규칙을 추가하여 호스트 포트로 들어오는 트래픽의 목적지 포트를 컨테이너 포트로 변경한다.
- 규칙을 통해 컨테이너의 IP 주소와 컨테이너의 포트 번호를 목적지로 설정한다.

![46-bridge-15.png](images%2F46-bridge-15.png)

- `iptables -t nat -L` 명령어를 사용하여 iptables 규칙을 확인할 수 있다.

#### 외부 사용자의 접근

- 포트 매핑을 통해 외부 사용자는 Docker 호스트의 IP 주소와 포트를 사용하여 컨테이너의 서비스에 접근할 수 있다.

### 핵심 개념

- **네트워크 네임스페이스**: 컨테이너의 네트워크 격리를 위한 핵심 기술이다.
- **Linux Bridge (Docker0)**: 컨테이너 간 통신을 위한 가상 스위치 역할을 한다.
- **veth 쌍**: 컨테이너와 Docker0 브리지를 연결하는 가상 이더넷 인터페이스다.
- **포트 매핑**: 외부 네트워크에서 컨테이너의 서비스에 접근할 수 있도록 한다.
- **iptables**: 네트워크 트래픽을 제어하고 포트 매핑을 구현하는 데 사용된다.

---

## Container Network Interface (CNI)

### 컨테이너 네트워킹의 표준화 필요성

![47-container-networking-interface-1.png](images%2F47-container-networking-interface-1.png)

- **다양한 컨테이너 런타임 환경**: Docker, Rocket, Mesos Containerizer, Kubernetes 등 다양한 컨테이너 런타임 환경이 존재한다.
- **네트워킹 문제의 유사성**: 각 런타임 환경은 네트워크 격리, 브리지 연결, IP 주소 할당 등 유사한 네트워킹 문제를 해결해야 한다.

![48-container-networking-interface-2.png](images%2F48-container-networking-interface-2.png)

- **중복 개발 문제**: 각 런타임 환경이 자체적으로 네트워킹 솔루션을 개발하면 중복 개발과 유지 보수 비용이 발생한다.
- **표준화된 접근 방식 필요**: 모든 런타임 환경에서 사용할 수 있는 표준화된 네트워킹 접근 방식이 필요하다.

### 컨테이너 네트워크 인터페이스(CNI) 소개

![49-conatiner-networking-interface-3.png](images%2F49-conatiner-networking-interface-3.png)

![50-conatiner-networking-interface-4.png](images%2F50-conatiner-networking-interface-4.png)

- **CNI의 정의**: CNI는 컨테이너 런타임 환경에서 네트워크 문제를 해결하기 위한 표준 인터페이스다.
- **CNI의 역할**: CNI는 컨테이너 런타임과 네트워크 플러그인 간의 상호 작용을 정의한다.
- **CNI의 목표**: 다양한 런타임 환경에서 일관된 네트워킹 환경을 제공하고 플러그인 개발을 단순화한다.

### CNI 플러그인(Plugins)

![51-conatiner-networking-interface-5.png](images%2F51-conatiner-networking-interface-5.png)

- **플러그인의 역할**: 플러그인은 컨테이너 네트워크 설정을 담당하는 프로그램이다.
- **플러그인의 종류**: Bridge, VLAN< IP VLAN< MAC VLAN< Host Local, DHCP 등 다양한 플러그인이 존재한다.
- **플러그인의 예시**: Veave, Flannel, Cilium, VMware NSX, Calico, Infoblox 등 타사 플러그인도 존재한다.
- **Bridge 플러그인**: "Bridge" 프로그램은 CNI 플러그인의 예시다. 컨테이너를 브리지 네트워크에 연결하는 역할을 한다.

### CNI의 작동 방식 

#### 컨테이너 런타임의 역할

- 컨테이너를 위한 네트워크 네임스페이스를 생성한다.
- 컨테이너가 연결해야 할 네트워크를 식별한다.
- 컨테이너 생성 시 `add` 명령어를 사용하여 플러그인을 호출한다.
- 컨테이너 삭제 시 `del` 명령어를 사용하여 플러그인을 호출한다.
- JSON 파일을 사용하여 네트워크 플러그인을 구성한다.

#### 플러그인의 역할

- `add`, `del`, `check` 명령줄 인수를 지원한다.
- 컨테이너 ID, 네트워크 네임스페이스 등 필요한 매개변수를 처리한다.
- 컨테이너에 IP 주소를 할당하고 필요한 라우팅 설정을 수행한다.
- 결과를 특정 형식으로 변환한다.

### CNI 장점

- **상호 운용성**: 다양한 컨테이너 런타임과 플러그인 간의 상호 운용성을 보장한다.
- **유연성**: 다양한 네트워크 요구 사항을 충족하는 플러그인을 선택하여 사용할 수 있다.
- **확장성**: 새로운 플러그인을 개발하여 기능을 확장할 수 있다.
- **표준화**: 일관된 네트워킹 환경을 제공하고 개발 및 유지 보수를 단순화한다.

### Docker와 CNI

![52-conatiner-networking-interface-6.png](images%2F52-conatiner-networking-interface-6.png)

- **Docker의 CNM**: Docker는 CNI 대신 CNM(Container Network Model)이라는 자체 표준을 사용한다.
- **CNI와 CNM의 차이점**: CNI와 CNM은 컨테이너 네트워킹 문제를 해결하기 위한 유사한 목표를 가지지만, 구현 방식에 차이가 있다.
- **Docker와 CNI 플러그인 사용**: Docker는 CNI 플러그인을 직접적으로 지원하지 않지만, 우회적인 방법을 통해 사용할 수 있다.
  - Docker 컨테이너를 네트워크 설정 없이 생성한다.
  - 수동으로 Bridge 플러그인을 호출하여 컨테이너 네트워크를 설정한다.
  - Kubernetes는 이러한 방식으로 Docker 컨테이너와 CNI 플러그인을 함께 사용한다.

### 핵심 개념

- **CNI**: 컨테이너 런타임 환경에서 네트워크 문제를 해결하기 위한 표준 인터페이스다.
- **CNI 플러그인**: 컨테이너 네트워크 설정을 담당하는 프로그램이다.
- **상호 운용성**: 다양한 컨테이너 런타임과 플러그인 간의 상호 운용성을 보장한다.
- **표준화**: 일관된 네트워킹 환경을 제공하고 플러그인 개발을 단순화한다.
- **CNM**: Docker는 CNI 대신 CNM(Container Network Model)이라는 자체 표준을 사용한다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)