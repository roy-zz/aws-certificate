# Kubernetes Architecture

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "쿠버네티스 아키텍처"에 대해서 알아보도록 한다.

---

### Nodes

![1-nodes-minions.png](images%2F1-nodes-minions.png)

- 노드는 쿠버네티스가 설치된 물리적 또는 가상 시스템으로, 작업자 시스템이고 컨테이너는 쿠버네티스에 의해 시작된다.
- 예전에는 미니언즈라고도 불렸으며 이러한 용어들이 상호 교환적으로 사용될 수 있다.
- 우리는 애플리케이션이 실행되고 있는 노드가 실패할 수도 있기 때문에, 두 대 이상의 노드를 구성하는 것이 권장된다.

---

### Cluster

![2-cluster.png](images%2F2-cluster.png)

- 클러스터는 함께 그룹화된 노드의 집합니다.
- 이렇게 구성되는 경우 하나의 노드가 실패하더라도 사용자는 다른 노드에서 실행되는 애플리케이션에 계속 액세스할 수 있다.
- 여러 노드를 사용하면 로드를 공유하는 데 도움이 된다.

---

### Master

![3-master.png](images%2F3-master.png)

- 마스터는 쿠버네티스가 설치된 또 다른 노드이며 마스터로 구성된다.
- 마스터는 클러스터의 노드를 감시하고 작업자 노드에서 컨테이너의 실제 오케스트레이션을 담당한다.
- 실제로 클러스터 구성원에 대한 정보를 저장하고 있으며, 노드가 실패하는 경우 워크로드를 다른 작업자 노드로 이동시키는 역할을 한다.

---

### 구성 요소

![4-contents.png](images%2F4-contents.png)

- 시스템에 쿠버네티스를 설치하면 API 서버, ETCD, Kubelet, Container Runtime, Controller, Scheduler와 같은 구성 요소들이 설치된다.

#### API 서버

- 쿠버네티스의 프론트엔드 역할을 한다.
- 사용자, 관리 장치, 명령줄 인터페이스는 모두 API 서버와 통신하여 쿠버네티스 클러스터와 상호 작용한다.

#### ETCD

- ETCD는 클러스터 관리에 사용되는 모든 데이터를 저장하기 위해 쿠버네티스에서 사용하는 신뢰할 수 있는 분산 Key-Value 저장소다.
- 클러스터에 여러 노드와 여러 마스터가 있는 경우 ETCD는 클러스터의 모든 노드에 대한 모든 정보를 분산 방식으로 저장한다.
- ETCD는 마스터 간에 충돌이 발생하지 않도록 클러스터 내에서 잠금을 구현하는 역할을 담당한다.

#### Scheduler

- 스케줄러는 여러 노드에 작업이나 컨테이너를 배포하는 역할을 하며, 새롭게 생성된 컨테이너를 찾아 노드에 할당한다.
- 컨트롤러는 오케스트레이션의 두뇌이며, 노드, 컨테이너 또는 엔드포인트가 다운될 때 이를 인지하고 대응하는 역할을 담당하며, 이러한 경우 컨트롤러는 새로운 컨테이너를 가져오는 결정을 한다.

#### Container Runtime

- 컨테이너 런타임은 컨테이너를 실행하는 데 사용되는 기본 소프트웨어로 도커가 많이 사용된다.

#### Kubelet

- Kubelet은 클러스터의 각 노드에서 실행되는 에이전트다.
- 에이전트는 컨테이너가 예상대로 노드에서 실행되고 있는지 확인하는 역할을 담당한다.

---

### 마스터 노드 & 워커 노드

![5-master-worker-nodes.png](images%2F5-master-worker-nodes.png)

- 도커 컨테이너가 있고 시스템에서 도커 컨테이너를 실행하려면 컨테이너 런타임이 설치되어 있어야 한다. 이미지에서는 컨테이너 런타임으로 도커가 설치되어 있다.
- 반드시 도커일 필요는 없으며 Rocket 또는 CRIO와 같은 다른 컨테이너 런타임이라는 대안이 있다.
- 마스터 서버에는 kube-apiserver가 있으므로 이를 마스터로 만든다.
- 마찬가지로 워커 노드에는 워커 노드의 상태 정보를 제공하고 워커 노드에서 마스터가 요청한 작업을 수행하기 위해 마스터와 상호 작용하는 역할을 담당하는 kubelet 에이전트가 있다.
- 수집된 모든 정보는 마스터의 Key-Value 저장소에 저장되며, Key-Value 저장소는 ETCD 프레임워크를 기반으로 한다.
- 마스터에는 컨트롤러 관리자와 스케쥴러가 있다.

---

### Kubectl

![6-kubectl.png](images%2F6-kubectl.png)

- kubectl은 쿠버네티스 클러스터에서 애플리케이션 배포 및 관리하고, 클러스터의 정보를 가져오며, 클러스터의 노드 상태를 가져오는 등 다양한 작업에 사용된다.
- `kubectl run` 명령은 클러스터에 애플리케이션을 배포하는 데 사용된다.
- `kubectl cluster-info` 명령은 클러스터에 대한 정보를 보는 데 사용되며 `kubectl get pod` 명령은 클러스터의 모든 노드 부분을 나열하는데 사용된다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)