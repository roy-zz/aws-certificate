# Security (Network Policy)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Network Policy"에 대해서 자세하게 알아보도록 한다.

---

## Docker

### Docker 컨테이너 격리

![1-security-1.png](images%2F1-security-1.png)

- Docker 컨테이너는 호스트 운영 체제와 커널을 공유하지만, 네임스페이스(namespaces)를 사용하여 격리된다.
- 호스트는 자체 네임스페이스를 가지고, 컨테이너는 별도의 네임스페이스를 가진다.
- 컨테이너 내부에서 실행되는 프로세스는 호스트에서 실행되지만, 컨테이너의 네임스페이스 내에서만 보인다.

![2-security-2.png](images%2F2-security-2.png)

- 컨테이너 내부와 호스트에서 프로세스 ID(PID)는 다를 수 있다.

### Docker 사용자 보안

![3-security-users-1.png](images%2F3-security-users-1.png)

- 기본적으로 Docker 컨테이너 내의 프로세스는 루트(root) 사용자로 실행된다.
- `docker run --user <사용자 ID>` 옵션을 사용하여 컨테이너 내에서 프로세스를 특정 사용자로 실행할 수 있다.

![4-security-users-2.png](images%2F4-security-users-2.png)

- Dockerfile의 `USER` 명령어를 사용하여 이미지 생성 시 기본 사용자 ID를 설정할 수 있다.

### 루트 사용자 권한 제한

- 컨테이너 내부의 루트 사용자는 호스트의 루트 사용자와 동일하지 않다.

![5-linux-capabilities-1.png](images%2F5-linux-capabilities-1.png)

- Docker는 Linux Capabilities를 사용하여 컨테이너 내부의 루트 사용자 권한을 제한한다.
- 루트 사용자는 시스템의 모든 권한을 가진다.

![6-linux-capabilities-2.png](images%2F6-linux-capabilities-2.png)

- Linux Capabilities는 이러한 권한을 세분화하여 특정 권한만 부여하거나 제거할 수 있도록 한다.
- Docker는 기본적으로 제한된 Capabilities 집합으로 컨테이너를 실행한다.
- `docker run --cap-add <capability>` 옵션을 사용하여 추가적인 Capabilities를 부여할 수 있다.
- `docker run --cap-drop <capability>` 옵션을 사용하여 특정 Capabilities를 제거할 수 있다.

![7-linux-capabilities-3.png](images%2F7-linux-capabilities-3.png)

- `docker run --privileged` 옵션을 사용하여 모든 Capabilities를 부여할 수 있다.

### 보안 중요성

- 컨테이너를 루트 사용자로 실행하는 것은 잠재적인 보안 위험을 초래할 수 있다.
- Linux Capabilities를 사용하여 컨테이너의 권한을 제한하는 것이 중요하다.
- 최소한의 권한만을 컨테이너에 부여하여야 한다.

---

## Security Context

### Docker 보안 설정의 쿠버네티스 적용

![8-conatiner-security-1.png](images%2F8-conatiner-security-1.png)

- Docker 컨테이너를 실행할 때 설정할 수 있는 다양한 보안 설정(사용자 ID, Linux Capabilities 등)을 쿠버네티스에서도 적용할 수 있다.

![9-kubernetes-security-1.png](images%2F9-kubernetes-security-1.png)

- 쿠버네티스에서 컨테이너는 파드 안에 캡슐화되어 있다.
- 보안 설정은 컨테이너 수준 또는 파드 수준에서 구성할 수 있다.
- 파드 수준에서 설정을 구성하면 파드 내의 모든 컨테이너에 설정이 적용된다.
- 파드와 컨테이너 모두에서 설정을 구성하면 컨테이너 수준의 설정이 파드 수준의 설정을 덮어쓴다.

### 파드 수준 보안 컨텍스트 설정

![10-security-context-1.png](images%2F10-security-context-1.png)

- 파드 정의 파일의 `spec` 영역 아래에 `securityContext` 필드를 추가하여 파드 수준 보안 설정을 구성한다.
- `runAsUser` 옵션을 사용하여 파드의 사용자 ID를 설정할 수 있다.

### 컨테이너 수준 보안 컨텍스트 설정

- 컨테이너 레벨에 보안 컨텍스트를 적용하기 위해, 파드의 보안 컨텍스트 설정을 컨테이너의 스펙안으로 옮겨서 적용한다.
- `capabilities` 옵션을 사용하여 파드에 추가할 Capabilities 목록을 지정한다.

---

## Networking

### Traffic

![11-traffic-1.png](images%2F11-traffic-1.png)

![12-ingress-egress-1.png](images%2F12-ingress-egress-1.png)

![13-ingress-egress-2.png](images%2F13-ingress-egress-2.png)

![14-ingress-egress-3.png](images%2F14-ingress-egress-3.png)

- 웹 서버(80 포트) -> 앱 서버(5000 포트) -> 데이터베이스 서버(3306 포트)
- Ingress: 들어오는 트래픽 (예: 웹 서버로의 사용자 요청)
- Egress: 나가는 트래픽 (예: 웹 서버에서 앱 서버로의 요청)
- 응답 트래픽은 Ingress, Egress를 정의할 때 고려하지 않는다.

![15-traffic-2.png](images%2F15-traffic-2.png)

- 예시에서 필요한 규칙 항목은 아래와 같다.
  - 웹 서버: 80번 포트 Ingress 허용, 5000번 포트 Egress 허용
  - 앱 서버: 5000번 포트 Ingress 허용, 3306번 포트 Egress 허용
  - 데이터베이스 서버: 3306번 포트 Ingress 허용

### 쿠버네티스 네트워크 보안

#### 쿠버네티스 네트워크 기본 설정

![16-network-security-1.png](images%2F16-network-security-1.png)

![17-network-security-2.png](images%2F17-network-security-2.png)

- 클러스터 내의 모든 파드와 서비스는 기본적으로 서로 통신 가능하다. (모든 허용 규칙)
- 각 노드, 파드, 서비스는 IP 주소를 가진다.
- 쿠버네티스 네트워크는 추가적인 설정 없이 파드들이 서로 통신이 가능해야한다.

#### 네트워크 정책의 필요성

![18-traffic-3.png](images%2F18-traffic-3.png)

- 특정 파드 간의 통신을 제한해야 하는 경우 (예: 웹 서버에서 데이터베이스 서버로의 직접 통신 차단)
- 네트워크 보안 감사 요구 사항 충족

#### 네트워크 정책(Network Policy) 객체

![19-network-policy-1.png](images%2F19-network-policy-1.png)

![20-network-policy-2.png](images%2F20-network-policy-2.png)

- 파드, 레플리카셋, 서비스와 같은 쿠버네티스 네임스페이스 내의 객체
- 하나 이상의 파드에 연결하여 적용
- 규칙(Rules)을 정의하여 트래픽 제어

### 네트워크 정책 적용

#### 파드 선택 (Pod Selection)

![21-network-policy-selectors.png](images%2F21-network-policy-selectors.png)

- 레이블(Labels)과 선택기(Selectors)를 사용하여 정책을 적용할 파드를 선택한다.
- 파드에 레이블을 부여하고, 네트워크 정책의 파드 선택기 필드에 동일한 레이블을 지정한다.

#### 규칙 정의

![22-network-policy-rules.png](images%2F22-network-policy-rules.png)

- **policyTypes**: Ingress, Egress, Ingress/Egress 중 하나를 선택한다.
- **ingress/egress**: 허용할 트래픽을 정의한다.
- **from**: 트래픽을 허용할 파드를 선택한다. (레이블 및 선택기 사용)
- **ports**: 허용할 포트를 지정한다.

#### 정책 정의 파일(YAML)

![23-network-policy-1.png](images%2F23-network-policy-1.png)

- `apiVersion: networking.k8s.io/v1`
- `kind: NetworkPolicy`
- `metadata:`
  - `name: <policy-name>`
  - `namespace: <namespace>`
- `spec:`
  - `podSelector:`
    - `matchLabels:`
      - `<label-key>: <label-value>`
  - `policyTypes:`
    - `Ingress`
  - `ingress:`
    - `- from:`
      - `- podSelector:`
        - `matchLabels:`
          - `<label-key>: <label-value>`
      - `ports:`
        - `- protocol: TCP`
          - `port: 80`

#### Ingress/Egress 격리

- `policyTypes`에 Ingress 또는 Egress를 명시적으로 지정해야 격리가 적용된다.
- 지정하지 않으면 해당 방향의 트래픽은 격리되지 않는다.
- 예를 들어, Ingress만 지정하면 Egress 트래픽은 모두 허용된다.

#### 네트워크 정책 구현 및 주의 사항

![24-note.png](images%2F24-note.png)

- 네트워크 솔루션 지원
  - 네트워크 정책은 쿠버네티스 네트워크 솔루션에 의해 구현된다.
  - 모든 네트워크 솔루션이 네트워크 정책을 지원하는 것은 아니다.
  - 지원 솔루션: Cube Router, Calico, Romana, WaveNet 등
  - 미지원 솔루션: Flannel
  - 네트워크 솔루션 문서를 참조하여 지원 여부를 확인해야 한다.
- 정책 생성과 구현
  - 네트워크 정책을 지원하지 않는 솔루션에서도 정책을 생성할 수는 있지만 적용되지는 않는다.
  - 오류 메시지가 표시되지 않으므로 주의해야 한다.

### Network Policies 예시

#### 기본 설정 및 초기 차단

![25-network-policies-1.png](images%2F25-network-policies-1.png)

- 쿠버네티스는 기본적으로 모든 파드 간의 트래픽을 허용한다.
- DB 파드에 대한 모든 트래픽 차단
  - `db-policy` 네트워크 정책을 생성한다.
  - `podSelector`를 사용하여 DB 파드에 정책을 연결한다. (예: `matchLabels: role: db`)
  - 이렇게 설정하면 DB 파드로의 모든 트래픽이 차단된다.

#### Ingress 규칙 정의

![26-network-policies-2.png](images%2F26-network-policies-2.png)

![27-network-policies-3.png](images%2F27-network-policies-3.png)

- API 파드의 접근 허용
  - DB 파드의 관점에서 API 파드로부터의 트래픽은 Ingress다.
  - `policyTypes: [Ingress]`를 설정한다.
  - `ingress` 영역을 추가하고 `from`과 `ports`를 정의한다.
  - `from`에는 `podSelector`를 사용하여 API 파드의 레이블을 지정한다.
  - `ports`에는 `port: 3306, protocol: TCP`를 설정한다.
- 응답 트래픽
  - Ingress 규칙을 통해 요청 트래픽이 허용되면 응답 트래픽은 자동으로 허용된다. 별도의 규칙이 필요하지 않다.
- Ingress와 Egress의 차이
  - 요청 트래픽의 시작 방향을 기준으로 Ingress와 Egress를 구분한다.
  - DB 파드에서 API 파드로의 트래픽은 Egress이며 별도의 Egress 규칙이 필요하다.

#### 다양한 선택기(Selectors) 사용

![28-network-policies-4.png](images%2F28-network-policies-4.png)

![29-network-policies-5.png](images%2F29-network-policies-5.png)

- **Namespace 선택기(namespaceSelector)**
  - 특정 네임스페이스의 파드만 접근을 허용할 때 사용한다.
  - `namespaceSelector`에 네임스페이스의 레이블을 지정한다.
  - `podSelector`와 함께 사용하여 특정 네임스페이스의 특정 파드만 허용할 수 있다.
  - `namespaceSelector`만 사용하면 해당 네임스페이스의 모든 파드가 접근 가능하다.

![30-network-policies-6.png](images%2F30-network-policies-6.png)

- **IP 블록 선택기(ipBlock)**
  - 쿠버네티스 클러스터 외부의 IP 주소로부터의 접근을 허용할 때 사용한다.
  - `ipBlock`에 CIDR(Classless Inter-Domain Routing) 표기법으로 IP 주소 범위를 지정한다.
  - 예: `ipBlock: cidr: 192.168.5.10/32`
- **선택기 조합**:
  - `from` 영역에 여러 선택기를 함께 사용할 수 있다.
  - `podSelector`와 `namespaceSelector`를 함께 사용하면 AND 조건으로 작동한다.
  - 여러 개의 `from` 규칙을 정의하면 OR 조건으로 작동한다.
  - 선택기 순서에 따라서 허용되는 트래픽이 크게 달라질수 있기 때문에 주의해야한다.

#### Egress 규칙 정의

![31-network-policies-7.png](images%2F31-network-policies-7.png)

- DB 파드에서 외부 서버로의 트래픽 허용
  - DB 파드는 외부 백업 서버로의 트래픽은 Egress다.
  - `policyTypes: [Ingress, Egress]`를 설정한다.
  - `egress` 섹션을 추가하고 `to`와 `ports`를 정의한다.
  - `to`에는 `ipBlock`을 사용하여 백업 서버의 IP 주소 범위를 지정한다.
  - `ports`에는 백업 서버의 포트를 설정한다.
- `to` 선택기
  - Egress의 `to` 영역에도 `podSelector`, `namespaceSelector`, `ipBlock`을 사용할 수 있다.

---

## Kubectx and Kubens

### 실제 쿠버네티스 환경의 복잡성

- 실제 운영 환경에서는 많은 네임스페이스와 클러스터를 관리해야 한다.
- `kubectl` 명령어만으로는 이런 운영 환경에서 컨텍스트와 네임스페이스를 관리하는 것이 번거롭고 혼란스러울 수 있다.

### `kubectx` 및 `kubens` 도구

- `kubectx`와 `kubens`는 쿠버네티스 컨텍스트 및 네임스페이스를 간편하게 전환할 수 있도록 도와주는 명령줄 도구다.
- 참고 자료: https://github.com/ahmetb/kubectx

#### `kubectx` (컨텍스트 전환 도구)

- `kubectl config` 명령어를 사용하지 않고 컨텍스트를 빠르게 전환할 수 있다.
- 다중 클러스터 환경에서 특히 유용하다.
- 설치:
  - `sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx`
  - `sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx`
- 사용법:
  - 모든 컨텍스트 목록 보기: `kubectx`
  - 새로운 컨텍스트로 전환: `kubectx <컨텍스트 이름>`
  - 이전 컨텍스트로 되돌아가기: `kubectx -`
  - 현재 컨텍스트 보기: `kubectx -c`

#### kubens (네임스페이스 전환 도구)

- 간단한 명령어로 네임스페이스를 빠르게 전환할 수 있다.
- 설치:
  - `sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx`
  - `sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens`
- 사용법:
  - 새로운 네임스페이스로 전환: `kubens <새로운 네임스페이스>`
  - 이전 네임스페이스로 되돌아가기: `kubens -`

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)