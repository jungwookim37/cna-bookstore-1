# cna-bookstore

## User Scenario
```
* 고객 또는 고객 운영자는 고객정보를 생성한다.
* 북 관리자는 판매하는 책의 종류와 보유 수량을 생성하고 수정할 수 있다.
* 고객은 책의 주문과 취소가 가능하며 주문 정보의 수정은 없다.
* 고객이 주문을 생성할 때 고객정보와 Book 정보가 있어야 한다.
  - Order -> Customer 동기호출
  - Order -> BookInventory 동기호출
* 주문 시에 재고가 없더라도 주문이 가능하다.
  - 주문 상태는 “Ordered”
* 주문 취소는 "Ordered" 상태일 경우만 가능하다.
* 배송준비는 주문 정보를 받아 이뤄지며 재고가 부족한 경우, 책 입고가 이뤄져서 재고 수량이 충분한 경우에 배송 준비가 완료되었음을 알린다.
* 배송은 주문 생성 정보를 받아서 배송을 준비하고 주문 상품 준비 정보를 받아서 배송을 시작하며 배송이 시작되었음을 주문에도 알린다.
  - 주문 생성 시 배송 생성
  - 상품 준비 시 배송 시작  
* 배송을 시행하는 외부 시스템(물류 회사 시스템) 또는 배송 담당자는 배송 단계별로 상태는 변경한다. 변경된 배송 상태는 주문에 알려 반영한다.
* 주문 취소되더라도 고객은 MyPage에서 주문 이력을 모두 조회할 수 있다.
```
## 아키텍처
```
* 모든 요청은 단일 접점을 통해 이뤄진다.

```

## Cloud Native Application Model
![Alt text](cna-bookstore.PNG?raw=true "Optional Title")

## 구현 점검

### 모든 서비스 정상 기동 
```
http http://gateway:8080/customers
http http://gateway:8080/myPages
http http://gateway:8080/books
http http://gateway:8080/deliverables
http http://gateway:8080/stockInputs
http http://gateway:8080/orders
http http://gateway:8080/deliveries
```

### 고객 생성
```
http POST http://gateway:8080/customers customerName="hong gil dong"
http POST http://gateway:8080/customers customerName="bak na re"
```

### 책 정보 생성
```
http POST http://gateway:8080/books bookName="alice in a wonderland" stock=100
http POST http://gateway:8080/books bookName="quobadis?" stock=50
```

### 주문 생성
```
http POST http://gateway:8080/orders bookId=1 customerId=1 deliveryAddress="bundang gu" quantity=50
http POST http://gateway:8080/orders bookId=1 customerId=2 deliveryAddress="incheon si" quantity=100
```
##### Message 전송 확인 결과

##### Deliveriy 확인 결과

##### Deliverables 확인 결과

### 주문 준비
```
http http://gateway:8080/deliverables
http POST http://gateway:8080/stockInputs bookId=1 quantity=200
```
##### 재고 수량 변경 확인 결과
##### 주문 상태 변경 확인 결과
```
http http://gateway:8080/orders
http http://gateway:8080/deliverables
http http://gateway:8080/deliveries
```

### 배송 상태 변경
```
```

### 고객 Mypage 이력 확인
```
```

## CI/CD 점검

## Circuit Breaker 점검

## Autoscale 점검
d오토스케일 작업 예정

## Readiness Probe 점검
### 설정 확인
```
readinessProbe:
  httpGet:
    path: '/orders'
    port: 8080
  initialDelaySeconds: 12
  timeoutSeconds: 2
  periodSeconds: 5
  failureThreshold: 10
```
### 점검 순서
```
1. Repli
http http://gateway:8080/orders
2. Siege 실행
siege -c2 -t120S  -v 'http://gateway:8080/orders'
3. Readiness 설정 제거 후 배포
4. Siege 결과 Availability 확인(100% 미만)
5. Readiness 설정 추가 후 배포
6. Siege 결과 Availability 확인(100%)
```

## Liveness Probe 점검
### 설정 확인
```
livenessProbe:
  httpGet:
    path: '/isHealthy'
    port: 8080
  initialDelaySeconds: 120
  timeoutSeconds: 2
  periodSeconds: 5
  failureThreshold: 5
```
### 점검 순서
```
1. 기동 확인
http http://gateway:8080/orders
2. Siege 실행
siege -c2 -t100S  -v 'http://gateway:8080/orders'
3. 서비스 상태 변경
http http://order:8080/makeZombie
4. Pod 재기동 확인
```
