# Networking (Ingress)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Ingress"에 대해서 자세하게 알아보도록 한다.

---

## Ingress

### 초기 배포 시나리오

![1-ingress-1.png](images%2F1-ingress-1.png)

- **온라인 스토어 애플리케이션**: `my-onlinestore.com`에서 접근 가능한 온라인 스토어 애플리케이션을 쿠버네티스 클러스터에 배포한다.
- **배포 및 서비스**: 애플리케이션은 파드로 배포되고 MySQL 데이터베이스는 `mysql-service` (ClusterIP) 서비스를 통해 애플리케이션에 연결된다.
- **외부 접근**: 애플리케이션을 외부에서 접근할 수 있도록 NodePort 서비스(예: 38080 포트)를 생성한다.

![2-ingress-2.png](images%2F2-ingress-2.png)

- **DNS 설정**: DNS 서버를 설정하여 노드 IP 주소에 `my-onlinestore.com` 도메인을 연결한다.

![3-ingress-3.png](images%2F3-ingress-3.png)

- **포트 번호 제거**: 사용자가 포트 번호를 입력하지 않도록 프록시 서버를 사용하여 80 포트 요청을 38080 포트로 전달한다.

![4-ingress-4.png](images%2F4-ingress-4.png)

### 클라우드 환경에서의 배포

![5-ingress-5.png](images%2F5-ingress-5.png)

- **LoadBalancer 서비스**: Google Cloud Platform(GCP)과 같은 클라우드 환경에서는 NodePort 대신 LoadBalancer 서비스를 사용할 수 있다.
- **클라우드 로드 밸런서 프로비저닝**: 쿠버네티스는 GCP에 네트워크 로드 밸런서 프로비저닝 요청을 보내고 GCP는 트래픽을 서비스 포트로 라우팅하는 로드 밸런서에 자동으로 배포한다.
- **외부 IP 주소**: 로드 밸런서는 외부 IP 주소를 가지며 DNS를 통해 `my-onlinestore.com` 도메인에 연결된다.

### 다중 서비스 배포 및 문제점

![6-ingress-6.png](images%2F6-ingress-6.png)

- **비디오 스트리밍 서비스 추가**: `my-onlinestore.com/watch`에서 접근 가능한 비디오 스트리밍 서비스를 추가한다.
- **별도 배포 및 서비스**: 비디오 스트리밍 서비스는 별도의 배포 및 LoadBalancer 서비스(`video-service`)로 배포된다.
- **별도 로드 밸런서 및 IP 주소**: 각 LoadBalancer 서비스는 별도의 로드 밸런서 및 IP 주소를 가지므로 클라우드 비용이 증가한다.

![7-ingress-7.png](images%2F7-ingress-7.png)

- **URL 기반 라우팅**: 사용자가 입력한 URL에 따라 트래픽을 다른 서비스로 라우팅해야 한다.
- **로드 밸런서 재구성**: 새로운 서비스가 추가될 때마다 로드 밸런서를 재구성해야 한다.
- **SSL/TLS 설정**: HTTPS를 사용하여 보안을 강화해야 하며, SSL/TLS 설정은 애플리케이션, 로드 밸랜서, 프록시 서버 등 다양한 수준에서 수행할 수 있따.
- **개발자 부담 증가**: 개발자가 애플리케이션 수준에서 SSL/TLS를 구현하는 것은 부담이 된다.

### 기존 방식의 문제점

- **복잡한 구성 관리**: 다중 서비스, URL 기반 라우팅, SSL/TLS 설정 등을 관리하는 것은 복잡하고 어렵다.
- **팀 간 협업 필요**: 방화벽 규칙 설정 등 다양한 팀의 협업이 필요하다.
- **비용 증가**: 각 서비스마다 클라우드 로드 밸런서를 프로비저닝하는 것은 비용이 많이 든다.

### Ingress의 역할

![9-ingress-9.png](images%2F9-ingress-9.png)

- **단일 외부 접근 URL**: Ingress는 사용자가 애플리케이션에 접근할 수 있도록 단일 외부 접근 URL을 제공한다.
- **URL 기반 라우팅**: URL 경로에 따라 클러스터 내의 다른 서비스로 트래픽을 라우팅하도록 구성할 수 있다.
- **SSL/TLS 보안**: SSL/TLS 보안을 구성하여 HTTPS를 통한 안전한 통신을 제공한다.
- **Layer 7 로드 밸런서**: Ingress는 쿠버네티스 클러스터에 내장된 Layer 7 로드 밸런서 역할을 한다.
- **쿠버네티스 기본 요소**: 다른 쿠버네티스 객체와 마찬가지로 쿠버네티스 기본 요소를 사용하여 구성할 수 있다.

### Ingress 접근 방식

![10-ingress-10.png](images%2F10-ingress-10.png)

![11-ingress-11.png](images%2F11-ingress-11.png)

- **외부 노출**: Ingress는 클러스터 외부에서 접근할 수 있도록 노출되어야 한다.
  - NodePort 서비스를 사용하거나 클라우드 네이티브 LoadBalancer를 사용한다.
  - 이는 일회성 구성이며, 이후 모든 로드 밸런싱, SSL/TLS, URL 기반 라우팅 설정은 Ingress 컨트롤러에서 수행된다.

### Ingress 작동 방식

![12-ingress-12.png](images%2F12-ingress-12.png)

- **리버스 프록시/로드밸런싱 솔루션**: Ingress는 Nginx, HAProxy, Traefik과 같은 리버스 프록시 또는 로드 밸런싱 솔루션을 기반으로 구현된다.
- **Ingress 컨트롤러**: 지원되는 솔루션을 쿠버네티스 클러스터에 배포하여 Ingress 컨트롤러를 구성한다.
- **Ingress 리소스**: URL 라우팅, SSL/TLS 인증서 등 Ingress 구성을 정의하는 규칙 집합이다.
  - 파드, 배포, 서비스와 마찬가지로 정의 파일을 사용하여 생성된다.

### Ingress 컨트롤러

- 쿠버네티스 클러스터는 기본적으로 Ingress 컨트롤러를 제공하지 않는다.
- Ingress 리소스를 사용하려면 Ingress 컨트롤러를 수동으로 배포해야 한다.
- Ingress 컨트롤러가 배포되지 않은 상태에서 Ingress 리소스를 생성해도 작동하지 않는다.

![13-ingress-controller-1.png](images%2F13-ingress-controller-1.png)

- **쿠버네티스 프로젝트 지원**: GCE 및 Nginx는 쿠버네티스 프로젝트에서 지원 및 유지 관리된다.
- **컨트롤러의 역할**: Ingress 컨트롤러는 단순한 로드 밸런서나 Nginx 서버가 아닌 쿠버네티스 클러스터를 모니터링하고 Ingress 리소스 변경 사항을 감지하여 Nginx 서버를 동적으로 구성하는 기능을 포함한다.

![14-ingress-controller-2.png](images%2F14-ingress-controller-2.png)

![15-ingress-controller-3.png](images%2F15-ingress-controller-3.png)

![16-ingress-controller-4.png](images%2F16-ingress-controller-4.png)

![17-ingress-controller-5.png](images%2F17-ingress-controller-5.png)

- **ConfigMap**: Nginx 설정 옵션을 분리하기 위해 ConfigMap을 사용한다. (초기에는 빈 객체도 사용이 가능하다.)
- **환경 변수**: 파드 이름 및 네임스페이스를 환경 변수로 전달하여 Nginx가 파드 내부에서 설정 데이터를 읽을 수 있도록 한다.
- **포트**: Ingress 컨트롤러는 80 및 443 포트를 사용한다.

![18-ingress-controller-6.png](images%2F18-ingress-controller-6.png)

![19-ingress-controller-7.png](images%2F19-ingress-controller-7.png)

- **서비스**: NodePort 유형의 서비스를 생성하여 Ingress 컨트롤러를 외부로 노출한다.
- **서비스 계정**: Ingress 컨트롤러가 쿠버네티스 클러스터를 모니터링하고 리소스를 구성할 수 있도록 적절한 권한을 가진 서비스 계정을 생성한다.

### Ingress 리소스 구성

![20-ingress-resource-1.png](images%2F20-ingress-resource-1.png)

- **Ingress 리소스**: Ingress 컨트롤러에 적용되는 규칙 및 구성 집합니다.

![21-ingress-resource-2.png](images%2F21-ingress-resource-2.png)

- **단일 애플리케이션 라우팅**: 모든 트래픽을 단일 애플리케이션으로 전달하는 규칙을 구성할 수 있다.
- **URL 기반 라우팅**: URL 경로에 따라 다른 애플리케이션으로 트래픽을 라우팅하는 규칙을 구성할 수 있다.
  - 예: `my-online-store.com/wear` -> wear 앱, `my-online-store.com/watch` -> video 앱
- **도메인 기반 라우팅**: 도메인 이름에 따라 다른 애플리케이션으로 트래픽을 라우팅하는 규칙을 구성할 수 있다.
  - 예: `wear.my-online-store.com` -> wear 앱, `watch.my-online-store.com` -> video 앱

![22-ingress-resource-3.png](images%2F22-ingress-resource-3.png)

- **Ingress 정의 파일**: Ingress 리소스는 쿠버네티스 정의 파일(`Ingress-wear.yaml`)을 사용하여 생성된다.
  - `apiVersion`: `extensions/v1beta1` (쿠버네티스 버전에 따라 변경 가능)
  - `kind`: `Ingress`
  - `metadata`: 이름 및 기타 메타데이터
  - `spec`: 규칙 및 구성
- **단일 백엔드**: `spec.backend` 섹션에 서비스 이름 및 포트를 지정하여 모든 트래픽을 단일 서비스로 라우팅한다.

![23-ingress-resource-4.png](images%2F23-ingress-resource-4.png)

![24-ingress-resource-5.png](images%2F24-ingress-resource-5.png)

![25-ingress-resource-6.png](images%2F25-ingress-resource-6.png)

![26-ingress-resource-7.png](images%2F26-ingress-resource-7.png)

![27-ingress-resource-8.png](images%2F27-ingress-resource-8.png)

![28-ingress-resource-9.png](images%2F28-ingress-resource-9.png)

![29-ingress-resource-10.png](images%2F29-ingress-resource-10.png)

- **규칙 기반 라우팅**: `spec.rules` 섹션에 규칙을 정의하여 다양한 조건에 따라 트래픽을 라우팅한다.
  - **호스트 기반 규칙**: `host` 필드를 사용하여 도메인 이름별로 규칙을 정의한다.
  - **경로 기반 규칙**: 각 규칙 내에 `paths` 필드를 사용하여 URL 경로별로 트래픽을 라우팅한다.

![30-ingress-resource-11.png](images%2F30-ingress-resource-11.png)

- **기본 백엔드**: `kubectl describe ingress` 명령어를 사용하여 Ingress 리소스의 세부 정보를 확인하면 기본 백엔드 서비스(default-http-backend)가 표시된다.
  - 정의된 규칙과 일치하지 않는 URL에 대한 요청은 기본 백엔드 서비스로 전달된다.
  - 404 오류 페이지를 표시하는 기본 백엔드 서비스를 배포해야 한다.

### URL 기반 라우팅 예시

![31-ingress-resource-12.png](images%2F31-ingress-resource-12.png)

- `my-online-store.com` 도메인에 대한 단일 규칙을 정의한다.

![32-ingress-resource-13.png](images%2F32-ingress-resource-13.png)

![33-ingress-resource-14.png](images%2F33-ingress-resource-14.png)

- `paths` 배열을 사용하여 `/wear` 및 `/watch` 경로에 대한 백엔드 서비스를 지정한다.
- `kubectl create` 명령어를 사용하여 Ingress 리소스를 생성하고 `kubectl describe ingress` 명령어를 사용하여 세부 정보를 확인한다.

### 도메인 기반 라우팅 예시

- `wear.my-online-store.com` 및 `watch.my-online-store.com` 도메인에 대한 두 개의 규칙을 정의한다.
- 각 규칙의 `host` 필드에 도메인 이름을 지정하고 백엔드 서비스를 지정한다.

### URL 기반 vs 도메인 기반 라우팅

- URL 기반 라우팅은 단일 도메인에 대한 여러 경로를 처리하는 데 사용된다.
- 도메인 기반 라우팅은 여러 도메인에 대한 트래픽을 처리하는 데 사용된다.
- 각 규칙 내에 여러 경로를 정의하여 다양한 URL 경로를 처리할 수 있다.

---

## Ingress의 변화

- **apiVersion, serviceName, servicePort**: 이전 버전과 현재 버전의 쿠버네티스에서 Ingress의 `apiVersion`, `serviceName`, `servicePort` 등과 같은 설정에 변경 사항이 있다.
- **쿠버네티스 1.20+의 변경 사항**: 쿠버네티스 1.20 버전 이상에서는 Ingress 리소스를 명령형 방식으로 생성할 수 있다.

### 명령형 Ingress 리소스 생성

- **`kubectl create ingress` 명령어**: `kubectl create ingress` 명령어를 사용하여 Ingress 리소스를 직접 생성할 수 있다.
- **형식**: `kubectl create ingress <ingress-name> --rule="host/path=service:port"`
  - `<ingress-name>`: 생성할 Ingress 리소스의 이름
  - `--rule`: 라우팅 규칙을 정의하는 옵션
    - `host`: 도메인 이름
    - `path`: URL 경로
    - `service`: 서비스 이름
    - `port`: 서비스 포트
- **예시**: `kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80`
  - `ingress-test`라는 이름의 Ingress 리소스를 생성한다.
  - `wear.my-online-store.com` 도메인의 `/wear*` 경로로 들어오는 트래픽을 `wear-service` 서비스의 80포트로 라우팅한다. (`*`은 와일드카드 문자다.)
- **장점**: yaml 파일을 작성하지않고 CLI 환경에서 빠르게 Ingress 리소스를 생성할 수 있다.
- **명령어**: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-
- **References**: 
  - https://kubernetes.io/docs/concepts/services-networking/ingress
  - https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types

---

## rewrite-target

### Ingress 컨트롤러별 옵션

- 각 Ingress 컨트롤러는 작동 방식을 사용자 정의하기 위한 다양한 옵션을 제공한다.
- NGINX Ingress 컨트롤러는 많은 옵션을 가지고 있으며, 자세한 내용은 공식 문서를 참조할 수 있다.
- 이번에는 `rewrite-target` 옵션에 대해서 알아본다.

### 시나리오

- `watch` 앱은 `http://<watch-service>:<port>/`에서 비디오 스트리밍 웹 페이지를 표시한다.
- `wear` 앱은 `http://<wear-service>:<port>/`에서 의류 웹 페이지를 표시한다.
- Ingress를 사용하여 다음과 같은 URL 라우팅을 구성해야 한다.
  - `http://<ingress-service>:<ingress-port>/watch` -> `http://<watch-service>:<port>/`
  - `http://<ingress-service>:<ingress-port>/wear` -> `http://<wear-service>:<port>/`
- `/watch` 및 `/wear` URL 경로는 Ingress 컨트롤러에서 구성되며, 백엔드 애플리케이션에는 구성되지 않는다.

### `rewrite-target` 옵션의 필요성

- `rewrite-target` 옵션이 없으면 다음과 같은 URL 라우팅이 발생한다.
  - `http://<ingress-service>:<ingress-port>/watch` -> `http://<watch-service>:<port>/watch`
  - `http://<ingress-service>:<ingress-port>/wear` -> `http://<wear-service>:<port>/wear`
- 백엔드 애플리케이션은 `/watch` 또는 `/wear` 경로를 처리하더라도 구성되지 않았으므로 404 오류가 발생한다.
- `rewrite-target` 옵션을 사용하여 요청이 `watch` 또는 `wear` 애플리케이션으로 전달될 때 URL을 재작성해야 한다.

### `rewrite-target` 옵션 작동 방식

- `rewrite-target` 옵션은 `rules->http->paths->path` 아래에 있는 URL 경로를 `rewrite-target` 값으로 바꾼다.
- 검색 및 바꾸기 함수와 유사하게 작동한다.
- 예시: `replace(path, rewrite-target)`
- 실습 예시: `replace("/watch","/")` 또는 `replace("/wear","/")`
- Ingress yaml파일의 annotation에 "nginx.ingress.kubernetes.io/rewrite-target: /"를 추가하여 경로를 재작성할 수 있따.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
```

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)