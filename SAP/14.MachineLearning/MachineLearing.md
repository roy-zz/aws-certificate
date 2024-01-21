# Machine Learning

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "머신러닝"에 대해서 알아보도록 한다.

---

### Amazon Rekognition

- ML을 사용하여 객체, 사람, 텍스트, 장면을 이미지 및 비디오에서 찾는다.
- 사용자 확인을 위한 얼굴 분석 및 얼굴 검색, 사용자 수 집계를 제공한다.
- "익숙한 얼굴" 데이터베이스를 작성 또는 유명인사와 비교한다.
- 다양한 사용 사례에서 사용된다.
  - 라벨링
  - 컨텐츠 조정
  - 텍스트 탐지
  - 얼굴 감지 및 분석 (성별, 연령대, 감정 등..)
  - 얼굴 검색 및 확인
  - 셀러브리티 인지도
  - 경로 지정 (예: 스포츠 경기 분석용)

#### Content Moderation

- 부적절하거나, 원하지 않거나, 불쾌감을 주는 콘텐츠를 탐지한다. (이미지 및 비디오)
- 소셜 미디어, 방송 미디어, 광고 및 전자 상거래 상황에서 사용되어 보다 안전한 사용자 환경을 만든다.
- 플래그가 지정될 항목에 대한 최소 신뢰 임계값을 설정한다.
- 선택적으로 Amazon Augmented AI(A2I)에서 플래그 민감 컨텐츠를 수동으로 사용자가 직접 확인할 수 있다.

![1-content-moderation.png](images%2F1-content-moderation.png)

- 회사의 규정 준수에 도움을 준다.

---

### Amazon Transcribe

- 자동으로 음성을 텍스트로 변환한다.
- 자동 음성 인식(ASR, Automatic Speech Recognition)이라는 딥 러닝 프로세스를 사용하여 빠르고 정확하게 음성을 텍스트로 변환한다.
- Redaction을 사용하여 PII(Personal Identify Information)을 자동으로 제거한다.
- 다국어 오디오 자동 언어 식별을 지원한다.
- 다양한 사용 사례에서 사용된다.
  - 고객 센터에서 서비스 콜을 기록한다.
  - 자막 및 자동 자막화를 지원한다.
  - 미디어 자산의 메타데이터를 생성하여 전체 검색 가능한 아카이브를 생성한다.

---

### Amazon Polly

- 딥 러닝을 사용하여 텍스트를 실제와 같은 음성으로 바꾼다.
- 대화가 가능한 애플리케이션을 만들 수 있다.

#### Lexicon & SSML

- 발음 어휘로 단어 발음을 맞출 수 있다.
  - 스타일화된 단어: St3ph4ne => "Stephane"
  - 두문자어: AWS => "Amazon Web Service"
- 렉시콘을 업로드하고 `SynthesizeSpeech` 작업에 사용한다.
- 일반 텍스트 또는 SSML(Speech Synthesis Markup Language)로 표시된 문서에서 음성을 생성한다.
  - 더 많은 사용자 정의를 지원한다.
  - 특정 단어나 구를 강조할 수 있다.
  - 발음을 사용할 수 있다.
  - 숨소리, 속삭임을 포함할 수 있다.
  - 뉴스캐스터 화법을 사용할 수 있다.

---

### Amazon Translate

- 자연스럽고 정확한 언어 번역을 지원한다.
- Amazon Translate를 사용하면 해외 사용자를 위해 웹 사이트 및 애플리케이션과 같은 콘텐츠를 현지화할 수 있고 많은 양의 텍스트를 쉽게 번역할 수 있다.

![2-amazon-translate.png](images%2F2-amazon-translate.png)

---

### Amazon Lex & Connect

- **Amazon Lex**
  - Alexa를 작동시키는 기술과 동일한 기술이다.
  - 음성을 텍스트로 변환하는 ASR(Automatic Speech Recognition)을 지원한다.
  - 텍스트의 의도를 인식하기 위한 자연어 이해, 호출자를 제공한다.
  - 챗봇, 콜센터 봇 구축을 돕는다.
- **Amazon Connect**
  - 전화 수신, 연락처 흐름 생성, 클라우드 기반 가상 콜센터를 생성한다.
  - 다른 CRM 시스템 또는 AWS와 통합할 수 있다.
  - 선불 없이 기존 콜센터 솔류션보다 80%저렴하다.

![3-amazon-lex-connect.png](images%2F3-amazon-lex-connect.png)

- 전화로 약속을 잡고 Amazon Connect가 지정하는 번호로 예약한다.
- Lex가 이 통화의 모든 정보를 송출하고 통화의 의도를 파악할 수 있다.
- 람다 함수를 작동시켜 CRM으로 이동하여 회의를 예약한다.

---

### Amazon Comprehend

- 자연어 처리(NLP, Natural Language Processing)를 제공한다.
- 완벽하게 관리되고 서버리스로 작동하는 서비스다.
- 머신러닝을 사용하여 텍스트에서 통찰력과 관계를 찾는다.
  - 텍스트 언어
  - 주요 구문, 장소, 사용자, 브랜드 또는 이벤트 추출한다.
  - 텍스트가 얼마나 긍정적 또는 부정적인지 이해한다.
  - 토큰화 및 품사를 사용하여 텍스트를 분석한다.
  - 텍스트 파일 모음을 주제별로 자동 구성한다.
- 여러 사용 사례가 있다.
  - 고객과의 상호작용(이메일)을 분석하여 긍정적이거나 부정적인 경험으로 이어지는 요소를 찾아낸다.
  - "Comprehend"가 밝혀낼 주제별 기사 작성 및 그룹화한다.

---

### Amazon Comprehend Medical

- Amazon Comprehend Medical은 비정형 임상 텍스트에서 유용한 정보를 감지하고 반환한다.
  - 의사의 소견서
  - 방전 요약
  - 시험 결과
  - 사례 노트
- NLP를 사용하여 PHI(Protected Health Information)을 탐지한다. (`DetectPHI` API)
- S3에 문서를 저장하거나, Kinesis Data Firehose로 실시간 데이터를 분석하거나, Amazon Transcript를 사용하여 환자의 이야기를 Comprehend Medical로 분석할 수 있는 텍스트로 번역한다.

---

### Amazon SageMaker

- ML 모델 구축을 위한 개발자/데이터 분석가를 위한 완벽한 관리 서비스다.
- 일반적으로 모든 프로세스를 한 곳에서 수행하기 어렵고 서버도 프로비저닝해야 한다.
- 학생들의 시험 점수를 예측하는 시스템을 구축해본다.

![4-sage-maker.png](images%2F4-sage-maker.png)

- 학생들의 실제 퍼포먼스 히스토리로부터 데이터를 수집한다.
  - IT 경력은 몇년이고 이전 시험에서 얼마나 참여하고 모의시험은 몇번을 보았는지 등의 데이터다.
- 많은 데이터를 수집하고 레이블을 붙인다.
- 어떤 열에 상응하는지 알아야 하고 일종의 점수도 매겨야 한다.
  - 그 점수는 다른 사람이 시험에서 받은 실제 점수다.
  - 예를 들어, 670점을 받아 통과하지 못한 사람이 있다면 과정을 완전히 수료하지 않았을 수도 있다.
  - 고득점을 받은 학생이 있다면 990점이나 934점을 받은 학생일 수도 있다.
- 기계 학습 모델을 만들어서 훈련을 조율해야 한다.
  - 시간에 따라 모델을 다듬는 방법으로 데이터와 출력을 더 잘 맞츨 수 있다.
- SageMaker의 모든 것에 라벨을 추가하였고 빌드하고 트레이닝에 도움이 된다.

---

### Amazon Forecast

- ML을 사용하여 매우 정확한 예측을 제공하는 완벽한 관리 서비스다.
- 예를 들어, 우비의 향후 판매량을 예측할 수 있다.
- 데이터 자체를 보는 것보다 50% 정확도를 향상시킬 수 있다.
- 예측 시간을 몇 개월에서 몇 시간으로 단축할 수 있다.
- 다양한 활용 사례가 있다.
  - 제품 수요 계획
  - 재무 계획
  - 자원 계획

![5-amazon-forecast.png](images%2F5-amazon-forecast.png)

- 이전의 시계열 데이터를 S3로 업로드한다.
- Amazon Forecast가 예측 모델을 활용하여 어느 정도의 매출이 발생할지 예측한다.

---

### Amazon Kendra

- 머신 러닝을 기반으로 한 완벽한 관리형 문서 검색 서비스다.
- 문서 내에서 답변을 추출한다. (텍스트, pdf, HTML, PowerPoint, MS Word, FAQs 등..)
- 자연어 검색 기능을 제공한다.
- 사용자 상호 작용/피드백을 통해 학습하여 선호하는 결과를 만들어낼 수 있다.
- 검색 결과를 수동으로 미세 조정하는 기능을 제공한다.
  - 데이터의 중요성, 새로움, 사용자 정의 등..

![6-amazon-kendra.png](images%2F6-amazon-kendra.png)

---

### Amazon Personalize

- 완벽하게 관리되는 ML 서비스를 통해 실시간 맞춤형 권장 사항으로 앱을 구축할 수 있다.
- 예를 들어, 개인별 맞춤형 제품 추천/재순위, 맞춤형 직접 마케팅을 할 수 있다.
- 구체적인 예시로, 사용자가 구매한 원예 도구, 다음에 구매할 도구에 대한 추천을 제공할 수 있다.
- Amazon.com에서 사용하는 것과 동일한 기술을 제공한다.
- 기존 웹 사이트, 애플리케이션, SMS, 이메일 마케팅 시스템 등에 통합된다.
- 수개월이 아닌 수일 내에 구현할 수 있다.
  - ML 솔루션을 구축, 교육 및 구축할 필요가 없다.
- 다양한 사례에 사용된다.
  - 소매점, 메디어 및 엔터테인먼트 등..

![7-amazon-personalize.png](images%2F7-amazon-personalize.png)

---

### Amazon Textract

- AI 및 ML을 사용하여 스캔한 문서에서 텍스트, 필기 및 데이터를 자동으로 추출한다.

![8-amazon-textract.png](images%2F8-amazon-textract.png)

- 양식 및 표에서 데이터를 추출한다.
- 모든 유형의 문서(pdf, 이미지 등..)를 읽고 처리한다.
- 다양한 사용 사례가 있다.
  - 금융 서비스 (예: 송장, 재무 보고서)
  - 의료 (예: 의료 기록, 보험 청구)
  - 공공 부문 (예: 세금 양식, 신분증 문서, 여권)

---

### Summary

- Rekogintion: 얼굴 감지, 라벨링, 유명인사 인지
- Transcribe: 오디오에서 텍스트로 변환
- Polly: 텍스트에어 오디오로 변환
- Translate: 번역
- Lex: 대화 봇 구축 (챗봇)
- Connect: 클라우드 콜센터
- Comprehend: 자연어 처리
- SageMaker: 모든 개발자 및 데이터 과학자를 위한 기계 학습
- Forecast: 매우 정확한 예측 구축
- Kendra: ML 기반의 검색 엔진
- Personalize: 실시간 개인화 추천
- Textract: 문서 내 텍스트 및 데이터 탐지

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)