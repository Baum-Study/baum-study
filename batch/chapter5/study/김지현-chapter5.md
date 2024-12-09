## JdbcPagingItemReader 
스프링 배치에서 제공하는 `ItemReader`로, 데이터베이스로부터 데이터를 페이지 단위로 읽는다. <br>
대규모 데이터 처리 효율성: 메모리 사용량을 최소화하고 커밋 간격을 설정해 대규모 데이터를 효율적으로 처리 가능하다. <br>
쿼리 최적화: SQL 쿼리를 직접 작성할 수 있어서 최적화된 데이터 읽기가 가능하다. <br>
커서 제어: 데이터베이스 커서를 사용해 데이터 순회를 제어 가능하다. <br>

### 주요 구성 요소
DataSource: 데이터베이스 연결 정보를 설정한다. <br>
SqlQuery: 데이터를 읽을 SQL 쿼리를 설정한다. <br>
RowMapper: SQL 쿼리 결과를 Item으로 변환하는 역할을 한다. <br>
PageSize: 페이지 크기를 설정한다. <br>
SkippableItemReader: 오류 발생 시 해당 Item을 건너뛸 수 있도록 한다. <br>
ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 한다. <br>
SaveStateCallback: job 중단 시 현재 상태를 저장하여 재시작 시 이어서 처리할 수 있도록 한다. <br>

## JdbcBatchItemWriter 
스프링 배치에서 제공하는 `ItemWriter` 인터페이스를 구현하는 구현체이다. <br>
데이터를 JDBC를 통해 데이터베이스에 저장하는 데 사용된다. <br>
데이터베이스 연동: JDBC를 통해 다양한 데이터베이스에 데이터를 저장 가능하다. <br>
성능: 대량의 데이터 빠르게 저장 가능하다. <br>
유연성: 다양한 설정을 통해 원하는 방식으로 데이터 저장 가능하다. <br>
설정 복잡성: JDBC 설정 및 쿼리 작성이 복잡할 수 있다. <br>
오류 가능성: 설정 오류 시 데이터 손상 가능성이 있다. <br>

## 주요 구성 요소
DataSource: 데이터베이스 연결 정보를 지정한다. <br>
SqlStatementCreator: INSERT 쿼리를 생성하는 역할을 한다. <br>
PreparedStatementSetter: INSERT 쿼리의 파라미터 값을 설정하는 역할을 한다. <br>
ItemSqlParameterSourceProvider: Item 객체를 기반으로 PreparedStatementSetter에 전달할 파라미터 값을 생성하는 역할을 한다. <br>
