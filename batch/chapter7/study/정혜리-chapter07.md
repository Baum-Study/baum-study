# MyBatisPagingItemReader / MyBatisPagingItemReader

## MyBatisPagingItemReader

spring mybatis 에서 제공하는 ItemReader 구현 클래스

MyBatis의 Object Relation Mapper를 이용

### 장점
- 간편한 설정: MyBatis 쿼리 매퍼를 직접 활용하여 데이터를 읽을 수 있어 설정이 간편
- 쿼리 최적화: MyBatis의 다양한 기능을 활용하여 최적화된 쿼리를 작성 가능
- 동적 쿼리 지원: 런타임 시 조건에 따라 동적으로 쿼리를 생성 가능

### 단점
- MyBatis 라이브러리에 의존함
- Chunk-oriented Processing 방식과 비교했을 때 커스터마이징이 더 복잡할 수 있음

### 구성요소
- SqlSessionFactory
  - MyBatis 설정 정보 및 SQL 쿼리 매퍼 정보를 담고 있는 객체
  - @Bean, Spring Batch XML, Java 코드와 같은 방법으로 설정할 수 있음
- QueryId
  - MyBatis의 쿼리 ID
  - setQueryId()를 통해 실행할 쿼리를 지정
  - 쿼리ID는 "namespace + 작성한 query id" 형식으로 지정
- PageSize
  - MyBatisPagingItemReader는 pageSize로 offset, limit 을 활용할 수 있음
- SkippableItemReader
  - 오류 발생 시 해당 Item을 건너 뛰도록 설정함
- ReadListener
  - 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리 가능
- SaveStateCallback
  - 잡의 중단 시 현재 상태를 저장해 재시작 시 이어서 처리 가능
- ParameterValues
  - Map 타입으로 실행할 쿼리의 파라메터 전달

<br />

## MyBatisItemWriter

Spring Batch에서 제공하는 ItemWriter 인터페이스 구현체

MyBatis를 통해 데이터베이스에 저장하는 데 사용

### 구성 요소
- SqlSessionTemplate
  - MyBatis SqlSession 생성 및 관리를 위한 템플릿 객체
- SqlSessionFactory
  - SqlSessionTemplate 생성을 위한 팩토리 객체
- StatementId
  - 실행할 MyBatis SQL 맵퍼의 Statement ID
- ItemToParameterConverter
  - 객체를 ParameterMap으로 변경할 수 있음

### 장점
- MyBatis를 통해 다양한 데이터베이스에 데이터 저장 가능
- SQL 쿼리를 Java 코드로부터 분리하여 관리 및 유지보수 용이
- 다양한 설정을 통해 원하는 방식으로 데이터 저장 가능

### 단점
- MyBatis 설정 및 SQL 맵퍼 작성이 복잡할 수 있음
- 특정 데이터베이스에 종속적임
- 설정 오류 시 데이터 손상 가능성이 있음