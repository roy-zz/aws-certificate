# Docker vs ContainerD

- 이번 장에서는 **Certified Kubernetes Administrator (CKA)** 을 준비하며 "Docker와 ContainerD의 차이점"에 대해서 알아보도록 한다.

---

![1-docker-conatinerd.png](images%2F1-docker-conatinerd.png)

- 오래된 글을 확인해보면 도커와 함께 쿠버네티스가 언급되는 것을 볼 수 있지만, 비교적 최신의 글을 보면 ContainerD와 쿠버네티스가 언급되는 것을 확인할 수 있다.
- ctl, nerdctl, crictl과 같은 여러 CLI 도구가 있으며 우리는 어떤 도구를 사용할 것인지 선택해야 한다.

![2-origin.png](images%2F2-origin.png)

- 처음에는 ContainerD가 존재하지 않았고, 도커 및 로켓과 같은 컨테이너 도구만 존재하였다.
- 하지만 도커의 사용자 경험으로 인해 컨테이너 작업은 아주 간단해져 도커는 대부분의 사용자가 사용하는 컨테이너 도구가 되었다.
- 쿠버네티스는 도커를 위해 초기부터 만들어졌으며, 목적과 같이 도커와 잘 동작하였다. 
- 이 당시의 쿠버네티스는 도커만 지원하고 다른 컨테이너 솔류션은 지원하지 않았기 때문에 컨테이너 오케스트레이터로 인기가 높아졌다.
- 도커가 아닌 로켓(rkt)이라는 컨테이너 도구가 등장하면서 쿠버네티스 사용자들은 도커가 아닌 컨테이너 런타임과 작업할 일이 생기기 시작하였다.
- 쿠버네티스는 표준을 만들기 위해서 CLI라고 불리는 Container Runtime Interface를 소개하게 된다.
- CLI는 어떤 도구가 되었든 OCI 표준을 준수한다면 쿠버네티스를 위한 컨테이너 런타임으로 사용될 수 있다.
- OCI는 Open Container Initiative의 약자로 이미지 스펙과 런타임 스펙으로 구성되어 있다.
- 이미지 스펙은 이미지가 빌드되는 방법을 스펙으로 정의하고 런타임 스펙은 모든 컨테이너 런타임이 어떻게 개발되어야 하는지에 대한 표준을 정의하고 있다.
- 이러한 표준을 준수한다면 누구나 컨테이너 런타임을 빌드할 수 있다.
- 로켓의 경우 OCI 표준을 준수하기 때문에 쿠버네티스의 컨테이너 런타임으로 지원된다.
- **하지만 도커는 CRI 기준을 만족하도록 만들어지지 않았다.** 도커는 CRI가 도입되기 훨씬 전에 만들어졌고 여전히 대부분의 사용자들이 사용하는 컨테이너 도구이기 때문이다.
- 하지만 사용자들이 도커를 여전히 사용했기 때문에 쿠버네티스는 "dockershim"을 도입하여 도커를 지원했지만 어디까지나 CRI 인터페이스 밖에서 도커를 계속 지원하는 임시 방법이다.

- ![3-dockershim.png](images%2F3-dockershim.png)

- 쿠버네티스는 "dockershim"을 유지하는 것은 불필요한 노력과 복잡성 때문에 버전 1로 결정하였고, 도커에 대한 지원을 더 이상하지 않는다.

![4-containerd.png](images%2F4-containerd.png)

- ContainerD는 도커에 속해 있지만 현재는 독립된 프로젝트이며 CNCF를 졸업하였다.
- 실제로 도커를 설치하지 않더라도 ContainerD를 설치할 수 있으며, 도커의 다른 기능들이 필요 없다면 ContainerD만 설치해도 상관없다.

![5-client-ctr.png](images%2F5-client-ctr.png)

- ContainerD를 설치하면 "ctr"이라는 명령줄 도구를 제공한다.
- 이 도구는 ContainerD 디버깅만을 위해 만들어졌고 제한된 기능만을 지원하기 때문에 사용자 친화적이지 않다.

![6-nerdctl.png](images%2F6-nerdctl.png)

- 더 나은 대안으로 "nerdctl"이 있으며 명령줄 도구로 도커와 아주 유사한 사용 방법을 가지고 있으며 도커가 지원하는 대부분의 옵션을 지원한다.
- 추가적인 이점으로는 ContainerD에 구현된 새로운 기능에 액세스할 수 있는 권한을 준다.
- 예를 들어, 암호화된 컨테이너 이미지로 작업할 수 있으며, 이는 미래에 도커에서 지원될 다소 새로운 기능이다.
- 이미지 풀링, P2P 이미지 배포 이미지 서명 및 확인, 네임스페이스를 지원하며 이는 도커에서는 불가능하다.
- 이러한 점을 보았을 때, 도커 대신 "nerdctl"로 대체하는 것이 가장 이상적인 방법이다.

![7-clictl.png](images%2F7-clictl.png)

- 또 다른 도구로는 "crictl"이 있다.
- 기본적으로 컨테이너 관련 작업을 수행하는데 사용되며, 이미지 가져오기, 기존 이미지 목록 조회하기 같은 기능을 제공한다.

![8-docker-crictl.png](images%2F8-docker-crictl.png)

- 공식 문서를 확인해보면 도커 명령어과 "crictl" 명령어를 비교하는 내용을 확인할 수 있다. 
  - 작성일 기준으로 위의 이미지 내용을 더 이상 공식 문서에서 확인할 수 없다. "dockershim" 지원을 멈추었기 때문으로 예상한다.

---

### 참고한 강의

- [Kubernetes for the Absolute Beginners](https://www.udemy.com/course/learn-kubernetes)
- [Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)