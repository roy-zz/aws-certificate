# JSON Path

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "JSON Path를 활용하는 방법"에 대해서 알아보도록 한다.

---

## JSON Path

### JSON Path의 필요성

- **대규모 환경에서의 정보 추출**: 수백 개의 노드와 수천 개의 오브젝트(Pod, Deployment, Service 등)가 존재하는 프로덕션 환경에서 필요한 정보를 효율적으로 추출해야 한다.
- **특정 필드 및 조건 기반 데이터 추출**: 모든 리소스의 특정 필드 값, 특정 조건에 맞는 리소스의 데이터 등 복잡한 요구사항을 충족해야 한다.
- **데이터 필터링 및 포맷팅**: 대량의 데이터를 원하는 형식으로 필터링하고 포맷팅하여 가독성을 높여야 한다.

### kubectl과 Kubernetes API의 작동 방식

![1-kubectl.png](images%2F1-kubectl.png)

- `kubectl`은 쿠버네티스 API 서버와 통신하여 쿠버네티스 오브젝트의 정보를 요청하고 응답을 받는다.
- 쿠버네티스 API 서버는 JSON 형식으로 데이터를 주고받는다.
- `kubectl`은 JSON 형식의 응답을 받아 사람이 읽기 쉬운 형식으로 변환하여 출력한다.

![2-kubectl.png](images%2F2-kubectl.png)

- `kubectl get` 명령어에 `-o wide` 옵션을 사용하면 더 많은 정보를 출력할 수 있지만, 여전히 모든 정보를 보여주지는 않는다.
  - 예: 노드의 리소스 용량, taint, 조건, 하드웨어 아키텍처, 이미지 목록 등
- `kubectl describe` 명령어를 사용하면 상세 정보를 확인할 수 있지만, 원하는 형식으로 데이터를 추출하고 포맷팅하는 데는 한계가 있다.

![3-kubectl-jsonpath.png](images%2F3-kubectl-jsonpath.png)

### JSONPath를 사용한 데이터 추출 및 포맷팅

- JSONPath를 사용하면 `kubectl` 명령어의 출력을 원하는 대로 필터링하고 포맷팅할 수 있다.

![4-kubectl-jsonpath.png](images%2F4-kubectl-jsonpath.png)

- **4단계 JSONPath 활용 프로세스**
  1. **필요한 정보 제공 명령어 확인**: `kubectl get nodes`, `kubectl get pods` 등 필요한 정보를 제공하는 명령어를 확인한다.
  2. **JSON 형식 출력 확인**: `kubectl get <리소스> -o json` 명령어를 사용하여 JSON 형식의 출력을 확인한다.
  3. **JSONPath 쿼리 작성**: JSON 출력 구조를 분석하여 필요한 정보를 추출하는 JSONPath 쿼리를 작성한다. (예: `$.items[*].metadata.name`, `$.items[*].status.nodeInfo.architecture`, `$.items[*].status.capacity.cpu`)
  4. **JSONPath 쿼리 적용**: `kubectl get <리소스> -o jsonpath='{<JSONPath> 쿼리}'` 명령어를 사용하여 JSONPath 쿼리를 적용하고 결과를 확인한다. (쿼리는 작은따옴표와 중괄호로 묶어야 한다.)

### JSONPath 쿼리 작성 및 활용 예시

![5-jsonpath-example.png](images%2F5-jsonpath-example.png)

- **노드 이름 추출**: `kubectl get nodes -o=jsonpath='{.items[*].metadata.name}'`
- **노드 하드웨어 아키텍처 추출**: `kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.architecture}'`
- **노드 CPU 개수 추출**: `kubectl get nodes -o=jsonpath='{.items[*].status.capacity.cpu}'`

![6-jsonpath-example.png](images%2F6-jsonpath-example.png)

- **여러 정보 결합 및 출력**: `kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {"\n"} {.items[*].status.capacity.cpu}'`

### JSONPath 루프 (범위 사용)

- **목표**: JSON 배열의 각 항목을 순회하며 특정 속성을 추출하고 원하는 형식으로 출력한다.
- **필요성**: 단순히 여러 속성을 나열하는 것이 아니라, 표 형태 또는 사용자 정의된 형식으로 데이터를 출력해야 하는 경우 유용하다.

![7-loop-range.png](images%2F7-loop-range.png)

- **사용법**:
  - `range .items[*]`: `items` 배열의 각 항목을 순회한다.
  - `.metadata.name`: 각 항목의 `metadata.name` 값을 추출한다.
  - `\t`: 탭 문자를 삽입한다.
  - `.status.capacity.cpu`: 각 항목의 `status.capacity.cpu` 값을 추출한다.
  - `\n`: 줄바꿈 문자를 삽입한다.
  - `end` 루프를 종료한다.

![8-loop-range.png](images%2F8-loop-range.png)

- **예시**: `kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name} {"\t"} {.status.capacity.cpu}{"\n"}{end}'`
  - 노드 이름과 CPU 개수를 탭으로 구분하고 각 노드를 줄바꿈하여 출력한다.

### 사용자 정의 열 (Custom Column)

- **목표**: `kubectl` 명령어를 사용하여 사용자 정의 열을 생성하고 데이터를 출력한다.
- **필요성**: 루프를 사용하지 않고도 표 형식의 데이터를 쉽게 출력할 수 있다.

![9-custom-column.png](images%2F9-custom-column.png)

- **사용법**:
  - `--template '{{range .items}}{{.metadata.name}}{\t}{{.status.capacity.cpu}}{\n}{end}}'`와 같은 방법도 사용가능하지만, `--custom-columns` 옵션이 더 직관적이다.
  - `--custom-columns="<열 이름>:<JSONPath 쿼리>,<열 이름>:<JSONPath 쿼리>,...` 형식으로 사용한다.
  - JSONPath 쿼리에서 `$.items[*]` 부분을 생략한다. (각 항목에 대한 쿼리로 가정)
- **예시**: `kubectl get nodes --custom-columns="NODE:.metadata.name,CPU:.status.capacity.cpu"`
  - "NODE" 열에 노드 이름, "CPU" 열에 CPU 개수를 출력한다.

### 정렬 (Sort By)

- **목표**: `kubectl` 명령어의 출력을 특정 속성 값을 기준으로 정렬한다.
- **필요성**: 대량의 데이터를 특정 기준으로 정렬하여 원하는 정보를 쉽게 찾을 수 있다.

![10-jsonpath-sort.png](images%2F10-jsonpath-sort.png)

- **사용법**:
  - `--sort-by=<JSONPath 쿼리>` 옵션을 사용한다.
  - JSONPath 쿼리는 정렬 기준으로 사용할 속성의 경로를 지정한다.
- **예시**:
  - `kubectl get nodes --sort-by=.metadata.name`: 노드 이름을 기준으로 오름차순 정렬하여 출력한다.
  - `kubectl get nodes --sort-by=.status.capacity.cpu`: CPU 개수를 기준으로 오름차순 정렬하여 출력한다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)
- [JSON Path Quiz](https://kodekloud.com/p/json-path-quiz)