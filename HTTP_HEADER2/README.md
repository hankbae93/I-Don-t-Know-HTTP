# HTTP 헤더 - 캐시와 조건부 요청

1. 캐시가 없을 때
   - 데이터가 변경되지 않아도 계속 네트워크를 통해서 데이터를 다운로드받아야 된다.
   - 인터넷 네트워크는 매우 느리고 비싸다
   - 느린 사용자 경험

## 캐시 적용

        HTTP/1.1 200 OK
        Content-Type: image/jpeg
        cache-control: max-age=60 //캐시가 유효한 시간

        asdjv212... // 이미지 파일 byte

1. 브라우저에서 응답결과를 캐시를 저장한 후 다시 요청할 때 그 유효시간동안 캐시를 조회해서 다시 사용한다.

## 검증 헤더와 조건부 요청

1. 캐시 만료 후에도 서버에서 데이터 변경하지 않음

- 이런 상황에서 같은 데이터를 또 요청하는 것이 아쉽기 때문에 <br/>
  클라이언트와 서버가 서로 같은 데이터를 가지고 있다는 검증 헤더가 필요하다

          -서버-
          HTTP/1.1 200 OK
          Content-Type: image/jpeg
          cache-control: max-age=60
          Last-Modified: 2020년 11월 10일 10:00:00 // 검증 헤더

          asdasdjjiqwdj... // 이미지 byte

          - 클라이언트 -
          Get /star.jpg
          if-modified-since: 2020년 11월 10일 10:00:00 // 조건부 요청

          서버는 이 헤더를 읽고 데이터가 수정이 됐는지 안됐는지 확인할 수 있다.

          - 서버 -
          HTTP/1.1 304 Not Modified
          Content-Type: image/jpeg
          cache-control: max-age=60
          Last-Modified: 2020년 11월 10일 10:00:00

          HTTP body없이 다시 전송

          클라이언트단은 응답 헤더를 읽고 캐시에서 다시 조회해 사용한다.

2. 검증 헤더

- 캐시 데이터와 서버 데이터가 같은지 검증
- Last-Modified, ETag

3. 조건부 요청 헤더

- 검증 헤더로 조건에 따른 분기
- If-Modified-Since: Last-Modified 사용
- If-None-Match: ETag 사용
- 조건 만족시 200 OK / 조건 불만족 시 304 Not Modified

        데이터 미변경 예시
        캐시 : 2020년 11월 10일 10:00:00 vs 서버 : 2020년 11월 10일 10:00:00
        304 Not Modified, 헤더 데이터만 전송 (BODY 미포함)
        전송 용량 0.1M (헤더 0.1M, 바디 1.0M)

        데이터 변경 예시
        캐시: 2020년 11월 10일 10:00:00 vs 서버: 2020년 11월 10일 11:00:00
        200 Ok, 모든 데이터 전송(BODY 포함)
        전송 용량 1.1M (헤더 0.1M, 바디 1.0M)

4. 단점

- 1초 미만 단위로 캐시 조정이 불가능
- 날짜 기반의 로직 사용
- 데이터를 수정해서 날짜가 다르지만 데이터 자체는 같은 경우
- 서버에서 별도의 캐시 로직을 관리하고 싶은 경우 <br/>데이터가 크게 영향없게 변한 경우

        ETag(Entity Tag)
        1. 캐시용 데이터에 임의의 고유 버전이름 추가
        ex) ETag: "v1.0" , ETag:"asdbasc"

        2. 데이터가 정말 변경 될때는 이름을 아예 바꾸어버린다.
        ex) ETag: "asdbasc" => "bbbbbb"

        3. 단순하게 ETag의 값만 같으면 유지, 아니면 다시 요청처리

## 캐시 제어 헤더

1.  Cache-Control

        Cache-Control: max-age
        캐시 유효시간, 초단위

        Cache-Control: no-cache
        데이터는 캐시해도 되지만 항상 origin 서버에서 검증하고 사용해야된다.

        Cache-Control: no-store
        데이터에 민감한 정보가 있으므로 메모리에서 쓰고 빨리 삭제해야된다.

2.  Pragma (하위 호환)

- 캐시 제어
- Pragma: no-cache

3. Expires (하위 호환)

- 캐시 만료일 지정
- 날짜로 지정해야되서 초단위의 max-age를 권장한다

## 프록시 캐시

    클라이언트 => 프록시 캐시 서버 (한국 어딘가) => 미국 Origin 서버

예를 들어 유투브가 있다고 하면 미국에 있는 서버에 직접 가는 것보다 중간 한국의 프록시 서버에서

캐시를 받아오면서 속도를 높일 수 있다.

        Cache-Control: public
        public 캐시에 저장됨

        Cache-Control: private
        해당 사용자만을 위한 private 캐시에 저장되어야함

        Cache-Control: no-store
        데이터에 민감한 정보가 있으므로 메모리에서 쓰고 빨리 삭제해야된다.

## 캐시 무효화

1.  브라우저가 GET 요청같은 경우 임의로 캐싱해버리는 경우가 많다.

        Cache-Control: no-cache, no-store, must-revalidate

        Cache-Control: must-revalidate
        캐시 만료 후 최초 조회 시 원서버에 검증해야됨

        no-cache vs must-revalidate ?

        원서버에 검증해야된다는 것은 똑같지만
        중간 프록시 서버에서 장애가 날 경우
        no-cache: Error를 주거나 예전 데이터라도 던져준다.
        must-revalidate: 504 Gateway TimeOut을 무조건 던져준다.

        => 조금이라도 민감한 정보의 경우 예외처리를 좀더 확실히 해주기 위해 추가된 것으로 보임.
