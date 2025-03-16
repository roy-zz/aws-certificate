# Helm

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Helm"에 대해서 자세하게 알아보도록 한다.

---

## Helm 개요

### 쿠버네티스의 복잡성

- 쿠버네티스 애플리케이션은 여러 상호 연결된 객체(Deployment, PersistentVolume, Service, Secret, 등)로 구성된다.

![1-helm.png](images%2F1-helm.png)

- 각 객체는 별도의 YAML 파일로 정의되며, `kubectl apply`를 사용하여 하나씩 생성해야 한다.
- 이 과정은 복잡하고 시간이 오래 걸리며, 특히 대규모 애플리케이션의 경우 더욱 그렇다.

### YAML 파일 관리의 어려움

- YAML 파일은 인터넷에서 다운로드한 후 사용자 정의를 위해 수정해야 하는 경우가 많다.
- 여러 YAML 파일에서 값을 변경하는 것은 번거롭고 오류 발생 가능성이 높다.
- 애플리케이션 업그레이드 또는 삭제 시 여러 YAML 파일을 수동으로 편집하고 관리해야 한다.

![2-helm.png](images%2F2-helm.png)

- 단일 YAML 파일에 모든 객체 정의를 포함하면 파일이 너무 커져서 관리 및 문제 해결이 어려워진다.

### Helm의 필요성

![3-helm.png](images%2F3-helm.png)

- 쿠버네티스는 애플리케이션을 개별 객체의 집합으로 취급하지만, Helm은 애플리케이션을 하나의 패키지로 관리한다.
- Helm은 쿠버네티스를 위한 패키지 관리자로서, 애플리케이션 배포, 업그레이드, 롤백 및 삭제를 간소화한다.
- Helm은 애플리케이션을 구성하는 모든 객체를 그룹화하여 관리하므로, 사용자는 개별 객체를 관리할 필요가 없다.

### Helm의 핵심 기능

![4-helm.png](images%2F4-helm.png)

- **패키지 관리**:
  - Helm은 애플리케이션을 차트(Chart)라는 패키지로 관리한다.
  - 차트는 애플리케이션을 구성하는 모든 객체의 YAML 파일과 설정 파일을 포함한다.
  - 사용자는 단일 명령어로 전체 애플리케이션을 설치, 업그레이드 또는 삭제할 수 있다.
- **사용자 정의 설정**:
  - `values.yaml` 파일에서 애플리케이션 설정을 사용자 정의할 수 있다.
  - 여러 YAML 파일을 수정하는 대신, 단일 파일에서 모든 설정을 관리할 수 있다.
- **릴리즈 관리**:
  - Helm은 애플리케이션의 릴리즈를 추적하고 업그레이드 및 롤백을 지원한다.
  - 사용자는 이전 릴리즈로 롤백하여 애플리케이션을 이전 상태로 복원할 수 있다.
- **자동화된 배포**:
  - Helm은 애플리케이션 배포를 자동화하여 수동 작업의 부담을 줄인다.
  - 사용자는 단일 명령어로 애플리케이션을 배포하고 Helm은 필요한 모든 쿠버네티스 객체를 자동으로 생성한다.

### Helm의 비유

![5-helm.png](images%2F5-helm.png)

- 컴퓨터 게임 설치 프로그램과 유사하게 Helm은 복잡한 애플리케이션 설치를 간소화한다.
- 게임 설치 프로그램이 여러 파일을 적절한 위치에 배치하는 것처럼, Helm은 여러 쿠버네티스 객체를 클러스터에 배포한다.

### Helm의 이점

![6-helm.png](images%2F6-helm.png)

- 쿠버네티스 애플리케이션 관리를 단순화하고 자동화한다.
- YAML 파일 관리의 복잡성을 줄이고 오류 발생 가능성을 낮춘다.
- 애플리케이션 배포, 업그레이드, 롤백 및 삭제를 간소화한다.
- 쿠버네티스 애플리케이션을 하나의 패키지로 관리하여 관리 효율성을 높인다.

---

## Helm 설치 및 설정

### Helm 설치 전 필수 사항

- **기능적인 쿠버네티스 클러스터**: Helm을 설치하고 사용하려면 먼저 작동하는 쿠버네티스 클러스터가 있어야 한다.
- **kubectl 설치 및 설정**: `kubectl`은 쿠버네티스 클러스터와 상호 작용하는 명령줄 도구다. 로컬 컴퓨터에 `kubectl`이 설치되어 있고 올바르게 구성되어 있어야 한다.
- **kubeconfig 파일 설정**: `kubeconfig` 파일은 `kubectl`이 쿠버네티스 클러스터에 연결하는 데 필요한 인증 정보를 포함한다. 올바른 로그인 세부 정보로 `kubeconfig` 파일이 설정되어 있어야 한다.

### Helm 설치 방법 (Linux)

![7-install-helm.png](images%2F7-install-helm.png)

- **Snap을 사용하는 경우**
  - `sudo snap install helm --classic` 명령어를 사용하여 Helm을 설치한다.
  - `--classic` 옵션은 Helm이 홈 디렉터리의 `kubeconfig` 파일에 쉽게 액세스할 수 있도록 더 완화된 샌드박스 환경을 제공한다.
- **APT 기반 시스템 (Debian, Ubuntu 등)**:
    ```bash
    $ curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
    $ sudo apt-get install apt-transport-https --yes
    $ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
    $ sudo apt-get update
    $ sudo apt-get install helm
    ```
- **PKG 기반 시스템**:
  - `pkg install helm`

---

## Helm 2 vs Helm 3

### Helm 버전별 역사

![8-helm-history.png](images%2F8-helm-history.png)

- Helm 1.0 (2016년 2월): 초기 버전
- Helm 2.0 (2016년 11월): 기능 개선
- Helm 3.0 (2019년 11월): 아키텍처 및 기능의 큰 변화

### Helm 2와 Helm 3의 주요 차이점

#### Tiller의 제거

![9-helm2.png](images%2F9-helm2.png)

- Helm 2: Tiller라는 서버 측 구성 요소가 쿠버네티스 클러스터에 설치되어 Helm 클라이언트와 쿠버네티스 API 서버 간의 중개 역할을 수행했다.

![10-helm3.png](images%2F10-helm3.png)

- Helm 3: Tiller을 제거하여 아키텍처를 단순화하고 보안을 강화했다. Helm 3는 직접 쿠버네티스 API 서버와 통신한다.
- Tiller의 제거는 쿠버네티스의 RBAC(Role-Based Access Control) 기능이 강화되면서 가능해졌다. Tiller의 "GOd mode" 권한으로 인한 보안 문제가 해결되었다.

#### 3-Way Strategic Merge Patch

![11-helm2.png](images%2F11-helm2.png)

![12-helm2.png](images%2F12-helm2.png)

- Helm 2: 롤백 및 업그레이드 시 이전 Chart와 현재 Chart의 차이점만 비교하겨 변경 사항을 적용했다. 
  - 이로 인해 `kubectl set image`와 같은 수동 변경 사항이 롤백 또는 업그레이드 시 손실되는 문제가 발생했다.

![13-helm3.png](images%2F13-helm3.png)

- Helm 3: 3-Way Strategic Merge Patch를 도입하여 이전 Chart, 현재 Chart, 그리고 쿠버네티스 클러스터의 현재 상태(live state)를 모두 비교한다. 
  - 이를 통해 수동 변경 사항을 보존하고 보다 정확한 롤백 및 업그레이드를 수행한다.

#### 릴리즈(Revisions) 관리

- Helm은 설치, 업그레이드, 롤백 시마다 릴리즈(Revision)를 생성하여 애플리케이션의 상태를 스냅샷처럼 저장한다.
- Helm 3는 3-Way Strategic Merge Patch를 통해 릴리즈 간의 차이점을 보다 정확하게 비교하고 관리한다.

### Helm 3의 장점

- **단순화된 아키텍처**: Tiller 제거로 설치 및 관리가 용이해졌다.
- **강화된 보안**: 쿠버네티스 RBAC를 활용하여 사용자 권한을 세밀하게 제어할 수 있다.
- **정확한 롤백 및 업그레이드**: 3-Way Strategic Merge Patch를 통해 수동 변경 사항을 보존하고 보다 안정적인 롤백 및 업그레이드를 수행한다.

### 핵심 요약

- Helm 3는 Helm 2의 Tiller를 제거하여 보안과 아키텍처를 개선했다.
- Helm 3는 3-Way Strategic Merge Patch를 통해 롤백 및 업그레이드 기능을 향상시켰다.
- Helm 3는 쿠버네티스 애플리케이션 관리를 보다 쉽고 안전하게 만들어준다.

---

## Component

### Helm의 주요 구성 요소 및 개념

![14-helm-components.png](images%2F14-helm-components.png)

- **Helm CLI (명령줄 유틸리티)**
  - 로컬 시스템에 설치되어 Helm 작업을 수행하는 도구다.
  - Chart 설치, 업그레이드, 롤백 등의 명령어를 실행한다.
- **Chart**:
  - 쿠버네티스 클러스터에 애플리케이션을 배포하기 위한 모든 지침을 담고 있는 파일들의 모음이다.
  - 쿠버네티스 객체 정의(YAML 파일), 설정 파일(values.yaml) 등을 포함한다.
  - Chart를 통해 Helm은 쿠버네티스 클러스터에 애플리케이션을 설치한다.
- **Release**
  - Helm Chart를 사용하여 쿠버네티스 클러스터에 애플리케이션을 설치한 단일 인스턴스다.
  - 하나의 Chart로 여러 개의 Release를 생성하여 동일한 애플리케이션을 여러 환경에 배포할 수 있다.
- **Revision**
  - Release의 특정 시점의 스냅샷이다.
  - 애플리케이션 변경(업그레이드, 설정 변경 등) 시마다 새로운 Revision이 생성된다.
  - Revision을 통해 이전 상태로 롤백할 수 있다.
- **Helm Repository (Helm 저장소)**
  - Helm Chart를 저장하고 배포하는 저장소다.
  - Docker Hub 또는 Vagrant Cloud와 유사하게 공개 저장소에서 다양한 애플리케이션의 Chart를 다운로드할 수 있다.
  - AppsCode, Community Operators, TrueCharts, Bitnami 등 다양한 제공업체가 Helm 저장소를 운영한다.
- **Metadata(메타데이터)**
  - Helm이 쿠버네티스 클러스터에서 수행한 작업(Release, Chart, Revision 등)에 대한 정보를 저장하는 데이터다.
  - 쿠버네티스 Secret으로 저장되어 클러스터 내에서 공유 및 유지된다.
  - 메타데이터는 Helm이 클러스터 상태를 추적하고 작업을 수행하는 데 필수적이다.
- **Artifact Hub (구 Helm Hub)**
  - 다양한 Helm 저장소의 Chart를 검색하고 다운로드할 수 있는 중앙 허브다.
  - 6,300개 이상의 패키지를 제공하며 공식 또는 검증된 게시자의 Chart를 사용할 수 있다.

### Chart의 구성 및 사용

![15-helm-charts.png](images%2F15-helm-charts.png)

![16-helm-charts.png](images%2F16-helm-charts.png)

- Chart는 쿠버네티스 객체 정의(Deployment, Service 등)와 설정 파일(values.yaml)을 포함한다.
- values.yaml 파일은 Chart 설정을 사용자 정의하는 데 사용된다.
- 템플릿 기능을 사용하여 Chart의 YAML 파일을 동적으로 생성할 수 있다.

### Release의 중요성

![17-helm-release.png](images%2F17-helm-release.png)

- 하나의 Chart로 여러 개의 Release를 생성하여 동일한 애플리케이션을 다양한 환경(개발, 운영 등)에 독립적으로 배포할 수 있다.
- Release를 통해 애플리케이션을 격리하고 독립적으로 관리할 수 있다.

### Helm Repository 및 Artifact Hub 활용

![18-helm-repositories.png](images%2F18-helm-repositories.png)

- Artifact Hub를 통해 다양한 Helm Chart를 검색하고 다운로드할 수 있다.
- 공식 또는 검증된 게시자의 Chart를 사용하는 것이 안전하다.

### Metadata 관리

- Helm은 쿠버네티스 Secret을 사용하여 메타데이터를 저장하고 관리한다.
- 메타데이터는 클러스터 상태를 추적하고 Helm 작업을 수행하는 데 필수적이다.

---

## Helm Chart

### Helm의 역할 및 작동 방식

- Helm은 쿠버네티스 애플리케이션 배포를 자동화하는 도구다.
- 사용자는 원하는 최종 상태(예: 애플리케이션 설치, 업그레이드, 롤백)를 선언적으로 지정한다.
- Helm은 필요한 모든 단계를 자동으로 수행하여 사용자의 요청을 처리한다.
- Helm은 차트(Charts)라는 일종의 "설치 설명서"를 사용하여 필요한 작업을 수행한다.

### 템플릿 엔진 및 `values.yaml` 활용

![19-helm-charts.png](images%2F19-helm-charts.png)

- 템플릿 엔진은 `values.yaml` 파일의 값을 사용하여 템플릿 파일을 동적으로 생성한다.
- 이를 통해 사용자는 `values.yaml` 파일만 수정하여 다양한 환경에 맞게 애플리케이션을 배포할 수 있다.
- 템플릿 엔진 문법(예: `{{ .Values.image.repository }}`)을 사용하여 `values.yaml` 파일의 값을 템플릿 파일에 삽입한다.

### `Chart.yaml` 상세 분석

![20-char.png](images%2F20-char.png)

- `apiVersion`: 차트 API 버전을 정의한다.
  - `v1`: Helm 2용 차트
  - `v2`: Helm 3용 차트
  - Helm 3는 `v2`를 사용하여 Helm 2와 호환되지 않는 새로운 기능을 구분한다.
  - Helm 2는 `apiVersion` 필드를 무시한다.
- `appVersion`: 차트가 배포하는 애플리케이션의 버전을 정의한다.
  - 예시: WordPress 버전
  - 정보 제공 목적으로만 사용된다.
- `version`: 차트 자체의 버전을 정의한다.
  - 애플리케이션 버전과 독립적으로 관리된다.
  - 차트 변경 사항을 추적하는 데 사용된다.
- `name`: 차트의 이름을 정의한다.
  - 예시: `wordpress`
- `description`: 차트에 대한 설명을 제공한다.
- `type`: 차트의 유형을 정의한다.
  - `application`: 애플리케이션 배포를 위한 기본 유형이다.
  - `library`: 다른 차트 개발을 위한 유틸리티를 제공하는 유형이다.
- `dependencies`: 현재 차트가 의존하는 다른 차트들을 정의한다.
  - 예시: WordPress 차트의 MariaDB 의존성
  - 의존성 관리를 통해 복잡한 애플리케이션 배포를 단순화한다.
- `keywords`: 차트를 검색하기 위한 키워드 목록을 제공한다.
  - 공용 차트 저장소에서 차트를 검색할 때 유용하다.
- `maintainers`: 차트 유지 관리자의 정보를 제고앟ㄴ다.
- `home` (선택 사항): 프로젝트 홈페이지 URL을 제공한다.
- `icon` (선택 사항): 프로젝트 아이콘 URL을 제공한다.

### Helm 차트의 구성 요소

![21-helm-chart-structure.png](images%2F21-helm-chart-structure.png)

- **템플릿(Templates)**: 쿠버네티스 매니페스트 파일(예: Deployment, Service)을 정의하는 텍스트 파일이다.
  - `values.yaml` 파일의 값을 사용하여 동적으로 생성된다.(템플릿 엔진 사용)
  - 예시: `deployment.yaml`, `service.yaml`
- `values.yaml`: 템플릿에 사용할 변수 및 기본값을 정의하는 파일이다.
  - 사용자는 이 파일을 수정하여 애플리케이션의 설정을 사용자 정의할 수 있다.
- `Chart.yaml` 차트 자체에 대한 메타데이터를 정의하는 파일이다.
  - API 버전, 애플리케이션 버전, 차트 버전, 의존성 등을 포함한다.
- `charts/`: 현재 차트가 의존하는 다른 차트들을 포함하는 디렉토리다.
- `LICENSE`: 차트의 라이선스 정보를 포함하는 파일이다.
- `README.md`: 차트에 대한 사용자 친화적인 설명을 포함하는 파일이다.

### 차트 의존성 관리

- `dependencies` 필드를 사용하여 다른 차트에 대한 의존성을 선언한다.
- Helm은 의존성 차트를 자동으로 다운로드하고 설치한다.
- 이를 통해 복잡한 애플리케이션 배포를 단순화하고 재사용성을 높인다.

---

## Helm CLI

### Helm CLI 기본 사용법

![22-helm-cli.png](images%2F22-helm-cli.png)

- **`helm` 명령어**: Helm CLI를 실행한다.
- **`helm help` 명령어**: Helm CLI의 도움말 정보를 표시한다.
  - 명령어 사용법, 하위 명령어 목록 등을 확인할 수 있다.
  - 예시: `helm rollback` 명령어를 찾을 때 유용하다.

![23-helm-cli.png](images%2F23-helm-cli.png)

- **하위 명령어 도움말**: `helm <하위 명령어> help` 형식으로 특정 하위 명령어의 도움말을 표시한다.
  - 예시: `helm repo help`
  - 하위 명령어의 기능과 매개변수를 자세히 확인할 수 있다.

### Helm 차트 검색 및 설치

#### 차트 검색

![24-wordpress.png](images%2F24-wordpress.png)

- **Artifact Hub(artifacthub.io)**: 웹 브라우저를 통해 차트를 검색한다.
  - 공식 또는 검증된 게시자 배지를 확인하여 고품질 차트를 찾을 수 있다.
  - 차트 설명, 구성 요소, 설정 옵션 등을 확인할 수 있다.

![25-wordpress.png](images%2F25-wordpress.png)

- **`helm search hub <검색어>` 명령어**: CLI에서 Artifact Hub를 검색한다.
  - 예시: `helm search hub wordpress`
- **`helm search repo <검색어>` 명령어**: 특정 저장소에서 차트를 검색한다.

#### 차트 설치

![26-deploying-wordpress.png](images%2F26-deploying-wordpress.png)

- **저장소 추가**: `helm repo add <저장소 이름> <저장소 URL>` 명령어로 차트 저장소를 Helm에 추가한다.
  - 예시: `helm repo add bitnami https://charts.bitnami.com/bitnami`
- **차트 설치**: `helm install <릴리즈 이름> <저장소 이름>/<차트 이름>` 명령어로 차트를 쿠버네티스 클러스터에 설치한다.
  - 예시: `helm install my-release bitnami/wordpress`
- 설치 후 출력되는 정보를 통해 설치된 애플리케이션 사용 방법을 확인할 수 있으며, 이는 차트내의 템플리세 의해 생성된다.

### 릴리즈 관리

![27-helm-release.png](images%2F27-helm-release.png)

- **릴리즈 목록**: `helm list` 명령어로 설치된 릴리즈 목록을 표시한다.
  - 설치된 애플리케이션 및 업데이트 상태를 확인할 수 있다.
- **릴리즈 삭제**: `helm uninstall <릴리즈 이름>` 명령어로 릴리즈를 삭제한다.
  - 릴리즈와 관련된 모든 쿠버네티스 객체를 삭제한다.

### Helm 저장소 관리

![28-helm-repo.png](images%2F28-helm-repo.png)

- **저장소 추가**: `helm repo add` 명령어를 사용하여 Helm 저장소를 추가한다.
- **저장소 목록**: `helm repo list` 명령어를 사용하여 등록된 Helm 저장소 목록을 표시한다.
- **저장소 업데이트**: `helm repo update` 명령어를 사용하여 로컬 저장소 정보를 최신 상태로 업데이트한다.
  - `sudo apt-get update`와 유사하게 저장소 메타데이터를 최신 상태로 유지한다.
  - 저장소 관리자가 저장소를 업데이트하면, 로컬 저장소 정보가 오래된 정보가 되므로, 해당 명령어를 통해 최신 정보를 갱신해야 한다.

### 핵심 개념

- **릴리즈(Release)**: Helm 차트가 쿠버네티스 클러스터에 설치된 인스턴스다.
- **저장소(Repository)**: Helm 차트를 저장하고 관리하는 온라인 저장소다.
- **Artifact Hub**: 모든 Helm 저장소를 목록화하는 중앙 허브다.

---

## Helm 차트 값 사용자 정의

### 기본값 및 사용자 정의 필요성

![29-customizing.png](images%2F29-customizing.png)

- Helm 차트는 `values.yaml` 파일에 기본 설정값을 가지고 있다.
- 애플리케이션 설치 시 기본값이 사용되지만, 사용자는 필요에 따라 이러한 값을 변경해야 할 수 있다.

![30-helm-charts.png](images%2F30-helm-charts.png)

- 예시: WordPress 블로그 이름 변경, 사용자 이메일 설정 등.

### `helm install` 명령어와 `--set` 옵션

![31-custom-parameter-set.png](images%2F31-custom-parameter-set.png)

- `helm install` 명령어를 사용하여 차트를 설치할 때 `--set` 옵션을 사용하여 `values.yaml` 파일의 특정 값을 명령줄에서 직접 변경할 수 있다.
- `--set` 옵션은 여러 번 사용하여 여러 값을 변경할 수 있다.
- 예시: `helm install my-wordpress bitnami/wordpress --set wordpressBlogName="Helm Tutorials" --set wordpressEmail="john@example.com"`
- 장점: 간단한 값 변경에 유용하다.
- 단점: 많은 값을 변경해야 할 경우 명령줄이 길어지고 복잡해진다.

### 사용자 정의 `values.yaml` 파일 사용

![32-custom-parameter-value.png](images%2F32-custom-parameter-value.png)

- 많은 값을 변경해야 할 경우, 사용자 정의 `values.yaml` 파일을 생성하여 `helm install` 명령에 `--values` 옵션으로 전달할 수 있다.
- 사용자 정의 `values.yaml` 파일은 기본 `values.yaml` 파일의 값을 덮어쓴다.
- 장점: 많은 값을 체계적으로 관리할 수 있다.
- 단점: 별도의 파일을 생성하고 관리해야 한다.

### 로컬 차트 수정 및 설치

![33-helm-pull.png](images%2F33-helm-pull.png)

- 기본 `values.yaml` 파일을 직접 수정해야 할 경우, `helm pull` 명령어를 사용하여 차트를 로컬로 다운로드하고 압축을 해제한 후 수정할 수 있다.
- `helm pull` 명령어에 `--untar` 옵션을 사용하면 압축 해제까지 자동으로 수행된다.
- `wordpress/values.yaml` 파일을 원하는 텍스트 편집기로 열어 수정한다.
- 수정한 로컬 차트를 `helm install` 명령어로 설치한다.
- 장점: 기본 `values.yaml` 파일을 직접 수정하여 모든 값을 변경할 수 있다.
- 단점: 차트를 로컬로 다운로드하고 관리해야 하며, 업데이트 시 충돌이 발생할 수 있다.

### `helm install` 명령어의 차트 경로 지정

- `helm install` 명령어는 차트 저장소 이름과 차트 이름을 함께 지정하거나, 로컬 차트 디렉토리 경로를 지정할 수 있다.
- `./`는 현재 디렉토리를 의미한다.
- 예시: `helm install my-wordpress ./wordpress` (현재 디렉토리의 `wordpress` 디렉토리에서 차트를 설치)

---

## Helm 릴리즈 생명주기 관리

### 릴리즈(Release) 개념

![34-helm-release.png](images%2F34-helm-release.png)

- 릴리즈는 Helm 차트를 설치하여 생성된 쿠버네티스 객체의 모음이다.
- 각 릴리즈는 독립적으로 관리되며 동일한 차트를 기반으로 하더라도 서로 영향을 주지 않는다.
- Helm은 각 릴리즈에 속한 쿠버네티스 객체를 추적하여 업그레이드, 롤백, 삭제 등의 작업을 수행한다.

### 릴리즈 업그레이드

![35-helm-upgrade.png](images%2F35-helm-upgrade.png)

![36-helm-upgrade.png](images%2F36-helm-upgrade.png)

- `helm install` 명령어의 `--version` 옵션을 사용하여 특정 버전의 차트를 설치할 수 있다.
- `helm upgrade <릴리즈 이름> <차트 이름>` 명령어를 사용하여 릴리즈를 업그레이드한다.
- 업그레이드 시 이전 버전의 파드가 삭제되고 새 버전의 파드가 생성된다.
- Helm은 업그레이드 과정을 추적하여 릴리즈의 리비전(Revision)을 증가시킨다.
- `kubectl describe pod` 명령어를 사용하여 파드의 이미지 버전을 확인할 수 있다.
- 업그레이드 시 쿠버네티스 객체의 매니페스트 파일 변경 사항(예: 환경 변수, 시크릿)이 자동으로 적용된다.

### 릴리즈 히스토리 및 롤백

![37-helm-rollback.png](images%2F37-helm-rollback.png)

- **히스토리 조회**
  - `helm list` 명령어를 사용하여 현재 설치된 릴리즈의 목록을 확인한다.
  - `helm history <릴리즈 이름>` 명령어를 사용하여 특정 릴리즈의 히스토리를 조회한다.
  - 히스토리에는 각 리비전의 차트 버전, 앱 버전, 작업 유형(설치, 업그레이드, 롤백) 등의 정보가 포함된다.
  - 히스토리를 통해 릴리즈의 생명주기를 추적하고 문제 발생 시 원인을 파악할 수 있다.
- **릴리즈 롤백**
  - `helm rollback <릴리즈 이름> <리비전 번호>` 명령어를 사용하여 릴리즈를 이전 리비전으로 롤백한다.
  - 롤백은 이전 리비전의 설정으로 새 리비전을 생성하는 방식으로 작동한다.
  - 롤백 후 `helm history` 명령어를 사용하여 롤백된 리비전을 확인할 수 있다.
  - 롤백은 쿠버네티스 객체의 매니페스트 파일만 복원하며, 애플리케이션 데이터(예: 파일, 데이터베이스)는 복원하지 않는다.

### 업그레이드 시 주의사항

![38-helm-upgrade.png](images%2F38-helm-upgrade.png)

- 일부 차트는 업그레이드 시 추가적인 매개변수나 설정을 요구할 수 있다.
- 예시: WordPress 차트는 데이터베이스 및 WordPress 관리자 비밀번호가 필요하다.
- 차트의 요구사항을 확인하고 필요한 설정을 제공해야 한다.

### 데이터 복원 및 백업

![39-helm-update.png](images%2F39-helm-update.png)

- Helm 롤백은 쿠버네티스 객체의 매니페스트 파일만 복원하며 데이터는 복원하지 않는다.
- 데이터 복원 및 백업은 별도의 방법을 사용해야 한다.
- 차트 훅(Chart Hooks)을 사용하여 데이터베이스 백업 및 복원을 자동화할 수 있다.
- 영구 볼륨(Persistent Volumes) 또는 외부 데이터베이스를 사용하는 경우 데이터 복원 전략을 수립해야 한다.

### 핵심 개념

- **리비전(Revision)**: 릴리즈의 변경 이력을 나타내는 버전 번호다.
- **차트 훅(Chart Hooks)**: 차트 설치, 업그레이드, 삭제 시 특정 작업을 수행하는 스크립트다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)
- [Helm Docs](https://helm.sh/docs/intro/install/)