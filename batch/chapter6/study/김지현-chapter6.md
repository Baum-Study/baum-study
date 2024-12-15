## JpaPagingItemReader
스프링 배치에서 제공하는 `ItemReader`로, JPA를 사용하여 데이터베이스로부터 데이터를 페이지 단위로 읽는다. <br>
JPA 기능 활용: JPA Entity 기반 데이터 처리, 객체 매핑 자동화 등 기존 JPA의 다양한 기능을 사용할 수 있다. <br>
쿼리 최적화: JPA 쿼리 기능을 사용해 최적화된 데이터 읽기가 가능하다. <br>
커서 제어: JPA Criteria API를 사용해 데이터 순회를 제어할 수 있다. <br>

### 주요 구성 요소
EntityManagerFactory: JPA 엔티티 매니저 팩토리를 설정한다. <br>
JpaQueryProvider: 데이터를 읽을 JPA 쿼리를 제공한다. <br>
PageSize: 페이지 크기를 설정한다. <br>
SkippableItemReader: 오류 발생 시 해당 Item을 건너뛸 수 있도록 한다. <br>
ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 한다. <br>
SaveStateCallback: job 중단 시 현재 상태를 저장하여 재시작 시 이어서 처리할 수 있도록 한다. <br>

## JpaItemWriter 
스프링 배치에서 제공하는 `ItemWriter` 인터페이스를 구현하는 구현체이다. <br>
데이터를 JPA를 통해 데이터베이스에 저장하는 데 사용된다. <br>
ORM 연동: JPA를 통해 다양한 데이터베이스에 데이터를 저장할 수 있다. <br>
객체 매핑: 엔티티 객체를 직접 저장해 코드 간결성을 높일 수 있다. <br>
유연성: 다양한 설정을 통해 원하는 방식으로 데이터를 저장할 수 있다. <br>
설정 복잡성: JPA 설정이 복잡할 수 있다. <br>
오류 가능성: 설정 오류 시 데이터 손상 가능성이 있다. <br>

## 주요 구성 요소
EntityManagerFactory: JPA EntityManager 생성을 위한 팩토리 객체 <br>
JpaQueryProvider: INSERT 쿼리를 생성하는 역할을 한다. <br>
