# Caching

- 이번 장에서는 **Solutions Architect Professional (SAP)** 을 준비하며 "캐싱"에 대해서 알아보도록 한다.

---

### Amazon CloudFront

- Content Delivery Network로 CDN으로 부른다.
- 읽기 성능 향상시키며 엣지에 콘텐츠를 캐시한다.
- 225개 이상의 포인트를 가지고 있다.
  - 215개 이상의 엣지 로케이션과, 13개의 지역 엣지 캐시
- 네트워크 및 애플리케이션 계층 공격(예: DDoS 공격)으로부터 보호한다.
- AWS Shield, AWS WAF 및 Route 53과 통합된다.
- 외부에 HTTPS를 노출하고 내부 HTTPS 백엔드와 통신할 수 있다.
- WebSocket 프로토콜을 지원한다.

![1-cloudfront.png](images%2F1-cloudfront.png)

- 애플리케이션ㅇ르 배포하거나 S3 버킷에 웹사이트나 파일을 배포할 수 있다.
- 글로벌 사용자가 액세스하기를 원한다.
- 미국의 사용자가 미국의 캐시에 접근해서 S3 버킷에 요청하고 그 요청 결과를 엣지에 캐싱한다.
- 중국의 사용자도 아시아에서 S3 버킷에 접속하고 아시아의 엣지에 결과가 캐싱된다.

#### Origins

- **S3 Bucket**
  - 파일 배포용으로 사용된다.
  - OAC(Origin Access Control)를 통해 보안을 강화한다.
  - 최신인 OAC가 OAI(Origin Access Identity)를 대체한다.
  - CloudFront를 사용하여 파일을 S3에 업로드할 수 있다.
- **웹 사이트로 구성된 S3 버킷**
  - 먼저 버킷에서 정적 웹 사이트 호스팅을 사용한다.
- **MediaStore Container & MediaPackage Endpoint**
  - AWS Media 서비스를 사용하여 VOD(Video On Demand) 또는 라이브 스트리밍을 제공할 수 있다.
- **Custom Origin (HTTP)**
  - EC2 인스턴스
  - Elastic Load Balancer (CLB or ALB)
  - API 게이트웨이 (더 많은 제어를 위해 사용하거나 API 게이트웨이 엣지 사용)
  - 사용자가 원하는 HTTP 백엔드

#### 오리진으로써 S3

![2-s3-as-an-origin.png](images%2F2-s3-as-an-origin.png)

- 오리진으로 사용되는 S3 버킷이 있다.
- OAC를 사용함으로써 CloudFront와 S3 버킷 사이의 밀접한 통합이 이루어진다.
- S3 버킷의 콘텐츠는 다양한 위치의 엣지에 캐싱된다.
  - LA 엣지, 뭄바이 엣지, 상파울루 엣지, 멜버른 엣지에 각각 캐싱된다.
- 캐싱된 후에는 오리진에서 데이터를 조회하는 것이 아니라 엣지에서 조회하여 서비스하게 된다.

#### CloudFront vs S3 Cross Region Replication

- **CloudFront**
  - 글로벌 엣지 로케이션이다.
  - 파일이 TTL용으로 캐싱된다. (하루정도)
  - 어디에서나 사용할 수 있어야 하는 **정적 컨텐츠**에 적합하다.
- **S3 Cross Region Replication**
  - 복제를 수행할 각 영역에 대해 설정해야 한다.
  - 거의 실시간으로 파일이 업데이트된다.
  - 읽기 전용이다.
  - 일부 지역에서 낮은 지연 시간에 사용할 수 있어야 하는 **동적 컨텐츠**에 적합하다.

#### 오리진으로써 EC2 & ALB

![3-ec2-alb-as-an-origin.png](images%2F3-ec2-alb-as-an-origin.png)

- EC2 인스턴스나 ALB와 같은 HTTP 유형을 CloudFront의 오리진으로 사용할 수 있다.
- 전자의 경우 엣지 로케이션이 액세스할 수 있도록 EC2 인스턴스가 공개되어 있어야 한다.
  - 보안 그룹을 사용자 정의해 엣지 로케이션의 공용 IP만 허용할 수 있도록 설정할 수 있다.
- ALB를 사용해서 EC2 인스턴스를 공개하는 것이 가능하다.
  - 이 경우에는 EC2 인스턴스가 공개될 필요없다.
  - ALB만 공개되면 되고 EC2 인스턴스는 비공개 상태로 보안 그룹에 ALB가 접속할 수 있도록 설정하면 된다.
- 허용해야 하는 CloudFront의 IP 주소 목록은 [여기](http://d7uri8nf7uskq.cloudfront.net/tools/list-cloudfront-ips)에서 확인할 수 있다.

#### ALB 및 사용자 지정 오리진에 대한 액세스 제한

- ALB 또는 사용자 지정 오리진에 대한 직접 액세스 방지를 설정할 수 있다.
  - 설정되는 경우 사용자는 CloudFront를 통해서만 접근해야 한다.
- 먼저 CloudFront를 구성하여 사용자 정의 HTTP 헤더를 추가하여 ALB로 전송하도록 요청한다.
- 다음으로, ALB가 해당 사용자 정의 HTTP 헤더가 포함된 요청만 전달하도록 구성한다.
- **사용자 정의 헤더 이름 및 값은 비밀로 유지한다.**

![4-restrict-access-alb-custom-origin.png](images%2F4-restrict-access-alb-custom-origin.png)

- 사용자들은 CloudFront에 접속할 수 있다.
- CloudFront에서 HTTP 요청을 수행하면 CloudFront은 비밀 값이 있는 사용자 지정 헤더를 추가한다. (`X-Custom-Header: djdfhsb12121`)
- 사용자들이 임의로 오리진에 접근하게 되면 헤더의 값이 없기 때문에 요청이 거부당한다.
- 보안 그룹을 이용하여 CloudFront의 공개 IP 주소만을 허용함으로써 네트워크 관점에서의 접근을 제한할 수 있다.

#### Origin Groups

- 고가용성(HA)을 높이고 Failover를 수행하기 위해 사용된다.
- Origin Group: 하나의 1차 오리진과, 하나의 2차 오리진으로 구성된다.
- 만약 1차 오리진이 실패하면 사용자들은 2차 오리진을 사용하게 된다.
- Origin은 여러 AWS 리전에 걸쳐있을 수 있다.

- **Cross-Region High Availability**

![5-cross-region-high-availability.png](images%2F5-cross-region-high-availability.png)

- 클라이언트는 CloudFront에 접속한다.
- `us-east-1` 리전에 있는 1차 오리진에 요청을 보낸다.
  - 1차 오리진은 응답과 함께 에러 상태 코드를 반환한다.
- 만약 에러가 발생한 경우 CloudFront는 2차 오리진인 `us-west-1`로 동일한 요청을 전달한다.

- **S3 + CloudFront - Regional-level High Availability**

![6-s3-cloudfront-region-level-high-availability.png](images%2F6-s3-cloudfront-region-level-high-availability.png)

- S3 버킷에도 1차 오리진과 2차 오리진이 있을 수 있다.
- 두 개의 버킷은 복제되도록 설정되어 있다.
- 하나의 버킷이 다운되면 CloudFront는 다른 리전에 있는 두 번째 버킷에 동일한 요청을 시도한다.
- 복제를 통해서 데이터가 최신 상태여야 한다.

#### CloudFront Geo Restriction

- 배포에 액세스할 수 있는 사용자를 제한할 수 있다.
  - Allow list: 사용자가 승인된 국가 목록에 있는 국가 중 하나에 있는 경우에만 컨텐츠에 액세스할 수 있다.
  - Block list: 사용자가 금지 국가 목록에 있는 국가 중 하나에 있는 경우 사용자가 컨텐츠에 액세스하지 못하도록 할 수 있다.
- "국가"는 3rd Party Geo-IP 데이터베이스를 사용하여 결정된다.
- 저작권법에 의해 컨텐츠에 대한 접근이 통제되어야 하는 경우에 사용된다.
- 자동으로 요청에 `CloudFront-Viewer-Country`라는 헤더를 추가하기 때문에 오리진이나 Lambda@Edge 같은 함수에 접근이 가능하다.

#### Pricing

- CloudFront 엣지 로케이션은 전 세계에 분포되어 있다.
- 엣지 로케이션에서 데이터가 외부로 나갈 때 로케이션에 따라 비용이 다르다.

![7-cloudfront-pricing.png](images%2F7-cloudfront-pricing.png)

- 비용 절감을 위헤 엣지 로케이션의 수를 줄일 수 있다.
- 세 가지 가격 등급이 있다.
  - Price Class All: 모든 리전을 사용하며 가장 성능이 높다.
  - Price Class 200: 대부분의 리전을 사용하지만 가장 비싼 지역은 제외된다.
  - Price Class 100: 가장 저렴한 지역만 해당된다.

![8-cloudfront-price-classes.png](images%2F8-cloudfront-price-classes.png)

![9-cloudfront-price-classes.png](images%2F9-cloudfront-price-classes.png)

#### Signed URL Diagram

- CloudFront의 컨텐츠에 대한 액세스를 제어하기 위해 만료가 포함된 서명된 URL이다.
- 서명된 URL은 신뢰할 수 있는 서명자로 CloudFront에 API 호출을 통해 생성된다.

![10-signed-url-diagram.png](images%2F10-signed-url-diagram.png)

- OAC를 통해 Amazon S3와 통합된 CloudFront가 있다.
- 애플리케이션 서버는 CloudFront에 있는 컨텐츠에 접근하고자 하지만 보호되어 있다.
- 클라이언트는 서명된 URL을 생성하기 위해서 애플리케이션 서버와 통신해야한다.
  - 애플리케이션 서버는 AWS SDK를 통해서 CloudFront로부터 서명된 URL을 응답받는다.
  - 응답받은 서명된 URL을 클라이언트로 전달한다.
- 클라이언트는 서명된 URL을 통해서 CloudFront에 접근할 수 있게 된다.

#### CloudFront Signed URL vs S3 Pre-Signed URL

- **CloudFront Signed URL**
  - 오리진에 상관없이 경로에 대한 액세스를 허용한다.
  - 계정 전체 키 쌍, 루트만 관리할 수 있다.
  - IP, 경로, 날짜, 만료별로 필터링이 가능하다.
  - 캐싱 기능 활용이 가능하다.

![11-cloudfront-signed-url.png](images%2F11-cloudfront-signed-url.png)

- **S3 Pre-Signed URL**
  - URL에 미리 서명한 사용자로 요청을 발행한다.
  - 서명 IAM 주체의 IAM 키를 사용한다.
  - 제한된 수명 시간이 있다.

![12-s3-presigned-url.png](images%2F12-s3-presigned-url.png)

#### Custom Error Pages

- 오리진에서 CloudFront로 HTTP 4xx 또는 5xx 상태 코드를 반환할 때 객체를 뷰어(예: html)로 반환할 수 있다.
- 오류 캐싱 최소 TTL을 사용하여 CloudFront가 사용자 지정 오류 페이지를 캐싱하는 기간을 지정할 수 있다.

![13-custom-error-pages.png](images%2F13-custom-error-pages.png)

- 클라이언트가 CloudFront에 요청을 하고 CloudFront는 오리진에 대한 요청을 전달한다.
- 오리진은 4xx 또는 5xx 오류 메시지를 응답한다.
- CloudFront는 발생한 오류를 클라이언트에 전달하는 대신에 사용자 지정 오류 페이지를 만들어 저장한다.
- S3 버킷에 저장되어 있는 오류 페이지를 사용자에게 전달한다.
  - 만약 캐싱되어 있다면 S3 버키승ㄹ 조회하지 않고 클라이언트에 오류 페이지를 전달한다.

#### Customization At The Edge

- 많은 현대의 애플리케이션은 엣지에서 어떤 형태의 로직을 실행한다.
- 엣지 함수
  - CloudFront 배포물에 작성 및 첨부하는 코드다.
  - 사용자에게 가깝게 실행하여 지연 시간을 최소화한다.
  - 캐시가 없으면 요청/응답만 변경하고 다시 사용자에게 보낸다.
  - CloudFront는 "CloudFront Functions"와 "Lambda@Edge" 두 가지 유형을 제공한다.
- 사용 사례
  - HTTP 요청 및 응답을 조작한다.
  - 애플리케이션에 도달하기 전에 요청 필터링을 구현한다.
  - 사용자 인증 및 권한을 부여한다.
  - 엣지에서 반환되는 HTTP 응답을 생성한다.
  - A/B 테스트를 진행한다.
  - 봇의 요청에 대해서 엣지에서 완화한다.
- 전 세계적으로 구축된 서버를 관리할 필요가 없다.

#### CloudFront Function & Lambda@Edge

![14-cloudfront-function-lambdaedge.png](images%2F14-cloudfront-function-lambdaedge.png)

- CloudFront의 전통적인 구조를 보면 클라이언트가 엣지 로케이션과 통신하고 리전 엣지 캐시와 통신하고 우리의 오리진과 통신한다.
- Lambda@Edge 함수는 지역 엣지 캐시 레벨에 배포된다.
- 반면 CloudFront Function은 엣지 로케이션에 배포된다.
- 이러한 이유로 함수들의 배포를 위해 두 개의 레이어가 있다.
  - CloudFront Function과 Lambda@Edge 함수 중 선택에 영향을 미치게 된다.

#### CloudFront Function

- Javascript로 작성되는 경량 함수다.
- 지연 시간에 민감한 대규모 CDN 사용자 지정의 경우에 사용된다.
- 0.001초 미만의 시작 시간이 소요되며 초당 수백만 개의 요청을 처리할 수 있다.
- 엣지 로케이션에서 실행된다.
- 프로세스 기반의 격리다.
- 뷰어 요청 및 응답을 변경하려면 아래의 단계를 따른다.
  - Viewer Request: CloudFront가 뷰어로부터 요청을 받은 후
  - Viewer Response: CloudFront가 뷰어에게 응답을 전달하기 전
- CloudFront의 기본 기능으로 전적으로 CloudFront 내에서 코드가 관리된다.

![15-cloudfront-function.png](images%2F15-cloudfront-function.png)

#### CloudFront Lambda@Edge

- Node.js 또는 Python으로 작성된 람다 함수다.
- 초당 1,000개의 요청을 처리할 수 있도록 확장될 수 있다.
- 가장 가까운 리전의 엣지 캐시에서 실행된다.
- VM 기반의 격리다.
- CloudFront 요청 및 응답 변경에 사용된다.
  - Viewer Request: CloudFront가 뷰어로부터 요청을 받은 후
  - Origin Request: CloudFront가 요청을 오리진으로 전달하기 전
  - Origin Response: CloudFront가 오리진에서 응답을 받은 후
  - Viewer Response: CloudFront가 뷰어에게 응답을 전달하기 전
- 하나의 AWS 리전(`us-east-1`)에서 함수를 작성한 다음 CloudFront가 해딩 위치에 복제한다.

![16-cloudfront-lambdaedge.png](images%2F16-cloudfront-lambdaedge.png)

- CloudFront Function과 Lambda@Edge는 함께 사용될 수 있다.

![17-cloudfront-function-lambdaedge.png](images%2F17-cloudfront-function-lambdaedge.png)

- 엣지 로케이션과 리전 엣지 캐시가 있다.
  - 엣지 로케이션에는 엣지 로케이션 캐시가 있고 리전 엣지에도 리전 엣지 캐시가 있다.
- CloudFront 함수는 뷰어 요청과 뷰어 응답 레벨에서 엣지 위치에서 실행된다.
- 반면 Lambda@Edge 함수는 오리진 요청과 오리진 응답 레벨에서 실행된다.

![18-lambdaedge-only.png](images%2F18-lambdaedge-only.png)

- Lambda@Edge만 사용하면 뷰어 요청/응답과 오리진 요청/응답 전부 처리한다.
- CloudFront 함수에서는 사용할 수 없는 Lambda@Edge의 일부 기능이 필요할 때 이러한 방식이 사용된다.
  - 실행 시간 연장, 네트워크 액세스 등..

- CloudFront 함수와 Lambda@Edge 함수의 차이는 아래의 표를 참고한다.

![19-cloudfront-function-lambdaedge.png](images%2F19-cloudfront-function-lambdaedge.png)

#### CloudFront 함수 & Lambda@Edge 사용 사례

- **CloudFront 함수**
  - 캐시 키 정규화
    - 요청 속성(헤더, 쿠키, 쿼리 문자열, URL)을 변환하여 최적의 캐시 키를 생성한다.
  - 헤더 조작
    - 요청 또는 응답에 HTTP 헤더를 삽입/수정/삭제할 수 있다.
  - URL 다시 쓰기 또는 리디렉션
  - 요청의 인증 및 인가
    - 요청을 허용/거부하기 위한 사용자 생성 토큰(예: JWT) 생성 및 검증을 수행한다.
- **Lambda@Edge**
  - 실행 시간이 길어진다. 
  - 조정 가능한 CPU 또는 메모리를 가지고 있다.
  - 코드의 3rd 라이브러리(예: 다른 AWS 서비스에 액세스하기 위한 AWS SDK)에 따라 달라진다.
  - 처리를 위해 외부 서비스를 사용하기 위한 네트워크 액세스가 가능하다.
  - 파일 시스템 액세스 또는 HTTP 요청 본문에 대한 액세스가 가능하다.

#### CloudFront 함수 & Lambda@Edge 인증 및 인가

- **CloudFront 함수**

![20-cloudfront-function-authentication-authorization.png](images%2F20-cloudfront-function-authentication-authorization.png)

- 엣지 로케이션에서 컨텐츠를 가로채고 헤더에 있는 JWT와 같은 토큰을 확인한다.
- 만약 오류가 나면 클라이언트가 오류 메시지를 직접 받게 되고 요청은 오리진으로 전달되지 않는다.

- **Lambda@Edge**

![21-lambdaedge-authentication-authorization.png](images%2F21-lambdaedge-authentication-authorization.png)

- Lambda@Edge는 리전 엣지 캐시 레벨에서 요청을 가로챈다.
- 타사 API를 호출하여 인증과 인가가 적절하게 설정되었는지 확인할 수 있다.

#### Lambda@Edge User-Agent 헤더를 사용한 컨텐츠 로딩

![22-loading-content-based-on-user-agent.png](images%2F22-loading-content-based-on-user-agent.png)

- Lambda@Edge는 User-Agent 헤더를 기반으로 컨텐츠를 불러올 수 있다.
- 데스크탑 또는 모바일에서 CloudFront로 접속하는 사용자가 있다.
  - 두 사용자 모두 고양이 사진 파일인 `cat-1920-1080.jpg`를 요청하고 있다.
  - 데스크탑의 요청인 경우 화면의 크기에 맞게 1920x1080 사이즈의 이미지를 반환한다.
  - 모바일의 요청인 경우 큰 크기의 이미지가 필요없기 때문에 620x320 사이즈의 이미지를 반환한다. 
- Lambda@Edge 함수가 디바이스의 종류에 따라서 요청을 리디렉션한다.

#### Lambda@Edge Global Application

![23-lambdaedge-global-application.png](images%2F23-lambdaedge-global-application.png)

- Lambda@Edge는 글로벌 애플리케이션을 위해서 사용될 수 있다.
- S3 버킷에서 웹 사이트를 받으면 동적 API를 요청을 위해 CloudFront로 보내 캐시를 활용한다.
- 하지만 Lambda@Edge는 DynamoDB 테이블에서 데이터를 쿼리하는 데에도 사용될 수 있다.
- AWS에서 API를 사용할 수 있는 완전히 동적이고 글로벌하며 서버리스 방법을 제공한다.

#### Route to Different Origin

![24-route-to-different-origin.png](images%2F24-route-to-different-origin.png)

- CloudFront 엣지로부터의 일부 호출의 지연 시간을 줄일 수 있다.
- S3 버킷을 통해서 웹 사이트를 호스팅하고 있으며 CloudFront 엣지 로케이션도 생성되어 있다.
- 만약 엣지 로케이션에서 S3 버킷까지의 거리가 멀다면 요청은 시간이 오래 걸릴 수 있다.
- 만약 유럽에 있는 사용자가 우리의 엣지 로케이션에 액세스하길 원하는 경우 콘텐츠가 이미 캐싱되어 있다면 문제가 되지 않는다.
  - 하지만 객체를 캐시로 옮기기 위해서는 유럽에 있는 엣지 로케이션에서 `us-east-1` 리전으로 요청을 하기 때문에 지연 시간이 늘어날 수 있다.
  - 문제를 해결하기 위해 `eu-west-2` 리전에 버킷을 생성하고 두 버킷이 서로 복제되도록 설정한다.
- Lambda@Edge 함수를 생성한다.
  - 생성된 함수는 CloudFront 엣지 로케이션에서 오리진에 액세스할 때마다 요청이 어디서 발생한 요청인지 확인하고 가까운 위치의 오리진을 확인하여 요청을 전달한다.

---

### Amazon ElastiCache

- RDS가 관리되는 관계형 데이터베이스를 얻는 것과 동일한 방식으로 관리되는 캐시를 얻을 수 있다.
- ElastiCache는 관리형 Redis 또는 관리형 Memcached다.
- 캐시는 매우 높은 성능과 낮은 지연 시간을 가진 인메모리 데이터베이스다.
- 읽기 집약적인 워크로드를 위해 데이터베이스의 부하를 줄이는 데 도움이 된다.
- 애플리케이션을 Stateless 상태로 만드는 데 도움이 된다.
- AWS에서 OS 관리/패치, 최적화, 설정, 구성, 모니터링, 장애 복구 및 백업 관리를 제공한다.
- ElastiCache를 사용하면 애플리케이션 코드가 크게 변경되어야 한다.

#### DB Cache

- 애플리케이션은 ElastiCache를 사용할 수 없는 경우 RDS에서 가져와 ElastiCache에 저장하도록 쿼리한다.
- RDS의 부하를 완화하는 데 도움이 된다.
- 캐시에 최신 데이터만 사용되도록 하려면 무효화 전략이 있어야 한다.

![25-elasticache-db-cache.png](images%2F25-elasticache-db-cache.png)

#### User Session Store

- 사용자가 애플리케이션에 로그인한다.
- 애플리케이션이 세션 데이터를 ElastiCache에 기록한다.
- 사용자가 우리 애플리케이션 중 다른 인스턴스에 요청을 전달한다.
- 인스턴스가 세션 저장소에서 데이터를 검색하고 사용자가 이미 로그인되어 있다고 판단한다.

![26-elasticache-user-session-store.png](images%2F26-elasticache-user-session-store.png)

#### Redis vs Memcached

- **Redis**
  - Multi AZ와 함께 자동 Failover를 제공한다.
  - 복제본 읽기를 통해 읽기를 확장하고 가용성이 높다.
  - 지속적인 데이터 내구성을 제공하며 AOF (Append Only File), 백업 및 복원 기능을 제공한다.
- **Memcached**
  - 데이터 파티셔닝을 위한 다중 노드(sharding)를 제공한다.
  - 데이터가 비영구적이다.
  - 백업 및 복원 기능이 없다.
  - 멀티 스레드 아키텍처를 제공한다.

---

### Handling Extreme Rates

![27-handling-extreme-rates.png](images%2F27-handling-extreme-rates.png)

- 엄청나게 많은 양의 요청을 처리해야 하는 상황을 가정한다.
- 글로벌 DNS인 Route53을 사용하고 있으며 매우 높은 비율의 요청을 처리하도록 만들어져 있다.
- CloudFront의 경우 캐싱 기능을 제공하기 때문에 초당 10만건의 요청을 쉽게 처리할 수 있다.
  - CloudFront는 오리진에 대한 요청을 감소시킬 수 있다.
- ALB는 유연하게 확장될 수 있지만 워밍업에 문제가 발생할 수 있다.
- API 게이트웨이는 초당 10,000개의 요청을 처리할 수 있는 소프트 제한이 설정되어 있다.
  - 단, API 게이트웨이의 경우 캐싱 기능이 제공된다.
- 만약 ASG, ECS를 사용한다면 VM이 생성된 후 인스턴스를 부트스트랩해야 하기 때문에 확장하는데 시간이 많이 소요된다.
- Fargate의 경우 컨테이너가 부팅되기 때문에 빠르게 스케일링될 수 있다.
- 람다 함수의 경우 동시 처리 1,000이라는 소프트 제한이 설정되어 있다.
- 데이터베이스 레이어에서는 RDS, Aurora, ElastiSearch등을 사용할 수 있다.
  - 이것들은 데이터베이스를 프로비저닝 해야 한다.
  - DynamoDB를 사용한다면 Auto-Scaling이나 On-Demand Scaling을 사용할 수 있다.
- 캐싱 레이어에서 Redis를 사용하는 경우 최대 200개의 노드까지 확장할 수 있다.
  - Redis의 200개의 노드에는 복제본과 샤딩이 포함되어 있다.
  - Memcached의 경우 20개의 노드까지 확장될 수 있지만 복제본이 아니라 샤딩된 데이터다.
  - DAX의 경우 최대 10개까지의 노드를 생성할 수 있다.
- 만약 디스크에 데이터를 저장해야 한다면 EBS를 사용할 수 있다.
  - gp2는 최대 16,000IOPS를 제공하며 io1은 최대 64,000IOPS를 제공한다.
  - 인스턴스 스토어를 사용한다면 수백만 IOPS까지 지원할 수 있다.
  - EFS를 사용해야 한다면 General, MaxIO 모드 중 원하는 모드를 생성해서 사용하면 된다.
- 애플리케이션을 분리해야 할 경우 SNS, SQS를 사용할 수 있으며 사실상 무한한 크기로 확장된다.
  - SQS FIFO는 배치 작업 시 초당 3,000개 또는 배치 작업없이 초당 300개의 요청을 처리할 수 있다.
- S3의 경우 PUT은 3,500, GET은 5,500개의 요청을 처리할 수 있다.
  - 하지만 SSE-KMS 암호화 방식을 사용한다면 KMS 암호화에서 제한이 발생할 수 있다.

---

### 참고한 강의

- [Solutions Architect Professional](https://www.udemy.com/course/aws-solutions-architect-professional)
- [Solutions Architect Associate](https://www.udemy.com/course/best-aws-certified-solutions-architect-associate)
- [SysOps Administrator Associate](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate)