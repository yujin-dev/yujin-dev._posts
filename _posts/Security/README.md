# Google Cloud authentication

## Google Cloud SSO
**SSO**는 한 번의 시스템 인증으로 여러 시스템에 재인증 절차 없이 접근할 수 있는 통합 인증 솔루션이다.( 단일 인증 ) 

SSO를 사용하도록 구글 클라우드에서 Cloud ID나 Google Workspace 계정을 구성할 수 있다.   
SSO를 사용하면 사용자가 엑세스하려고 할 때 비밀번호를 입력하라는 메시지가 표시되지 않는 대신 외부 ID 공급업체(IdP)로 리디렉션된다.
- 기존 IdP가 사용자 인증을 위한 레코드 시스템으로 유지된다.
- 비밀번호를 Cloud ID 또는 Google Workspace에 동기화할 필요가 없다.

### 인증 프로세스
![](https://cloud.google.com/architecture/identity/images/console-access-with-sso.svg?hl=ko)  
![](https://cloud.google.com/architecture/identity/images/saml-exchange-using-sso.svg?hl=ko)   

SSO에 대해 SAML 2.0을 지원한다. SAML(Security Assertion Markup Language)은 SAML IdP와 SAML 서비스 제공업체 간에 인증 및 승인 데이터를 교환하기 위한 개방형 표준이다. 
1. 콘솔이나 인증이 필요한 다른 google 리소스를 가리킨다.
2. 인증 전이므로 google 로그인으로 리디렉션한다.
3. 이메일 주소를 입력하라는 로그인 페이지가 표시된다.
4. 이메일 주소를 입력하여 정보를 제출한다.
5. google 로그인은 내부적으로 이메일 주소와 연결된 Cloud ID 또는 Google workspace 계정을 조회한다.
6. SSO 사용 설정이 되어 있으면 Google 로그인은 외부 IdP의 URL로 리디렉션한다. 
7. 외부 IdP는 ACS URL로 HTTP-POST 요청을 위한 HTML 페이지를 반환한다. 
8. SAML assertion을 Google ACS 엔드포인트에 게시한다.
9. ACS 엔드포인트가 SAML assertion의 디지털 서명을 확인한다.
10. ACS 엔드포인트는 SAML assertion의 `NameID`를 사용자의 이메일 주소와 일치시켜 사용자 계정을 찾는다.
11. 엔드포인트가 원래 액세스하려고 했던 리소스의 URL을 결정하면 사용자가 콘솔로 리디렉션된다.

## BigQuery 인증으로 보는 Google Cloud 인증
BigQuery API를 사용하려면 인증을 거쳐 클라이언트 ID를 확인해야 한다. BigQuery는 ID를 기반으로 리소스에 대한 액세스 권한을 승인한다.

### 서비스 계정
Google Cloud project와 연결된 Google 계정이다. 
### OAuth2.0 인증
OAuth는 access token을 기반으로 한다.
### 사용자 계정
사용자 인증 정보를 통해 사용자가 접근 가능한 BigQuery 테이블만 액세스하도록 한다. application project가 아닌 cloud project에 대해서만 쿼리를 실행한다.
이에 따라 application이 아닌 쿼리에 대한 요금이 청구된다.

### API 요청 승인
클라이언트에서 리로스에 액세스할 수 있도록 토큰을 BigQuery API에 전달한다.


# KeyCloak
국제적인 인증, 인가 표(SAML, OAuth 2.0 등)을 모두 제공하는 오픈소스로 Kubernetes, MSA 환경에 최적화된 솔루션이다.
- SSO
- ID 중개 및 소셜 로그인
- 관리자/계정관리 콘솔
- 표준 프로토콜( OpenID Connect + OAuth 2.0, SAML ) 지원
- 기본적으로 Java 기반의 H2 DB를 제공한다. 
- 세분화된 권한 부여 정책 및 다양한 액세스 제어 매커니즘으로 리소스 보호
	- Attribute-Based-Access-Contorl
	- Role-Based-Access-Control
	- User-Based-Access-Control
	- Context-Based-Access-Control
	- Rule-Based-Access-Control
	- Time-Based-Access-Control
- 중앙 집중형 정책 결정 
- REST 기반
- 권한 부여 워크플로 및 사용자 관리 액세스

리소스는 keycloak 관리 콘솔이나 protection API를 통해 관리 가능하다. 

### 인증 서비스
권한 부여 서비스는 RESTfull 엔드포인트로 구성된다.
- 토큰 엔드포인트 : 서버에서 액세스 토큰을 얻어 리소스에 액세스할 수 있다. 
- 리소스 관리 엔드포인트
- 권한 관리 엔드포인트 

### OpenID Connect(OIDC) vs. OAuth 2.0
keycloak은 인증 방식이 OAuth 2.0을 기반으로 한 OIDC이다.
- OIDC : Oauth 2.0의 확장 인증 프로토콜로, 인증에 초점을 맞춘다.
- OAuth 2.0 : 데이터에 대한 액세스 권한 부여에 초점을 맞춘다.

### Google Cloud SSO
SAML 제휴를 통해 KeyCloak과 Cloud ID/Google workspace 계정 간에 SSO를 설정할 수 있다.

다음과 같이 KeyCloak 서버를 구성한다.
- Client ID : `google.com`
	- Name : `Google Cloud`
	- Sign Assertions : 사용
    ..
- Client Protocol : SAML
- Client SAML endpoint : -

## architecture
keycloak은 모든 메타 데이터, 구성을 관리할 수 있는 네임스페이스와 같다.   
![](https://developers.redhat.com/sites/default/files/styles/article_floated/public/blog/2019/11/keycloak1.png?itok=dlMycurG)



### 프로세스
![](https://sp-ao.shortpixel.ai/client/to_auto,q_lossless,ret_img,w_765,h_484/https://www.comakeit.com/wp-content/uploads/keycloak-1.jpg)
> 출처  
https://alice-secreta.tistory.com/28  
https://www.keycloak.org/docs/latest/authorization_services/#_overview_architecture  
https://www.comakeit.com/blog/quick-guide-using-keycloak-identity-access-management/  
https://developers.redhat.com/blog/2019/12/11/keycloak-core-concepts-of-open-source-identity-and-access-management#wrapping_up

# IAM
IAM(ID 및 Access 관리)는 사용자가 누구인지, 사용자가 어떤 권한을 갖는지 알려준다. 
IAM은 사용자의 ID와 ID별 연계된 권한을 관리하는 수단이다. 

**ID(Identity)**  
컴퓨터 시스템에서 ID는 사용자 고유의 특징을 평가하는 요소이며 일종의 신분증이다.   
보통 아래와 같은 인증 요소가 있다.
- 사용자가 아는 것 : ex. 비밀번호
- 사용자가 갖고 있는 것 : 토큰 ex. 일회용 인증코드
- 사용자의 속성 : ex. 안면인식, 지문스캔

**Access**   
액세스는 사용자가 로그인하여 접근할 수 있는 데이터와 실행 가능한 작업이다.

**ID Provider(IdP)**  
ID provider(IdP)는 ID관리에 도움을 준다. IdP는 실제 로그인 프로세스를 처리하는 경우가 많으며 SSO provider도 포함된다.

**IDaaS(IDentity-as-a-Service)**  
IDaaS는 ID 관리를 위한 클라우드 서비스이다.

> 출처  
https://www.cloudflare.com/ko-kr/learning/access-management/what-is-identity-and-access-management/  


# Token
### stateless
statueful의 경우 클라이언트로부터 요청을 받을 때 마다 상태를 계속해서 유지하고 정보를 제공한다.
stateless의 경우 상태를 유지하지 않는다. 서버는 클라이언트에서 받은 요청만으로만 작업을 처리한다. 

기존을 세션을 이용한 인증은 statuful 방식으로 세션 정보를 계속 저장하고 있어야 하나 토큰을 사용하면 stateless 방식으로 가능하다.

인증 과정은 대략적으로 아래와 같다.
1. ID, 비밀번호로 로그인한다.
2. 서버에서 계정정보를 검증한다.
3. 계정정보가 유효하면 유저에게 *signed* 토큰을 발급한다( 정상적으로 발급된 토큰임을 증명하는 서명을 갖고 있다는 뜻).
4. 클라이언트에서 토큰을 저장해두고, 서버에 요청을 할 때마다 토큰과 함께 전달한다.
5. 서버에서 토큰을 검증하고 응답한다. 

> 출처  
https://velopert.com/2350


# AWS Cognito
AWS Cognito를 통해 웹, 모바일 앱에서 손쉽게 사용자 가입, 로그인 및 액세스 제어가 가능하다.
수백만 사용자로 확장할 수 있고 Google, Facebook 같은 소셜 자격 증명 공급자와 SAML 2.0, OpenID Connect 같은 엔터프라이즈 자격 증명 공급자를 통한 로그인도 지원한다.  
managed service로 수백만 사용자로 확장 가능한 자격 증명을 제공하는 이점이 있다.

## User Pool & Identity Pool
![](https://docs.aws.amazon.com/ko_kr/cognito/latest/developerguide/images/scenario-cup-cib2.png)

1. user pool을 통해 로그인하여 인증 성공 후 토큰을 받는다.
2. identity pool을 통해 user pool 토큰을 AWS credential과 교환한다.
3. credential을 기반으로 AWS 서비스에 접근한다.

### User Pool
**User Pool**은 사용자 디렉토리이다. user pool의 사용자는 AWS Cognito나 IdP를 통해 페더레이션하여 웹이나 모바일 앱에 로그인한다.   
가입이나 로그인 관련 서비스를 여러 방식으로 제공하고 MFA, 이상 있는 credential 확인 등 보안 기능을 제공한다. AWS Lambda 트리거를 통해 사용자 지정 워크플로우나 사용자 migration도 가능하다. 

- 로그인 : 사용자는 User Pool을 통해 직접 로그인하거나 타사 자격 증명 공급자(IdP)를 통해 연동 로그인할 수 있다.
- 인증( 토큰 처리 ): Facebook, Google을 통한 소셜 로그인에서 받은 토큰과 OpenID Connect(OIDC) 및 SAML IdP에서 받은 토큰을 처리한다. 토큰을 받아 인증을 성공하면, 
	- 자체 서버 리소스나 Amazon API Gateway에 대한 액세스 권한을 부여할 수 있다.
	- AWS 자격 증명으로 토큰을 교환한다.
- 클라이언트 : 클라이언트 측 User Pool 토큰 처리는 Cognito SDK에서 제공한다. 


### Identity Pool
사용자는 **Identity Pool**을 통해 임시 AWS 자격 증명을 얻어 여러 AWS 서비스에 액세스할 수 있다. 
사용자의 고유한 credential을 만들고 자격 증명 공급자와 페더레이션할 수 있다. 

>출처  
https://docs.aws.amazon.com/ko_kr/cognito/latest/developerguide/what-is-amazon-cognito.html  


