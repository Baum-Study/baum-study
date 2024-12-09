# JdbcPagingItemReader로 DB내용을 읽고, JdbcBatchItemWriter로 DB에 쓰기

## JdbcPagingItemReader

JdbcPagingItemReader는 스프링 배치에서 제공하는 ItemReader로, 데이터베이스로부터 데이터를 페이지 단위로 읽는다.   
대규모 데이터 처리 효율성을 높이기 위해 메모리 사용량을 최소화하고 커밋 간격을 설정해 대규모 데이터를 효율적으로 처리할 수 있다.   
또한 SQL 쿼리를 직접 작성할 수 있어서 최적화된 데이터 읽기가 가능하고 데이터베이스 커서를 사용해 데이터 순회를 제어할 수 있다.

### 주요 구성 요소

- DataSource: 데이터베이스 연결 정보를 설정한다.
- SqlQuery: 데이터를 읽을 SQL 쿼리를 설정한다.
- RowMapper: SQL 쿼리 결과를 Item으로 변환하는 역할을 한다.
- PageSize: 페이지 크기를 설정한다.
- SkippableItemReader: 오류 발생 시 해당 Item을 건너뛸 수 있도록 한다.
- ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 한다.
- SaveStateCallback: 잡 중단 시 현재 상태를 저장하여 재시작 시 이어서 처리할 수 있도록 한다.

## Query Provider

Query Provider는 JdbcPagingItemReader에서 SQL 쿼리를 생성하는 역할을 한다.  
Query Provider를 통해 커스텀 SQL 쿼리를 생성할 수 있다.

### 주요 구성 요소

- SqlPagingQueryProviderFactoryBean: 쿼리 프로파이더 팩토리
- setDataSource: 데이터소스를 설정한다.
- setSelectClause: select에서 프로젝션할 필드 이름을 지정한다.
- setFromClause: 조회할 테이블
- setWhereClause: 조건절
- setSortKeys: 소트 키를 지정한다.

## JdbcBatchItemWriter

JdbcBatchItemWriter는 스프링 배치에서 제공하는 ItemWriter 인터페이스를 구현하는 구현체이다.  
데이터를 JDBC를 통해 데이터베이스에 저장하는 데 사용된다.  
성능은 대량의 데이터를 빠르게 저장할 수 있고 커스터마이징을 통해 원하는 방식으로 데이터를 저장할 수 있다.  
하지만 JDBC 설정 및 쿼리 작성이 복잡하고 설정 오류 시 데이터 손상 가능성이 있다.

### 주요 구성 요소

- DataSource: 데이터베이스 연결 정보를 지정한다.
- SqlStatementCreator: INSERT 쿼리를 생성하는 역할을 한다.
- PreparedStatementSetter: INSERT 쿼리의 파라미터 값을 설정하는 역할을 한다.
- ItemSqlParameterSourceProvider: Item 객체를 기반으로 PreparedStatementSetter에 전달할 파라미터 값을 생성하는 역할을 한다.