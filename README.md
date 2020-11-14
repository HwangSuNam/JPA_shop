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

# API 개발 고급 - 지연 로딩과 조회 성능 최적화
### 1. 간단한 주문 조회 V1 : 엔티티를 직접 노출
    // 1. json 변환할 때(jackson), 무한 루프 발생, 양방향 연관관계
    // 해결 방법 : 양방향 연관관계 중, 하나를 JsonIgnore 처리
    // 2. json 변환할 때(jackson), lazy로 설정된 필드가 본래 클래스로 초기화 된 것이 아닌, 프록시 클래스라 문제 발생
    // 해결 방법 : Hibernate5Module 모듈 등록(지연 로딩 설정된 필드는 무시하는 기능이며 null로 셋팅됨, FORCE_LAZY_LOADING 옵션도 존재)
    // 엔티티를 그대로 리턴하는 것도 엔티티 정보를 전부 노출하는 것이기 때문에 문제가 됨. DTO로 변환해서 리턴하자.
    @GetMapping("/api/v1/simple-orders")
    public List<Order> odersV1() {
        // JPQL은 SQL로 그대로 번역되기 때문에 EAGER로 설정해도 성능 최적화가 되지 않음(em.find로 실행해야 eager로 한 번에 끌고 올 수 있는데..)
        List<Order> all = orderRepository.findAllByString(new OrderSearch());

        // 루프 주석 처리하고 EAGER로 설정하면 되긴 하지만 그렇게 하면 안된다!
        // 다른 경우에 즉시 로딩으로 연관관계 데이터가 필요 없는 경우에도 쿼리가 수행되는 성능 문제가 발생할 수 있다.
        // 성능 튜닝이 매우 어려워지므로 LAZY를 항상 기본으로 설정하고 필요한 경우, 성능 최적화가 필요한 경우, fetch join을 사용한다.
        for (Order order : all) {
            order.getMember().getAddress();
            order.getDelivery().getAddress();
        }

        return all;
    }
