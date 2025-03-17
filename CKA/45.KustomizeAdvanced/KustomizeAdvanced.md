# Kustomize

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Kustomize의 심화 내용들"에 대해서 알아보도록 한다.

---

## 다중 디렉토리 관리

### 여러 디렉토리의 쿠버네티스 매니페스트 관리

![1-multiple-directories.png](images%2F1-multiple-directories.png)

- 초기에는 모든 쿠버네티스 매니페스트 파일이 하나의 디렉토리(예: `k8s/`)에 있다.

![2-multiple-directories.png](images%2F2-multiple-directories.png)

- 파일 수가 증가함에 따라 관리가 어려워져서 관련 파일들을 하위 디렉토리로 분리하게 된다. (예: `k8s/api/`, `k8s/database/`)

![3-multiple-directories.png](images%2F3-multiple-directories.png)

- 하위 디렉토리로 분리된 매니페스트 파일을 배포하려면 각 디렉토리로 이동하여 `kubectl apply -f` 명령어를 실행해야 한다.
- 디렉토리 수가 증가하면 배포 및 관리가 번거로워진다.

### Kustomize를 사용한 해결 방법 (1): 루트 `kustomization.yaml` 파일

![4-multiple-directories.png](images%2F4-multiple-directories.png)

- 루드 디렉토리(`k8s/`)에 `kustomization.yaml` 파일을 생성한다.
- `resources` 섹션에 모든 하위 디렉토리의 매니페스트 파일 경로를 상대 경로로 나열한다.

![5-apply-kustomize-configs.png](images%2F5-apply-kustomize-configs.png)

- `kustomize build k8s | kubectl apply -f` 또는 `kubectl apply -k k8s` 명령어를 실행하여 모든 매니페스트 파일을 배포한다.

![6-multiple-directories.png](images%2F6-multiple-directories.png)

- 장점: 한 번의 명령어로 모든 매니페스트 파일을 배포할 수 있다.
- 단점: 하위 디렉토리 수가 증가하면 루트 `kustomization.yaml` 파일이 복잡해진다.

### Kustomize를 사용한 해결 방법 (2): 하위 디렉토리별 `kustomization.yaml` 파일

![7-multiple-directories.png](images%2F7-multiple-directories.png)

- 각 하위 디렉토리(예: `k8s/api/`, `k8s/database/`)에 `kustomization.yaml` 파일을 생성한다.
- 각 하위 디렉토리의 `kustomization.yaml` 파일의 `resources` 섹션에 해당 디렉토리의 매니페스트 파일 경로를 나열한다.
- 루트 `kustomization.yaml` 파일의 `resources` 섹션에 하위 디렉토리 경로를 나열한다.

![8-apply-kustomize-configs.png](images%2F8-apply-kustomize-configs.png)

- `kustomize build k8s | kubectl apply -f -` 또는 `kubectl apply -k k8s` 명령어를 실행하여 모든 매니페스트 파일을 배포한다.
- 장점: 하위 디렉토리로 독립적인 설정을 적용할 수 있다.
- 단점: 하위 디렉토리마다 `kustomization.yaml` 파일을 생성해야 한다.

### 핵심 개념

- **루트 `kustomization.yaml` 파일**: 전체 매니페스트 파일 배포를 관리한다.
- **하위 디렉토리 `kustomization.yaml` 파일**: 특정 하위 디렉토리의 매니페스트 파일 배포를 관리한다.
- **`resources` 섹선**: Kustomize가 관리할 매니페스트 파일 또는 하위 디렉토리 경로를 나열한다.
- **상대 경로**: `kustomization.yaml` 파일을 기준으로 매니페스트 파일 또는 하위 디렉토리 경로를 지정한다.

---

## Transformer

### Kustomize 변환(Transformers)

- Kustomize는 쿠버네티스 설정을 수정하거나 변환하기 위해 변환(Transformers)라는 기능을 제공한다.
- Kustomize는 다양한 내장 변환을 제공하며, 사용자는 사용자 정의 변환을 생성할 수도 있다.

### 공통 변환(Common Transformations)의 필요성

![9-common-transformers.png](images%2F9-common-transformers.png)

- 여러 쿠버네티스 설정 파일(YAML)에 공통 설정을 적용해야 하는 경우가 있다.
- 예를 들어, 모든 리소스에 동일한 레이블을 추가하거나, 이름에 접두사/접미사를 추가하거나, 특정 네임스페이스에 배포하거나, 공통 어노테이션을 추가하는 경우가 있다.
- 수동으로 각 파일에 설정을 추가하는 것은 비효율적이고 오류 발생 가능성이 높다.
- 공통 변환은 이러한 작업들을 자동화하여 효율성을 높인다.

### 공통 변환 종류

![10-common-transformers.png](images%2F10-common-transformers.png)

- `commonLabels`: 모든 쿠버네티스 리소스에 공통 레이블을 추가한다.
- `namePrefix`/`nameSuffix`: 모든 쿠버네티스 리소스 이름에 접두사 또는 접미사를 추가한다.
- `namespace`: 모든 쿠버네티스 리소스를 특정 네임스페이스에 배포한다.
- `commonAnnotations`: 모든 쿠버네티스 리소스에 공통 어노테이션을 추가한다.

### 공통 변환 사용 방법

- `kustomization.yaml` 파일에 해당 변환 속성을 추가하고 값을 설정한다.
- `commonLabels` 예시:

![11-commonlabel-transformer.png](images%2F11-commonlabel-transformer.png)

- `namespace` 예시:

![12-namespace-transformer.png](images%2F12-namespace-transformer.png)

- `namePrefix` / `nameSuffix` 예시:

![13-prefix-suffix-transformer.png](images%2F13-prefix-suffix-transformer.png)

- `commonAnnotations` 예시:

![14-commonannotation-transformer.png](images%2F14-commonannotation-transformer.png)

### 공통 변환 작동 방식

- `kustomization.yaml` 파일에 정의된 변환은 해당 파일을 통해 가져온 모든 쿠버네티스 리소스에 적용된다.
- `kustomize build` 명령어를 실행하면 변환이 적용된 최종 쿠버네티스 설정 파일이 생성된다.

### 공통 변환 핵심 요약

- Kustomize 공통 변환은 여러 쿠버네티스 설정 파일에 공통 설정을 효율적으로 적용하는 기능을 제공한다.
- `commonLabels`, `namePrefix` / `nameSuffix`, `namespace`, `commonAnnotations` 등의 변환을 사용하여 레이블, 이름, 네임스페이스, 어노테이션을 일괄적으로 설정할 수 있다.
- `kustomization.yaml` 파일에 변환 설정을 추가하여 사용할 수 있다.

### 이미지 변환기(Image Transformer)의 역할

- 이미지 변환기는 Kustomize를 사용하여 특정 배포 또는 컨테이너가 사용하는 이미지를 수정할 수 있도록 한다.
- 이를 통해 이미지 이름, 태그 또는 둘 다를 변경할 수 있다.

### 이미지 이름 변경

![15-image-transformer.png](images%2F15-image-transformer.png)

- `kustomization.yaml` 파일에서 `images` 속성을 사용하여 이미지 변경을 정의한다.
- `name` 속성은 변경할 이미지의 이름을 지정한다.
- `newName` 속성은 새 이미지의 이름을 지정한다.

### 컨테이너 이름과 이미지 이름의 차이

- `kustomization.yaml` 파일의 `name` 속성은 이미지 이름을 지정하며, 컨테이너 이름을 지정하는 것이 아니다.
- 배포 YAML 파일의 컨테이너 이름과 혼동하지 않도록 주의해야 한다.

### 이미지 태그 변경

![16-image-transformer.png](images%2F16-image-transformer.png)

- `newTag` 속성을 사용하여 이미지 태그를 변경할 수 있다.

### 이미지 이름과 태그 동시 변경

![17-image-transformer.png](images%2F17-image-transformer.png)

- `newName`과 `newTag` 속성을 함께 사용하여 이미지 이름과 태그를 동시에 변경할 수 있다.

### 이미지 변환기 핵심 요약

- 이미지 변환기는 Kustomize를 사용하여 쿠버네티스 배포의 컨테이너 이미지를 유연하게 수정할 수 있도록한다.
- `name`, `newName`, `newTag` 속성을 사용하여 이미지 이름과 태그를 변경할 수 있다.
- 이미지 이름과 태그를 동시에 변경할 수도 있다.

---

## Patches

### 패치(Patches) 기능 소개

![18-patches.png](images%2F18-patches.png)

- 패치는 쿠버네티스 설정 파일을 수정하는 또 다른 방법이다.
- 공통 변환(Common Transformations)과 달리, 패치는 특정 쿠버네티스 리소스의 특정 부분을 수정하는 데 사용된다.
- 공통 변환은 전체 리소스에 공통 설정을 적용하는 반면, 패치는 특정 리소스의 특정 설정만 변경한다.
- 예를 들어, 배포의 복제본 수(replicas)를 변경하는 데 사용된다.

### 패치(Patches) 구성 요소

![19-patches.png](images%2F19-patches.png)

![20-patches.png](images%2F20-patches.png)

![21-patches.png](images%2F21-patches.png)

![22-patches.png](images%2F22-patches.png)

![23-patches.png](images%2F23-patches.png)

- **작업 유형(Operation Type)**: 수행할 작업을 지정한다.
  - `add`: 리소스에 항목을 추가한다.
  - `remove`: 리소스에서 항목을 제거한다.
  - `replace`: 리소스의 값을 다른 값으로 대체한다.
- **대상(Target)**: 패치를 적용할 쿠버네티스 리소스를 지정한다.
  - `kind`, `apiVersion`, `name`, `namespace`, `labelSelector`, `annotationSelector` 등의 속성을 사용하여 리소스를 식별한다.
  - 여러 속성을 조합하여 특정 리소스를 정확하게 지정할 수 있다.
- **값(Value)**: 추가 또는 대체할 값을 지정한다.
  - `remove` 작업에는 값을 지정할 필요가 없다.

### JSON 6902 패치 & 전략적 병합 패치(Strategic Merge Patch)

![24-json-6902-strategic-merge-patch.png](images%2F24-json-6902-strategic-merge-patch.png)

#### JSON 6902 패치

- `kustomization.yaml` 파일의 `patches` 속성에 패치 설정을 정의한다.
- `target` 속성은 패치를 적용할 대상을 지정한다.
- `patch` 속성은 수행할 작업과 값을 지정한다.
- `op` 속성은 작업 유형을 지정한다.
- `path` 속성은 수정할 YAML 경로를 지정한다.
- `value` 속성은 새로운 값을 지정한다.

#### 전략적 병합 패치(Strategic Merge Patch)

- 쿠버네티스 설정 파일과 유사한 형식으로 패치를 정의한다.
- 수정할 부분만 포함하는 새로운 설정 파일을 생성하고, Kustomize가 기전 설정 파일과 병합한다.
- `target` 속성은 패치를 적용할 대상을 지정한다.
- `patch` 속성은 수정할 부분을 포함하는 새로운 설정을 지정한다.
- 기존 설정 파일에서 수정할 부분만 지정하면 Kustomize가 자동으로 병합한다.

#### 패치 정의 파일 선택

- JSON 6902 패치와 전략적 병합 패치 모두 사용할 수 있다.
- 전략적 병합 패치는 쿠버네티스 설정 파일과 유사한 형식을 사용하여 가독성이 높다.
- JSON 6902 패치는 YAML 경로를 사용하여 세밀한 수정을 할 수 있다.
- 개인적인 선호에 따라 적절한 방법을 선택할 수 있다.
- 두 가지 방법을 혼합하여 사용할 수도 있다.

### 패치 정의 방법: 인라인(Inline) vs 별도 파일(Separate File)

- Kustomize 패치(JSON 6902 또는 전략적 병합)는 두 가지 방식으로 정의할 수 있다.
  - **인라인(Inline) 방식**: `kustomization.yaml` 파일 내에 패치 내용을 직접 정의한다.
  - **별도 파일(Separate File) 방식**: 패치 내용을 별도의 YAML 파일에 정의하고, `kustomization.yaml` 파일에서 해당 파일의 경로를 지정한다.

#### 인라인(Inline) 방식

- 지금까지 살펴본 방식으로 `kustomization.yaml` 파일 내에 `patches` 속성을 사용하여 패치 내용을 직접 정의한다.
- 장점: 간단하고 빠르게 패치를 정의할 수 있다.
- 단점: 패치 수가 많아지면 `kustomization.yaml` 파일이 복잡해질 수 있다.

#### 별도 파일(Separate File) 방식

- 패치 내용을 별도의 YAML 파일(예: `replica-patch.yaml`)에 정의하고 `kustomization.yaml` 파일에서 해당 파일의 경로를 지정한다.
- `kustomization.yaml` 파일에서는 `target` 속성과 패치 파일의 경로(`path`)를 지정한다.
- 아래는 JSON 6902 패치 예시다.

![25-patch-separate-file.png](images%2F25-patch-separate-file.png)

- 아래는 Strategic merge 패치 예시다.

![26-separate-file.png](images%2F26-separate-file.png)

#### 선택 기준

- 패치 수가 적으면 인라인 방식을 사용하는 것이 편리하다.
- 패치 수가 많아지면 별도 파일 방식을 사용하여 `kustomization.yaml` 파일을 깔끔하게 유지하는 것이 좋다.
- 두 가지 방법을 혼합하여 사용할 수도 있다.

---

## Patches Dictionary

### JSON 6902 패치를 이용한 딕셔너리 키 수정

![27-replace-dictionary.png](images%2F27-replace-dictionary.png)

![28-replace-disctionary.png](images%2F28-replace-disctionary.png)

- **목표**: `api-depl.yaml` 파일의 `labels` 딕셔너리에 `component: api`를 `component: web`으로 수정한다.
- **설명**:
  - `target`: `kind: Deployment`와 `name: api-deployment`를 가진 리소스를 대상으로 지정한다.
  - `op: replace`: 기존 값을 새로운 값으로 대체하는 작업을 수행한다.
  - `path: /spec/template/metadata/labels/component`: 수정할 키의 경로를 지정한다.
  - `value: web`: 새로운 값을 지정한다.

### 전략적 병합 패치를 이용한 딕셔너리 키 수정

![29-strategic-merge-patch.png](images%2F29-strategic-merge-patch.png)

- **목표**: 동일하게 `component: api`를 `component: web`으로 수정한다.
- **설명**:
  - `label-patch.yaml`: 원본 `api-depl.yaml` 파일에서 변경할 부분(labels)만 남기고 나머지는 삭제한다.
  - Kustomize는 `label-patch.yaml` 파일과 원본 `api-depl.yaml` 파일을 병합하여 변경된 부분을 적용한다.

### JSON 6902 패치를 이용한 딕셔너리 키 추가

![30-add-dictionary.png](images%2F30-add-dictionary.png)

![31-add-dictionary.png](images%2F31-add-dictionary.png)

- **목표**: `labels` 딕셔너리에 `org: KodeKloud` 키-값을 추가한다.
- **설명**:
  - `op: add`: 새로운 키-값 쌍을 추가하는 작업을 수행한다.
  - `path: /spec/template/metadata/labels/org`: 추가할 키의 경로를 지정한다.
  - `value: KodeKloud`: 추가할 값을 지정한다.

### 전략적 패치를 이용한 딕셔너리 키 추가

![32-add-dictionary.png](images%2F32-add-dictionary.png)

- **목표**: 동일하게 `org: KodeKloud` 키-값 쌍을 추가한다.
- **설명**:
  - `label-patch.yaml`: 원본 `api-depl.yaml` 파일에서 추가할 부분(labels)만 남기고 나머지는 삭제한다.
  - Kustomize는 `labels-patch.yaml` 파일과 원본 `api-depl.yaml` 파일을 병합하여 새로운 키-값 쌍을 추가한다.

### JSON 6902 패치를 이용한 딕셔너리 키 삭제

![33-remove-dictionary.png](images%2F33-remove-dictionary.png)

- **목표**: `labels` 딕셔너리에서 `org: KodeKloud` 키-값을 삭제한다.
- **설정**:
  - `op: remove`: 키-값 쌍을 삭제하는 작업을 수행한다.
  - `path: /spec/template/metadata/labels/org`: 삭제할 키의 경로를 지정한다.

### 전략적 병합 패치를 이용한 딕셔너리 키 삭제

![34-remove-dictionary.png](images%2F34-remove-dictionary.png)

- **목표**: 동일하게 `org: KodeKloud` 키-값 쌍을 삭제한다.
- **설명**: 
  - `label-patch.yaml` 파일에 삭제할 키의 값을 `null`로 설정한다.
  - Kustomize는 `label-patch.yaml` 파일과 원본 `api-depl.yaml` 파일을 병합하여 해당 키-값 쌍을 삭제한다.

### 핵심 요약

- JSON 6902 패치는 `op`, `path`, `value` 속성을 사용하여 딕셔너리 키를 추가, 수정 삭제한다.
- 전략적 병합 패치는 원본 파일에서 변경할 부분만 남기고 나머지는 삭제하여 병합을 통해 변경을 적용한다.
- 전략적 병합 패치는 삭제 시 키의 값을 `null`로 설정한다.
- 두 가지 방법 모두 딕셔너리 키를 효과적으로 관리할 수 있으며 상황에 따라 적절한 방법을 선택할 수 있다.

---

## Patch List

### JSON 6902 패치를 이용한 리스트 항모 수정

![35-replace-list.png](images%2F35-replace-list.png)

- **목표**: `api-depl.yaml` 파일의 `containers` 리스트에서 첫 번째 컨테이너의 이름과 이미지를 `haproxy`로 변경한다.
- **설명**:
  - `target`: `kind: Deployment`와 `name: api-deployment`를 가진 리소스를 대상으로 지정한다.
  - `op: replace`: 기존 값을 새로운 값으로 대체하는 작업을 수행한다.
  - `path: /spec/template/spec/containers/0/name`: 수정할 컨테이너 이름의 경로를 지정한다.
    - `/0`은 리스트의 첫 번째 항목(인덱스 0)을 의미한다.
  - `value: haproxy`: 새로운 컨테이너 이름을 지정한다.
  - `path: /spec/template/spec/containers/0/image`: 수정할 컨테이너 이미지의 경로를 지정한다.
  - `value: haproxy`: 새로운 컨테이너 이미지를 지정한다.

### 전략적 병합 패치를 이용한 리스트 항목 수정

![36-replace-list.png](images%2F36-replace-list.png)

- **목표**: 동일하게 첫 번째 컨테이너의 이미지를 `haproxy`로 변경한다.
- **설명**:
  - `label-path.yaml`: 원본 `api-depl.yaml` 파일에서 변경할 부분(containers)만 남기고 나머지는 삭제한다.
  - Kustomize는 `label-patch.yaml` 파일과 원본 `api-depl.yaml` 파일을 병합하여 변경된 부분을 적용한다.

### JSON 6902 패치를 이용한 리스트 항목 추가

![37-add-list.png](images%2F37-add-list.png)

- **목표**: `containers` 리스트에 새로운 컨테이너를 추가한다.
- **설명**:
  - `op: add`: 새로운 항목을 추가하는 작업을 수행한다.
  - `path: /spec/template/spec/containers/-`: 추가할 위치를 지정한다.
    - `-`는 리스트의 마지막에 추가함을 의미한다.
    - 특정 인덱스를 지정하여 원하는 위치에 추가할 수도 있다.
  - `value`: 추가할 컨테이너의 이름과 이미지를 지정한다.

### 전략적 병합 패치를 이용한 리스트 항목 추가

![38-add-list.png](images%2F38-add-list.png)

- **목표**: 동일하게 새로운 컨테이너를 추가한다.
- **설명**: 
  - `label-patch.yaml`: 원본 `api-depl.yaml` 파일에서 추가할 컨테이너를 정의한다.
  - Kustomize는 `label-patch.yaml` 파일과 원본 `api-depl.yaml` 파일을 병합하여 새로운 컨테이너를 추가한다.

### JSON 6902 패치를 이용한 리스트 항목 삭제

![39-delete-list.png](images%2F39-delete-list.png)

- **목표**: `containers` 리스트에서 두 번째 컨테이너를 삭제한다.
- **설명**:
  - `op: remove`: 항목을 삭제하는 작업을 수행한다.
  - `path: /spec/template/spec/containers/1`: 삭제할 컨테이너의 인덱스를 지정한다.

### 전략적 병합 패치를 이용한 리스트 항목 삭제

![40-delete-list.png](images%2F40-delete-list.png)

- **목표**: 동일하게 두 번째 컨테이너를 삭제한다.
- **설명**:
  - `label-patch.yaml`: 삭제할 컨테이너를 정의하고 `$patch: delete`를 사용하여 삭제를 명시한다.
  - Kustomize는 `label-patch.yaml` 파일과 원본 `api-depl.yaml` 파일을 병합하여 해당 컨테이너를 삭제한다.

### 핵심 요약

- JSON 6902 패치는 `op`, `path`, `value` 속성을 사용하여 리스트 항목을 수정, 추가 삭제한다.
  - `path`에 인덱스를 사용하여 특정 항목을 지정하고, `-`를 사용하여 마지막에 추가할 수 있다.
- 전략적 병합 패치는 원본 파일에서 변경할 부분만 남기고 나머지는 삭제하여 병합을 통해 변경을 적용한다.
  - 삭제 시 `$patch: delete`를 사용한다.
- 두 가지 방법 모두 리스트 항목을 효과적으로 관리할 수 있으며, 상황에 따라 적절한 방법을 선택할 수 있다.

---

## Overlay

### Kustomize 오버레이의 필요성 및 개념

- **문제점**: 동일한 쿠버네티스 설정 파일을 여러 환경(개발, 스테이징, 프로덕션)에서 사용할 때, 환경별로 일부 설정 값을 변경해야 하는 경우가 발생한다.
- **해결책**: Kustomize 오버레이를 사용하여 기본 설정(base)을 유지하고 환경별 설정(overlay)을 추가하여 관리한다.
- **핵심 개념**:
  - **Base**: 모든 환경에서 공유되는 기본 쿠버네티스 설정 파일들을 포함하는 디렉토리다.
  - **Overlay**: 특정 환경에 특화된 설정 파일 및 패치 파일들을 포함하는 디렉토리다.
  - **Patch**: Base 설정 파일의 특정 부분을 변경하는 데 사용되는 파일들이다.

### 일반적인 Kustomize 프로젝트 폴더 구조

![41-overlays.png](images%2F41-overlays.png)

- `k8s`: 루트 디렉토리
- `base`: 기본 설정 파일들을 포함하는 디렉토리
- `overlays`: 환경별 설정 파일들을 포함하는 디렉토리
- `dev`, `stg`, `prod`: 각 환경에 특화된 설정 파일들을 포함하는 디렉토리

### Overlay 설정 (overlays/dev/kustomization.yaml, etc.)

![42-overlays.png](images%2F42-overlays.png)

- Overlay 디렉토리에는 각 환경에 특화된 설정 파일 및 패치 파일들이 포함된다.
- `kustomization.yaml` 파일은 `bases` 속성을 사용하여 Base 디렉토리의 경로를 정의한다.
- `patch` 속성을 사용하여 Base 설정 파일의 특정 부분을 변경하는 패치 파일들을 정의한다.
- `resources` 속성을 사용하여 Overlay 환경에만 존재하는 새로운 리소스를 추가할 수 있다.

### `bases` 속성

- `bases` 속성은 Overlay 디렉토리의 `kustomization.yaml` 파일에서 Base 디렉토리의 경로를 지정하는 데 사용된다.
- 상대 경로를 사용하여 Base 디렉토리의 경로를 지정한다.
- `../..`와 같은 형식으로 상위 디렉토리를 참조할 수 있다.

### `patches` 속성

- `patches` 속성은 Base 설정 파일의 특정 부분을 변경하는 패치 파일들을 정의하는 데 사용된다.
- 패치 파일은 JSON Patch 또는 Strategic Merge Patch 형식을 사용할 수 있다.
- 각 환경에 특화된 설정 값을 변경하는 데 사용된다.

### Overlay 환경에 새로운 리소스 추가 (overlays/prod/grafana-depl.yaml)

![43-overlays.png](images%2F43-overlays.png)

- Overlay 디렉토리에는 Base 디렉토리에 없는 새로운 리소스 파일을 추가할 수 있다.
- `Kustomization.yaml` 파일의 `resources` 속성을 사용하여 새로운 리소스 파일을 정의한다.
- 특정 환경에만 필요한 리소스를 추가하는 데 유용하다.

### 디렉토리 구조의 유연성

![44-overlays.png](images%2F44-overlays.png)

- Kustomize는 디렉토리 구조에 대한 엄격한 규칙을 강제하지 않는다.
- Base 및 Overlay 디렉토리를 하위 디렉토리로 분할하여 관리할 수 있다.
- 기능별, 컴포넌트별로 디렉토리를 분할하여 관리하는 것이 가능하다.
- 각 환경별로 다른 디렉토리 구조를 사용할 수 있다.
- `kustomization.yaml` 파일의 `resources` 속성을 사용하여 올바른 리소스 파일을 가져오도록 설정해야 한다.

---

## Component

### 컴포넌트의 필요성 및 개념

![45-components.png](images%2F45-components.png)

![46-components.png](images%2F46-components.png)

![47-components.png](images%2F47-components.png)

- **문제점**:
  - 여러 오버레이에서 동일한 설정 블록(예: 특정 기능 활성화 설정)을 방복적으로 사용해야 하는 경우, 코드 중복 및 설정 불일치 문제가 발생할 수 있다.
  - 특정 기능이 일부 오버레이에서만 활성화되어야 하는 경우, 베이스 설정에 해당 설정을 포함할 수 없다.

![48-components.png](images%2F48-components.png)

- **해결책**:
  - Kustomize 컴포넌트를 사용하여 재사용 가능한 설정 블록을 정의하고, 필요한 오버레이에서 해당 컴포넌트를 가져와 사용한다.
- **핵심 개념**:
  - **컴포넌트**: 특정 기능을 구현하는 데 필요한 모든 쿠버네티스 설정(리소스, 패치, 설정 맵, 시크릿 등)을 포함하는 재사용 가능한 설정 블록이다.
  - **재사용성**: 컴포넌트를 한 번 정의하면 여러 오버레이에서 재사용할 수 있다.
  - **모듈화**: 기능을 컴포넌트로 분리하여 설정 관리를 단순화하고 유지보수성을 향상시킨다.

### 컴포넌트 사용 시나리오

- 애플리케이션이 여러 선택적 기능을 지원하고, 각 기능이 일부 오버레이에서만 활성화되어야 하는 경우.
- 여러 오버레이에서 공통적으로 사용되는 설정 블록을 재사용해야 하는 경우.
- 설정 코드 중복을 제거하고 유지보수성을 향상시켜야 하는 경우.

### 컴포넌트 구성 요소

- **kustomization.yaml**: 컴포넌트의 메타데이터 및 리소스 목록을 정의한다.
- **리소스 파일**: 컴포넌트가 생성하는 쿠버네티스 리소스(배포, 서비스, 설정 맵, 시크릿 등)를 정의한다.
- **패치 파일**: 베이스 설정의 특정 부분을 변경하는 패치를 정의한다.

### 예시

- 컴포넌트 리소스 파일 (`/components/database/postgres-depl.yaml`)

![49-postgres-depl.png](images%2F49-postgres-depl.png)

- 컴포넌트 정의 파일 (`/components/database/kustomization.yaml`)

![50-kustomization.png](images%2F50-kustomization.png)

- 컴포넌트 패치 파일 (`/components/database/deployment-patch.yaml`)

![51-deployment-patch.png](images%2F51-deployment-patch.png)

- 오버레이에서 컴포넌트 사용 (`overlays/premium/kustomization.yaml`)
  - `components` 속성을 사용하여 사용할 컴포넌트의 경로를 정의한다.
  - 상대 경로를 사용하여 컴포넌트 디렉토리를 지정한다.

![52-kustomization.png](images%2F52-kustomization.png)

### 컴포넌트 사용 장점

- **코드 재사용성**: 동일한 설정 블록을 여러 오버레이에서 재사용하여 코드 중복을 제거한다.
- **유지보수성 향상**: 컴포넌트를 수정하면 해당 컴포넌트를 사용하는 모든 오버레이에 변경 사항이 적용된다.
- **모듈화**: 기능을 컴포넌트로 분리하여 설정 관리를 단순화하고 가독성을 높인다.
- **설정 일관성 유지**: 여러 오버레이에서 동일한 컴포넌트를 사용하므로 설정 일관성을 유지할 수 있다.
- **기능별 관리**: 기능별로 설정을 분리하여 관리하므로 설정 변경 시 영향 범위를 최소화할 수 있다.

### 핵심 요약

- Kustomize 컴포넌트는 재사용 가능한 설정 블록을 정의하여 코드 중복을 제거하고 설정 관리를 단순화한다.
- 컴포넌트는 리소스, 패치, 설정 맵, 시크릿 등을 포함할 수 있다.
- `components` 속성을 사용하여 오버레이에서 컴포넌트를 가져와 사용할 수 있다.
- 컴포넌트를 사용하면 설정의 재사용성, 유지보수성, 일관성을 높일 수 있다.
- 기능별 컴포넌트의 구성으로 설정 변경에 대한 영향도를 최소화할 수 있다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)