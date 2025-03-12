# Networking (CNI)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Container Network Interface (CNI)"에 대해서 자세하게 알아보도록 한다.

---

## CNI (Container Network Interface)

### CNI의 역할과 쿠버네티스의 책임

![1-container-network-interface.png](images%2F1-container-network-interface.png)

- **CNI의 역할**: CNI는 컨테이너 런타임이 컨테이너 네트워크 네임스페이스를 생성하고 네트워크에 연결하는 방법을 정의한다.
- **쿠버네티스의 책임**: 쿠버네티스는 CNI 표준에 따라 컨테이너 네트워크 네임스페이스를 생성하고, 적절한 네트워크 플러그인을 호출하여 컨테이너를 네트워크에 연결해야 한다.

### CNI 플러그인 설치 위치

![2-configuring-cni.png](images%2F2-configuring-cni.png)

- **`/opt/cni/bin` 디렉토리**: CNI 플러그인 실행 파일은 이 디렉토리에 설치된다.
  - **예시 플러그인**: bridge, DHCP, flannel 등 다양한 플러그인 실행 파일이 존재한다.
- **`/etc/cni/net.d` 디렉토리**: CNI 플러그인 구성 파일은 이 디렉토리에 위치한다.
  - **구성 파일**: 각 플러그인에 대한 구성 파일이 존재하며, 컨테이너 런타임은 이 파일을 참조하여 사용할 플러그인을 결정한다.
  - **파일 선택 순서**: 여러 구성 파일이 존재할 경우, 알파벳 순서로 파일이 선택된다.

### CNI 플러그인 확인

![3-view-kubelet-options-1.png](images%2F3-view-kubelet-options-1.png)

- `ps -aux | grep kubelet` 명령어를 사용하여 현재 kubelet에 적용되어 있는 CNI 옵션을 확인할 수 있다.


### CNI 플러그인 구성 파일 내용 (예: bridge)

![4-view-kubelet-options-2.png](images%2F4-view-kubelet-options-2.png)

- **JSON 형식**: 구성 파일은 CNI 표준에 정의된 JSON 형식을 따른다.
- **주요 설정 항목**:
  - `name`: 네트워크 이름 (예: "mynet")
  - `type`: 플러그인 유형 (예: "bridge")
  - `isGateway`: 브리지 인터페이스에 IP 주소를 할당하여 게이트웨이 역할을 수행할지 여부
  - `ipMasq`: IP 마스커레이딩(NAT) 규칙을 추가할지 여부
  - `ipam`: IP 주소 관리(IPAM) 구성
  - `bridge`: 브리지 네트워크 구성`
    - `type`: IPAM 유형 (예: "host-local", "dhcp")
    - `subnet` 또는 `range`: 파드에 할당된 IP 주소 서브넷 또는 범위
    - `routes`: 파드에 필요한 라우팅 정보
- **`host-local` IPAM**: 로컬 호스트에서 IP 주소를 관리한다.
- **`dhcp` IPAM**: 외부 DHCP 서버를 사용하여 IP 주소를 관리한다.

### 쿠버네티스의 CNI 플러그인 사용 과정

- **컨테이너 런타임**: 쿠버네티스의 컨테이너 런타임(Containerd, CRI-O 등)이 컨테이너 생성 시 CNI 플러그인을 호출한다.
- **CNI 구성 확인**: 컨테이너 런타임은 `/etc/cni/net.d` 디렉토리에서 사용할 플러그인 구성 파일을 찾는다.
- **플러그인 실행**: 컨테이너 런타임은 `/opt/cni/bin` 디렉토리에서 해당 플러그인 실행 파일을 찾아 실행한다.
- **플러그인 설정 적용**: 플러그인은 구성 파일에 정의된 설정을 적용하여 컨테이너 네트워크를 설정한다.

### 핵심 개념

- **CNI (Container Network Interface)**: 컨테이너 네트워크 네임스페이스를 생성하고 네트워크에 연결하는 방법을 정의하는 표준 인터페이스
- **플러그인**: 컨테이너 네트워크 설정을 담당하는 프로그램 (bridge, DHCP, flannel 등)
- `/opt/cni/bin`: CNI 플러그인 실행 파일이 위치하는 디렉토리
- `/etc/cni/net.d`: CNI 플러그인 구성 파일이 위치하는 디렉토리
- **IPAM (IP Address Management)**: IP 주소 할당 및 관리를 담당하는 서브시스템 (host-local, DHCP 등)

---

## CNI Weave

### 기존 파드 네트워킹 방식의 한계

![5-cni-weave-1.png](images%2F5-cni-weave-1.png)

- **라우팅 테이블 관리의 어려움**: 수백 개의 노드와 수천 개의 파드가 있는 대규모 환경에서는 라우팅 테이블 관리가 매우 복잡하고 비효율적이다.
- **라우터 설정의 복잡성**: 각 노드의 라우팅 테이블을 수동으로 설정하거나 라우터에 복잡한 라우팅 규칙을 설정하는 것은 확장성이 떨어진다.

### Weave CNI 플러그인의 작동 방식

![6-cni-weave-2.png](images%2F6-cni-weave-2.png)

- **Weave Peer 에이전트**: 각 노드에 Weave Peer 에이전트(서비스 또는 데몬)를 배포한다.

![7-cni-weave-3.png](images%2F7-cni-weave-3.png)

- **에이전트 간 통신**: Weave Peer 에이전트는 서로 통신하여 노드, 네트워크, 파드에 대한 정보를 교환하고 전체 클러스터의 토폴로지를 구축한다.

![8-cni-weave-4.png](images%2F8-cni-weave-4.png)

- **Weave 브리지 네트워크**: 각 노드에 Weave 브리지 네트워크(예: `weave`)를 생성하고 IP 주소를 할당한다.

![9-cni-weave-5.png](images%2F9-cni-weave-5.png)

- **파드 라우팅 설정**: Weave는 파드에 에이전트로 향하는 올바른 라우팅 경로를 설정하으 파드가 에이전트를 통해 다른 파드와 통신할 수 있도록 한다.
- **패킷 캡슐화 및 역캡슐화**:
  - 다른 노드의 파드로 패킷을 전송할 때 Weave는 패킷을 캡슐화하여 새로운 소스 및 대상 주소를 가진 패킷으로 만든다.
  - 대상 노드의 Weave 에이전트는 패킷을 역캡슐화하여 원래 패킷을 복원하고 대상 파드로 전달한다.

### Weave 배포 방법

![10-deploy-weave.png](images%2F10-deploy-weave.png)

- **수동 배포**: 각 노드에 Weave 서비스 또는 데몬을 수동으로 배포할 수 있다.
- **쿠버네티스 배포**: 쿠버네티스 클러스터가 이미 설정된 경우 `kubectl apply` 명령어를 사용하여 Weave를 파드로 배포하는 것이 더 쉽다.
- **DaemonSet 배포**: Weave Peer 에이전트는 DaemonSet으로 배포되어 클러스터의 모든 노드에 하나의 파드가 배포되도록 한다.
- **Kubeadm 설치**: Kubeadm 도구를 사용하여 쿠버네티스 클러스터를 설치하고 Weave 플러그인을 선택하면 Weave Peer 파드가 각 노드에 자동으로 배포된다.

### 문제 해결

![11-weave-peers.png](images%2F11-weave-peers.png)

- **로그 확인**: `kubectl logs` 명령어를 사용하여 Weave Peer 파드의 로그를 확인하여 문제를 해결할 수 있다.

### 핵심 개념

- **Weave Peer 에이전트**: 노드 간 통신 및 파드 라우팅을 담당하는 Weave 구성 요소다.
- **Weave 브리지 네트워크**: 파드 간 통신을 위한 가상 스위치 역할을 한다.
- **패킷 캡슐화 및 역캡슐화**: 다른 노드의 파드로 패킷을 전송하는 데 사용되는 기술이다.
- **DaemonSet**: 클러스터의 모든 노드에 하나의 파드를 배포하는 쿠버네티스 클러스터다.

### 추가 사항

- Weave는 복잡한 라우팅 테이블 관리 없이 파드 간 통신을 가능하게 하는 효과적인 CNI 플러그인이다.
- Weave Peer 에이전트 간 통신을 통해 클러스터 전체의 네트워크 토폴로지를 관리한다.
- Weave 배포는 쿠버네티스 환경에서 매우 간단하며, DaemonSet을 사용하여 모든 노드에 에이전트를 자동으로 배포할 수 있다.

---

## Weave Cloud 서비스 종료

- Weaveworks에서 Weave Cloud 서비스를 종료했다.
- 자세한 내용은 다음 게시물에서 확인할 수 있다.
  - https://www.weave.works/blog/weave-cloud-end-of-service

### 기존 설치 링크 작동 중단

- 기존 Weave Net 설치 링크(`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`)는 더 이상 작동하지 않는다.

### 새로운 설치 링크 제공

- 다음 최신 링크를 사용하여 Weave Net을 설치해야 한다: `kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml`
- 자세한 내용은 아래의 게시물을 확인한다.
  - https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation
  - https://github.com/weaveworks/weave/releases

---

## IP Address Management (IPAM)

### IP 주소 관리 범위

![12-ipam-1.png](images%2F12-ipam-1.png)

- **노드 IP 주소**: 사용자가 직접 관리하거나 외부 IPAM 솔루션을 사용할 수 있다.
- **가상 브리지 네트워크 및 파드 IP 주소**: 노드의 가상 브리지 네트워크에 IP 서브넷을 할당하고 파드에 IP 주소를 할당할 수 있다.

### IP 주소 할당 책임

![13-ipam-2.png](images%2F13-ipam-2.png)

- **CNI 플러그인**: CNI 표준에 따라 CNI 플러그인(네트워크 솔루션 제공 업체)이 컨테이너에 IP 주소를 할당할 책임을 가진다.
- **자체 플러그인**: 자체 제작된 CNI 플러그인에서도 파드에 IP 주소를 할당할 수 있다. 

### IP 주소 관리 방법

![14-ipam-3.png](images%2F14-ipam-3.png)

- **중복 IP 주소 방지**: 쿠버네티스는 IP 주소 관리 방법을 지정하지 않지만, 중복 IP 주소를 할당하지 않고 적절하게 관리해야 한다.
- **파일 저장**: IP 주소 목록을 파일에 저장하고 스크립트에서 파일을 관리하는 방식으로 IP 주소를 관리할 수 있다.

![15-ipam-4.png](images%2F15-ipam-4.png)

- **CNI 내장 플러그인**: CNI는 호스트 로컬(host-local) 및 DHCP와 같은 두 가지 내장 플러그인을 제공하여 IP 주소 관리를 아웃소싱할 수 있다.

### 호스트 로컬(host-local) 플러그인

![16-ipam-5.png](images%2F16-ipam-5.png)

- **로컬 관리**: 각 호스트에서 IP 주소를 로컬로 관리한다.
- **CNI 구성 파일**: CNI 구성 파일의 `ipam` 섹션에서 사용할 플러그인 유형, 서브넷, 라우팅 정보를 지정할 수 있다.
- **동적 플러그인 지원**: 스크립트를 동적으로 만들어 다양한 플러그인을 지원할 수 있다.

### Weaveworks IP 주소 관리

![17-ipam-6.png](images%2F17-ipam-6.png)

- **기본 IP 범위**: Weave는 기본적으로 `10.32.0.0/12` IP 범위를 전체 네트워크에 할당한다.
- **노드별 IP 범위 분할**: Weave Peer는 IP 범위를 노드 수에 따라 균등하게 분할하고 각 노드에 IP 범위를 할당한다.
- **파드 IP 주소 할당**: 각 노드의 파드는 해당 노드에 할당된 IP 범위에서 IP 주소를 할당받는다.
- **구성 가능**: Weave 플러그인 배포 시 추가 옵션을 사용하여 IP 범위를 구성할 수 있다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)