# Networking (Service Networking)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Service Networking"에 대해서 자세하게 알아보도록 한다.

---

## Service Networking

### 파드 네트워킹

![1-service-networking-1.png](images%2F1-service-networking-1.png)

- 각 노드에 비리지 네트워크를 생성한다.
- 파드를 위한 네트워크 네임스페이스를 생성한다.
- 네임스페이스에 인터페이스를 연결한다.
- 노드 서브넷 내에서 파드에 IP 주소를 할당한다.
- 라우팅 또는 오버레이 기술을 통해 다른 노드의 파드와 통신한다.

### 서비스 네트워킹의 필요성

- 파드 간 직접 통신은 드물게 사용된다.
- 서비스를 통해 다른 파드에 호스팅된 서비스에 접근한다.
- 서비스는 파드의 IP 주소 및 이름을 추상화하여 제공한다.

### 서비스 유형

#### ClusterIP

![2-clusterip-1.png](images%2F2-clusterip-1.png)

- 클러스터 내부에서만 접근 가능한 서비스다.
- 서비스에 IP 주소 및 이름을 할당한다.
- 클러스터 내의 모든 파드에서 접근이 가능하다.
- 특정 노드에 바인딩되지 않으며, 클러스터 전체에 호스팅된다.

#### NodePort

![3-nodeport-1.png](images%2F3-nodeport-1.png)

- 클러스터 외부에서 접근 가능한 서비스다.
- ClusterIP와 동일하게 작동하며, 추가적으로 클러스터의 모든 노드에서 특정 포트를 통해 애플리케이션을 노출한다.
- 외부 사용자 및 애플리케이션이 서비스에 접근 가능하다.

### 서비스 네트워킹 작동 방식

![4-service-networking-2.png](images%2F4-service-networking-2.png)

- **kubelet**
  - 각 쿠버네티스 노드에서 실행되는 프로세스다.
  - kube API 서버를 통해 클러스터의 변경 사항을 감지한다.
  - 새로운 파드 생성 시 파드 생성 및 CNI 플러그인을 호출하여 파드 네트워킹을 설정한다.
- **kube-proxy**
  - 각 쿠버네티스 노드에서 실행되는 컴포넌트다.
  - kube API 서버를 통해 클러스터 변경 사항을 감시한다.
  - 새로운 서비스 생성 시 동작한다.
  - 서비스는 각 노드에 생성되거나 할됭되지 않으며, 클러스터 전체의 개념이다.
  - 서비스는 실제로 존재하지 않으며 가상의 객체다.
  - 서비스를 위한 프로세스, 네임스페이스, 인터페이스는 없다.
- **서비스 IP 주소 할당**
  - 서비스 객체 생성 시 미리 정의된 범위에서 IP 주소가 할당된다.
  - kube-proxy는 각 노드에서 서비스 IP 주소를 가져와 포워딩 규칙을 생성한다.
  - 포워딩 규칙: "서비스 IP 주소로 들어오는 모든 트래픽은 파드 IP 주소로 전달"
  - 파드는 클러스터의 모든 노드에서 접근이 가능하다.
- **IP 및 포트 조합**
  - 서비스는 IP 주소와 포트 조합을 사용한다.
  - 서비스 생성 또는 삭제 시 kube-proxy는 포워딩 규칙을 생성하거나 삭제한다.

### kube-proxy의 작동 방식

![5-service-networking-3.png](images%2F5-service-networking-3.png)

- **kube-proxy 모드**: kube-proxy는 서비스 트래픽을 처리하기 위해 다양한 모드를 지원한다.
  - **userspace**: kube-proxy가 각 서비스에 대한 포트를 수신하고 파드에 연결을 프록시한다.
  - **IPVS**: IPVS 규칙을 생성하여 파드에 트래픽을 분산한다.
  - **iptables (기본값)**: iptables 규칙을 사용하여 트래픽을 포워딩 한다.
- **proxy mode 옵션**: kube-proxy 서비스 구성 시 `proxy-mode` 옵션을 사용하여 모드를 설정할 수 있다. 
  - 설정하지 않으면 기본적으로 iptables 모드가 사용된다.

### iptables를 사용한 서비스 네트워킹 규칙 생성

![6-service-networking-4.png](images%2F6-service-networking-4.png)

- **파드 및 서비스 생성**: 파드(예: DB)를 노드에 배포하고 ClusterIP 유형의 서비스를 생성한다.
- **서비스 IP 주소 할당**: 쿠버네티스는 `service cluster IP range` 옵션에 지정된 범위에서 서비스에 IP 주소를 할당한다. (기본값: 10.0.0.0/24, 예시: 10.96.0.0/12)
- **파드 IP 주소 할당**: 파드는 `pod network CIDR range` 옵션에 지정된 범위에서 IP 주소를 할당받는다. (예시: 10.244.0.0/16)
- **IP 주소 범위 중복 방지**: 서비스 및 파드 IP 주소 범위는 중복되지 않아야 한다.

![7-service-networking-5.png](images%2F7-service-networking-5.png)

- **iptables NAT 테이블 규칙 생성**: kube-proxy는 iptables NAT 테이블에 규칙을 생성하여 서비스 IP 주소로 들어오는 트래픽을 파드 IP 주소로 포워딩한다.
- **DNAT 규칙**: DNAT (Destination Network Address Translation) 규칙을 사용하여 대상 주소 및 포트를 파드 IP 주소 및 포트로 변경한다.
- **서비스 이름 주석**: kube-proxy가 생성한 모든 규칙에는 서비스 이름이 주석으로 포함된다.
- **NodePort 서비스 규칙 생성**: NodePort 유형의 서비스를 생성하면 kube-proxy는 모든 노드의 특정 포트로 들어오는 트래픽을 백엔드 파드로 포워딩하는 iptables 규칙을 생성한다.

![8-service-networking-6.png](images%2F8-service-networking-6.png)

- **kube-proxy 로그 확인**: kube-proxy 로그에서 iptables 규칙 생성 내용을 확인할 수 있다.
  - 사용된 프록시 종류 (iptables)
  - 새 서비스 추가시 로그 항목
- **로그 파일 위치 및 상세 수준**: 로그 파일 위치는 설치 환경에 따라 다를 수 있으며, 로그 상세 수준을 조정해야 로그 항목을 확인할 수 있다.

### 핵심 개념

- **kube-proxy**: 서비스 트래픽 포워딩을 담당하는 쿠버네티스 컴포넌트다.
- **iptables**: Linux 방화벽 및 네트워크 주소 변환 도구다.
- **DNAT (Destination Network Address Translation)**: 대상 주소 및 포트 변경이다.
- **service cluster IP range**: 서비스 IP 주소 할당 범위다.
- **pod network CIDR range**: 파드 IP 주소 할당 범위다.
- **NodePort**: 클러스터 외부에서 접근 가능한 서비스 유형이다.

### 추가 사항

- kube-proxy는 iptables 외에 userspace 및 IPVS 모드도 지원한다.
- 서비스 네트워킹 규칙은 iptables NAT 테이블에서 확인할 수 있다.
- kube-proxy 로그를 통해 서비스 네트워킹 관련 문제를 해결할 수 있다.
- 서비스 IP 주소 및 파드 IP 주소 범위는 중복되지 않도록 설정해야 한다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)