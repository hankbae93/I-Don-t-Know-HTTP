# 클라이언트가 서버로 데이터 전송하는 방식

## 정적 데이터 조회

        클라이언트
        GET /static/star.jpg HTTP/1.1
        Host: localhost:8080


        서버
        HTTP/1.1 200 OK
        Content-Type: image/jpeg
        Content-Length: 34012

        이미지소스 (byte)

- 이미지 정적 텍스트 문서
- GET을 많이 사용
- 정적 데이터는 일반적으로 쿼리 파라미터 사용없이 리소스경로로 단순하게 조회 가능

<br/>

## 동적 데이터 조회

        클라이언트
        GET /search?q=hello&hl=ko HTTP/1.1 => 쿼리 파라미터 사용
        Host: localhost:8080

- 주로 검색, 게시판 목록에서 정렬 필터(검색어)
- 조회 조건을 줄여주는 필터, 결과 정렬 조건에 주로 사용
- 조회는 GET 사용
- GET은 쿼리 파라미터 사용해서 데이터 전달

<br/>

## HTML Form 데이터 전송

1. POST 전송 - 저장

```html
<form action="/save" method="post">
  <input type="text" name="username" />
  <input type="text" name="age" />
  <button type="submit">전송</button>
  <!-- submit하는 순간 폼의 데이터를 읽어서 HTTP 메시지를 생성해준다 -->
</form>
```

        클라이언트
        POST /save HTTP/1.1
        Host: localhost:8080
        Content-Type: application/x-www-form-urlencoded
        (한글 같은 경우가 인코딩되서 넘어간다) abc김 => abc%EA%....

        username=kim&age=20

- POST가 아닌 GET으로 요청할 시 폼데이터를 쿼리 파라미터에 적게 된다.
- 리소스 변경이 발생하는 곳에 사용하면 안됨
- ex) 회원가입, 상품 주문, 데이터 변경

<br/>

2. multipart/form-data

```html
<form action="/save" method="post" enctype="multipart/form-data">
  <input type="text" name="username" />
  <input type="text" name="age" />
  <input type="file" name="file1" />
  <button type="submit">전송</button>
</form>
```

        클라이언트
        POST /save HTTP/1.1
        Host: localhost:8080
        Content-Type: multipart/form-data; boundary=---XXX (브라우저가 알아서 바운더리를 나눠 메세지를 생성한다)

        ---XXX
        Content-Disposition: form-data; name="username"

        ranja
        ---XXX
        Content-Disposition: form-data; name="age"

        20
        ---XXX
        Content-Disposition: form-data; name="file1"; filename="ranja_profile.jpg"
        Content-Type: image/png

        이미지소스 (byte)

- 파일 업로드 같은 바이너리 데이터 전송 시 사용
- 여러 종류의 파일과 폼 내용 전송 가능
- HTML Form 전송은 GET 과 POST만 지원한다.

<br/>

## HTTP API 데이터 전송

1. 서버 to 서버
2. 앱 클라이언트
3. 웹 클라이언트
   - HTML에서 폼 전송 대신 AJAX
   - React, VueJS 같은 웹 클라이언트와 API 통신
4. POST, PUT, PATCH : 메시지 바디를 통해 데이터 전송
5. Content-Type: application/json (표준, xml보다 json을 많이 활용한다.)
