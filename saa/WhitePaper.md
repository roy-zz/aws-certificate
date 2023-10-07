# White Papers and Architectures

이번 장에서는 SAA를 준비하며 **백서(White Paper)와 시험 팁** 에 대해서 알아보도록 한다.

---

### Well Architected Framework - 일반 지침

- [여기](https://aws.amazon.com/architecture/well-architected)에서 자세한 내용을 확인할 수 있다.
- 필요한 용량을 추측하지 않는다.
- 운영 환경 규모에서 시스템을 테스트한다.
- 자동화를 통해 아키텍처 테스트를 보다 쉽게 한다.
- 진화적 아키텍처를 허용한다.
  - 변화하는 요구사항을 기반으로 설계한다.
- 데이터를 사용한 아키텍처를 구동한다.
- 게임 데이를 통해 향상 시킨다.
  - 플래시 세일 기간동안 애플리케이션 시뮬레이션

- Well Architected Framework 6가지 원칙은 아래와 같다.
  - 운영 신뢰성(Operational Excellence)
  - 보안(Security)
  - 신뢰성(Reliability)
  - 성능 효율성(Performance Efficiency)
  - 비용 최적화(Cost Optimization)
  - 지속 가능성(Sustainability)
- 이러한 원칙들은 균형을 맞추거나 포기해야 하는 부분이 아니라, 함께 함으로써 시너지를 발생시킨다.

---

### AWS Well-Architected Tool

- 6개의 기본 요소를 바탕으로 아키텍처를 검토하고 아키텍처 모범 사례를 채택할 수 있는 무료 도구다.
- 워크로드를 선택하고 질문에 답한다.
- 6개의 요소를 기준으로 답변을 검토한다.
- 비디오 및 설명서, 보고서 생성, 대시보드에서 결과를 확인하는 방식으로 조언을 얻을 수 있다.
- [여기](https://console.aws.amazon.com/wellarchitected)에서 자세한 내용을 확인할 수 있다.

![well-architected-tool.png](images%2FWhitePaper%2Fwell-architected-tool.png)

---

### Trusted Advisor

- 아무것도 설치할 필요가 없다. AWS 계쩡 평가 수준이 높다.
- AWS 계정을 분석하고 5가지 카테고리에 대한 권장 사항을 제공한다.
  - 비용 최적화(Cost Optimization)
  - 성능(Performance)
  - 보안(Security)
  - 내결함성(Fault Tolerance)
  - 서비스 제한(Service Limits)

![trusted-advisor.png](images%2FWhitePaper%2Ftrusted-advisor.png)

#### 지원 계획

- 7 Core Checks (Basic & 개발자 지원 계획)
  - S3 버킷 권한
  - Security Group - 특정 포트 제한 없음 등
  - IAM 사용(IAM 사용자 최소 1명)
  - 루트 계정의 MFA
  - EBS 공개 스냅샷
  - RDS 공개 스냅샷
  - 서비스 제한
- Full Checks (Business & 엔터프라이즈 지원 계획)
  - 5가지 카테고리에서 Full Check를 사용할 수 있다.
  - 한계치에 도달했을 때, CloudWatch 경보를 설정할 수 있다.
  - AWS Support API를 이용한 프로그래밍 액세스를 지원한다.

---

### 시험 팁

- [공식 홈페이지](https://aws.amazon.com/certification/certified-solutions-architect-associate/)를 통해 자세한 내용을 확인할 수 있다.
- 대부분의 문제는 시나리오 기반으로 진행된다.
- 모든 질문에 대해 확실히 틀린 것을 배제한다.
- 나머지 답변에 대해서는 어떤 것이 가장 합리적인지 이해한다.
- 해결책이 실현 가능해 보이지만 매우 복잡하다면 오답일 가능성이 높다.

#### FAQ

- 각 서비스의 [FAQ](https://aws.amazon.com/vpc/faqs/)를 확인하는 것이 좋다.
- FAQ는 시험에서 출제된 많은 문제를 다루고 있다.
- 서비스에 대한 이해를 확인하는 데 도움이 된다.

#### 시험

- 온라인 등록은 [홈페이지](https://www.aws.training/)에서 가능하다.
- 시험 비용은 $150다.
- 하나의 신분증을 제공해야 한다.(ID, Passport, 세부정보는 사용자에게 전달된 이메일에 있음)
- 메모, 펜 사용, 발언이 금지된다.
- 130분 동안 65개의 문제를 풀어야 한다.
- "플래그" 기능을 사용하여 다시 확인하고 싶은 문제를 표시할 수 있다.
- 마지막에는 선택적으로 모든 질문/답변을 검토할 수 있다.
- 합격을 위해서는 1000점 중에 720점 이상의 점수가 필요하다.
- 합격/불합격 여부는 5일 이내에 알 수 있다.(대부분 이보다 이전에 알 수 있다.)
- 며칠 후에 이메일을 통해서 전체 점수를 확인할 수 있다.
- 어떤 문제를 맞추고 틀렸는지 확인할 수 있다.
- 불합격하는 경우 14일 후에 다시 재시험을 볼 수 있다.

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03