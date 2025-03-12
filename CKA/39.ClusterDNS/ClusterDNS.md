# Networking (Cluster DNS)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Cluster DNS"에 대해서 자세하게 알아보도록 한다.

---

## Cluster DNS

### 쿠버네티스 클러스터 DNS

![1-cluster-dns-1.png](images%2F1-cluster-dns-1.png)

- 쿠버네티스 클러스터 설정 시 기본적으로 내장 DNS 서버가 배포된다.
- 수동으로 쿠버네티스를 설정한 경우 직접 DNS 서버를 구성해야 한다.

![2-cluster-dns-2.png](images%2F2-cluster-dns-2.png)

- 클러스터 네트워킹이 올바르게 설정되어 모든 파드와 서비스가 IP 주소를 할당받고 서로 통신할 수 있어야 한다.

### 서비스 DNS 레코드

![3-cluster-dns-3.png](images%2F3-cluster-dns-3.png)

- **서비스 생성**: 웹 서버를 테스트 파드에서 접근할 수 있도록 서비스를 생성한다.
- **서비스 이름 및 IP 주소**: 서비스에 이름(예: `web-service`)과 IP 주소(예: `10.107.37.188`)가 할당된다.
- **DNS 레코드 생성**: 쿠버네티스 DNS 서비스는 서비스 이름과 IP 주소를 매핑하는 DNS 레코드를 생성한다.

![4-cluster-dns-4.png](images%2F4-cluster-dns-4.png)

![5-cluster-dns-5.png](images%2F5-cluster-dns-5.png)

- **동일 네임스페이스 내 접근**: 테스트 파드는 서비스 이름을 사용하여 웹 서비스에 접근할 수 있다.

![6-cluster-dns-6.png](images%2F6-cluster-dns-6.png)

- **다른 네임스페이스 접근**: 웹 서비스가 다른 네임스페이스(예: `app`)에 있는 경우, 테스트 파드는 `web-service.apps`와 같이 네임스페이스를 포함한 전체 이름을 사용해야 한다.

![7-cluster-dns-7.png](images%2F7-cluster-dns-7.png)

- **DNS 서브도메인**:
  - 네임스페이스별 서브도메인 생성 (예: `apps`)
  - 서비스는 `svc` 서브도메인에 그룹화된다.
  - 클러스터 루트 도메인은 기본적으로 `cluster.local`로 설정된다.
- **FQDN (Fully Qualified Domain Name)**: 서비스의 완전한 도메인 이름은 `web-service.apps.svc.cluster.local`과 같다.

### 파드 DNS 레코드

![8-cluster-dns-8.png](images%2F8-cluster-dns-8.png)

- **기본적으로 파드 레코드 생성 X**: 파드 레코드는 기본적으로 생성되지 않지만 명시적으로 활성화할 수 있다.
- **파드 이름 생성**: 파드 IP 주소의 점(`.`)을 대시(`-`)로 대체하여 이름을 생성한다. (예: `10.244.1.5` -> `10-244-1-5`)
- **파드 레코드 구성**
  - 파드 이름 (IP 주소 변환)
  - 네임스페이스 (예: `default`)
  - 레코드 유형 (예: `pod`)
  - 루트 도메인 (예: `cluster.local`)
- 예시: IP 주소가 `10.244.1.5`인 테스트 파드는 `10-244-1-5.default.pod.cluster.local`와 같은 DNS 레코드를 갖는다.

![9-cluster-dns-9.png](images%2F9-cluster-dns-9.png)

### 핵심 개념

- **DNS 서비스**: 쿠버네티스 클러스터 내에서 이름 해결을 제공한다.
- **서비스 DNS 레코드**: 서비스 이름과 IP 주소를 매핑한다.
- **파드 DNS 레코드**: 파드 IP 주소를 변환한 이름과 IP 주소를 매핑한다.
- **네임스페이스**: 클러스터 내에서 리소스를 그룹화한다.
- **서브도메인**: 네임스페이스 및 서비스별로 DNS 서브도메인을 구성한다.
- **FQDN**: 서비스 또는 파드의 완전한 도메인 이름이다.

---

## CoreDNS

### DNS 해결 방법의 진화

![10-implement-dns-1.png](images%2F10-implement-dns-1.png)

![11-implement-dns-2.png](images%2F11-implement-dns-2.png)

- **`/etc/hosts` 파일**: 초기에는 각 파드의 `/etc/hosts` 파일에 다른 파드의 IP 주소와 이름을 수동으로 추가하여 이름 해결을 구현하였다.
- **중앙 DNS 서버**: 수천 개의 파드가 있는 클러스터에서는 중앙 DNS 서버를 사용하여 이름 해결을 관리하는 것이 더 효율적이다.
- **`/etc/resolv.conf` 파일**: 각 파드의 `/etc/resolv.conf` 파일에 DNS 서버의 IP 주소를 설정하여 DNS 서버를 사용하도록 구성한다.

### 쿠버네티스 DNS 구현

![12-kube-dns.png](images%2F12-kube-dns.png)

- **kube-dns** (이전): 쿠버네티스 1.12 이전 버전에서는 kube-dns가 사용되었다.

![13-core-dns-1.png](images%2F13-core-dns-1.png)

- **CoreDNS**: 쿠버네티스 1.12 버전부터 CoreDNS가 기본 DNS 서버로 사용된다.

![14-core-dns-2.png](images%2F14-core-dns-2.png)

- **CoreDNS 파드**: CoreDNS는 `kube-system` 네임스페이스에 파드로 배포된다.
- **CoreDNS 구성 파일**: CoreDNS는 `/etc/coredns/Corefile` 구성 파일을 사용한다.
- **Corefile 플러그인**: Corefile에는 오류 처리, 상태 확인, 메트릭 모니터링, 캐싱 등 다양한 플러그인이 구성되어 있다.
- **쿠버네티스 플러그인**: CoreDNS가 쿠버네티스와 연동되도록 하는 핵심 플러그인이다.
  - **`cluster.local` 도메인**: 클러스터의 최상위 도메인을 설정한다.
  - **`pod` 옵션**: 파드 IP 주소를 대시로 변환하여 파드 레코드를 생성하는 기능을 활성화한다. (기본적으로 비활성화)

![15-core-dns-3.png](images%2F15-core-dns-3.png)

- **상위 DNS 서버**: CoreDNS는 해결할 수 없는 도메인 이름(예: `www.google.com`)을 CoreDNS 파드의 `/etc/resolv.conf` 파일에 지정된 상위 DNS 서버로 전달한다.
- **ConfigMap**: Corefile은 ConfigMap 객체로 파드에 전달되어 필요에 따라 구성을 수정할 수 있다.

### 파드의 DNS 설정

![16-core-dns-4.png](images%2F16-core-dns-4.png)

- **CoreDNS 서비스**: CoreDNS는 클러스터 내 다른 컴포넌트에서 접근할 수 있도록 서비스(기본적으로 `kube-dns`)로 배포된다.
- **파드의 `/etc/resolv.conf` 파일**: 파드 생성 시 쿠버네티스는 자동으로 파드의 `/etc/resolv.conf` 파일에 CoreDNS 서비스의 IP 주소를 네임서버로 설정한다.
- **kubelet**: kubelet이 파드의 DNS 설정을 담당한다.
- **kubelet 구성 파일**: kubelet 구성 파일에서 DNS 서버 IP 주소와 도메인을 확인할 수 있다.

### 서비스 및 파드 이름 해결

![17-core-dns-5.png](images%2F17-core-dns-5.png)

- **서비스 이름 해결**: 파다의 서비스 이름(예: `web-service`), 네임스페이스를 포함한 이름(예: `web-service.default`), 또는 FQDN(예: `web-service.default.svc.cluster.local`)을 사용하여 서비스에 접근할 수 있다.
- **`/etc/resolv.conf` 검색 항목**: 파드의 `/etc/resolv.conf` 파일에는 `default.svc.cluster.local`, `svc.cluster.local`, `cluster.local`과 같은 검색 항목이 설정되어 있어 축약된 이름으로 서비스를 찾을 수 있다.

![18-core-dns-6.png](images%2F18-core-dns-6.png)

- **파드 이름 해결**: 파드는 파드의 FQDN을 사용하여 접근해야 한다. 검색 항목은 서비스에만 적용된다.
- **`host` 또는 `nslookup` 명령**: `host` 또는 `nslookup` 명령어를 사용하여 서비스 이름을 조회하면 서비스의 FQDN이 반환된다.

### 핵심 개념

- **CoreDNS**: 쿠버네티스 클러스터의 기본 DNS 서버
- **kube-dns**: 이전 버전의 쿠버네티스 DNS 서버
- **`/etc/coredns/Corefile`**: CoreDNS 구성 파일
- **쿠버네티스 플러그인**: CoreDNS가 쿠버네티스와 연동되도록 하는 플러그인
- **ConfigMap**: CoreDNS 구성 파일을 파드에 전달하는 쿠버네티스 객체
- **`/etc/resolv.conf`**: 파드의 DNS 설정 파일
- **kubelet**: 파드의 DNS 설정을 담당하는 쿠버네티스 컴포넌트
- **FQDN (Fully Qualified Domain Name)**: 서비스 또는 파드의 완전한 도메인 이름

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)