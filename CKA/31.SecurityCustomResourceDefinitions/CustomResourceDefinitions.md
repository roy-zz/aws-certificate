# Security (Custom Resource Definitions)

- 이번 장에서는 **Certified Ku
- bernetes Administrator (CKA)** 을 준비하며 "Custom Resource Definitions(CRD)"에 대해서 자세하게 알아보도록 한다.

---

## Custom Resource Definition (CRD)

### 쿠버네티스 리소스의 기본 이해

#### 리소스와 ETCD

![1-resource-controller-1.png](images%2F1-resource-controller-1.png)

- 쿠버네티스에서 리소스(예: Deployment)를 생성하면 해당 정보는 ETCD 데이터 저장소에 저장된다.
- `kubectl create`, `kubectl get`, `kubectl delete` 등의 명령어를 통해 ETCD에 저장된 리소스를 생성, 조회, 삭제할 수 있다.

#### 컨트롤러의 역할

![2-resource-controller-2.png](images%2F2-resource-controller-2.png)

- Deployment를 생성하면 설정된 복제본 수만큼 Pod가 생성된다. 이 작업은 Deployment 컨트롤러가 수행한다.
- 컨트롤러는 백그라운드에서 실행되는 프로세스로 관리 대상 리소스의 상태를 지속적으로 모니터링한다.
- 리소스의 생성, 수정, 삭제에 따라 클러스터 상태를 원하는 상태로 유지하기 위한 작업을 수행한다.
- 예: Deployment 컨트롤러는 ReplicaSet을 생성하고, ReplicaSet은 Pod를 생성한다.

#### 기본 제공 리소스

![3-resource-controller-3.png](images%2F3-resource-controller-3.png)

- ReplicaSet, Deployment, Job, CronJob, StatefulSet, Namespace 등 많은 기본 제공 리소스가 있으며, 각 리소스에는 담당 컨트롤러가 존재한다.

### 사용자 정의 리소스(Custom Resource)의 필요성

#### 새로운 리소스 정의

![4-resource-controller-4.png](images%2F4-resource-controller-4.png)

- 기존 제공 리소스 외에 사용자 정의 리소스를 생성하여 특정 요구 사항을 충족할 수 있다.
- 예: 항공권 예약 시스템에서 `FlightTicket` 리소스를 정의하여 항공권 예약을 관리할 수 있다.

![5-resource-controller-5.png](images%2F5-resource-controller-5.png)

#### 사용자 정의 컨트롤러(Custom Controllers)의 역할

- **실제 동작 수행**:
    - 사용자 정의 리소스를 생성해도 실제 동작(예: 항공권 예약)은 수행되지 않는다.
    - 사용자 정의 컨트롤러를 통해 리소스 생성, 수정, 삭제 시 필요한 작업을 수행한다.
    - 예: `FlightTicket` 컨트롤러는 항공권 예약 API를 호출하여 실제 예약을 처리한다.
- **컨트롤러 구현**:
    - 컨트롤러는 Go 언어 등으로 구현되며, 리소스의 상태를 감시하고 필요한 작업을 수행한다.

### 사용자 정의 리소스 정의(Custom Resource Definition, CRD)

![6-custom-resource-1.png](images%2F6-custom-resource-1.png)

- **CRD의 필요성**
  - 사용자 정의 리소스를 사용하기 전에 CRD를 통해 쿠버네티스 API에 해당 리소스를 등록해야 한다.
  - CRD는 쿠버네티스에 새로운 리소스 종류를 알리는 역할을 한다.

![7-custom-resource-2.png](images%2F7-custom-resource-2.png)

- CRD 설정 항목
  - `scope`: 네임스페이스 범위 또는 클러스터 범위 설정
  - `group`: API 그룹 이름
  - `names`: 리소스 종류, 단수/복수 이름, 축약 이름 설정
  - `versions`: API 버전 설정 (alpha, beta, v1 등)
  - `schema`: 리소스 정의의 스키마 및 유효성 검사 규칙 설정 (OpenAPI v3 스키마 사용)

### CRD 생성 및 사용자 정의 리소스 사용

- **CRD 생성**:
  - `kubectl create -f <CRD 정의 파일>` 명령어를 사용하여 CRD를 생성한다.
- **사용자 정의 리소스 생성**:
  - CRD 생성 후 사용자 정의 리소스를 생성하고 조회, 삭제할 수 있다.
- **컨트롤러의 필요성**:
  - CRD는 리소스 정의 및 유효성 검사만 수행하며, 실제 동작은 컨트롤러가 담당한다.

---

## Custom Controller

### 사용자 정의 컨트롤러의 필요성

#### CRD와 컨트롤러

![8-custom-resource-custom-controller-1.png](images%2F8-custom-resource-custom-controller-1.png)

- CRD를 생성하여 사용자 정의 리소스(`FlightTicket`)를 정의하고 ETCD에 데이터를 저장할 수 있다.
- 하지만 실제 동작(항공권 예약 API 호출)은 컨트롤러를 통해 수행해야 한다.

#### 컨트롤러의 역할

- 컨트롤러는 ETCD에 저장된 사용자 정의 리소스의 상태를 감시하고, 변경 사항에 따라 필요한 작업을 수행한다.
- 예: `FlightTicket` 컨트롤러는 항공권 예약 API를 호출하여 항공권을 예약, 수정, 취소한다.

### 컨트롤러 개발 방법

#### 기본 개념

- 컨트롤러는 특정 객체의 변경 이벤트를 감시하고, 루프를 통해 지속적으로 작업을 수행하는 프로세스 또는 코드다.
- 이론적으로는 어떤 언어로든 컨트롤러를 개발할 수 있다. (예: Python)

#### Go 언어와 Kubernetes Go 클라이언트

![9-custom-resource-custom-controller-2.png](images%2F9-custom-resource-custom-controller-2.png)

- Python으로 컨트롤러를 개발하는 것은 API 호출 비용, 큐 및 캐싱 메커니즘 구현 등의 어려움이 있다.
- Go 언어와 Kubernetes Go 클라이언트를 사용하면 `shared informers`와 같은 라이브러리를 통해 캐싱 및 큐잉 메커니즘을 쉽게 구현할 수 있다.

#### 샘플 컨트롤러(sample-controller) 사용

![10-custom-controller-1.png](images%2F10-custom-controller-1.png)

- GitHub의 `sample-controller` 저장소를 클론하여 사용자 정의 컨트롤러 개발을 시작할 수 있다.
- `controller.go` 파일을 사용자 정의 로직으로 수정한다.
- Go 언어 설치가 필요하다.

#### 컨트롤러 코드 수정

- `controller.go` 파일에 사용자 정의 로직을 추가한다.
- 예: 항공권 예약 API를 호출하는 코드를 추가한다.

#### 컨트롤러 빌드 및 실행

![11-custom-controller-2.png](images%2F11-custom-controller-2.png)

- 수정된 `controller.go` 파일을 빌드한다.
- `kubeconfig` 파일을 지정하여 컨트롤러를 실행한다.
- 컨트롤러는 로컬에서 실행되며, `FlightTicket` 객체 생성을 감시하고 필요한 API 호출을 수행한다.

### 컨트롤러 배포

#### Docker 이미지 생성

![12-custom-controller-3.png](images%2F12-custom-controller-3.png)

- 사용자 정의 컨트롤러를 Docker 이미지로 패키징한다.

#### 쿠버네티스 클러스터 배포

![13-custom-controller-4.png](images%2F13-custom-controller-4.png)

- Docker 이미지를 사용하여 쿠버네티스 클러스터 내에 Pod 또는 Deployment로 컨트롤러를 배포한다.
- 클러스터 내에서 실행되면서 API를 감시하고 작업을 수행한다.

---

## Operator Framework

### CRD와 사용자 정의 컨트롤러의 결합

#### 기존 방식의 문제점

- CRD와 사용자 정의 컨트롤러는 별도의 엔티티로 관리되며, 수동으로 생성하고 배포해야 한다.
- CRD 생성, 리소스 생성, 컨트롤러 배포 등의 과정을 개별적으로 수행해야 한다.

#### 오퍼레이터 프레임워크의 도입

![14-operator-framework-1.png](images%2F14-operator-framework-1.png)

- 오퍼레이터 프레임워크를 사용하면 CRD와 사용자 정의 컨트롤러를 하나의 엔티티로 패키징하여 배포할 수 있다.
- 오퍼레이터를 배포하면 CRD, 리소스, 컨트롤러가 자동으로 생성 및 배포된다.
- 배포 과정을 자동화하고 관리 효율성을 높일 수 있다.

### 오퍼레이터의 기능 및 활용

#### 단순 배포 이상의 기능

- 오퍼레이터는 단순히 CRD와 컨트롤러를 배포하는 것 외에도 다양한 기능을 제공한다.

![15-operator-framework-2.png](images%2F15-operator-framework-2.png)

- 애플리케이션의 설치, 유지 관리, 백업/복구, 문제 해결 등의 작업을 자동화한다.

#### ETCD 오퍼레이터 예시

- EtcdCluster CRD와 사용자 정의 컨트롤러를 포함한다.
- EtcdCluster 리소스를 감시하고 쿠버네티스 클러스터 내에 etcd를 배포한다.
- 백업 및 복구 기능을 제공하며 Backup 및 Restore 오퍼레이터가 추가 작업을 수행한다.

#### 운영자 역할 자동화

- 오퍼레이터는 사람인 운영자가 특정 애플리케이션을 관리하는 작업을 자동화한다.
- 설치, 유지 관리, 백업/복구, 문제 해결 등의 작업을 자동화하여 운영 효율성을 높인다.

### 오퍼레이터 허브(OperatorHub) 및 설치

#### 다양한 오퍼레이터 제공

![16-operator-hub-1.png](images%2F16-operator-hub-1.png)

- OperatorHub에서 etcd, MySQL, Prometheus, Grafana, Argo CD, Istio 등 다양한 애플리케이션의 오퍼레이터를 찾을 수 있따.
- 각 오퍼레이터의 상세 정보 및 설치 지침을 확인할 수 있다.

#### 간편한 설치

![17-operator-hub-2.png](images%2F17-operator-hub-2.png)

- 오퍼레이터 설치는 3단계로 구성된다.
  - **Operator Lifecycle Manager (OLM) 설치**: 오퍼레이터 수명 주기를 관리한다.
  - **오퍼레이터 설치**: OperatorHub에서 원하는 오퍼레이터를 설치한다.
  - **사용자 정의 리소스 생성**: 오퍼레이터에서 제공하는 CRD를 사용하여 원하는 리소스를 생성한다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)