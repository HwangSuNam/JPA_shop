# JPA_shop
- JPA 실전 학습 및 shop 프로젝트
- JPA + Spring boot

# 엔티티 설계
![image](https://user-images.githubusercontent.com/8254744/99077197-b5af8800-25ff-11eb-9f77-47e65c146a4c.png)

# 테이블 설계
![image](https://user-images.githubusercontent.com/8254744/99077327-ec859e00-25ff-11eb-9b81-16af67482e12.png)

# 구조
### 1. Entity
- @Entity 선언
- 여기서는 편의를 위해 @Getter, @Setter를 선언했지만 실무에서는 이렇게 하면 안된다. 추적이 어려워지기 때문
  - Member
  - Order
  - OrderItem
  - Item (Single table 전략)
    - Album, Book, Movie
  - Delivery
  - Category
  - Category_item(다대다를 풀어내기 위한 중간 테이블, 여기서 엔티티는 아님)
  - Address(엔티티는 아니며 임베디드 타입임)
### 2. Repository
- @Repository 선언, @RequiredArgsConstructor를 이용해 DI EntityManaer 초기화
- EntityManaer을 이용해 쿼리 수행
  - ItemRepository
  - MemberRepository
  - OrderRepository
  - OrderSearch(레파지토리 역할이 아니라 값 바인딩을 위한 클래스로 사용됨)
### 3. Service
- @Service, @Transactional(readOnly = true), @RequiredArgsConstructor 선언
- @Transactional(readOnly = true)을 하는 이유는 JPA의 모든 데이터 명령이나 로직이 트랜잭션 안에서 수행되어야 하기 때문. 참고로 readOnly = true로 설정하면 Flush 모드를 never로 설정하여 더티 체킹을 하지 않도록 하고 DB한테 SELECT만 할 것을 알려주므로 큰 리소스를 쓰지 않는다.
  - ItemService
  - MemberService
  - OrderService
### Controller는 생략
