# JpaPagingItemReader로 DB내용을 읽고, JpaItemWriter로 DB에 쓰기

## JpaPagingItemReader

JpaPagingItemReader도 이전 챕터에서 공부했던 JdbcPagingItemReader와 마찬가지로 Spring Batch에서 지원해주는 데이터베이스로부터 데이터를 페이지 단위로 읽는 ItemReader이다.  
JPA에서 지원해 주는 JPA 엔티티 기반 데이터 처리, 객체 매핑 자동화 등 다양한 기능을 활용할 수 있다.  
JPA Criteria API를 사용하여 데이터 순회를 제어할 수 있다.  

- EntityManagerFactory: JPA 엔티티 매니저 팩토리를 설정한다.
- JpaQueryProvider: 데이터를 읽을 JPA 쿼리를 제공한다.
- PageSize: 페이지 크기를 설정한다.
- SkippableItemReader: 오류 발생 시 해당 Item을 건너뛸 수 있도록 한다.
- ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 한다.
- SaveStateCallback: 잡시 중단 시 현재 상태를 저장하여 재시작 시 이어서 처리할 수 있도록 한다.

실습 코드는 여기에 첨부하진 않지만 jdbc보다 편리하고 단순한 문법, 큰 데이터를 가독성있게 처리할 수 사용할 수 있다.  
하지만 페이징은 페이지 오프셋이 증가할때마다 데이터의 변경이 가해지면 누락이 되는 문제가 있다던가 성능이 jdbc보다 10%정도 떨어진다는 단점이 있다.  
  

## JpaItemWriter

JpaItemWriter는 JpaPagingItemReader와 마찬가지로 Spring Batch에서 제공하는 ItemWriter 인터페이스를 구현하는 구현체이다.

JPA 설정을 통해 다양한 데이터베이스에 엔터티 객체 형식으로 데이터를 저장할 수 있다.
하지만 JPA 설정이 생각보다 복잡하고 설정 오류 시 데이터 손상 가능성이 있다.

- EntityManagerFactory: JPA EntityManager 생성을 위한 팩토리 객체
- JpaQueryProvider: 저장할 엔터티를 위한 JPA 쿼리를 생성하는 역할

## 결론
내 생각으로는 별도의 Batch Server를 구축할때는 JDBC를 사용하고, 이미 jpa를 사용하고 있는 모놀리틱 서버에 Batch를 추가할때는 JPA를 사용하는게 좋을 것 같다.