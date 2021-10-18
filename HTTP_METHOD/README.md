# 😉 API URI 설계

URI는 리소스만 식별!

URI는 리소스 대상 행위를 의미하지 않는다.

그래서 naming을 지을 땐 되도록 리소스 개념으로 짓고 행위는 메소드를 활용해 표현한다.

        회원 목록 조회
        /read-memeber-list (X) ==> /members

        여기서 리소스는 member 회원 정보를 의미한다.

        회원 조회
        /read-memeber-by-id (X) ==> GET /members/{id}

        회원 등록
        /regist-memeber-by-id (X) ==> POST /members/{id}

        회원 수정
        /modify-memeber-by-id (X) ==> PUT /members/{id}

        회원 삭제
        /delete-memeber-by-id (X) ==> DELETE /members/{id}

# 😉 HTTP 메서드

- GET : 리소스 조회

- POST : 요청 데이터 처리, 주로 등록에 사용

- PUT : 리소스를 대체(덮어씌운다), 해당 리소스가 없으면 생성

- PATCH : 리소스 부분 변경

- HEAD : GET과 동일하지만 메시지(body)부분 제외하고 상태줄, 헤더만 반환

- OPTIONS : 대상 리소스에 대한 통신 가능 옵션 설명 (CORS에서 주로 사용)

## GET

- 리소스 조회

- 서버에 전달하고 싶은 데이터는 query 통해서 전달

- BODY 통해서도 가능하긴 한데 지원 안하는 서버가 많음

## POST

- 요청 데이터 처리 (등록 이상의 의미를 가지고 있다.)

- 메시지 바디를 통해 서버로 요청 데이터 전달!

- 서버는 요청 데이터 처리

- 신규 데이터 등록, 프로세스 처리에 많이 사용됨

        POST는 대상 리소스가 리소스의 고유한 의미 체계에 따라 요청에 포함된 표현 처리하도록 요청합니다.
        1. HTML 양식 입력된 필드와 같은 데이터 블록을 데이터 처리 프로세스에 제공
        - HTML FORM의 입력된 정보로 회원가입, 주문 등 처리
        - POST /orders/{orderdId}/start-delivery (컨트롤 URI) :
        결제완료 => 배달 시작 => 배달 완료 처럼 단순히 값 변경을 넘어
        프로세스의 상태가 바뀌는 경우 POST와 저런 동사형 URI를 붙이기도 한다.

        2. 메시지 게시
        - 게시판 글쓰기, 댓글 달기

        3. 서버가 아직 식별하지 않은 새 리소스 생성
        - 신규 주문 생성

        4. 기존 자원에 데이터 추가
        - 한 문서 끝에 내용 추가하기

        5. 다른 메서드로 처리하기 애매한 경우
        - GET메서드를 사용하기 어려울 때 (ex. JSON 데이터를 서버에 보내야될 때)
        => 결국 POST로 뭘해도 된다는 얘기지만 우리가 그래도 다른 메서드들을 활용해야되는 이유는
        서버가 get 요청을 받을때 caching을 하는 등 고려해야될 점들이 있기 때문이다.

        결론 : 리소스마다 POST요청이 처리할 프로세스를 정해놔야 된다.

## PUT

1. 리소스를 대체

   - 리소스가 있으면 대체

   - 리소스가 없으면 생성

   - 쉽게 말해 덮어씌운다!

2. 클라이언트가 리소스를 식별한다.

   - 클라이언트가 리소스 위치를 알고 URI 지정

     PUT /members/100 HTTP/1.1
     Content-Type: application/json
     {
     "username": "Ranja",
     "age": "29.8"
     }

- POST와 가장 큰 차이점

3. 주의사항

- 리소스를 완전히 대체한다

       PUT /members/100 HTTP/1.1
       Content-Type: application/json
       {
           "age": "29.8"
       }

       서버단
       /members/100
       {
           "username": "Ranja", => 덮어씌워지면서 username의 정보가 사라질 수 있다.
           "age": "29.8"
       }

## PATCH

- PUT이 리소스를 완전히 대체하는 단점이 있어서 부분변경을 위해 태어났다.

- 문제가 있을때는 POST로 처리하면 된다.

        PATCH /members/100 HTTP/1.1
        Content-Type: application/json
        {
            "age": "29.8"
        }

        서버단
        /members/100
        {
            "username": "Ranja", => age만 보냈지만 age만 수정된다.
            "age": "29.8"
        }

## DELETE

- 리소스 제거

        DELETE /members/100 HTTP/1.1
        Content-Type: application/json

        서버단 /members/100 삭제 처리

# HTTP 메서드의 속성

1.  안전 (safe)

    - 호출해도 리소스를 변경하지 않는다

    - 안전은 해당 리소스만 고려한다.

    => 계속 호출해서 로그가 쌓여 장애 발생 이런 부분까지 고려하지 않는다.

2.  멱등 (idempotent)

    - 한 번 호출하든 100번 호출하든 결과가 똑같다.

            GET - 몇번 가져오던 같은 결과 조회
            PUT - 몇 번 대체하던 최종 결과는 같다
            DELETE - 몇번 삭제하던 삭제된 결과는 같다.
            *POST - 두 번 호출하면 두번 요청이 되버린다. 결제가 두번 될수도 있다.

    - 외부 요인으로 중간에 리소스가 변경되는 것까지 고려하지 않는다.

3.  캐시 가능 (Cacheable)

    - GET, HEAD, POST, PATCH 캐시 가능 (실제로는 GET,HEAD 정도만 캐시로 사용)
