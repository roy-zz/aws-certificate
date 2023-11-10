# Amazon CloudFront

이번 장에서는 **SysOps Administrator**를 준비하며 **CloudFront 서비스**에 대해서 알아보도록 한다.

---

### Amazon CloudFront

- Content Delivery Network (CDN)
- **읽기 성능이 향상되고 콘텐츠가 엣지에서 캐싱**된다.
- 사용자 경험을 개선할 수 있다.
- 전 세계적으로 216개의 PoP(Point of Presence, Edge Location)를 가지고 있다.
- **DDoS 보호(전 세계적), Shield, AWS Web Application Filewall(WAF)과 통합**된다.

![1-cloudfront.png](images%2F1-cloudfront.png)

- 호주의 S3 버킷에서 웹 사이트를 제공하고 있다.
- 미국의 사용자가 해당 웹사이트를 요청한다면 CloudFront를 이용해서 미국에 있는 Edge Location에서 콘텐츠를 요청하면 호주에서 콘텐츠를 가져올 수 있다.
- 중국의 사용자가 요청한 경우에도 CloudFront를 사용하여 중국에 있는 Edge Location에서 콘텐츠를 저장하고 중국 사용자에게 제공할 수 있다.

#### Origin

- **S3 Bucket**
  - 파일 배포 및 엣지 캐싱용으로 사용된다.
  - CloudFront OAC(Origin Access Control)을 사용하여 보안을 강화할 수 있따.
  - OAC가 OAI(Origin Access Identity)를 대체한다.
  - CloudFront를 수신(S3에 파일 업로드)으로 사용할 수 있다.
- **Custom Origin (HTTP)**
  - Application Load Balancer
  - EC2 Instance
  - S3 Website (버킷을 정적 S3 웹사이트로 활성화 필요)
  - 모든 HTTP 백엔드

![2-cloudfront-high-level.png](images%2F2-cloudfront-high-level.png)

- Origin과 연결되는 CloudFront 엣지 로케이션이 있다.
- 사용자가 엣지 로케이션에 HTTP 요청을 하면 엣지 로케이션은 캐시 안에 데이터가 있는지 확인한다.
- 캐시에 데이터가 없다면 Origin에서 데이터를 가져와서 로컬 캐시에 저장한다.
- 다른 사용자가 동일한 요청을 하는 경우 Origin까지 갈 필요없이 캐시에 있는 데이터를 반환할 수 있다.

#### S3 Origin

![3-cloudfront-s3-origin.png](images%2F3-cloudfront-s3-origin.png)

- S3가 Origin인 경우 전 세계의 엣지 로케이션으로 Private 네트워크를 통해 데이터를 전송한다.
- OAC(Origin Access Control)과 S3 Bucket Policy를 통해서 Origin에 접근할 수 있는 엣지 로케이션을 제한할 수 있다.

#### CloudFront vs S3 Cross Region Replication

- CloudFront
  - 글로벌 엣지 네트워크
  - 파일은 TTL 동안 캐시된다.
  - 어디에서나 사용할 수 있어야 하는 정적 콘텐츠에 적합하다.
- S3 Cross Region Replication
  - 복제를 수행하려는 각 지역에 대해 설정해야 한다.
  - 파일은 거의 실시간으로 업데이트된다.
  - 읽기 전용이다.
  - 몇몇 지역에서 짧은 지연 시간으로 제공되어야 하는 동적 콘텐츠에 적합하다.

---

### ALB & EC2 Origin

![4-alb-ec2-origin.png](images%2F4-alb-ec2-origin.png)

- EC2 인스턴스에 백엔드 서버를 운영하고 CloudFront를 사용하여 사용자가 액세스할 수 있다.
- 이러한 경우 Private VPC 연결이 없기 때문에 EC2 인스턴스는 Public으로 설정되어 있어야 한다.
- CloudFront 요청을 허용하도록 모든 엣지 로케이션의 Public IP를 허용하는 보안 그룹도 필요하다.
- Load Balancer를 사용하는 경우 인스턴스는 Private으로 설정할 수 있다.
- Load Balancer는 Public으로 설정되어 있어야 하고, 인스턴스의 보안 그룹은 Load Balancer의 요청을 허용할 수 있도록 설정한다.
- Load Balancer는 모든 엣지 로케이션의 Public IP를 허용하는 보안 그룹을 가지고 있어야 한다.

---

### Geo Restriction

- 배포에 액세스할 수 있는 사용자를 제한할 수 있다.
  - 허용 목록: 사용자가 승인된 국가 목록에 있는 국가 중 하나에 있는 경우에만 콘텐츠에 액세스할 수 있도록 허용한다.
  - 차단 목록: 사용자가 금지 국가 목록에 있는 국가 중 하나에 있는 경우 사용자가 콘텐츠에 액세스하지 못하도록 차단한다.
- 접속 국가를 결정하는 것은 3rd Party Geo-IP 데이터베이스다.
- 콘텐츠에 대한 액세스를 제어하는 저작권법 등에 사용된다.

---

### Access Logs

- CloudFront에 대한 모든 요청을 로깅 S3 버킷에 기록한다.

![5-access-logs.png](images%2F5-access-logs.png)

- 사용자가 CloudFront에 요청을 하는 경우 로그 데이터는 중앙 S3 버킷에 저장되낟.
- 버킷 안에 접두사를 추가해 엣지 로케이션 별로 저장되는 경로를 분리할 수 있다.

#### Report

- 아래의 항목에 대한 보고서 생성이 가능하다.
  - 캐시 통계 보고서
  - 인기 객체 보고서
  - 상위 추천자 보고서
  - 사용 보고서
  - 시청자 보고서
- 이러한 리포트를 만들기 위해 CloudFront는 액세스 로그로부터의 데이터를 사용한다.
- 하지만 이런 리포트가 생성되는 동안 액세스 로그를 S3로 보낼 필요는 없다.

#### Troubleshooting

- CloudFront는 S3나 오리진 서버에서 반환되는 HTTP 4xx 및 5xx 상태 코드를 캐시한다.
- 4xx 오류 코드는 사용자가 기본 버킷에 액세스할 수 없거나(403) 사용자가 요청한 객체를 찾을 수 없음(404)을 나타낸다.
- 5xx 오류 코드는 게이트웨이 문제를 나타낸다.

---

### Caching

![6-caching.png](images%2F6-caching.png)

- 캐시 기반
  - 헤더
  - 세션 쿠키
  - 쿼리 문자열 파라미터
- 캐시는 각 CloudFront 엣지 로케이션에 있다.
- 캐시 적중률을 최대화하여 오리진에 대한 요청을 최소화해야 한다.
- TTL(0초 ~ 1년)을 제어하고 `Cache-Control` 헤더, `Expires` 헤더를 사용하여 오리진에서 설정할 수 있다.
- `CreateInvalidation` API를 사용하여 캐시의 일부를 무효화할 수 있다.

![7-cache-behaviour-header.png](images%2F7-cache-behaviour-header.png)

- CloudFront를 세 가지 방법으로 구성할 수 있다.

1. 모든 헤더를 오리진으로 전달
   - 캐싱을 사용하지 않으며 모든 요청이 오리진으로 전달된다.
   - TTL을 0으로 설정해야 한다.
2. 헤더 허용 목록 전달 
   - 지정된 모든 헤더의 값을 기반으로 캐싱
3. (None) == 기본값만 전달
   - 요청 헤더 기반 캐싱이 없다.
   - 최고의 캐싱 성능을 제공한다.

#### Whitelist Headers

![8-whitelist-headers.png](images%2F8-whitelist-headers.png)

- 클라이언트의 많은 헤더들은 엣지 로케이션으로 전달된다.
- 엣지 로케이션을 통한 요청의 헤더는 값이 줄어들기 때문에 캐싱할 값이 적어져서 캐싱이 더 용이해진다.

#### Origin Header vs Cache Behavior

- Origin Custom Header
  - 오리진 레벨에서 설정한다.
  - 오리진에 대한 모든 요청에 대해 일정한 헤더/헤더 값을 설정한다.
  - 예를 들어, CloudFront에서 요청이 전달될 때, 어떤 엣지 로케이션에서 전달된 요청인지 값을 정의할 수 있다.

![9-origin-custom-header.png](images%2F9-origin-custom-header.png)

- Behavior Setting
  - 캐시 관련 설정을 한다.
  - 전달할 헤더의 화이트리스트가 포함되어 있다.
  - 오리진 커스텀 헤더는 캐싱에 사용되지 않고 오리진으로 헤더를 전달하는 데만 사용된다.

![10-cache-behavior.png](images%2F10-cache-behavior.png)

#### TTL

- `Cache-Control: max-age`는 `Expires` 헤더보다 선호된다.
- 오리진이 항상 `Cache-Control` 헤더를 다시 보내는 경우 TTL이 해당 헤더에 의해서만 제어되도록 설정할 수 있다.
- 최소/최대 경게를 설정하려는 경우 객체 캐싱 설정에 대해 "사용자 정의"를 선택한다.

![11-ttl-1.png](images%2F11-ttl-1.png)

- `Cache-Control` 헤더가 누락된 경우 기본값을 사용한다.

![12-ttl-2.png](images%2F12-ttl-2.png)

#### Cache Behaviour (Cookie)

![13-cache-behaviour-for-cookie.png](images%2F13-cache-behaviour-for-cookie.png)

- 쿠키는 특정 요청의 헤더이며, 세 가지로 설정할 수 있다.
1. 기본값은 쿠키를 처리하지 않는다.
   - 캐싱은 쿠키를 기반으로 하지 않는다.
   - 쿠키는 전달되지 않는다.
2. 쿠키 화이트리스트 전달
   - 지정된 모든 쿠키의 값을 기반으로 캐싱한다.
3. 모든 쿠키 전달
   - 최악의 캐싱 성능이지만 앱이 모든 쿠키를 사용할 수 있게 해준다.
- **앱에서 쿠키를 어떻게 사용하는지가 중요**하다.

![14-caching-cookie.png](images%2F14-caching-cookie.png)

#### Cache Behaviour (Query String)

![15-cache-behaviour-for-query-string.png](images%2F15-cache-behaviour-for-query-string.png)

- 쿼리 문자열 파라미터는 URL에 있으며 세 가지 옵션이 있다.
1. 기본적으로 쿼리 문자열을 처리하지 않는다.
   - 캐싱은 쿼리 문자열을 기반으로 하지 않는다.
   - 파라미터는 전달되지 않는다.
2. 쿼리 문자열 화이트리스트 전달
   - 파라미터 화이트리스트 기반으로 캐싱한다.
3. 모든 쿼리 문자열을 전달한다.
   - 모든 파라미터를 기반으로 한 캐싱이기 때문에 최악의 성능을 제공한다.

![16-caching-query-string-parameter.png](images%2F16-caching-query-string-parameter.png)

#### 캐시 히트 최대화

![17-maximize-cache-hits.png](images%2F17-maximize-cache-hits.png)

- CloudWatch 지표 중에 `CacheHitRate`를 모니터링한다.
- `Cache-Control max-age`헤더를 통해서 객체를 캐시할 기간을 지정한다.
- 아무것도 지정하지 않거나 최소한으로 필요한 헤더를 지정한다.
- 쿠키를 전혀 지정하지 않거나 최소한으로 필요한 쿠키를 지정한다.
- 아무것도 지정하지 않거나 최소한으로 필요한 쿼리 문자열 파라미터를 지정한다.
- 별도의 정적 및 동적 배포(두개의 오리진)를 구성한다.

---

### ALB Sticky Session

- 세션 선호도가 작동할 수 있도록 세션 선호도를 제어하는 쿠키를 오리진으로 전달/화이트리스트에 추가해야 한다.
- 아래의 예시를 보면 `AWSALB` 쿠키를 화이트리스트에 추가하여 동일한 사용자는 동일한 인스턴스로 요청이 전달되도록 하고 있다.
- TTL을 인증 쿠키가 만료되는 시점보다 작은 값으로 설정해야 한다.

![18-alb-sticky-session.png](images%2F18-alb-sticky-session.png)

---

- [참고한 강의](https://www.udemy.com/course/ultimate-aws-certified-sysops-administrator-associate/)
- [CloudFront Source](https://aws.amazon.com/cloudfront/features/?nc=sn&loc=2)
- [CloudFront Public IPs](http://d7uri8nf7uskq.cloudfront.net/tools/list-cloudfront-ips)