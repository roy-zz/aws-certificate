# Networking (Gateway API)

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Gateway API"에 대해서 자세하게 알아보도록 한다.

---

## Gateway API

### 멀티 테넌시 문제

![1-ingress-limitation-1.png](images%2F1-ingress-limitation-1.png)

- **단일 Ingress 리소스**: 여러 서비스가 하나의 Ingress 리소스를 공유하는 경우, 각 서비스가 달느 팀이나 조직에 의해 관리된다면 충돌이 발생할 수 있다.

![2-ingress-limitation-2.png](images%2F2-ingress-limitation-2.png)

- **협업의 어려움**: 예를 들어, `wear` 서비스는 팀 A가, `video` 서비스는 팀 B가 관리하는 경우, 단일 Ingress 리소스를 공유하고 수정하는 과정에서 협업이 어려워지고 충돌이 발생할 수 있다.
- **멀티 테넌시 지원 부족**: Ingress는 멀티 테넌시 환경을 충분히 지원하지 못한다.

### 규칙 설정의 제한 사항

![3-ingress-limitation-3.png](images%2F3-ingress-limitation-3.png)

- **L7 기반 규칙만 지원**: Ingress는 호스트 매칭, 경로 매칭과 같은 L7(Application Layer) 기반의 규칙만 지원한다.
- **추가 기능 지원 부족**: TCP/UDP 라우팅, 트래픽 분할, 헤더 조작, 인증, 속도 제한 등과 같은 추가 기능은 지원하지 않는다.
- **어노테이션을 통한 컨트롤러 설정**: 추가 기능은 애노테이션을 통해 Ingress 컨트롤러에 전달된다.
  - 애노테이션을 사용하여 NGINX, Traefik 등 Ingress 컨트롤러별 설정을 추가할 수 있다.
  - 예시: SSL 리디렉션, CORS 설정 등

### 애노테이션 사용의 문제점

![4-ingress-limitation-4.png](images%2F4-ingress-limitation-4.png)

- **컨트롤러 종속성**: 애노테이션 설정은 특정 Ingress 컨트롤러(예: NGINX< Traefik)에 종속적이다.
  - 동일한 기능을 구현하더라도 컨트롤러별로 다른 애노테이션 설정을 사용해야 한다.
- **쿠버네티스의 유효성 검사 불가**: 쿠버네티스는 애노테이션 설정을 인식하지 못하므로 유효성을 검사할 수 없다.
  - 잘못된 애노테이션 설정이 적용되어도 쿠버네티스는 오류를 감지할 수 없다.
- **설정 복잡성**: 복잡한 애노테이션 설정은 가독성을 떨어뜨리고 유지 관리를 어렵게 만든다.

### Gateway API의 등장 배경

![5-gateway-api-1.png](images%2F5-gateway-api-1.png)

- **Ingress의 한계**: Ingress는 멀티 테넌시 지원 부족, L7 기반 규칙만 지원, 컨트롤러 종속적인 애노테이션 사용 등 여러 한계를 가지고 있다.
- **Gateway API**: Gateway API는 L4/L7 라우팅에 초점을 맞춘 공식 쿠버네티스 프로젝트로 Ingress의 차세대 버전이다.
- **멀티 테넌시 강화**: Gateway API는 역할 분담을 통해 멀티 테넌시를 강화한다.

### Gateway API의 핵심 구성 요소

![6-gateway-api-2.png](images%2F6-gateway-api-2.png)

- **GatewayClass**: 인프라 제공자가 구성하며 Nginx, Traefik 등 기반 네트워크 인프라(로드 밸런서)를 정의한다.
- **Gateway**: 클러스터 운영자가 구성하며 GatewayClass의 인스턴스다.
- **HTTPRoute, TCPRoute, GRPCRoute 등**: 애플리케이션 개발자가 구성하며 라우팅 규칙을 정의한다. Ingress와 달리 다양한 프로토콜을 지원한다.

### Gateway API 구성 예시

![7-gateway-api-3.png](images%2F7-gateway-api-3.png)

### Ingress와의 차이점 및 개선 사항

![8-gateway-api-4.png](images%2F8-gateway-api-4.png)

![9-ingress-gateway-api-1.png](images%2F9-ingress-gateway-api-1.png)

- **TLS 설정**:
  - **Ingress**: `spec.tls` 섹션에 기본 TLS 설정, HTTP -> HTTPS 리디렉션은 컨트롤러별 애노테이션 사용.
  - **Gateway API**: `listeners` 섹션에 HTTPS 엔드포인트 명시, `tls` 섹션에 명시적인 TLS 설정, `certificateRefs`로 시크릿 직접 참조, `allowedRoutes`로 허용된 경로 종류 지정.

![10-ingress-gateway-api-2.png](images%2F10-ingress-gateway-api-2.png)

- **카나리 배포**:
  - **Ingress**: 컨트롤러별 애노테이션을 사용하여 트래픽 분할.
  - **Gateway API**: `backendRefs` 섹션에 명시적인 트래픽 분할 설정 (v1:80%, v2:20%), 컨트롤러 독립적인 기능.

![11-ingress-limitation-1.png](images%2F11-ingress-limitation-1.png)

- **CORS 설정**:
  - **Ingress**: 컨트롤러별 복잡한 애노테이션 사용.
  - **Gateway API**: `responseHeaderModifier` 필터를 사용하여 중앙 집중식 CORS 설정, 애노테이션 불필요, 가독성 및 유지 관리 용이.

### Gateway API의 장점

- **멀티 테넌시 강화**: 역할 분담을 통해 인프라 관리자와 애플리케이션 개발자의 책임을 분리한다.
- **다양한 프로토콜 지원**: HTTP, TCP, UDP, GRPC 등 다양한 프로토콜을 지원한다.
- **컨트롤러 독립적인 설정**: 애노테이션 대신 명시적인 설정을 사용하여 컨트롤러 종속성을 제거한다.
- **가독성 및 유지 관리 용이**: 설정이 명확하고 자체 문서화되어 가독성이 높고 유지 관리가 용이하다.

### Gateway API 구현 현황

![12-gateway-controller-implementation.png](images%2F12-gateway-controller-implementation.png)

### 요약

- Gateway API는 Ingress의 한계를 극복하고 L4/L7 라우팅을 위한 차세대 쿠버네티스 API다.
- 멀티 테넌시, 다양한 프로토콜 지원, 컨트롤러 독립적인 설정, 가독성 및 유지 관리 용이성 등 다양한 장점을 제공한다.
- 다양한 컨트롤러에서 Gateway API를 지원하고 있으며 앞으로 더욱 널리 사용될 것으로 예상된다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)