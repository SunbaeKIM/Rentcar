# Rentcar 서비스 : Rentcar

## 서비스 소스 레파지토리
- https://github.com/SunbaeKIM/rentcar-assignment.git
- https://github.com/SunbaeKIM/rentcar-car.git
- https://github.com/SunbaeKIM/rentcar-customercenter.git
- https://github.com/SunbaeKIM/rentcar-gateway.git
- https://github.com/SunbaeKIM/rentcar-rent.git


## 서비스 시나리오

### 기능적 요구사항
1. 고객이 렌트카 키오스크에서 렌터카를 예약 한다. 
2. 자동차 재고 여부를 확인하여 렌트가능 여부를 알려준다.
3. 구입가능 시 렌트카 재고가 감소하고 자동으로 렌트정보가 배정시스템으로 전달된다.
4. 배정시스템에서는 근처 렌터카 회사를 배정한다.( 랜던함수 )
5. 자동차가 배정되면  [RentRequest] 상태를 예약관리로 보낸다. 
6. 고객은 자동차 수령 이전에 예약주문을 취소할 수 있다.
7. 예약취소가 접수 [RentCancel]되면 배정된 주문데이터를 삭제하고, 해당 재고를 원복한다.
8. 정상 취소처리가 되면 [RentCancel] 상태가 된다.

### 비기능적 요구사항

#### 트랜잭션
 - 자동차 재고가 없는 경우 예약은 접수처리가 되지 않는다. Sync 호출(Req/Res)

#### 장애격리
 - 배정관리 서비스가 되지않더라도 예약접수는 정상적으로 처리가 되어야한다. Async (event-driven)

## 이벤트스토밍

![캡처0](https://user-images.githubusercontent.com/31124658/93425572-4108e780-f8f5-11ea-9191-b88cf2c4e7aa.JPG)

## 구현
- 분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트로 구현
- 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

### DDD 의 적용
- 각 서비스내에 도출된 핵심 객체를 Entity 로 선언
  - 렌트 -> rent
  - 배정 -> assignment
  - 자동차 -> car

### 적용 후 REST API 의 테스트

- 한정판 자동차 수량 등록 : http http://car:8080/cars carName=Sonata quantity=20
- 자동차 예약 : http POST http://rent:8080/rents carId=1
- 자동차 예약 취소 : http PATCH http://rent:8080/rents/1 status="RentCancel"
- 예약확인 : http GET http://rent:8080/rents
- 배정확인 : http GET http://assignment:8080/assignments
- 자동차 재고확인 : http GET http://car:8080/cars
- 뷰 확인 : http GET http://customercenter:8080/customercenters

## SAGA 패턴

- 예약을 하면, 재고가 감소하고 배정이 되며 예약상태값이 RentAccepted 변경된다. 
- 예약을 취소하면 재고가 원복되고 배정은 삭제되며 예약상태값이 RentCanceled 변경된다. 

![캡처](https://user-images.githubusercontent.com/31124658/93425581-42d2ab00-f8f5-11ea-91cb-4c2262b6e7f6.JPG)

![캡처1](https://user-images.githubusercontent.com/31124658/93425580-42d2ab00-f8f5-11ea-9e5b-12f9da817201.JPG)

![캡처1 1](https://user-images.githubusercontent.com/31124658/93425573-41a17e00-f8f5-11ea-968e-f775a42bb95c.JPG)

![캡처2](https://user-images.githubusercontent.com/31124658/93425578-423a1480-f8f5-11ea-8ab4-f58aa335ea7c.JPG)

![캡처3](https://user-images.githubusercontent.com/31124658/93425577-423a1480-f8f5-11ea-8d3f-87a4975d4a9b.JPG)

![캡처4](https://user-images.githubusercontent.com/31124658/93425575-41a17e00-f8f5-11ea-9a70-45e7f9fc3243.JPG)


## CQRS

- 고객은 자신의 예약 상태를 뷰(CustomerCenter) 를 통해 확인할 수 있다. 
![캡처51](https://user-images.githubusercontent.com/31124658/93425571-40705100-f8f5-11ea-888c-61570ff80db8.JPG)


## 동기식 호출 

- 예약 시 재고확인하는 부분을 FeignClient를 사용하여 동기식 트랜잭션으로 처리 

![캡처52](https://user-images.githubusercontent.com/31124658/93425570-40705100-f8f5-11ea-8aff-ce8e726223e7.JPG)

![캡처53](https://user-images.githubusercontent.com/31124658/93425568-3fd7ba80-f8f5-11ea-8641-da40fbded884.JPG)


## 폴리글랏

- view페이지인 page 서비스에서는 DB hsql를 적용함

![캡처54](https://user-images.githubusercontent.com/31124658/93425567-3f3f2400-f8f5-11ea-80ec-9a487b35b3b6.JPG)


## 서킷 브레이킹

 - istio를 이용한 CB TEST 

<img src="https://user-images.githubusercontent.com/68719151/93408579-e3ae6f80-f8cf-11ea-8459-ac5f7ef7cd08.JPG"></img>

<img src="https://user-images.githubusercontent.com/68719151/93408602-ec06aa80-f8cf-11ea-8ee6-a48d9c01a7d7.JPG"></img>

<img src="https://user-images.githubusercontent.com/68719151/93408584-e5783300-f8cf-11ea-9bf1-982a5e87ebc5.JPG"></img>

<img src="https://user-images.githubusercontent.com/68719151/93408587-e6a96000-f8cf-11ea-8834-f284425f45ff.JPG"></img>


## 오토스케일 아웃

 - Mertric 서버 설치 후 오토스케일링 TEST
 - rent의 deployment.yaml수정 후 배포 

<img src="https://user-images.githubusercontent.com/68719151/93408895-9f6f9f00-f8d0-11ea-9c2c-e40e4335e90b.JPG"></img>

<img src="https://user-images.githubusercontent.com/68719151/93408910-aac2ca80-f8d0-11ea-90ff-75d3c01f52b8.JPG"></img>

<img src="https://user-images.githubusercontent.com/68719151/93408953-c1692180-f8d0-11ea-9ef0-dde5cf3e84e4.JPG"></img>

<img src="https://user-images.githubusercontent.com/68719151/93408929-b4e4c900-f8d0-11ea-8d57-8013d1d2840a.JPG"></img>

<img src="https://user-images.githubusercontent.com/68719151/93408943-bc0bd700-f8d0-11ea-9fa9-44cea466bf4b.JPG"></img>

<img src="https://user-images.githubusercontent.com/68719151/93408956-c332e500-f8d0-11ea-9b59-cd46fe189dd2.JPG"></img>

