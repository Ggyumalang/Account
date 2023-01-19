# Account API

## 프로젝트 설명

Account 시스템은 사용자 , 계좌 , 거래의 정보를 저장하고 있으며,<br>
외부 시스템에서 거래 요청 시 정보를 받아 각각의 도메인 별로 하기의 서비스를 제공합니다.<br>

사용자 - 편의를 위해 DB로 수기 입력 <br>
계좌 - 생성 , 해지 , 확인  기능 제공 <br>
거래 - 잔액 사용 , 잔액 사용 취소 , 거래 확인 기능<br>

-----
## 패키지 구조
- aop : AOP로 중복 거래 방지 락을 걸 때 사용될 어노테이션 등을 위치
- config : redis 관련 설정 및 클라이언트 빈 등록, JPA 관련 설정 등록
- controller : API의 endpoint를 등록하고, 요청/응답의 형식을 갖는 클래스 패키지
- domain : jpa entity
- dto : DTO(Data Transafer Object)를 위치
  * Controller에서 요청/응답에 사용할 클래스
  * 로직 내부에서 데이터 전송에 사용할 클래스
- exception : 커스텀 Exception, Exception Handler 클래스 패키지
- repository : Repository(DB에 연결할 때 사용하는 인터페이스)가 위치하는 패키지
- service : 비즈니스 로직을 담는 서비스 클래스 패키지
- type : 상태타입, 에러코드, 거래종류 등의 다양한 enum class들의 패키지
- test : controller 및 service 들의 테스트 코드를 담은 패키지

-----
## API 상세명세서
1. 계좌 생성 API (POST /account)
- 파라미터 : 사용자 ID, 초기 잔액
- 정책 : 사용자가 없는 경우, 계좌가 10개(사용자당 최대 보유 가능 계좌 수)인 경우 실패 응답
- 성공 응답 : 사용자 ID, 계좌번호, 등록일시

2. 계좌 해지 API (DELETE /account)
- 파라미터 : 사용자 ID, 계좌 번호
- 정책 :  사용자 또는 계좌가 없는 경우, 사용자 ID와 계좌 소유주의 ID가 다른 경우, <br>
          계좌가 이미 해지 상태인 경우, 잔액이 있는 경우 실패 응답
- 성공 응답 : 사용자 ID, 계좌번호, 해지일시

3. 계좌 확인 API (GET /account?user_id={userId})
- 파라미터 : 사용자 ID
- 정책 : 사용자가 없는 경우 실패 응답
- 성공 응답 : List<계좌번호 , 잔액>의 구조로 응답

4. 잔액 사용 API (POST /transaction/use)
- 파라미터 : 사용자 ID, 계좌 번호, 거래 금액
- 정책 : 사용자 없는 경우, 사용자 ID와 계좌 소유주의 ID가 다른 경우, <br>
         계좌가 이미 해지 상태인 경우, 거래금액이 잔액보다 큰 경우, <br>
         거래금액이 너무 작거나 큰 경우 실패 응답 <br>
        - 해당 계좌에서 거래(사용, 사용 취소)가 진행 중일 때 <br>
          다른 거래 요청이 오는 경우 해당 거래가 동시에 잘못 처리되는 것을 방지해야 한다. <br>
- 성공 응답 : 계좌번호 , 거래 결과 코드(성공/실패) , 거래 ID , 거래금액, 거래일시

5. 잔액 사용 취소 API (POST /transaction/cancel)
- 파라미터 : 거래 ID, 계좌번호, 취소 요청 금액
- 정책 :  거래 ID에 해당하는 거래가 없는 경우, 계좌가 없는 경우, 거래와 계좌가 일치하지 않는 경우 <br>
          거래금액과 거래 취소 금액이 다른경우(부분 취소 불가능) 실패 응답 <br>
        - 1년이 넘은 거래는 사용 취소 불가능 <br>
        - 해당 계좌에서 거래(사용, 사용 취소)가 진행 중일 때 다른 거래 요청이 오는 경우 해당 거래가 동시에 잘못 처리되는 것을 방지해야 한다.
- 성공 응답 : 계좌번호 , 거래 결과 코드(성공/실패) , 거래 ID , 거래금액, 거래일시

6. 거래 확인 API (GET /transaction/{transactionId})
- 파라미터 : 거래 ID
- 정책 : 해당 거래 ID의 거래가 없는 경우 실패 응답
- 성공 응답 : 계좌번호, 거래종류(잔액 사용, 잔액 사용 취소), 거래 결과 코드(성공/실패), 거래 ID, 거래금액, 거래일시

-----

## JDK version
OpenJDK version 11.0.11

## Spring version
2.6.8

## Dependencies
- Lombok
- H2DB
- JPA
- Embedded redis
- Spring Web
- Spring MVC
- Validation



