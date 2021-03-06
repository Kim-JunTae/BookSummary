# Chapter9 HTTP 헤더

# 01 HTTP 헤더의 중요성

### 헤더

- 메시지와 바디에 대한 부가적인 정보(메타 데이터)를 표현합니다.
- 클라이언트와 서버는 헤더를 보고 메시지에 대한 동작을 결정
- 헤더의 기능
    - 리소스에 대한 접근권한을 설정하는 인증
    - 클라이언트와 서버의 통신횟수와 양을 감소시키는 캐시 같은 HTTP의 기능

# 02 HTTP 헤더의 탄생

- HTTP 0.9에서는 헤더가 X 
→ 이후 스펙 책정을 하면서 **전자메일**의 메시지 스펙 헤더형식을 빌려오는 식으로 추가 
→ 공통되는 부분이 있음
    - Content-Type: Text/Plain; charset=us-ascii
    - Content-Length: 14
    - Date: Tue, 22 Jul 2010 09:40:53 +0900(JST)
- 전자메일과의 차이점
    - 메일 프로토콜은 한 방향으로밖에 메시지를 주고 받지 않음
    - HTTP는 한 번의 통신으로 요청/응답의 두 가지 메시지를 주고받는다는 점

# 03 날짜와 시간

- Date와 Expires

![Chapter9%20HTTP%20%E1%84%92%E1%85%A6%E1%84%83%E1%85%A5%20e84be7d831b64d988621cbfdbeeda6f7/Untitled.png](Chapter9%20HTTP%20%E1%84%92%E1%85%A6%E1%84%83%E1%85%A5%20e84be7d831b64d988621cbfdbeeda6f7/Untitled.png)

# 04 MIME 미디어 타입

### MIME(Multipurpose Internet Mail Extensions)

메시지로 주고받는 리소스 표현의 종류를 지정하는 것

HTTP에서는 Content-Type 헤더 등 몇 개만을 이용(원래는 복수의 메일헤더를 정의하는 스펙)

### Content-Type - 미디어 타입을 지정한다.

메시지의 바디 내용이 어떠한 종류인가를 미디어 타입으로 나타냅니다.

XHTML(Extensible Hypertext Markup Language)

ex) Content-Type: **application/xhtml+xml**; charset=utf-8
                                   **타입         서브타입**

![Chapter9%20HTTP%20%E1%84%92%E1%85%A6%E1%84%83%E1%85%A5%20e84be7d831b64d988621cbfdbeeda6f7/Untitled%201.png](Chapter9%20HTTP%20%E1%84%92%E1%85%A6%E1%84%83%E1%85%A5%20e84be7d831b64d988621cbfdbeeda6f7/Untitled%201.png)

![Chapter9%20HTTP%20%E1%84%92%E1%85%A6%E1%84%83%E1%85%A5%20e84be7d831b64d988621cbfdbeeda6f7/Untitled%202.png](Chapter9%20HTTP%20%E1%84%92%E1%85%A6%E1%84%83%E1%85%A5%20e84be7d831b64d988621cbfdbeeda6f7/Untitled%202.png)

- 타입의 종류는 임의로 늘릴 수 없음
- 서브타입은 비교적 자유롭게 늘릴 수 있음

### charset 파라미터 - 문자 인코딩을 지정한다

- 생략 가능하지만 타입이 text인 경우에는 주의(디폴트 문자 인코딩 ISO 8859-1로 정의)

# 05 언어 태그

Content-Language 헤더의 값은 '언어 태그 Language Tag'

ex) Content-Language: ko-KR
                           언어코드-지역코드

# 06 콘텐트 네고시에이션

- 서버가 일방적으로 결정하는 것이 아닌 클라이언트와 교섭(네고시에이션)해서 정하는 방법
- 미디어 타입, 문자 인코딩, 언어 태그 등

### Accept - 처리할 수 있는 미디어 타입을 전달한다.

- Accept: text/html, application/xhtml+xml, application/xml; q=0.9, **/**;q=0.8
q= : qvalue, 그 미디어 타입의 우선 순위, 소수점 이하 세 자리 이내의 0~1까지의 수치

### Accept-Charset - 처리할 수 있는 문자 인코딩 전달하기

### Accept-Langauge - 처리할 수 있는 언어를 전달한다.

# 07 Content-Length와 청크(chunk) 전송

### Content-Length - 바디의 길이를 지정한다.

메시지가 바디를 가지고 있는 경우, 그 사이즈를 10진수의 바이트로 나타냄

사이즈를 알고 있는 리소스인 정적인 파일 등을 전송할 때 이용

### 청크 전송 - 바디를 분할하여 전송한다.

동적으로 이미지를 생성하는 웹 서비스의 경우, 파일 사이즈가 정해질 때까지 응답할 수 없기 때문에 응답 성능이 저하됨. 이 때 Transfer-Encoding 헤더 사용

- Transfer-Encoding: chunked

최종적으로는 사이즈를 모르는 바디를 조금씩 전송할 수 있다.

# 08 인증

**URI 공간**
URI에서 패스 이하를 가리키는 것
ex) [http://example.com/foo라는](http://example.com/foo라는) URI 공간에서
- [http://example.com/foo](http://example.com/foo라는)
- [http://example.com/foo](http://example.com/foo라는)/bar
- [http://example.com/foo](http://example.com/foo라는)/bar/baz
- [http://example.com/foo](http://example.com/foo라는)?q=a
모두 이 URI 공간에 속함
클라이언트는 동일한 URI 공간에 속한 리소스에는 같은 인증 정보를 송신할 수 있다고 가정하게 되어 있어 매번 401 Unauthorized가 반환되는 것을 피할 수 있습니다.

### Basic 인증

유저 이름과 패스워드에 의한 인증방식

```java
DELETE /test HTTP/1.1
Host: example.com
Authorization: Basc dXNlcjpwYXNzd29yZA==
```

유저 이름과 패스워드를 ':'으로 연결하고 Base64 인코딩으로 한 문자열이 됩니다.

보안 강도를 검토해야됨.(SSL, TLS을 사용하여 HTTPS 통신을 하고 통신선로 상에서 암호화할 것인지)

**HTTPS**
HTTP와 SSL/TLS를 조합한 통신, 통신로를 암호화하고 클라이언트와 서버 간에 주고받는 데이터를 보호하여, 도청을 방지할 목적으로 주로 사용합니다.
SSL/TLS에서는 3가지 기능을 제공합니다.
1. 암호화 : 공통키 암호에 기반한 암호화 기능
2. 인증 : 공개키 증명서에 기반한 인증 기능
3. 변경감지 : 해시 공통키에 기반한 변경 감지 기능
HTTPS에서 통신할 경우, URI는 https 스키마를 사용합니다. 기본 포트번호는 443입니다.

### Digest 인증

메시지 다이제스트, 메시지에 대해 해시함수를 적용한 해시값을 말함.

- 첼린지 : WWW-Authenticate 헤더의 값을 첼린지라 부릅니다.

![Chapter9%20HTTP%20%E1%84%92%E1%85%A6%E1%84%83%E1%85%A5%20e84be7d831b64d988621cbfdbeeda6f7/Untitled%203.png](Chapter9%20HTTP%20%E1%84%92%E1%85%A6%E1%84%83%E1%85%A5%20e84be7d831b64d988621cbfdbeeda6f7/Untitled%203.png)

- nonce : number used once, 모든 요청에 대해 변화하는 문자열
- qop : quality of protection, 클라이언트가 송신할 다이제스트의 작성방법에 영향을 미침
- opaque : 클라이언트에는 불투명한 문자열, 동일 URI 공간에 대한 요청에는 공통되게 클라이언트에서 서버로 보냄
- 다이제스트의 생성과 송신 : 서버로부터 인증에 필요한 정보를 얻은 클라이언트는 다이제스트를 생성
    1. 유저이름, realm, 패스워드는 ':'로 연결하고, MD5(Message Digest Algorithm 5) 해시 값 구함
    2. 메서드와 URI 패스를 ':'로 연결하고, MD5 해시 값을 구함
    3. (1), 서버로부터 얻은 nonce, 클라이언트가 nonce를 보낸 횟수, 클라이언트가 생성한 nonce, qop값, (2)의 값을 ':'로 연결하고 MD5 해시 값을 구한다.
    4. response 필드에 넣고 응답을 송신한다.

### Digest 인증의 이점과 결점

- 이점
    1. 패스워드를 도둑맞을 위험성이 없다.
    2. 해시값만 서버에 저
- 결점
    1. 요청할 때마다 401 Unauthorized 응답을 얻어야 한다. → 조작이 번잡하다.
    2. 웹 서버에서는 Digest 인증이 옵션으로 되어 있어 호스팅 서비스에서는 지원하지 않을 가능성도 있음

### WSSE 인증

HTTP 1.1의 표준 외의 인증 방식, AtomPob 등의 웹 API의 인증에 사용되고 있음

SSL/TLS의 이용이 불가능해서 Basic 인증을 사용하지 못하고, 호스팅 서비스 상의 CGI 스크립트 등으로 Digest 인증도 사용할 수 없는 경우에 사용. 민간에 의해 책정

WS-Security의 UsernameToken이라는 인증방식, WSSE(WS-Security Extension)

Basic 인증과 Digest 인증의 중간에 위치하는 인증 방식

**OpenID와 OAuth**
- OpenID - 심플한 싱글 사인 온
- OAuth - 웹 서비스 사이의 권한의 위임

# 09 캐시 : 이해부족

- 캐시 : 서버로부터 가져온 리소스를 로컬 스토리지(하드디스크 등)에 저장하여 재사용하는 방법

![Chapter9%20HTTP%20%E1%84%92%E1%85%A6%E1%84%83%E1%85%A5%20e84be7d831b64d988621cbfdbeeda6f7/Untitled%204.png](Chapter9%20HTTP%20%E1%84%92%E1%85%A6%E1%84%83%E1%85%A5%20e84be7d831b64d988621cbfdbeeda6f7/Untitled%204.png)

### 캐시용 헤더

어떤 리소스가 캐시 가능한지는 그 리소스를 취득했을 때의 헤더로 판단합니다.

리소스가 캐시 가능한지, 그 유효기간이 언제까지인지는 Pragma, Expire, Cache-Control 헤더를 이용해 서버가 지정합니다.

- Pragma - 캐시를 억제한다.
- Expires - 캐시의 유효기한을 나타낸다.
- Cache-Control - 상세한 캐시 방법을 지정한다.
- 캐시용 헤더의 사용 구분
    1. 캐시를 시키지 않을 경우는 Pragma와 Cache-Control의 no-cache를 동시에 지정한다.
    2. 캐시의 유효기간이 명확하게 정해져 있는 경우는 Expires를 지정한다.
    3. 캐시의 유효기간을 상대적으로 지정하고자 하는 경우는 Cache-Control의 max-age로 상대시간을 지정한다.

### 조건부 GET

클라이언트가 Expires와 Cache-Control 헤더를 검증한 결과, 로컬 캐시를 그대로 재사용할 수 없다고 판단한 경우라도 조건부 GET을 송신하면 캐시를 재사용할 수 있는 가능성이 있습니다.

조건부 GET은 서버측에 있는 리소스가 클라이언트 로컬 캐시로부터 변경되어 있는지 여부를 조사하는 조건을 요청 헤더에 포함시킴으로써, 캐시를 그대로 사용할 수 있는지 검토하는 구조입니다.

- If-Modified-Since : 리소스의 갱신일시를 조건으로 한다.
- If-None-Match : 리소스의 ETag를 조건으로 한다.

**ETag의 계산**
정적 파일
  Apache는 기본적으로 정적 파일의 ETag의 값은 inode번호, 파일 사이즈, 갱신일시를 이용해 자동으로 계산해 줍니다.

동적 파일
  동적으로 생성한 HTML 페이지와 피드 등의 경우, Apache 등의 웹 서버가 자동 계산 X

- If-Modified-Since와 If-None-Match의 사용 구분 : ETag 헤더 유무

# 10 지속적 접속

HTTP 1.1에서의 신기능

HTTP 1.0에서 클라이언트가 TCP 커넥션을 확립해 요청을 송신하고 서버가 응답을 반환할 때마다 TCP 커넥션을 절단함.

HTTP 1.1.에서 매 요청과 응답시마다 절단X, 계속 접속을 유지하여 모아두는 방법을 개발
지속적 접속에서는 클라이언트가 응답을 기다리지 않고 같은 서버에 요청을 송신할 수 있습니다.
→ 파이프라인화 Pipelining

# 11 그 밖의 HTTP 헤더

표준은 아니지만 자주 사용하는 헤더

### Content-Disposition : 파일명을 지정한다.

```java
Content-Disposition: attachment; filename="rest.txt"
```

- 문자 인코딩 방식, 브라우저마다 서버의 동작을 변경하는 구현이 필요함

### Slug : 파일명과 힌트를 지정한다.

```java
Slug: %ED%85%8C%EC%8A%A4%ED%8A%B8
//테스트
```

- Slug 헤더에서는 비 ASCII 문자열의 인코딩으로 UTF-8의 %인코딩을 채용.

# 12 HTTP 헤더를 활용하기 위해서

이 장에서는 HTTP 헤더에 대해 개념과 이용법 설명.

헤더는 메서드와 스테이터스 코드를 조합하여 인증이나 캐시같은 HTTP의 중요한 기능을 구현

다른 표준을 활용하는 것도 HTTP 헤더의 특징