# Kustomize

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Kustomize의 기본적인 내용들"에 대해서 알아보도록 한다.

---

## Kustomize 개요

### Kustomize의 필요성

![1-why-kustomize.png](images%2F1-why-kustomize.png)

- 쿠버네티스 환경에서 애플리케이션을 여러 환경(개발, 스테이징, 프로덕션 등)에 배포할 때 환경별로 다른 설정이 필요하다.

![2-why-kustomize.png](images%2F2-why-kustomize.png)

- 기존 방식(환경별 디렉토리 복사)은 설정 중복 및 관리의 어려움을 야기한다.
- Kustomize는 설정 재사용 및 환경별 맞춤 설정을 위한 솔루션을 제공한다.

### 기존 방식의 문제점

![3-why-kustomize.png](images%2F3-why-kustomize.png)

- **환경별 디렉토리 복사**:
  - 각 환경별로 쿠버네티스 설정 파일(YAML)을 복사하여 관리한다.
  - 환경별로 설정 파일의 일부 값(예: replicas)을 수정한다.
  - `kubectl apply -f <환경 디렉토리>` 명령어를 사용하여 배포한다.

![4-deploying-dev.png](images%2F4-deploying-dev.png)

![5-deploying-stg.png](images%2F5-deploying-stg.png)

- **문제점**
  - 설정 중복으로 인한 관리의 어려움 증가
  - 새로운 리소스 추가 시 모든 환경 디렉토리에 복사해야 함
  - 설정 변경 시 모든 환경 디렉토리에 적용해야 함
  - 실수로 인한 환경별 설정 불일치 발생 가능성 증가
  - 확장성이 떨어짐

![6-why-kustomize.png](images%2F6-why-kustomize.png)

### Kustomize의 해결 방식

![7-overlays.png](images%2F7-overlays.png)

- **Base Config (기본 설정)**:
  - 모든 환경에서 공통으로 사용되는 쿠버네티스 설정 파일을 포함한다.
  - 기본값을 정의한다.
  - 예시: `nginx-deployment.yaml` (replicas: 1)
- **Overlays (오버레이)**:
  - 환경별로 변경해야 하는 설정을 정의한다.
  - Base Config의 설정을 덮어쓴다.
  - 예시:
    - `overlays/dev/kustomization.yaml` (replicas: 1)
    - `overlays/staging/kustomization.yaml` (replicas: 2)
    - `overlays/prod/kustomization.yaml` (replicas: 5)
- **Kusomize 작동 방식**:
  - Base Config와 Overlays를 결합하여 최종 쿠버네티스 매니페스트 파일을 생성한다.
  - `kubectl apply -k <오버레이 디렉토리>` 명령어를 사용하여 배포한다.

### Kustomize 폴더 구조

![8-folder-structure.png](images%2F8-folder-structure.png)

- `base/`: Base Config 파일들을 포함한다.
- `overlays/`: Overlays 폴더들을 포함한다.
  - `overlays/dev/`: 개발 환경 설정을 포함한다.
  - `overlays/stg/`: 스테이징 환경 설정을 포함한다.
  - `overlays/prod/`: 프로덕션 환경 설정을 포함한다.

### Kustomize 사용 흐름

![9-kustomize.png](images%2F9-kustomize.png)

- Base Config 파일들을 `base/` 디렉토리에 생성한다.
- Overlays 폴더들을 `overlays/` 디렉토리에 생성하고 환경별 설정을 정의한다.
- `kubectl apply -k <오버레이 디렉토리>` 명령어를 사용하여 배포한다.

### Kustomize의 장점

![10-kustomize.png](images%2F10-kustomize.png)

- **설정 재사용**: Base Config를 통해 설정 중복을 최소화한다.
- **환경별 맞춤 설정**: Overlays를 통해 환경별로 필요한 설정만 변경한다.
- **YAML 기반**: 별도의 템플릿 언어를 학습할 필요 없이 표준 YAML을 사용한다.
- **kubectl 내장**: `kubectl`에 내당되어 별도의 설치가 필요하지 않다. (단, 최신 버전 사용을 위해 별도 설치 권장)
- **단순성**: 설정 변경 및 관리가 용이하다.
- **가독성**: 일반 YAML을 사용하여 가독성이 높다.

---

## Kustomize vs Helm

### Helm의 환경별 설정 관리 방식

- Helm은 Go 템플릿 문법을 사용하여 쿠버네티스 매니페스트 파일에 변수를 할당한다.
- 변수를 통해 환경별로 다른 설정을 적용할 수 있다.

### Helm 템플릿 문법

![11-kustomize-helm.png](images%2F11-kustomize-helm.png)

- `{{ .Values.<변수명> }}` 형식으로 변수를 사용한다.
- 예시: `replicas: {{ .Values.replicaCount }}`


### `values.yaml` 파일

![12-kustomize-helm.png](images%2F12-kustomize-helm.png)

- 템플릿에 사용되는 변수의 값을 정의한다.
- 환경별로 다른 `values.yaml` 파일을 사용하여 설정을 관리할 수 있다.

### Helm 프로젝트 구조

![13-kustomize-helm.png](images%2F13-kustomize-helm.png)

- `templates/`: 템플릿 파일(쿠버네티스 매니페스트 파일)을 포함한다.
- `environment/`: 환경별 `values.yaml` 파일을 포함한다.
  - `values.dev.yaml`
  - `values.stg.yaml`
  - `values.prod.yaml`

### Helm 추가 기능

- **패키지 관리자**: 쿠버네티스 애플리케이션의 패키지 관리 기능을 제공한다. (Linux의 YUM/APT와 유사)
- **조건문, 반복문, 함수, 훅(Hooks) 지원**: Kustomize보다 더 많은 기능을 제공한다.

### Helm의 단점

- **복잡성**: Go 템플릿 문법으로 인해 설정 파일의 가독성이 떨어진다.
- **템플릿의 유효성**: 템플릿 파일은 유효한 YAML 형식이 아니다.
- **학습 곡선**: Kustomize보다 학습 곡선이 높다.

### Kustomize vs Helm

![14-kustomize-helm.png](images%2F14-kustomize-helm.png)

| 특징       | Kustomize              | Helm                      |
|----------|------------------------|---------------------------|
| 설정 관리 방식 | Base Config 및 Overlays | Go 템플릿 문법 및 `values.yaml` |
| 기능       | 설정 재사용, 환경별 맞춤 설정      | 패키지 관리, 조건문, 반복문, 함수, 훅   | 
| 복잡성      | 단순                     | 복잡                        |
| 가독성      | 높음 (YAML)              | 낮음 (템플릿 문법)               |
| 학습 곡선    | 낮음                     | 높음                        |
| 유효성      | YAML                   | 템플릿 문법 (YAML 아님)          |

### 선택 기중

- **단순성 및 가독성**: Kustomize
- **추가 기능 및 패키지 관리**: Helm
- 프로젝트의 복잡성과 요구사항에 따라 적절한 도구를 선택해야 한다.

---

## Install

### 사전 준비 사항

- 정상적으로 작동하는 쿠버네티스 클러스터가 필요하다.
- `kubectl`이 로컬 머신에 설치되어 있어야 하며, 쿠버네티스 클러스터에 연결되도록 구성되어 있어야 한다.

### Kustomize 설치 방법

![15-install-kustomize.png](images%2F15-install-kustomize.png)

- Kustomize는 Linux, Windows, macOS에서 설치할 수 있다.
- Kustomize 팀에서 제공하는 설치 스크립트를 사용하여 간편하게 설치할 수 있다.
- 터미널에서 명령어를 실행하여 설치 스크립트를 다운로드하고 실행한다.
  - 이 스크립트는 운영 체제를 자동으로 감지하고 적절한 버전의 Kustomize를 설치한다.

### 설치 확인

![16-install-kustomize.png](images%2F16-install-kustomize.png)

- Kustomize가 올바르게 설치되었는지 확인하려면 다음 명령어를 사용하여 버전을 확인한다.
  - `kustomize version`
- 만약 출력 결과가 예상과 다르다면, 설치 과정에서 문제가 발생했을 가능성이 높다.
- 환경 변수가 현재 터미널 세션에 업데이트되지 않았을 수도 있다.

---

## YAML

### `kustomization.yaml` 파일의 필요성

- Kustomize는 특정 디렉토리의 쿠버네티스 설정 파일들을 관리한다.
- Kustomize는 디렉토리 내의 모든 YAML 파일을 자동으로 처리하지 않고, `kustomization.yaml` 파일에 명시된 파일들만 처리한다.
- `kustomization.yaml` 파일은 Kustomize가 관리해야 할 리소스와 적용할 변경 사항(transformation)을 정의한다.

### `kustomization.yaml` 파일의 구성 요소

![17-kustomization-yaml.png](images%2F17-kustomization-yaml.png)

- `resources`: Kustomize가 관리할 쿠버네티스 설정 파일(YAML) 목록을 정의한다.
  - 예시: `resources: [ nginx-deployment.yaml, nginx-service.yaml ]`
- **변경 사항(Transformations)**: 쿠버네티스 리소스에 적용할 변경 사항을 정의한다.
  - 예시: `commonLabels: { company: KodeKloud }` (모든 리소스에 `company: KodeKloud` 레이블 추가)

### `kustomization.yaml` 파일 생성 및 설정

- 관리할 쿠버네티스 설정 파일들이 있는 디렉토리(예: `k8s/`)에 `kustomization.yaml` 파일을 생성한다.
- `resources` 섹션에 관리할 YAML 파일 목록을 추가한다.
- 필요한 변경 사항(transformations)을 추가한다.

### `kustomize build` 명령어

![18-kustomize-build.png](images%2F18-kustomize-build.png)

- `kustomize build <디렉토리>` 명령어를 사용하여 Kustomize 설정을 기반으로 최종 쿠버네티스 설정 파일을 생성한다.
- 예시: `kustomize build k8s/`
- 이 명령어는 `kustomization.yaml` 파일을 찾아 `resources`에 정의된 파일들을 가져와 변경 사항(transformations)을 적용하고 결과를 표준 출력으로 보여준다.
- 실제 쿠버네티스 클러스터에 배포하지 않고 최종 설정 결과를 미리 확인할 수 있다.

### `kustomize build` 명령어 결과 확인

- `kustomize build` 명령어 실행 결과로 최종 쿠버네티스 설정 파일(YAML)이 터미널에 출력된다.
- 출력 결과를 통해 변경 사항(transformations)이 올바르게 적용되었는지 확인할 수 있다.
- 예시: `commonLabels`에 정의된 레이블이 모든 리소스에 추가되었는지 확인한다.

### `kustomization.yaml` 파일 요약

- Kustomize가 관리할 리소스 목록(`resources`)과 적용할 변경 사항(transformations)을 정의한다.
- `kustomize build` 명령어를 통해 최종 쿠버네티스 설정 파일을 생성하고 결과를 확인할 수 있다.
- `kustomize build` 명령어는 실제 배포를 수행하지 않으며, 결과를 `kubectl apply` 명령어에 전달하여 배포해야 한다.

---

## Output

### Linux 파이프(|) 활용

![19-apply-kustomize.png](images%2F19-apply-kustomize.png)

- `kustomize build` 명령어의 결과를 `kubectl apply` 명령어의 입력으로 전달하기 위해 Linux 파이프(|)을 사용한다.
- 파이프는 첫 번째 명령어의 출력을 두 번째 명령어의 입력으로 연결한다.
- 전체 명령어는 다음과 같다.
  - `kustomize build k8s/ | kubectl apply -f -`
  - `kustomize build k8s/`: `k8s` 디렉토리의 `kustomization.yaml` 파일을 기반으로 쿠버네티스 설정 파일을 생성한다.
  - `|` 파이프를 사용하여 `kustomize build` 명령어의 출력을 `kubectl apply` 명령어의 입력으로 전달한다.
  - `kubectl apply -f -`: 표준 입력(-)에서 쿠버네티스 설정 파일을 읽어 클러스터에 배포한다.

### `kubectl apply -k` 명령어 (Native Kustomize)

- `kubectl`은 Kustomize를 내장하고 있으며, `-k` 플래그를 사용하여 직접 Kustomize 설정을 적용할 수 있다.
- 명령어는 다음과 같다.
  - `kubectl apply -k k8s`
  - `kubectl apply`
  - `-k k8s/`: `k8s/` 디렉토리의 `kustomization.yaml` 파일을 기반으로 Kustomize 설정을 적용한다.

### Kustomize를 사용한 리소스 삭제

![20-delete-kustomize.png](images%2F20-delete-kustomize.png)

- 리소스를 삭제할 때도 파이프와 `kubectl delete` 명령어를 사용한다.
- 명령어는 다음과 같다.
  - `kustomize build k8s | kubectl delete -f -`
  - `kustomize build k8s`: 쿠버네티스 설정 파일을 생성한다.
  - `|`: 파이프를 사용하여 `kustomize build` 명령어의 출력을 `kubectl delete` 명령어의 입력으로 전달한다.
  - `kubectl delete -f -`: 표준 입력(-)에서 쿠버네티스 설정 파일을 읽어 클러스터에서 삭제한다.

### `kubectl delete -k` 명령어 (Native Kustomize 삭제)

- `kubectl`의 `-k` 플래그를 사용하여 직접 Kustomize 설정을 기반으로 리소스를 삭제할 수 있다.
- 명령어는 다음과 같다.
  - `kubectl delete -k k8s`
  - `kubectl delete`: 쿠버네티스 리소스를 클러스터에서 삭제한다.
  - `-k k8s`: `k8s` 디렉토리의 `kustomization.yaml` 파일을 기반으로 Kustomize 설정을 적용한다.

### apiVersion & kind

![21-apiversion-kind.png](images%2F21-apiversion-kind.png)

- **`apiVersion`과 `kind`속성의 설정**:
  - 다른 쿠버네티스 리소스 파일과 마찬가지로 `kustomization.yaml` 파일에도 `apiVersion`과 `kind` 속성을 설정할 수 있다.
  - `kind` 속성은 `Kustomization`으로 설정해야 한다.
- **선택적 속성이지만 설정 권장**:
  - `apiVersion`과 `kind` 속성은 기술적으로 선택 사항이며, Kustomize는 기본값을 자동으로 설정한다.
  - 하지만 향후 Kustomize 버전에서 호환성이 깨지는 변경 사항이 발생할 수 있으므로 명시적으로 설정하는 것을 권장한다.
- **설정 이유**:
  - 명시적으로 설정하면 향후 Kustomize 버전 업데이트 시 발생할 수 있는 잠재적인 문제를 방지할 수 있다.
  - 코드의 명확성을 높이고 다른 개발자가 `kustomization.yaml` 파일을 이해하기 쉽게 만든다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)