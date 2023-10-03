# Machine Learning

이번 장에서는 SAA를 준비하며 **AWS의 머신러닝 서비스**에 대해서 알아보도록 한다.

---

### Amazon Rekognition

- ML을 사용하여 이미지 및 비디오에서 객체, 사람, 텍스트, 장면을 찾을 수 있다.
- 사용자 확인을 위해 얼굴 분석 및 얼굴 검색, 사람 수 파악을 할 수 있다.
- "친숙한 얼굴" 데이터베이스를 만들거나 유명인과 비교할 수 있다.
- 라벨링, 내용 조절, 텍스트 감지, 얼굴 감지 및 분석(성별, 연령대, 감정 등), 얼굴 검색 및 확인, 유명인사 인지도, 경로 지정(예: 스포츠 경기 분석) 등에 사용된다.

#### Content Moderation

- 부적절하거나 원치 않거나 불쾌감을 주는 컨텐츠를 탐지할 수 있다.(이미지 및 비디오)
- 소셜 미디어, 방송 미디어, 광고 및 전자 상거래 상황에서 사용하여 보다 안전한 사용자 환경을 구축할 수 있다.
- 플래그가 지정될 항목에 대한 최소 신뢰 임계값을 설정할 수 있다.
- "Amazon Augmented AI(2AI)"를 통해 수동 검토를 위한 민감한 컨텐츠 플래그를 지정할 수 있다.
- 규정을 준수하도록 지원할 수 있다.

![rekognition-content-moderation.png](images%2FMachineLearning%2Frekognition-content-moderation.png)

---

### Amazon Transcribe

- 자동으로 음성을 텍스트로 변환한다.
- "Automatic Speech Recognition(ASR)"이라는 딥 러닝 프로세스를 사용하여 빠르고 정확하게 음성을 텍스트로 변환할 수 있다.
- Redaction을 사용하여 PII(Personal Identifiable Information)를 자동으로 제거한다.
- 다국적 오디오 자동 언어 식별을 지원한다.
- 고객 서비스 호출을 녹음, 폐쇄 자막 및 자막 자동화, 미디어 자산의 메다데이터 생성 및 검색 가능한 아카이브 생성 등에 사용된다.

---

### Amazon Polly

- 딥 러닝을 사용하여 텍스트를 실물과 같은 말로 변형한다.
- 대화할 수 있는 응용프로그램을 생성할 수 있다.

#### Lexicon & SSML(Speech Synthesis Markup Language)

- 발음 어휘를 사용하여 단어의 발음을 사용자 지정한다.
  - Stylized words: St3ph4ne => Stephane
  - Acronyms: AWS => "Amazon Web Service"
- 어휘를 업로드하고 **SynthesizeSpeech** 작업을 한다.
- 일반 텍스트 또는 SSML(Speech Synthesis Markup Language)로 표시된 문서에서 음성을 생성한다. 보다 많은 사용자 정의를 지원한다.
  - 특정 단어나 구를 강조할 수 있다.
  - 음성 발음을 사용할 수 있다.
  - 숨소리를 포함하거나, 속삭임을 포함할 수 있다.
  - 뉴스캐스터 화법으로 생성할 수 있다.

---

### Amazon Translate

- 자연스럽고 정확한 언어 변역을 지원한다.
- "Amazon Translate"를 사용하면 해외 사용자를 위해 웹 사이트 및 응용 프로그램 등의 콘텐츠를 현지화할 수 있으며, 대량의 텍스트를 효율적으로 쉽게 변역할 수 있다.

---

### Amazon Lex & Connect

- **Amazon Lex**: (Alexa에게 힘을 주는 기술과 동일)
  - 음성을 텍스트로 변환하는 자동 음성 인식(ASR)
  - 문자, 발신자의 의도를 인식할 수 있는 자연어 이해
  - 챗봇, 콜센터 봇 구축에 도움을 준다.
- **Amazon Connect**:
  - 전화 수신, 연락처 흐름 생성, 클라우드 기반 가상 연락처 센터
  - 다른 CRM 시스템이나 AWS와 통합할 수 있다.
  - 선불금이 없으며 기존 Contact Center보다 80% 저렴하다.

![lex-connect.png](images%2FMachineLearning%2Flex-connect.png)

---

### Amazon Comprehend

- 자연어 처리용(NLP, Natural Language Processing)
- 완전 관리형 서버리스 서비스다.
- 기계 학습을 사용하여 텍스트에서 통찰력과 관계를 찾을 수 있다.
  - 텍스트 언어
  - 주요 문구, 장소, 사용자, 브랜드 또는 이벤트를 추출한다.
  - 텍스트가 얼마나 긍정적인지 또는 부정적인지 이해한다.
  - 토큰화 및 품사를 사용하여 텍스트를 분석한다.
  - 주제별로 텍스트 파일 모음을 자동으로 구성한다.
- 사용 예시:
  - 고객 상호 작용(email)을 분석하여 긍정적 또는 부정적 경험을 유도하는 것을 찾는다.
  - 이해할 수 있는 주제별로 기사를 작성하고 그룹화한다.

---

### Amazon Comprehend Medical

- 비구조화된 임상 테스트 결과에서 유용한 정보를 감지하고 반환한다.
  - 의사 소견서
  - 배출 요약
  - 시험 결과
  - 사례 노트
- NLP를 사용하여 PHI(Protected Health Information)을 탐지한다.
- 문서를 "Amazon S3"에 저장하고, "Kinesis Data Firehose"로 실시간 데이터를 분석하거나, "Amazon Transcribe"를 사용하여 환자 설명을 "Amazon Comprehend Medical"에서 분석할 수 있는 텍스트로 전부 복사할 수 있다.

---

### Amazon SageMaker

- 개발자/데이터 사이언티스트가 ML 모델을 구축할 수 있도록 지원하는 완전 관리형 서비스다.
- 일반적으로 모든 프로세스를 한 곳에서 수행하거나 프로비저닝 하는 것은 쉽지 않다.
- 머신러닝 프로세스를 간소화하여 아래와 같이 시험 점수 예측과 같은 기능을 구현할 수 있다.

![sagemaker.png](images%2FMachineLearning%2Fsagemaker.png)

---

### Amazon Forecast

- ML을 사용하여 매우 정확한 예측을 제공하는 완전 관리형 서비스다.
- 데이터 자체를 보는 것보다 50% 정도 정확도를 향상 시킬 수 있다.
- 예측 시간을 몇 달에서 몇 시간으로 단축할 수 있다.
- 제품 수요 계획, 재무 계획, 리소스 계획 등에 사용된다.
- 아래의 이미지와 같이 "우비의 향후 판매"를 예측할 수 있다.

![forcest.png](images%2FMachineLearning%2Fforcest.png)

---

### Amazon Kendra

- 머신 러닝을 기바능로 한 완전 관리형 문서 검색 서비스다.
- 문서 내에서 답변을 추출(텍스트, pdf, HTML, PowerPoint, MS Word, FAQ 등)할 수 있다.
- 자연어 검색기능을 지원한다.
- 사용자 상호 작용/피드백을 통해 학습하여 선호하는 결과를 촉진(Incremental Learning)할 수 있다.
- 검색 결과를 수동으로 미세 조정하는 기능을 지원한다. (데이터의 중요성, 새로 고침, 사용자 지정 등)

![kendra.png](images%2FMachineLearning%2Fkendra.png)

---

### Amazon Personalize

- 실시간으로 맞춤형 추천을 제공하는 앱을 구축하기 위한 완전관리형 ML 서비스다.
- 개인별 맞춤형 제품 추천/순위를 조정하거나, 맞춤형 마케팅을 할 수 있다.
  예를 들어, 정원 도구를 구매한 사용자에게 다음 구매 물건을 추천할 수 있다.
- `Amazon.com`에서 사용하는 것과 동일한 기술이다.
- 기존 웹 사이트, 애플리케이션, SMS, 이메일 마케팅 시스템 등과 통합된다.
- 몇 달이 아닌 며칠 만에 구현할 수 있으며 ML 솔루션을 구축, 교육 및 배포할 필요가 없다.
- 소매점, 미디어 및 엔터테인먼트 등에 사용된다.

![personalize.png](images%2FMachineLearning%2Fpersonalize.png)

---

### Amazon Textract

- AI 및 ML을 사용하여 스캔한 문서에서 텍스트, 필기 및 데이터를 자동으로 추출한다.

![textract.png](images%2FMachineLearning%2Ftextract.png)

- 양식 및 테이블에서 데이터를 추출한다.
- 모든 유형의 문서(PDF, 이미지 등)를 읽고 처리한다.
- 재무 서비스(송장, 재무 보고서 등), 의료(진료기록, 보험금 청구), 공공 부문(세금 양식, 신분증 문서, 여권) 등에 사용된다.

---

### AWS 머신러닝 - 요약

- **Rekognition**: 얼굴인식, 라벨링, 연예인 인식
- **Transcribe**: 음성을 문자로 변환
- **Polly**: 문자를 음성으로 변환
- **Translate**: 번역
- **Lex**: 대화형 봇 구축 - 챗봇
- **Connect**: 클라우드 컨택 센터
- **Comprehend**: 자연어 처리 프로세싱
- **SageMaker**: 모든 개발자 및 데이터 과학자를 위한 기계 학습
- **Forecast**: 매우 정확한 예측 구축
- **Kendra**: ML 기반의 검색 엔진
- **Personalize**: 실시간 개인화 추천
- **Textract**: 문서에서 텍스트 및 데이터 탐지

---

### 참고한 자료

- https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c03