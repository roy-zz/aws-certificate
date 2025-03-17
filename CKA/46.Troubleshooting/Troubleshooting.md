# Troubleshooting

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "문제를 해결하는 방법들"에 대해서 알아보도록 한다.

---

## Application Failure

### 문제 해결 접근 방식의 중요성

- **체계적인 접근**: 장애 발생 시 당황하지 않고 체계적인 문제 해결 과정을 따르는 것이 중요하다.
- **애플리케이션 아키텍처 이해**: 애플리케이션의 구성 요소와 연결 관계를 정확히 이해하고 있어야 한다.
- **단계별 검증**: 각 구성 요소의 상태를 단계별로 검증하여 문제의 근본 원인을 찾아야 한다.

### 2티어 애플리케이션 예시

- **구성 요소**:
  - **웹 서버 (Web Pod)**: 사용자 요청을 처리하고 데이터베이스 서버와 통신한다.
  - **데이터베이스 서버 (DB Pod)**: 데이터를 저장하고 웹 서버의 요청에 응답한다.
  - **웹 서비스 (Web Service)**: 웹 서버 파드에 대한 접근을 제공한다.
  - **데이터베이스 서비스 (DB Service)**: 데이터베이스 서버 파드에 대한 접근을 제공한다.
- **흐름**: 사용자 요청 -> 웹 서비스 -> 웹 파드 -> 데이터베이스 서비스 -> DB 파드

### 문제 해결 단계

- **1단계: 사용자 접근성 확인**
  - **증상**: 사용자로부터 애플리케이션 접근 오류 보고
  - **진단**:
    - `curl` 명령어를 사용하여 웹 서버의 노드 포트 IP 주소에 접근 가능 여부 확인
    - 웹 브라우저를 사용하여 애플리케이션 URL에 접근 가능 여부 확인

![1-check-service-status.png](images%2F1-check-service-status.png)

- **2단계: 웹 서비스 상태 확인**
  - **진단**:
    - `kubectl describe service <웹 서비스 이름>` 명령어를 사용하여 서비스의 엔드포인트 확인
    - 엔드포인트가 웹 파드를 가리키는지 확인 (서비스와 파드의 레이블 셀렉터 일치 여부 확인)

![2-check-pod.png](images%2F2-check-pod.png)

- **3단계: 웹 파드 상태 확인**
  - **진단**:
    - `kubectl get pods <웹 파드 이름>` 명령어를 사용하여 파드의 상태 확인 (Running 상태인지 확인)
    - `kubectl describe pods <웹 파드 이름>` 명령어를 사용하여 파드의 이벤트 확인 (재시작 횟수, 오류 메시지 등)
    - `kubectl logs <웹 파드 이름>` 명령어를 사용하여 애플리케이션 로그 확인
    - `kubectl logs -p <웹 파드 이름>` 명령어를 사용하여 이전 파드의 로그 확인 (파드 재시작 시 유용)
    - `kubectl logs -f <웹 파드 이름>` 명령어를 사용하여 실시간 로그 확인 (재현 가능한 문제에 유용)

![3-check-dependent-service.png](images%2F3-check-dependent-service.png)

- **4단계: 데이터베이스 서비스 상태 확인**
  - **진단**: 2단계와 동일한 방법으로 데이터베이스 서비스의 엔드포인트 확인

![4-check-dependent-application.png](images%2F4-check-dependent-application.png)

- **5단계: 데이터베이스 파드 상태 확인**
  - **진단**: 3단계와 동일한 방법으로 데이터베이스 파드의 상태 및 로그 확인 (데이터베이스 오류 메시지 확인)

### 주요 진단 도구 및 명령어

- `kubectl get pods`: 파드의 목록 및 상태 확인
- `kubectl describe pods`: 파드 상세 정보 및 이벤트 확인
- `kubectl logs`: 파드의 애플리케이션 로그 확인
- `kubectl describe service`: 서비스 상세 정보 및 엔드포인트 확인
- `curl`: 웹 서버 접근성 확인

### 문제 해결 팁

- **애플리케이션 아키텍처 도식화**: 문제 해결 과정을 시각화하여 이해도를 높인다.
- **로그 분석**: 애플리케이션 로그 및 쿠버네티스 이벤트 로그를 면밀히 분석하여 오류 원인을 파악한다.
- **재현 가능한 문제**: `-f` 옵션으로 로그를 실시간으로 확인하여 문제 발생 시점을 관찰한다.
- **이전 파드 로그 확인**: `-p` 옵션을 사용하여 파드 재시작 전 로그를 확인한다.

![5-references.png](images%2F5-references.png)

- **쿠버네티스 문서 활용**: [쿠버네티스 공식 문서](https://kubernetes.io/docs/tasks/debug/debug-application/)의 문제 해결 가이드를 참고한다.

---

## Control Plane Failure

### 컨트롤 플레인 장애 해결의 중요성

- **클러스터 안정성**: 컨트롤 플레인 장애는 클러스터 전체의 안정성을 저해한다.
- **애플리케이션 운영**: 컨트롤 플레인 장애 시 애플리케이션 배포, 관리, 운영에 문제가 발생한다.
- **신속한 복구**: 장애 발생 시 신속하게 원인을 파악하고 복구하는 것이 중요하다.

### 컨트롤 플레인 장애 해결 단계

![6-check-node-status.png](images%2F6-check-node-status.png)

- **1단계: 노드 상태 확인**
  - `kubectl get nodes` 명령어를 사용하여 모든 노드의 상태를 확인한다.
  - `Ready` 상태가 아닌 노드가 있는지 확인한다.
  - 노드 상태가 `NotReady`인 경우, 네트워크 문제, 리소스 부족, kubelet 문제 등을 의심할 수 있다.

![7-check-controlplane-pods.png](images%2F7-check-controlplane-pods.png)

- **2단계: 파드 상태 확인 (kubeadm 설치 클러스터)**
  - 컨트롤 플레인 컴포넌트가 파드 형태로 배포된 경우 (kubeadm 설치 클러스터), `kubectl get pods -n kube-system` 명령어를 사용하여 `kube-system` 네임스페이스의 파드 상태를 확인한다.
  - `Running` 상태가 아닌 파드가 있는지 확인한다.
  - 컨트롤 플레인 컴포넌트 파드가 재시작되거나 오류가 발생하는 경우, 로그를 확인하여 원인을 파악한다.

![8-check-controlplane-service.png](images%2F8-check-controlplane-service.png)

![9-check-controlplane-service.png](images%2F9-check-controlplane-service.png)

- **3단계: 서비스 상태 확인 (네이티브 설치 클러스터)**
  - 컨트롤 플레인 컴포넌트가 서비스 형태로 배포된 경우 (네이티브 설치 클러스터), 각 컴포넌트의 서비스 상태를 확인한다.
  - **마스터 노드**: kube-apiserver, kube-controller-manager, kube-scheduler 서비스 상태 확인
  - **워커 노드**: kubelet, kube-proxy 서비스 상태 확인
  - `systemctl status <서비스 이름>` 명령어를 사용하여 서비스 상태를 확인한다.
  - 서비스가 `active (running)` 상태가 아닌 경우, 서비스 재시작 또는 로그 확인이 필요하다.

![10-check-service-logs.png](images%2F10-check-service-logs.png)

- **4단계: 컨트롤 플레인 컴포넌트 로그 확인**
  - **kubeadm 설치 클러스터**: `kubectl logs -n kube-system <파드 이름>` 명령어를 사용하여 파드의 로그를 확인한다.
  - **네이티브 설치 클러스터**: 호스트의 로깅 솔루션을 사용하여 서비스 로그를 확인한다.
    - 예: `journalctl -u kube-apiserver` 명령어를 사용하여 kube-apiserver 로그를 확인한다.
    - 로그를 분석하여 오류 메시지, 경고 메시지 등을 확인하고 원인을 파악한다.

### 주요 진단 도구 및 명령어

- `kubectl get nodes`: 노드 상태 확인
- `kubectl get pods -n kube-system`: `kube-system` 네임스페이스의 파드 상태 확인 (kubeadm)
- `systemctl status <서비스 이름>`: 서비스 상태 확인 (네이티브)
- `kubectl logs -n kube-system <파드 이름>`: 파드 로그 확인 (kubeadm)
- `journalctl -u <서비스 이름>`: 서비스 로그 확인 (네이티브)

### 문제 해결 팁

- **로그 분석**: 로그를 면밀히 분석하여 오류 원인을 파악한다.
- **재시작 및 재배포**: 문제가 해결되지 않는 경우, 컴포넌트 재시작 또는 재배포르 ㄹ시도한다.
- **네트워크 확인**: 컨트롤 플레인 컴포넌트 간 통신 문제를 확인한다.
- **리소스 확인**: 컨트롤 플레인 컴포넌트의 리소스 부족 문제를 확인한다.
- **설정 파일 확인**: 컨트롤 플레인 컴포넌트 설정 파일의 오류를 확인한다.

### 핵심 요약

- 컨트롤 플레인 장애 해결은 단계벌 접근과 로그 분석이 중요하다.
- 노드 상태, 파드 상태(kubeadm), 서비스 상태(네이티브)를 확인한다.
- `kubectl` 및 호스트 로깅 도구를 사용하여 로그를 분석한다.
- 네트워크, 리소스 설정 등 다양한 요인을 고려해야 한다.

---

## Worker Node Failure

### 워커 노드 장애 해결의 중요성

- **애플리케이션 가용성**: 워커 노드 장애는 애플리케이션의 가용성에 직접적인 영향을 미친다.
- **리소스 부족**: 워커 노드 장애는 클러스터 전체의 리소스 부족을 초래할 수 있다.
- **신속한 복구**: 장애 발생 시 신속하게 원인을 파악하고 복구하여 애플리케이션 운영을 정상화해야 한다.

### 워커 노드 장애 해결 단계

- **1단계: 노드 상태 확인**
  - `kubectl get nodes` 명령어를 사용하여 모든 노드의 상태를 확인한다.
  - `Ready` 상태가 아닌 노드가 있는지 확인한다.
  - `NotReady` 상태의 노드는 워커 노드 장애를 나타낼 수 있다.

![11-check-node-status.png](images%2F11-check-node-status.png)

- **2단계: 노드 상세 정보 확인**
  - `kubectl describe nodes <노드 이름>` 명령어를 사용하여 노드의 상세 정보를 확인한다.
  - 노드의 `Conditions` 섹션을 확인하여 장애 원인을 파악한다.
    - `OutOfDisk`: 노드의 디스크 공간 부족 (true)
    - `MemoryPressure`: 노드의 메모리 부족 (true)
    - `DiskPressure`: 노드의 디스크 용량 부족 (true)
    - `PIDPressure`: 노드의 프로세스 과다 (true)
    - `Ready`: 노드의 정상 상태 (true)
    - `Unknown`: 노드와 마스터 노드 간 통신 단절 (노드 크래시 가능성)
  - `LastHeartbeatTime` 필드를 확인하여 노드 크래시 시점을 추정한다.

![12-check-node.png](images%2F12-check-node.png)

- **3단계: 노드 자체 상태 확인**
  - 노드가 온라인 상태인지 확인한다.
  - 노드가 크래시된 경우, 노드를 재시작한다.
  - 노드의 CPU, 메모리, 디스크 공간을 확인하여 리소스 부족 문제를 해결한다.

![13-check-kubelet-status.png](images%2F13-check-kubelet-status.png)

- **4단계: kubelet 상태 확인**
  - `systemctl status kubelet` 명령어를 사용하여 kubelet 서비스 상태를 확인한다.
  - kubelet 서비스가 `active (running)` 상태인지 확인해야 한다.
  - kubelet 서비스가 비정상적인 경우, 서비스를 재시작한다.

- **5단계: kubelet 로그 확인**
  - `journalctl -u kubelet` 명령어를 사용하여 kubelet 로그를 확인한다.
  - 로그를 분석하여 오류 메시지, 경고 메시지 등을 확인하고 원인을 파악한다.

![14-check-certificates.png](images%2F14-check-certificates.png)

- **6단계: kubelet 인증서 확인**
  - kubelet 인증서가 만료되지 않았는지 확인한다.
  - kubelet 인증서가 올바른 그룹에 속해 있는지 확인한다.
  - kubelet 인증서가 올바른 CA에서 발급되었는지 확인한다.

### 주요 진단 도구 및 명령어

- `kubectl get nodes`: 노드 상태 확인
- `kubectl describe node <노드 이름>`: 노드 상세 정보 확인
- `systemctl status kubelet`: kubelet 서비스 상태 확인
- `journalctl -u kubelet`: kubelet 로그 확인

### 문제 해결 팁

- **로그 분석**: 로그를 면밀히 분석하여 오류 원인을 파악한다.
- **리소스 모니터링**: 노드의 리소스 사용량을 지속적으로 모니터링한다.
- **인증서 관련**: kubelet 인증서를 주기적으로 관리한다.
- **네트워크 확인**: 노드와 마스터 노드 간 네트워크 통신 문제를 확인한다.

### 핵심 요약

- 워커 노드 장애 해결은 노드 상태 확인, 상세 정보 확인, kubelet 상태 확인, 로그 분석이 중요하다.
- `kubectl` 및 호스트 시스템 도구를 사용하여 노드 상태 및 kubelet 상태를 확인한다.
- 노드의 리소스 부족, kubelet 문제, 인증서 문제 등을 확인하고 해결한다.
- 네트워크 리소스, 인증서 등 다양한 요인을 고려해야 한다.

---

## Network Troubleshooting

### CNI (Container Network Interface) 네트워크 플러그인

- **역할**: 쿠버네티스 클러스터 내 파드 간 통신 및 외부 네트워크 연결을 가능하게 한다.
- **종류**:
  - **Weave Net**:
    - 설치 명령어: `kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml`
  - **Flannel**:
    - 설치 명령어: `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml`
    - 현재 쿠버네티스 네트워크 정책을 지원하지 않는다.
  - **Calico**:
    - 설치 명령어: `curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O | kubectl apply -f calico.yaml`
    - 가장 진보된 CNI 플러그인으로 알려져 있다.
- **주의사항**
  - CKA/CKAD 시험에서는 CNI 플러그인 설치를 직접적으로 요구하지 않지만, 설치 URL이 제공될 수 있다.
  - kubelet은 디렉토리 내 여러 CNI 설정 파일이 존재할 경우, 사전순으로 가장 먼저 오는 설정 파일을 사용한다.
- **참고 문서**:
  - https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy

### DNS (CoreDNS)

- **역할**: 쿠버네티스 클러스터 내 서비스 및 파드의 DNS 이름 해석을 제공한다.
- **CoreDNS**: 쿠버네티스 클러스터 DNS로 사용되는 유연하고 확장 가능한 DNS 서버다.
- **리소스**:
  - `serviceaccount` (coredns)
  - `clusterrole` (coredns, kube-dns)
  - `clusterrolebinding` (coredns, kube-dns)
  - `deployment` (coredns)
  - `configmap` (coredns)
  - `service` (kube-dns)
- **Corefile 플러그인**: CoreDNS 설정은 `configmap`에 정의된 Corefile 플러그인을 통해 관리된다.
- **포트**: 53번 포트를 사용하여 DNS 해석을 수행한다.
- **설정 예시**:
  - `kubenetes cluster.local in-addr.arpa ip6.arpa { ... }`: `cluster.local` 및 역방향 도메인에 대한 쿠버네티스 백엔드 설정.
  - `proxy . /etc/resolv.conf`: 클러스터 외부 도메인을 권한 있는 DNS 서버로 전달.
- **문제 해결**:
  - **CoreDNS 파드가 Pending 상태**: 네트워크 플러그인 설치 여부를 확인한다.
  - **CoreDNS 파드가 CrashLoopBackOff 또는 Error 상태**:
    - 도커 버전 업그레이드 또는 SELinux 비활성화 (SELinux 및 도커 버전 문제인 경우)
    - `allowPrivilegeEscalation: true`: 설정 (권한 상승 문제인 경우)
    - CoreDNS 루프 문제 해결
      - kubelet 설정에 `resolvConf` 경로 설정
      - 호스트 노드의 로컬 DNS 캐시 비활성화
      - Corefile에서 `forward . /etc/resolv.conf`를 업스트림 DNS IP 주소로 변경 (임시 해결책)
  - **CoreDNS 파드 및 kube-dns 서비스 정상 작동**: `kube-dns` 서비스의 엔드포인트 확인 (`kubectl -n kube-system get ep kube-dns`)
    - 엔드포인트가 없는 경우, 서비스의 셀렉터 및 포트 설정을 확인한다.
- **참고 문서**:
  - https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/

### kube-proxy

- **역할**: 클러스터 내 각 노드에서 실행되는 네트워크 프록시로 노드에 네트워크 규칙을 유지하여 파드 간 통신을 가능하게 한다.
- **kubeadm 클러스터**: `daemonset` 형태로 배포된다.
- **기능**: 서비스 및 엔드포인트 감시, 가상 IP를 실제 파드로 트래픽 전달.
- **설정**: `config.conf` 파일에서 설정되며, `hostname-override` 옵션을 사용하여 노드 이름을 오버라이드할 수 있다.
- **문제 해결**:
  - kube-proxy 파드가 실행 중인지 확인 (`kubectl describe ds kube-proxy -n kube-system`)
  - kube-proxy 로그 확인.
  - configmap 및 설정 파일 확인.
  - kube-config 설정 확인
  - kube-proxy 프로세스 실행 여부 확인 (`netstat -plan | grep kube-proxy`)
- **참고 문서**:
  - https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/

### 핵심 요약

- 쿠버네티스 네트워크 문제 해결은 CNI 플러그인, DNS, kube-proxy의 이해를 바탕으로 접근해야 한다.
- 각 구성 요소의 상태를 확인하고 로그를 분석하여 문제의 원인을 파악한다.
- `kubectl` 및 호스트 시스템 도구를 활용하여 네트워크 문제를 진단하고 해결한다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)
- [Debug Applications Docs](https://kubernetes.io/docs/tasks/debug/debug-application/)
- [Networking and network policy](https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy)
- [DNS debugging resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)
- [Debug service](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)