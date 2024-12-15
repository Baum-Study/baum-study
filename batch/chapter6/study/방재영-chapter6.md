## JpaPagingItemReader

- Spring Batch에서 제공하는 ItemReader, JPA를 사용하여 데이터베이스로부터 데이터를 페이지 단위로 읽는다.
- JPA 엔티티 기반 데이터 처리, 객체 매핑 자동화 등 JPA의 다양한 기능을 활용할 수 있다.
- JPA 쿼리 기능을 사용하여 최적화된 데이터 읽기가 가능하다.
- JPA Criteria API를 사용하여 데이터 순회를 제어할 수 있다.
- 페이징은 페이지 오프셋이 증가할 때마다 데이터의 변경이 가해지면 누락이 되는 문제가 있다.
- 하지만, 가장 단순하고 큰 데이터를 청크 단위로 처리할 수 있는 좋은 방법이다.

### JpaPagingItemReader 구성 요소

- EntityManagerFactory: JPA EntityManager 생성을 위한 팩토리 객체이다.
- JpaQueryProvider: 데이터를 읽을 JPA 쿼리를 제공한다.
- PageSize: 페이지 크기를 설정한다.
- SkippableItemReader: 오류 발생 시, 해당 Item을 건너뛸 수 있도록 한다.
- ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 한다.
- SaveStateCallback: 잡시 중단 시 현재 상태를 저장하여 재시작 시 이어서 처리할 수 있도록 한다.

## JpaItemWriter

- Spring Batch에서 제공하는 ItemWriter 인터페이스를 구현하는 클래스이다.
- JPA를 통해 데이터를 데이터베이스에 저장하는 데 사용된다.
- 장점
    - ORM 연동: JPA를 통해 다양한 데이터베이스에 데이터를 저장할 수 있다.
    - 객체 매핑: 엔터티 객체를 직접 저장하여 코드 간결성을 높일 수 있다.
    - 유연성: 다양한 설정을 통해 원하는 방식으로 데이터를 저장할 수 있다.
- 단점
    - 설정 복잡성: JPA 설정이 복잡할 수 있다.
    - 오류 가능성: 설정 오류 시 데이터 손상 가능성이 있다.

### JpaItemWriter 구성 요소

- EntityManagerFactory: JPA EntityManager 생성을 위한 팩토리 객체이다.
- JpaQueryProvider: 저장할 엔터티를 위한 JPA 쿼리를 생성하는 역할을 한다.
