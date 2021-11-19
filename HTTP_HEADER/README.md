# HTTP 헤더 개요

- HTTP 전송에 필요한 모든 정보

- RFC2616 => 2014년 RFC7230 으로 변함

## HTTP BODY

- 메시지 본문을 통해 표현 데이터 전달
- 표현은 요청이나 응답에서 전달할 실제 데이터
- 표현 헤더는 표현 데이터를 해석할 수 있는 정보 제공 ex) 데이터 유형(html, json), 데이터 길이, 압축 정보 등등

        HTTP/1.1 200 OK
        Content-Type: text/html;charset=UTF-8 // 표현헤더
        Content-Length: 3423

        // 메시지 본문
        <html> // 표현 데이터
            <body>...</body>
        </html>

### 1. 표현 헤더

1. Content-Type: 표현 데이터의 형식

- 미디어 타입, 문자
- html, json, image
- ex) aplication/json, text/html, mutipart-formdata, image/file

2. Content-Encoding: 표현 데이터의 압축 방식

- 데이터를 읽는 쪽에서 인코딩 헤더의정보로 압축 해제
- ex) gzip, deflate, identity

3. Content-Language: 표현 데이터의 자연 언어

- ex) ko, en, en-US

4. Content-Length: 표현 데이터의 길이

- 바이트 단위
- Transfer-encoding 같은 방식에는 사용하지 않는다

### 2. 협상 (Contents Negociation)

- 클라이언트가 선호하는 표현 요청

- Accept-Language

        Content-Language: en 인 Response가 온다.

        GET /event
        Accept-Language: ko 로 다시 한국어 지원으로 요청해볼 수 있다.

        그러나 서버에서 독일어와 영어만 지원한다고 했을때
        협상 헤더에서 우선순위를 설정할 수가 있다.

        Quality Values(q)값 사용
        Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
        q의 값이 1에 가까운 순으로 언어 우선순위를 요청할 수있다.
        ko-KR,ko 한국어 => en-US => en
        서버단에서는 q의 값으로 판단해서 알맞는 언어로 보내줄 수 있다.

- Accept

        Accept: text/*, text/plain, text/plain;format=flowed
        => 제일 구체적인 것으로 우선한다.
        text/plain;format=flowed이 제일 우선된다

        그다음 구체적인 순으로 미디어타입도 맞춰준다.

- Accept-Encoding, Accept-Charset

### 전송 방식

1.  단순 전송(Content-Length)

2.  압축 전송(Content-Encoding)

3.  분할 전송(Transfer)

        HTTP/1.1 200 OK
        Content-Type: text/plain
        // Content-length가 예상이 안되고 분할해서 보내기 때문에 넣어서도 안된다.
        Transfer-Encoding: chunked

        5
        Hello // 5byte마다 분할해서 보낸다
        5
        World
        0
        \r\n

4.  범위 전송(Content-Range)

- 통신을 하던 중 중간에 끊겼을 때 클라이언트가 중간부터 지정해서 다시 받을 수 있다.

        GET /event
        Range: bytes=1001-2000

        Response
        Content-Range: bytes 1001-2000 / 2000
        2000byte 중에 1001부터 2000바이트까지 다시 전송

### 일반 정보

1. From: 유저 에이전트의 이메일정보 (검색 엔진 같은 곳에서 주로 사용)

2. Referer: 이전 웹 페이지 주소

- 유입 경로 분석 가능
- 요청에서 사용

3. user-agent: 웹브라우저 정보, 클라이언트 애플리케이션 정보, 어떤 종류의 브라우저에서 장애가 발생하는지 파악가능

4. server: 요청을 처리하는 ORIGIN 서버의 소프트웨어 정보

5. Date: 메시지가 발생한 내용

- 응답에서만 사용

### 특별한 정보

1. Host

- 요청한 호스트 정보(도메인)
- 필수헤더
- 하나의 서버가 여러 도메인을 처리해야할 때

        GET /hello HTTP/1.1
        Host: aaa.com

        서버 IP: 200.200.200.2 에서
        aaa.com, bbb.com, ccc.com을 처리하고 있을 때
        Host 헤더가 없으면 이 요청이 어느사이트를 위한 요청인지 알 수 없다.

2. Location

- 페이지 리다이렉션

- 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면 자동으로 그 주소로 이동한다.

3. Allow

- 허용 가능한 메서드 알림 처리

4. Retry-After

- 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간

### 인증

1. Authorization

- 클라이언트 인증 정보를 서버에 전달
- Authorization: Basic xxxxxx

2. www-Authenticate

- 리소스 접근 시 필요한 인증 방법 정의

### 쿠키

- Set-Cookie: 서버에서 클라이언트로 쿠키 전달(응답)
- Cookie: 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달

        -클라이언트-
        Post /login HTTP/1.1
        user=홍길동

        -서버-
        HTTP/1.1 200 OK
        Set-Cookie: user=홍길동

        홍길동님이 로그인했습닏.

        ===>

        -클라이언트-
        GET /welcome HTTP/1.1
        Cookie: user=홍길동

- 쿠키는 모든 요청에 쿠키 정보 자동 포함

1. 사용처

- 주로 사용자 로그인 세션 관리
- 광고 정보 트래킹

2. 쿠키 정보는 항상 서버에 전송됨

- 최소한의 정보만 사용(세션 id, 인증 토큰)
- 서버에 전송하지 않고 웹 스토리지에 저장했다가 필요할때만 쓰기도 한다.

3. 보안에 민감한 데이터는 절대 저장하면 안된다.

4. 도메인을 정할 수 있다.

5. 보안

- Secure : https인 경우에만 전송
- HttpOnly : 자바스크립트에서 접근 불가, XSS 공격 방지, HTTP 전송에만 사용
- SameSite: 요청 도메인과 쿠키에 설정된 도메인이 같은 경우만 사용
