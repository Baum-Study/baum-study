## JdbcPagingItemReader

- Spring Batch에서 제공하는 ItemReader로, 데이터베이스로부터 데이터를 페이지 단위로 읽음
- 대규모 데이터 처리 효율성: 메모리 사용량을 최소화하고 커밋 간격을 설정하여 대규모 데이터를 효율적으로 처리 가능
- 쿼리 최적화: SQL 쿼리를 직접 작성하여 최적화된 데이터 읽기 가능
- 커서 제어: 데이터베이스 커서를 사용하여 데이터 순회를 제어 가능

### 주요 구성 요소

- DataSource: 데이터베이스 연결 정보 설정
- SqlQuery: 데이터를 읽을 SQL 쿼리 설정
- RowMapper: SQL 쿼리 결과를 Item으로 변환 역할
- PageSize: 페이지 크기 설정
- SkippableItemReader: 오류 발생 시, 해당 Item을 건너뛸 수 있음
- ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리 가능
- SaveStateCallback: Job 중단 시 현재 상태를 저장하여 재시작 시 이어서 처리할 수 있음

<br/>

---

## JdbcBatchItemWriter

- Spring Batch에서 제공하는 ItemWriter 인터페이스를 구현하는 클래스
- JDBC를 통해 데이터를 데이터베이스에 저장하는 데 사용
- 장점
    - 데이터베이스 연동: JDBC를 통해 데이터를 다양한 데이터베이스에 저장 가능
    - 성능: 대량의 데이터를 빠르게 저장 가능
    - 유연성: 다양한 설정을 통해 원하는 방식으로 데이터 저장 가능
- 단점
    - 설정 복잡성: JDBC 설정 및 쿼리 작성 복잡할 수 있음
    - 오류 가능성: 설정 오류 시 데이터 손상 가능성이 있음

### 주요 구성 요소

- DataSource: 데이터베이스 연결 정보 설정
- SqlStatementCreator: INSERT 쿼리 생성 역할
- PreparedStatementSetter: INSERT 쿼리의 파라미터 값 설정 역할
- ItemSqlParameterSourceProvider: Item 객체를 기반으로 PreparedStatementSetter에 전달할 파라미터 값 생성 역할