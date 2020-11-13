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
### 4. Controller는 생략

# 작업시, 신경 쓴 부분
### 1. DDD 설계
  - 가능한 비즈니스 로직은 엔티티 내부에 작업
  - 엔티티의 생성자 작업시, 가능한 비즈니스 로직 추가(ex. order.addOrderItem(orderItem);)
  - 여기서는 많은 부분이 미비하지만 실무에서는 setter로 엔티티 객체에 값을 넣지 말고 DDD 방식으로 엔티티에 메소드를 만들어야 한다.
### 2. 엔티티에 연관관계 메소드 작업(양 쪽 중에서 핵심점으로 컨트롤 하는 쪽에 코딩)
### 3. 기본적으로 fetch는 모두 lazy로 설정
### 4. 엔티티에 다른 생성자가 있다면 @NoArgsConstructor(access = AccessLevel.PROTECTED)를 선언해서 비즈니스 로직에서 함부로 객체를 생성하지 못하도록 한다.
### 5. 머지 방식 보다는 update는 더티체킹을 이용한다.
  - 머지 방식은 파라미터가 영속되지 않음. 리턴된 값만 영속됨
### 6. API 설계시, 신경 쓴 부분
  - 엔티티를 컨트롤러 파라미터로 받으면 문제가 되니 절대 그대로 사용하지 않고 별도의 DTO를 만들어 사용한다.
    - 별도의 DTO를 만들면 엔티티가 바뀌더라도 API 스펙이 바뀌지 않는다.
  - 엔티티를 리턴 타입으로 그대로 사용하지 않는다.
    - 민감한 정보가 유출될 수 있다.
    - 위와 마찬가지로 API 스펙이 바뀌면 문제가 된다. 엔티티에 @JsonIgnore와 같은 프레젠테이션 계층을 위한 로직이 추가된다.
  - array로 반환하지 않고 한 번 감싼 후, 반환한다.
  - 전체를 다 갈아버리기 때문에 몇몇 프로퍼티가 null로 업데이트 되는 경우가 있다.
