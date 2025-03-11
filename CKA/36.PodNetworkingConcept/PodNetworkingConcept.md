# Networking (Pod Networking)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Pod Networking"에 대해서 자세하게 알아보도록 한다.

---

## Pod Networking

### 쿠버네티스 파드 네트워킹의 필요성

![1-pod-networking-1.png](images%2F1-pod-networking-1.png)

- **노드 간 네트워크 구성**: 쿠버네티스 클러스터의 노드들은 서로 통신할 수 있도록 네트워크로 연결되어 있다.
- **컨트롤 플레인 구성 요소 통신**: 방화벽 및 네트워크 보안 그룹 설정을 통해 컨트롤 플레인 구성 요소 간 통신을 허용한다.
- **파드 네트워킹의 중요성**: 파드는 쿠버네티스 클러스터에서 애플리케이션을 실행하는 기본 단위다.
  - 파드 간 통신은 클러스터의 핵심 기능이다.

![2-networking-model-2.png](images%2F2-networking-model-2.png)

- **쿠버네티스 파드 네트워킹 요구 사항**
  - 모든 파드는 고유한 IP 주소를 가져야 한다.
  - 동일 노드의 파드는 IP 주소를 사용하여 통신할 수 있어야 한다.
  - 다른 노드의 파드도 동일한 IP 주소를 사용하여 서로 통신할 수 있어야 한다.
  - NAT 규칙 없이 통신이 가능해야 한다.

### 파드 네트워킹 구현 방법

#### 네트워크 네임스페이스 및 브리지 네트워크

![3-networking-model-3.png](images%2F3-networking-model-3.png)

- 각 파드는 네트워크 네임스페이스를 사용하여 격리된 네트워크 환경을 갖는다.

![4-networking-model-4.png](images%2F4-networking-model-4.png)

- 각 노드에는 브리지 네트워크를 생성하여 파드의 네트워크 네임스페이스를 연결한다.

#### IP 주소 할당

![5-networking-model-5.png](images%2F5-networking-model-5.png)

- 각 브리지 네트워크는 고유한 서브넷을 갖는다. (예: `10.240.1.0/24`, `10.240.2.0/24`, `10.240.3.0/24`)
- 각 파드는 네트워크 네임스페이스와 브리지 네트워크 사이에 veth 쌍을 생성한다.

#### 가상 이더넷 쌍(veth) 연결

![6-networking-model-6.png](images%2F6-networking-model-6.png)

- 각 파드의 네트워크 네임스페이스와 브리지 네트워크 사이에 veth 쌍을 생성한다.

#### 라우팅 테이블 생성

![7-networking-model-7.png](images%2F7-networking-model-7.png)

![8-networking-model-8.png](images%2F8-networking-model-8.png)

- 각 노드의 라우팅 테이블에 다른 노드의 서브넷 정보를 추가하여 파드 간 통신을 가능하게 한다.
- 라우터가 있는 경우, 라우터에 라우팅 정보를 설정하여 노드 간 통신을 관리할 수 있다.

#### 스크립트 자동화

![9-networking-model-9.png](images%2F9-networking-model-9.png)

- 파드 생성 시 자동으로 실행되는 스크립트를 작성하여 veth 쌍 생성, IP 주소 할당, 라우팅 테이블 설정 등을 자동화할 수 있다.

### CNI (Container Network Interface) 활용

![10-networking-model-10.png](images%2F10-networking-model-10.png)

- **CNI의 역할**: CNI는 쿠버네티스가 파드 네트워킹을 설정하는 데 사용하는 표준 인터페이스다.
- **CNI 플러그인**: CNI 플러그인은 파드 네트워킹 설정을 담당하는 프로그램이다. (예: 브리지 플러그인)
- **CNI 구성**:
  - CNI 구성 파일은 파드 생성 시 사용할 CNI 플러그인을 지정한다.
  - CNI 플러그인은 `add` 및 `del` 명령을 지원하여 파드 네트워크 설정을 추가하고 제거한다.
- **파드 생성 시 CNI 플러그인 실행**
  - 컨테이너 런타임은 파드 생성 시 CNI 구성을 확인하고 해당 CNI 플러그인을 실행한다.
  - CNI 플러그인은 파드 네트워크 설정을 자동으로 수행한다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)