# Elastic Beanstalk

이번 장에서는 **SysOps Administrator**를 준비하며 **Elastic Beanstalk**에 대해서 알아보도록 한다.

---

### Elastic Beanstalk

![1-typical-architecture.png](image%2F1-typical-architecture.png)

- 대부분의 웹/앱은 동일한 아키텍처(ALB+ASG)를 갖는다.
- 대부분의 서비스가 동일한 아키텍처를 따른다면 매번 새롭게 생성하는 것은 비효율적이다.

- "Elastic Beanstalk"는 애플리케이션 배포에 대한 개발자 중심의 관점이다.
- 위에서 살펴본 이미지의 구성요소(EC2, ASG, ELB, RDS 등)를 사용한다.
- 관리형 서비스다.
  - 용량 프로비저닝, 로드 밸런식, 확장, 애플리케이션 자동 처리 상태 모니터링, 인스턴스 구성 등
  - 애플리케이션 코드만 개발자의 책임이다.
- 완전 관리되는 서비스지만 개발자가 구성을 완전히 제어할 수 있다.
- "Beanstalk"에 대한 비용은 무료이며 생성된 서비스들에 대한 비용만 지불하면 된다.

#### Component

- Application: "Elastic Beanstalk"의 구성 요소 모음이다.(Environment, Version, Configuration 등)
- Application Version: 애플리케이션 코드의 반복이다.
- Environment:
  - 애플리케이션 버전을 실행하는 AWS 리소스 모음이다.(한 번에 하나의 애플리케이션 버전만 가질 수 있다.)
  - Tiers: 웹 서버 환경 계층 및 작업자 환경 계층이다.
  - 여러 환경(Dev, Test, Prod 등)을 생성할 수 있다.
- 새로운 버전을 업로드하고 환경을 시작하고, 이후 라이프사이클을 관리한다.
- 반복하고 싶다면 새로운 버전을 업로드해서 버전을 업데이트하고 새로운 환경을 다시 배포하여 애플리케이션 스택을 업데이트할 수 있다.

![2-beanstalk-component.png](image%2F2-beanstalk-component.png)

#### Supported Platform

- 여러 플랫폼을 지원하며 원하는 플랫폼이 없는 경우 사용자 정의 플랫폼을 작성할 수 있다.
  - Go
  - Java SE
  - Java with Tomcat
  - .NET Core on Linux
  - .NET on Windows Server
  - Node.js
  - PHP
  - Python
  - Ruby
  - Packer Builder
  - Single Container Docker
  - Multi-Container Docker
  - Preconfigured Docker

#### Web Server Tier vs Worker Tier

![3-web-server-tier.png](image%2F3-web-server-tier.png)

- 웹 계층은 전통적인 아키텍처로 부하 응답이 있다.
- 웹 서버가 될 여러 EC2 인스턴스를 가진 "Auto Scaling Group"으로 트래픽을 전송한다.

![4-worker-tier.png](image%2F4-worker-tier.png)

- Beanstalk 아키텍처는 Worker 환경을 중심으로 한다.
- EC2 인스턴스에 직접 액세스하는 클라이언트가 없다.
- SQS를 이용하여 EC2 인스턴스로 메시지를 전송하여 처리하도록 한다.
- Worker 환경의 경우 SQS의 메시지가 많을수록 EC2 인스턴스의 수도 늘릴 것이다.

#### Deployment Mode

![5-deployment-mode.png](image%2F5-deployment-mode.png)

- Beanstalk에는 두 가지 배포 모드가 있다.
- 첫 번째는 단일 인스턴스를 사용한다. 
  - 단일 EC2 인스턴스, 단일 RDS를 사용하여 개발 용도로 적합하다.
- 두 번째는 확장이 가능한 고가용성 모드다.
  - 다중 EC2 인스턴스, Load Balancer, 다중 AZ RDS를 통해서 운영 환경에서의 가용성을 높일 수 있다.

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)