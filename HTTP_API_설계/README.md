## HTTP API - 컬렉션 (POST 기반)

1.  클라이언트는 등록될 리소스의 경로(URI)를 모른다
    - 회원 등록 /members => POST
    - POST /members
2.  서버가 새로 등록된 리소스 URI를 생성해준다.

        HTTP/1.1 201 Created
        Location: /members/100

3.  컬렉션(Collection)
    - 서버가 관리하는 리소스 디렉토리
    - 서버가 리소스의 URI를 생성하고 관리
    - 여기서 컬렉션은 /members

<br />

## HTTP API - 스토어 (PUT 기반)

1. 리소스가 없으면 생성하고 있으면 덮어씌운다.

2. 클라이언트가 리소스 URI를 알고 있어야 한다.

   - 파일 등록 /files/{filename}
   - PUT /files/star.jpg

3. 클라이언트가 직접 리소스의 URI를 지정해준다.

4. 스토어(Store)
   - 클라이언트가 관리하는 리소스 저장소
   - 클라이언트가 리소스의 URI를 정한다.

## HTML FORM 전송

1.  GET, POST만 지원
2.  컨트롤 URI

        회원 등록 POST /members/new
        회원 수정 폼 GET /members/{id}/edit
        회원 수정 POST /members/{id}/edit
        (new, edit 같은 것들이 컨트롤 URI)

    - 동사로 된 리소스 경로 사용
    - HTTP 메서드로 해결하기 애매한 경우 사용 (실무적으로 쓸 수 밖에 없다)

## URI 설계 개념 정리

1. 문서 (document)

- 단일 개념 (파일 하나, 객체 인스턴스, 데이터베이스 row)
- /members/100 , /files/star.jpg

2. 컬렉션 (collection)

- 서버가 리소스의 URI를 생성하고 관리한다.

3. 스토어 (store)

- 클라이언트가 리소스의 URI를 알고 관리한다.
- 파일이나 게시판 등에 쓰일 수 있다.

4. 컨트롤러 (controller)

- 위 세가지만으로 해결하기 어려운 추가 프로세스 실행
- /members/{id}/delete
