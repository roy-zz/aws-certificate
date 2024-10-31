# Kubernetes Overview

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "쿠버네티스 개요"에 대해서 알아보도록 한다.

---

### 쿠버네티스란

- K8S라고도 알려진 쿠버네티스는 프로덕션 환경에서 컨테이너를 실행한 경험을 바탕으로 구글에서 구축하였다.
- K8S는 현재 오픈 소스 프로젝트이며 작성일 기준으로 가장 많이 사용되는 컨테이너 오케스트레이션 기술 중 하나이다.
- 쿠버네티스를 이해하기 위해서는 "컨테이너"와 "오케스트레이션"에 대한 이해가 필요하다.

---

### 컨테이너란

![1-why-do-you-need-containers.png](images%2F1-why-do-you-need-containers.png)

- 이미지의 프로젝트는 NodeJS를 사용하는 웹 서비스, MongoDB/CouchDB와 같은 데이터베이스, Redis와 같은 메시징 시스템, Ansible과 같은 오케스트레이션 도구와 같은 다양한 기술을 포함하는 엔드투엔드 스택을 설정해야 하는 요구 사항이 있다.
- 이러한 시스템을 운영할 때, 운영자가 겪게 되는 여러 문제점이 있다.
  - 첫째로, 기본 OS와의 호환성이다. 이렇게 다양한 서비스가 우리가 사용하려는 OS 버전과 호환되는지 확인해야 한다. 이러한 서비스의 특정 버전이 OS와 호환되지 않는 경우가 있었고, 우리는 이러한 다양한 서비스 모두와 호환되는 다른 OS를 찾아야 한다.
  - 둘째로, 이러한 서비스와 라이브러리 간의 호환성과 OS에 대한 종속성을 확인해야 한다. 한 서비스에는 한 버전의 종속 라이브러리가 필요한 반면 다른 서비스에는 다른 버전이 필요한 문제가 발생할 수 있다. 시간이 지남에 따라 애플리케이션의 아키텍처가 변경되었고, 구성 요소들을 최신 버전으로 업그레이드하거나 데이터베이스 등을 변경해야 했으며, 다양한 구성 요소와 기본 인프라 간의 호환성을 확인하는 동일한 프로세스를 거쳐야 한다. 이러한 문제를 메트릭스 지옥이라 한다.
  - 세번째로, 새로운 개발자가 합류할 때마다 새로운 환경을 설정하는 것이 어렵다. 새로운 개발자는 최종적으로 환경을 설정하기 위해 수많은 지침을 따르고 수백 개의 명령을 실행해야 한다. 그들은 올바른 운영 체제, 각 구성 요소의 올바른 버전을 사용하고 있는지 확인해야 하며, 개발자는 매번 모든 것을 직접 설정해야 한다.
  - 네번째로, 개발 환경과 운영환경이 동일하지 않다. 어떤 개발자는 A라는 OS를 사용하는 것이 편할 수 있고, 어떤 개발자는 B라는 OS를 사용하는 것이 편할 수 있다. 우리가 구축하고 있는 애플리케이션이 다른 환경에서 동일한 방식으로 실행될 것이라고 보장할 수 없다.

![2-what-can-it-do.png](images%2F2-what-can-it-do.png)

- 호환성 문제를 해결하는 데 도움이 될 수 있는 것이 필요했으며, 다른 구성 요소에 영향을 주지 않고 이러한 구성 요소를 수정하거나 변경할 수 있으며 필요에 따라 기본 운영 체제도 수정할 수 있는 방법이 필요하였다.
- 이런 상황에서, "도커"를 사용하면 자체 라이브러리와 종속성을 갖춘 별도의 컨테이너에서 각 구성 요소를 실행할 수 있다. 모두 동일한 VM 및 OS에 있지만 별도의 환경이나 컨테이너 내에 독립적으로 실행된다.
- 사용자는 도커 구성을 한 번만 구축하면 되고, 모든 개발자는 간단하게 "docker run" 명령으로 컨테이너를 실행시킬 수 있다.

![3-what-are-containers.png](images%2F3-what-are-containers.png)

- 컨테이너는 모두 동일한 OS 커털을 공유한다는 점을 제외하면 가상 머신과 마찬가지로 자체 프로세스나 서비스, 자체 네트워크 인터페이스, 자체 마운트를 가질 수 있다는 점에서 완전히 격리된 환경이다.
- 컨테이너는 현재 약 10년 동안 존재해 왔으며 다양한 유형의 컨테이너 중 일부는 LXC, LXD 컨테이너이며 이러한 컨테이너 환경을 설정하는 것은 매우 낮은 수준으로 어렵다.
- 도커는 최종 사용자가 쉽게 사용할 수 있도록 몇 가지 강력한 기능을 갖춘 높은 수준의 도구를 제공하고, LXCFS 등 도커는 LXC를 활용한다.

![4-operating-system.png](images%2F4-operating-system.png)

- 도커의 작동방식을 이해하기 위해서는 Ubuntu, Fedora, Suse, Centos와 같은 운영 체제의 작동 방식을 이해해야 한다.
- 이러한 운영 체제는 모두 두 가지 구성 요소인 OS 커널과 소프트웨어 세트로 구성되어 있다.
- OS 커널은 기본 하드웨어와 상호 작용하는 역할을 한다. OS 커널은 그대로 유지되지만 이러한 운영 체제를 서로 다르게 만드는 것은 그 위에 있는 소프트웨어다.
- 소프트웨어는 다양한 사용자 인터페이스, 드라이버, 컴파일러, 파일 관리자, 개발자 도구 등으로 구성될 수 있다.

![5-sharing-the-kernel.png](images%2F5-sharing-the-kernel.png)

- 도커 컨테이너는 기본적으로 커널을 공유하고 있다. 
- 이미지에서 도커가 설치된 Ubuntu OS가 있고 여러 소프트웨어가 작동되고 있다.
- 도커는 모두 동일한 커널을 기반으로 하는 모든 종류의 OS를 실행할 수 있다. 
- 기본 OS가 Ubuntu인 경우 도커는 Debian, Fedora, Suse, Centos와 같은 다른 배포판을 기반으로 컨테이너를 실행할 수 있다.
- 도커 컨테이너에는 추가 소프트웨어만 포함되어 있는 이러한 운영 체제를 다르게 만들고 도커는 위의 모든 소프트웨어와 작동하는 도커 호스트의 기본 커널을 활용한다.
- 도커는 하이퍼바이저와 달리 동일한 하드웨어에서 다양한 운영 체제와 커널을 가상화하고 실행하도록 설계되어 있지 않으며 주요 목적은 "애플리케이션을 컨테이너화"하는 것이다.

![6-container-virtual-macines.png](images%2F6-container-virtual-macines.png)

- 우측 이미지에서 도커의 경우 기본 하드웨어 인프라와 OS가 있고 OS에 도커가 설치되어 있다. 여기서 도커는 라이브러리 및 종속성만으로 실행되는 컨테이너를 관리한다.
- 좌측 가상 머신의 경우 기본 하드웨어 OS가 있고, ESX와 같은 일종의 가상화와 같은 하이퍼바이저와, 가상 머신이 있다. 각각의 가상 머신에는 자체 OS가 있고 종속성과 애플리케이션이 있다.
- 여러 가상 운영 체제와 커널이 실행 중이므로 오버헤드가 발생하고 기본 리소스 사용량이 높아진다.
- 가상 머신은 각각의 VM이 무겁고 일반적으로 크기가 기가바이트이므로 더 높은 디스크 공간을 소비하며, 도커 컨테이너는 가볍고 일반적으로 크기가 메가바이트이다.
- 이러한 이유로 도커 컨테이너는 일반적으로 몇 초 만에 빠르게 부팅할 수 있지만, VM은 전체 OS를 부팅해야 하기 때문에 부팅하기 위해 몇 분의 시간이 소요된다.
- 커널 등과 같은 컨테이너 간에 더 많은 리소스가 공유되므로 도커는 격리 수준이 낮다는 점을 유의해야 한다. 반면 VM은 서로 완전히 격리되어 있다.
- VM은 기본 OS나 커널에 의존하지 않으므로 리눅스 기반이나 동일한 하이퍼바이저 기반 Windows 등 다양한 유형의 OS를 실행할 수 있다.

![7-how-is-it-done.png](images%2F7-how-is-it-done.png)

- 쉽게 사용할 수 있는 컨테이너화된 버전의 애플리케이션이 많이 있으며, 대부분의 회사는 이미 도커 허브 또는 도커 스토어라는 공개 도커 레지스트리에서 서비스를 컨테이너화하고 사용할 수 있다.
- 예를 들어, 가장 일반적인 운영 체제, 데이터베이스, 기타 서비스 및 도구의 이미지를 찾을 수 있고, 해당 이미지를 식별하고 호스트에서 도커를 설치만 하면 된다.
- 이미지에서 `docker run ansible` 명령을 실행하면 도커 호스트에서 ansible 인스턴스가 실행된다.
- 마찬가지로 `docker run` 명령을 사용하여 mongodb, redis, nodejs의 인스턴스를 실행시킬 수 있다. 
- 만약 웹 서비스의 여러 인스턴스를 실행해야 하는 경우 필요한 만큼 인스턴스를 추가하고 앞단에 로드 밸런서를 구성할 수 있다. 이렇게 구성되는 경우 인스턴스 중 하나가 실패하는 경우 해당 인스턴스를 삭제하고 새로운 인스턴스를 시작하면 된다.

![8-container-vs-image.png](images%2F8-container-vs-image.png)

- 이미지는 가상화에서 VM 템플릿과 마찬가지로 패키지 또는 템플릿이며 하나 이상의 컨테이너를 만드는 데 사용된다.
- 컨테이너는 격리되어 있고 자체 환경과 프로세스 세트가 있으며 이미지에서 인스턴스를 실행한다.
- 만약 공개 저장소에서 원하는 이미지를 찾을 수 없는 경우, 이미지를 직접 만들어 도커 허브 저장소에 올려서 공개적으로 사용할 수 있다.

![9-container-advantage.png](images%2F9-container-advantage.png)

- 전통적인 프로세스를 살펴보면 개발자는 애플리케이션을 개발하고, 그런 다음 운영팀에 넘겨 프로덕션 환경에 배포하고 관리한다.
- 호스트를 설정하는 방법, 호스트에 설치해야 하는 필수 구성 요소, 종속성을 구성하는 방법 등에 대한 정보와 같은 일련의 지침을 제공하여 이를 수행한다.
- 운영팀은 애플리케이션을 개발하지 않았기 때문에 스스로 설정하는 데 어려움을 겪을 수밖에 없으며, 문제가 발생하면 개발자와 협력하여 문제를 해결한다.
- 도커를 사용하면 인프라 설정과 관련된 작업의 주요 부분이 도커 파일 형식으로 개발자의 손에 넘어가게 된다.
- 개발자가 이전에 인프라를 설정하기 위해 구축한 가이드를 이제 "Dockerfile"에 쉽게 통합하여 애플리케이션용 이미지를 생성할 수 있다.
- 이러한 이미지는 모든 컨테이너 플랫폼에서 실행될 수 있으며, 어디에서나 동일한 방식으로 실행되도록 보장되기 때문에, 운영팀은 이미지를 사용하여 애플리케잇녀을 배포할 수 있다.
- 개발자가 이미지를 구축할 때 이미지가 정상작동 하였다면, 운영 환경에서도 동일하게 작동하는 것이 보장된다. 

---

### 오케스트레이션이란

![10-container-orchestration.png](images%2F10-container-orchestration.png)

- 플랫폼은 컨테이너 간의 연결을 조정하고 부하에 따라 자동으로 확장 또는 축소해야 하며, 컨테이너를 자동으로 배포하고 관리하는 전체 프로세스를 컨테이너 오케스트레이션이라고 한다.

![11-orchestration-technologies.png](images%2F11-orchestration-technologies.png)

- 쿠버네티스를 이러한 오케스트레이션 기술이라고 하며, Docker Swarm, Mesos와 같은 여러 기술들이 있다.
- Docker Swarm은 설정 및 시작이 간단하지만 복잡한 애플리케이션에 필요한 일부 고급 자동 스케일링 기능은 부족하다.
- 반면 Mesos는 설정과 시작이 매우 어렵지만 많은 고급 기능을 지원하고 있다.
- 가장 많이 사용되는 쿠버네티스는 설정 및 시작이 약간 어렵지만 배포를 사용자 정의할 수 있는 다양한 옵션을 제공하고 복잡한 아키텍처의 배포를 지원한다.

![12-orchestration-technologies.png](images%2F12-orchestration-technologies.png)

- 컨테이너 오케스트레이션에는 다양한 장점이 있다.
- 서로 다른 노드에서 실행 중인 애플리케이션의 여러 인스턴스가 있기 때문에 하드웨어 오류로 인해 애플리케이션이 중단되지 않으므로 애플리케이션의 가용성이 높다.
- 사용자 트래픽을 다양한 컨테이너에 걸쳐 로드 밸런싱할 수 있으므로, 수요가 증가하면 더 많은 애플리케이션 인스턴스를 단 몇 초 만에 원활하게 배포할 수 있다.
- 하드웨어 리소스가 부족하면 애플리케이션을 중단하지 않고도 노드의 수를 늘리거나 줄일 수 있다.
- 이러한 작업들을 선언적 구성 파일 세트를 사용하여 이 모든 작업을 쉽게 수행할 수 있다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)