# 상태 코드

- 클라이언트가 보낸 요청의 처리 상태를 읍당에서 알려주는 기능

        1xx (Informational): 요청 수신되어 처리중 (거의 안씀)
        2xx (Successful): 요청 정상 처리
        3xx (Redirection): 요청 완료하려면 추가 행동이 필요
        4xx (Client Error): 클라이언트 오류, 잘못된 문법 등으로 서버가 요청 수행할 수 없음
        5xx (Server Error): 서버 오류, 서버가 정상 요청을 처리하지 못함

- 만약 모르는 상태 코드가 나타나면? 앞자리 숫자로 대략적인 예상이 가능하다.

## 2xx (Successful) - 성공

        200 OK
        201 Created
        202 Accepted (거의 안씀)
        204 No Content

1.  200 OK

    - 요청이 성공적으로 처리됐을때

2.  201 Created

        HTTP/1.1 201 Created
        Content-Type: application/json
        Content-Length: 34
        Location: /members/100 => 생성된 리소스는 응답의 Location 헤더 필드로 식별

        *로그인 인증 때는 헤더에 Set-Cookie와 토큰값이 적혀있기도하다.

        {
            "username": "young",
            "age": 200
        }

3.  202 Accepted

    - 요청이 접수되었으나 처리가 완료되지 않았음
    - 배치 처리 같은 곳에서 사용
    - 요청 접수 후 1시간 뒤 배치 프로세스 요청 처리

4.  204 No Content
    - 서버가 요청을 성공적으로 수행했지만, 응답 페이로드 본문에 보낼 데이터가 없음
    - ex) 웹 문서 편집기에서 save 버튼을 누름
    - 요청의 결과로 아무 내용이 없어도 될때 성공을 인식하게만 하면 될 때

## 3xx (Redirection)

- 요청을 완료하기 위해 유저 에이전트의 추가 조치 필요

        리다이렉션이란?
        웹 브라우저는 3xx 응답의 Location 헤더 있을 시 해당 위치로 자동 이동 (리다이렉트)

        클라이언트가 이제 쓰지 않는 url 요청할때
        GET /event HTTP/1.1
        Host: localhost:8080

        서버
        HTTP/1.1 301 Moved Permanently
        Location: /new-event

        클라이언트는 /new-event 페이지로 이동한다.

### 😑 영구 리다이렉션

특정 리소스의 URI가 영구적으로 이동

    - 원래의 URL를 사용x, 검색엔진등에서도 변경 인지
    - ex) /event => /new    -event로 주소가 바뀌었을때

1. 301 Moved Permanently

   - 리다이렉트시 요청 메서드가 GET으로 변하고 본문이 제거될 수 있음

2. 308 Permanent Redirect (실무적으로 잘 쓰지 않는다.)

   - 리다이렉트시 요청 메서드를 유지하며 본문은 유지됨

### 😑 일시적인 리다이렉션

리소스의 URI가 일시적으로 변경

    - ex) 주문 완료 후 주문 내역 화면으로 이동

1.  302 Found

    - 스펙의 의도는 HTTP 메서드를 유지하는 것
    - 대부분 요청 메서드를 GET으로 변경시키지만 명확하지않아 303이 생겻다.
    - 리다이렉트시 요청 메서드가 GET으로 변하고 본문이 제거될 수 있음

2.  307 Temporary Redirect

    - 302와 기능 같음
    - 리다이렉트시 요청 메서드와 본문 유지 (메서드는 절대 변경하면 안된다.)

3.  303 See Other

    - 리다이렉트시 요청 메서드가 GET으로 변함

4.  PRG: POST/Redirect/Get

        POST로 주문 후에 웹 브라우저를 새로고침하면?

    - POST로 주문 후에 새로고침으로 인한 중복 주문 방지
    - POST로 주문 후 주문 결과 화면을 GET 메서드로 리다이렉트
    - 새로고침해도 결과화면을 GET으로 조회
    - 중복 주문 대신에 결과 화면만 GET으로 다시요청

### 😑 기타 리다이렉션

1. 304 Not Modified

   - 캐시를 목적으로 사용
   - 클라이언트에게 리소스가 수정되지 않았음을 알려준다

   => 클라이언트는 로컬PC에 저장된 캐시를 재사용한다.
   => 캐시로 리다이렉트한다.
   => 응답에 메시지 바디를 포함하면 안된다.
