## MyBatisItemReader

- ItemReader 인터페이스 구현 클래스, MyBatis의 Object Relation Mapper를 이용
- 장점
    - MyBatis 쿼리 매퍼를 직접 활용한 데이터 읽기 가능하여 설정이 간편
    - MyBatis의 다양한 기능을 활용하여 최적화된 쿼리 작성 가능
    - 런타임 시, 조건에 따라 동적으로 쿼리 생성 가능
- 단점
    - MyBatis 라이브러리에 의존
    - Chunk-oriented Processing 방식과 비교했을 때, 커스터마이징이 더 복잡할 수 있음

<br/>

### 구성 요소

- SqlSessionFactory
    - MyBatis 설정 정보 및 SQL 쿼리 매퍼 정보를 담고 있는 객체
    - MyBatisPagingItemReader SqlSessionFactory 객체를 통해 MyBatis와 연동
    - 설정 방법
        - @Bean 어노테이션을 사용하여 직접 생성
        - Spring Batch XML 설정 파일에서 설정
        - Java 코드에서 직접 설정
- QueryId
    - 데이터를 읽을 MyBatis 쿼리 ID,
    - MyBatisPagingItemReader setQueryId() 메소드를 통해 데이터를 읽을 MyBatis 쿼리 ID 설정
    - 쿼리 ID는 com.example.mapper.CustomerMapper.selectCustomers와 같은 형식으로 지정
- PageSize
    - 페이징 쿼리를 위한 페이지 크기 지정
    - pageSize를 이용하여 offset, limit을 이용하는 기준 설정 가능

<br/>

---

## MyBatisItemWriter

- ItemWriter 인터페이스 구현 클래스, MyBatis를 통해 데이터를 데이터베이스에 저장하는 데에 사용
- 장점
    - MyBatis를 통해 다양한 데이터베이스에 데이터 저장 가능
    - SQL 쿼리를 Java 코드로부터 분리하여 관리 및 유지 보수 용이
    - 다양한 설정을 통해 원하는 방식으로 데이터 저장 가능
- 단점
    - MyBatis 설정 및 SQL 맵퍼 작성이 복잡할 수 있음
    - 특정 데이터베이스에 종속적
    - 설정 오류 시, 데이터 손상 가능성

<br/>

### 구성 요소

- SqlSessionTemplate: MyBatis SqlSession 생성 및 관리를 위한 템플릿 객체
- SqlSessionFactory: SqlSessionTemplate 생성을 위한 팩토리 객체
- StatementId: 실행할 MyBatis SQL 맵퍼의 Statement ID
- ItemToParameterConverter: 객체를 ParameterMap으로 변경할 수 있음
