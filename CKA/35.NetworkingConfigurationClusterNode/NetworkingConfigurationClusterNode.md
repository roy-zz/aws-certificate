# Networking (Cluster Networking)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Cluster Networking"에 대해서 자세하게 알아보도록 한다.

---

## Cluster Networking

### 쿠버네티스 클러스터 네트워크 기본 요구 사항

![1-ip-fqdn-1.png](images%2F1-ip-fqdn-1.png)

- **노드 구성**:
  - 쿠버네티스 클러스터는 마스터 노드와 워커 노드로 구성된다.
  - 각 노드는 네트워크에 연결된 하나 이상의 인터페이스를 가져야 한다.
- **IP 주소 설정**:
  - 각 인터페이스에는 IP 주소가 구성되어야 한다.
- **호스트 이름 및 MAC 주소**:
  - 각 호스트는 고유한 호스트 이름을 설정해야 한다.
  - 각 호스트는 고유한 MAC 주소를 가져야 한다. (특히 VM을 복제하여 생성한 경우 주의해야 한다.)

### 마스터 노드 및 워커 노드에서 열어야 하는 포트

#### 마스터 노드

![2-ports-1.png](images%2F2-ports-1.png)

- **6443 (API 서버)**:
  - 워커 노드, `kubectl` 도구, 외부 사용자 및 기타 컨트롤 플레인 구성 요소가 kube API 서버에 액세스하는 데 사용된다.
- **10250 (kubelet)**:
  - 마스터 노드와 워커 노드의 kubelet이 수신하는 포트다.
- **10259 (kube-scheduler)**:
  - kube-scheduler가 사용하는 포트다.
- **10257 (kbe-controller-manager)**:
  - kube-controller-manager가 사용하는 포트다.
- **2379 (etcd)**:
  - etcd 서버가 수신하는 포트다.

#### 워커 노드

![3-ports-2.png](images%2F3-ports-2.png)

- **10250 (kubelet)**:
  - 워커 노드의 kubelet이 수신하는 포트다.
- **30000 ~ 32767 (NodePort 서비스)**:
  - 외부 액세스를 위해 워커 노드가 서비스를 노출하는 포트 범위다.

#### 다중 마스터 노드 구성

![4-ports-3.png](images%2F4-ports-3.png)

- 다중 마스터 노드 구성에서는 위 포트 외에 2380 포트도 열어야 한다.
- **2380 (etcd 클라이언트 통신)**:
  - etcd 클라이언트 간 통신에 사용된다.

#### 네트워킹 구성 시 고려 사항

- **방화벽 및 IP Table 규칙**:
  - 노드 간 통신을 허용하도록 방화벽 및 IP Table 규칙을 설정해야 한다.
- **클라우드 환경의 네트워크 보안 그룹**:
  - GCP, Azure, AWS와 같은 클라우드 환경에서는 네트워크 보안 그룹을 사용하여 포트 액세스를 제어해야 한다.

![5-documentation-1.png](images%2F5-documentation-1.png)

- **문제 해결**:
  - 쿠버네티스 클러스터가 제대로 작동하지 않으면 포트 설정 및 네트워크 구성을 확인해야 한다.
  - 쿠버네티스 공식 문서를 참고하여 포트 목록을 확인하는 것이 좋다.

### 핵심 개념

- **마스터 노드 및 워커 노드**: 쿠버네티스 클러스터의 기본 구성 요소다.
- **kubelet, kube-scheduler, kube-controller-manager, etcd**: 쿠버네티스 컨트롤 플레인의 핵심 구성 요소다.
- **NodePort 서비스**: 외부에서 쿠버네티스 서비스에 접근할 수 있도록 하는 서비스 유형이다.
- **방화벽 및 네트워크 보안 그룹**: 네트워크 트래픽을 제어하는 데 사용된다.

---

## 시험에서 CNI의 중요성

### 네트워크 애드온의 중요성

- 애드온을 사용하여 쿠버네티스 클러스터의 네트워킹 기능을 확장한다.
- weave-net을 포함한 다양한 네트워크 플러그인을 사용할 수 있다.

### 사용 가능한 네트워크 플러그인

- 쿠버네티스 공식 문서에서 사용 가능한 네트워크 플러그인 목록을 확인할 수 있다.
  - https://kubernetes.io/docs/concepts/cluster-administration/addons/
  - https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model

### CKA 시험 관련 사항

- CAK 시험에서 네트워크 애드온 배포 문제가 출제될 경우, 별도의 지시가 없다면 위 링크에 설명된 플러그인 중 어느 것이든 사용할 수 있다.
- 현재 쿠버네티스 문서에서는 타사 네트워크 애드온 배포 명령어가 직접적으로 포함되어 있지 않다.
- 위 링크는 타사/벤더 사이트 또는 GitHub 저장소로 연결되는데, 이는 시험에서 사용할 수 없다. 쿠버네티스 문서의 벤더 중립성을 유지하기 위해 의도적으로 이렇게 구성되었다.
- 공식 시험에서는 모두 필수 CNI 배포 세부 정보가 제공된다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)
- [Check required ports](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)