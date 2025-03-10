# Security (Cluster Role)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Cluster Role"에 대해서 자세하게 알아보도록 한다.

---

## Roles & Role Bindings

![1-roles.png](images%2F1-roles.png)

- Role과 Role Binding은 네임스페이스(namespace) 범위 내에서 권한을 관리한다.
- 네임스페이스를 지정하지 않으면 기본(default) 네임스페이스에 생성된다.
- 특정 네임스페이스 내에서만 리소스에 대한 접근 권한을 제어한다.

### Namespace와 클러스터 범위(Cluster-scoped) 리소스

![2-namespaces-1.png](images%2F2-namespaces-1.png)

- 네임스페이스는 파드, 디플로이먼트, 서비스와 같은 리소스를 그룹화하거나 격리하는 데 사용된다.
- 하지만 노드와 같은 일부 리소스는 클러스터 전체 범위(cluster-wide 또는 cluster-scoped)에 적용된다.
- 클러스터 범위 리소스는 특정 네임스페이스에 연결할 수 없다.
- 리소스는 네임스페이스 범위 또는 클러스터 범위로 분류된다.

![3-namespaces-2.png](images%2F3-namespaces-2.png)

![4-namespaces-3.png](images%2F4-namespaces-3.png)

- 네임스페이스 범위 리소스 예시: 파드, 레플리카셋, 잡, 디플로이먼트, 서비스, 시크릿, 역할, 역할 바인딩
- 클러스터 범위 리소스 예시: 노드, 퍼시스턴트 볼륨, 클러스터 역할, 클러스터 역할 바인딩, 인증서 서명 요청, 네임스페이스 자체
- `kubectl api-resources --namespaced` 명령어를 사용하여 네임스페이스 범위 및 비네임스페이스 범위 리소스의 전체 목록을 확인할 수 있다.

### Cluster Role & Cluster Role Bindings

![5-clusterroles-1.png](images%2F5-clusterroles-1.png)

- 역할과 역할 바인딩은 네임스페이스 범위 리소스에 대한 사용자 권한 부여에 사용된다.
- 클러스터 범위 리소스(예: 노드, 퍼시스턴트 볼륨)에 대한 사용자 권한 부여에는 클러스터 역할과 클러스터 역할 바인딩이 사용된다.
- 클러스터 역할은 클러스터 범위 리소스에 대한 권한을 정의한다.

![6-clusterrolebinding.png](images%2F6-clusterrolebinding.png)

- 예를 들어, 클러스터 관리자 역할(cluster admin role)은 클러스터 관리자에게 노드 조회, 생성, 삭제 권한을 부여한다.
- 예를 들어, 스토리지 관리자 역할(storage admin role)은 스토리지 관리자에게 퍼시스턴트 볼륨 및 클레임 생성 권한을 부여한다.
- 클러스터 역할 정의 파일은 종류(kind)를 클러스터 역할(cluster role)로 설정하고 규칙(rules)을 지정한다.

### 클러스터 역할 바인딩 (Cluster Role Binding) 생성

- 클러스터 역할 바인딩은 사용자를 클러스터 역할에 연결한다.
- 예: 클러스터 관리자 역할 바인딩(cluster admin role binding)은 클러스터 관리자 사용자(cluster admin user)를 클러스터 관리자 역할에 연결한다.
- 클러스터 역할 바인딩 정의 파일은 종류를 클러스터 역할 바인딩으로 설정하고, 주제(subject)에 사용자 세부 정보를 지정하며 역할 참조(role ref)에 클러스터 역할 세부 정보를 제공한다.
- `kubectl create` 명령어를 사용하여 클러스터 역할 바인딩을 생성한다.

### 클러스터 역할(Cluster Role)의 네임스페이스 범위 리소스 적용

- 클러스터 역할은 클러스터 범위 리소스뿐만 아니라 네임스페이스 범위 리소스에도 사용할 수 있다.
- 클러스터 역할을 네임스페이스 범위 리소스에 적용하면 사용자는 모든 네임스페이스에서 해당 리소스에 접근할 수 있다.
- 역할을 사용하여 파드에 대한 접근 권한을 부여하면 사용자는 특정 네임스페이스의 파드에만 접근할 수 있다.
- 클러스터 역할을 사용하여 파드에 대한 접근 권한을 부여하면 사용자는 클러스터 전체의 모든 파드에 접근할 수 있다.
- 쿠버네티스 클러스터는 초기 설정 시 여러 기본 클러스터 역할을 생성한다.

---

## Service

### 서비스 계정의 개념

![7-user-account-service-account.png](images%2F7-user-account-service-account.png)

- 쿠버네티스에는 사용자 계정(User Account)과 서비스 계정(Service Account) 두 가지 계정이 있다.
- 사용자 계정은 사람이 클러스터에 접근하여 관리 작업을 수행하거나 애플리케이션을 배포하는 데 사용된다.
- 서비스 계정은 애플리케이션이 쿠버네티스 클러스터와 상호 작용하는 데 사용된다.

![8-kubernetes-dashboard.png](images%2F8-kubernetes-dashboard.png)

- 예를 들어, Prometheus와 같은 모니터링 애플리케이션이나 Jenkins와 같은 자동화 빌드 도구가 서비스 계정을 사용하여 쿠버네티스 API에 접근한다.

### 서비스 계정 생성 및 사용

![9-kubectl-create-get-describe.png](images%2F9-kubectl-create-get-describe.png)

- `kubectl create serviceaccount <계정 이름>` 명령어를 사용하여 서비스 계정을 생성한다.
- 서비스 계정을 생성하면 자동으로 토큰(token)이 생성되며 이 토큰은 시크릿(Secret) 객체에 저장된다.
- `kubectl get serviceaccount` 명령어를 사용하여 서비스 계정들을 확인한다.

![10-describe-service-account-1.png](images%2F10-describe-service-account-1.png)

- `kubectl describe secret <시크릿 이름>` 명령어를 사용하여 토큰을 확인할 수 있다.

![11-curl-service-account-1.png](images%2F11-curl-service-account-1.png)

- 애플리케이션은 이 토큰을 사용하여 쿠버네티스 API에 인증할 수 있다.
- 외부의 애플리케이션에서 API를 사용하기위해 curl 명령어와 Authorization 헤더에 Bearer token을 사용해 API를 호출할 수 있다.

### 클러스터 내부 애플리케이션 서비스 계정 사용

![12-my-kubernetes-dashboard.png](images%2F12-my-kubernetes-dashboard.png)

- 클러스터 내부에서 실행되는 애플리케이션(예: Prometheus, 사용자 정의 대시보드)은 서비스 계정 토큰을 자동으로 마운트하여 사용할 수 있다.
- 각 네임스페이스에는 기본 서비스 계정(default service account)이 자동으로 생성된다.
- 파드를 생성할 때 서비스 계정을 명시적으로 지정하지 않으면 기본 서비스 계정이 자동으로 마운트된다.
- `/var/run/secrets/kubernetes.io/serviceaccount` 디렉터리에 토큰이 마운트된다.

![13-describe-service-account-2.png](images%2F13-describe-service-account-2.png)

- `kubectl describe pod <파드 이름>` 명령어를 사용하여 파드의 상세 정보를 확인하면 마운트된 서비스 계정 토큰을 확인할 수 있다.
- 기본 서비스 계정은 제한된 권한을 가지고 있다. 필요한 경우 사용자 정의 서비스 계정을 생성하여 파드에 연결할 수 있다.

![14-describe-service-account-3.png](images%2F14-describe-service-account-3.png)

- 파드가 이미 실행중인 경우에는 서비스 계정의 변경이 불가능하고, 디플로이먼트의 경우에는 파드 정의 파일의 변경시 롤아웃이 발생하면서 새로운 파드가 생성되면서 서비스 계정의 변경이 적용된다.

![15-automount-service-account-token.png](images%2F15-automount-service-account-token.png)

- 파드 정의 파일에서 `automountServiceAccountToken: false`를 설정하여 서비스 계정 토큰 자동 마운트를 비활성화할 수 있다.

### Service Account 업데이트 사항

#### 1.22 버전 이번의 서비스 계정 토큰

![16-service-account-updated-1.png](images%2F16-service-account-updated-1.png)

![17-service-account-updated-2.png](images%2F17-service-account-updated-2.png)

![18-service-account-updated-3.png](images%2F18-service-account-updated-3.png)

- 각 네임스페이스에는 기본 서비스 계정이 있으며 이 계정에는 토큰이 포함된 시크릿 객체가 연결되어 있다.
- 파드가 생성되면 서비스 계정이 자동으로 연결되고 토큰이 파드 내부의 특정 위치(`/var/run/secrets/kubernetes.io/serviceaccount`)에 마운트된다.
- 이 토큰을 만료일이 설정되지 않아 서비스 계정이 존재하는 한 유효하다.
- 이런 방식은 보안 및 확장성 문제를 야기한다.

#### 1.22 버전의 변경 사항 (토큰 요청 API 도입)

![19-service-account-updated-4.png](images%2F19-service-account-updated-4.png)

![20-service-account-updated-5.png](images%2F20-service-account-updated-5.png)

![21-service-account-updated-6.png](images%2F21-service-account-updated-6.png)

- 보안 및 확장성 문제를 해결하기 위해 토큰 요청 API (Token Request API)가 도입되었다. (KEP-1205)
- 파드가 생성될 때 서비스 계정 어드미션 컨트롤러 (Service Account Admission Controller)가 토큰 요청 API를 통해 수명이 정의된 토큰을 생성한다.
- 이 토큰은 프로젝티드 볼륨(Projected Volume)으로 파드에 마운트된다.
- 기존의 서비스 계정의 Secret이 직접적으로 마운트 되는 방식에서, 토큰 요청 API를 통해 토큰을 발급받아 프로젝티드 볼륨에 마운트하는 방식으로 변경되었다.

#### 1.24 버전의 변경 사항 (시크릿 기반 서비스 계정 토큰 감소)

![22-service-account-updated-7.png](images%2F22-service-account-updated-7.png)

- 서비스 계정 생성 시 자동으로 시크릿 및 토큰이 생성되지 않는다. (KEP-2799)
- `kubectl create token <서비스 계정 이름>` 명령어를 사용하여 필요한 경우 토큰을 생성해야 한다.

![23-service-account-updated-8.png](images%2F23-service-account-updated-8.png)

- 이 명령어로 생성된 토큰은 만료일이 설정된다. (기본적으로 1시간)
- 만료 시간을 늘리는 추가 옵션을 명령어에 전달할 수 있다.
- 기존의 서비스 계정 생성시 자동으로 생성되던 시크릿이 더이상 자동으로 생성되지 않는다.

#### 1.24 버전 이후의 이전 방식 시크릿 생성

![24-service-account-updated-9.png](images%2F24-service-account-updated-9.png)

![25-service-account-updated-10.png](images%2F25-service-account-updated-10.png)

- 만료되지 않는 토큰을 사용하는 이전 방식의 시크릿을 생성하려면 시크릿 객체를 생성하고 유형(type)을 `kubernetes.io/service-account-token`으로 설정해야 한다.
- 메타데이터(metadata) 영역의 어노테이션(annotations)에 서비스 계정 이름을 지정해야 한다.
- 서비스 계정을 먼저 생성한 후 시크릿 객체를 생성해야 한다.

#### 권장 사항

- 쿠버네티스 문서에 따르면 토큰 요청 API를 사용하여 토큰을 얻을 수 없는 경우에만 서비스 계정 토큰 시크릿을 생성해야 한다.
- `kubectl create token` 명령어나, 토큰 요청 API를 사용하는것이 권장된다.
- 만료되지 않는 토큰 자격 증명을 유지하는 보안 노출이 허용되는 경우에만 서비스 계정 토큰 시크릿을 생성해야 한다.
- 토큰 요청 API는 서비스 계정 토큰 시크릿보다 안전하고 수명이 제한되어 권장된다.

---

## Image Security

### Image

#### 이미지 이름의 기본

![26-image-1.png](images%2F26-image-1.png)

- 컨테이너 이미지 이름은 Docker의 이미지 명명 규칙을 따른다.
- `User` 또는 `User/Image` 형식으로 구성된다.
- `User` 부분을 생략하면 기본적으로 `library`로 간주된다.
- `library`는 Docker의 공식 이미지가 저장되는 기본 계정이다.
- 예시: `nginx`는 `library/nginx`와 같다.

#### 이미지 저장소 (Registry)

![27-image-2.png](images%2F27-image-2.png)

- 이미지는 이미지 저장소에 저장되고 가져온다.
- 기본 이미지 저장소는 Docker Hub(docker.io)이다.
- 다른 인기 있는 저장소로는 Google Container Registry(gcr.io) 등이 있다.
- 사내에서 사용하는 비공개 이미지를 위한 내부 사설 저장소도 구축할 수 있다.
- AWS, Azure, GCP와 같은 클라우드 서비스 제공업체는 기본적으로 사설 저장소를 제공한다.

### Private Repository

#### 비공개 이미지 저장소 사용

![28-private-repository-1.png](images%2F28-private-repository-1.png)

- 비공개 저장소의 이미지를 사용하려면 인증이 필요하다.
- Docker에서는 `docker login` 명령어를 사용하여 비공개 저장소에 로그인한다.
- 쿠버네티스에서는 시크릿(Secret) 객체를 사용하여 인증 정보를 전달한다.

#### 쿠버네티스에서 비공개 이미지 사용 설정

![29-private-repository-2.png](images%2F29-private-repository-2.png)

- 파드 정의 파일에서 `image` 필드에 비공개 저장소의 이미지 전체 경로를 지정한다.
- 시크릿 객체를 생성하여 저장소 인증 정보를 저장한다.
- 시크릿의 타입은 `docker-registry`로 생성한다.
- 시크릿 객체에는 저장소 서버 이름, 사용자 이름, 암호, 이메일 주소를 포함한다.
- 파드 정의 파일의 `imagePullSecrets` 영역에 생성한 시크릿 이름을 지정한다.
- 쿠버네티스(kubelet)는 시크릿의 인증 정보를 사용하여 이미지를 가져온다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)