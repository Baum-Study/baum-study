# JpaPagingItemReader, JpaItemWriter

## JpaPagingItemReader
- Spring Batch 에서 제공하는 ItemReader 중 하나로, JPA를 활용해 데이터를 페이지 단위로 읽음
- JPA 기능 활용하므로 엔티티기반 데이터 처리와 객체 매핑 자동화가 가능하다
- JPA 쿼리 기능을 활용해 최적화된 데이터 읽기가 가능하다
- JPA Criteria API를 활용해 데이터 순회를 제어할 수 있다

### 구성 요소
- EntityManagerFactory: JPA 엔티티 매니저 팩토리를 설정한다
- JpaQueryProvider: 데이터를 읽을 JPA 쿼리를 제공한다
- PageSize: 페이지 크기를 지정한다
- SkippableItemReader: 오류 발생 시 해당 Item을 건너 뛸 수 있다
- ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있다
- SaveStateCallback: 잠시 중단 시 상태를 저장해 재시작 시 이어서 처리 가능하다

> [!WARNING]
> 페이징의 특성으로 인해, 데이터의 변경이 발생하는 경우 누락이 발생하는 문제가 있다.

<br />

## JpaItemWriter
- Spring Batch에서 제공하는 ItemWriter 구현 클래스이다
- 데이터를 JPA를 통해 DB에 저장하는 데 사용한다

### 구성요소
- EntityManagerFactory: JPA EntityManager 생성을 위한 팩토리 객체
- JpaQueryProvider: 저장할 엔티티를 위한 JPA 쿼리 생성 역할

### 장단점
- 장점
  - ORM 연동: JPA를 통해 다양한 DB에 데이터 저장 가능
  - 객체 매핑: 엔티티 객체를 직접 저장해 코드 간결성을 높인다
  - 유연성: 다양한 설정을 통해 원하는 방식으로 데이터 저장 가능
- 단점
  - 설정 복잡성: JPA 설정이 복잡할 수 있음
  - 오류 가능성: 설정 오류 시 데이터 손상 가능성이 있음