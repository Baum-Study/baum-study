# MyBatisItemReader

- Spring Mybatis에서 제공하는 ItemReader 인터페이스를 구현하는 클래스이다.
- MyBatis의 Object Relation Mapper를 이용하며 다음과 같은 특징을 가지고 있다.

## 장점

- 간편한 설정: MyBatis 쿼리 매퍼를 직접 활용하여 데이터를 읽을 수 있어 설정이 간편하다.
- 쿼리 최적화: MyBatis의 다양한 기능을 활용하여 최적화된 쿼리를 작성할 수 있다.
- 동적 쿼리 지원: 런타임 시 조건에 따라 동적으로 쿼리를 생성할 수 있다.

## 단점

- MyBatis 의존성: MyBatis 라이브러리에 의존해야 한다.
- 커스터마이징 복잡: Chunk-oriented Processing 방식과 비교했을 때 커스터마이징이 더 복잡할 수 있다.

## **구성 요소**

- SqlSessionFactory: MyBatis 설정 정보 및 SQL 쿼리 매퍼 정보를 담고 있는 객체이다.
- QueryId: 데이터를 읽을 MyBatis 쿼리 ID이다.
- PageSize: 페이징 쿼리를 위한 페이지 크기를 지정한다.

### **SqlSessionFactory:**

- MyBatisPagingItemReader SqlSessionFactory 객체를 통해 MyBatis와 연동된다.
- SqlSessionFactory는 다음과 같은 방법으로 설정할 수 있다.
- @Bean 어노테이션을 사용하여 직접 생성
- Spring Batch XML 설정 파일에서 설정
- Java 코드에서 직접 설정

### **QueryId:**

- MyBatisPagingItemReader setQueryId() 메소드를 통해 데이터를 읽을 MyBatis 쿼리 ID를 설정한다.
- 쿼리 ID는 com.example.mapper.CustomerMapper.selectCustomers 와 같은 형식으로 지정된다.

### **PageSize:**

- MyBatisItemReader는 pageSize를 이용하여 offset, limit 을 이용하는 기준을 설정할 수 있다.

### **추가 구성 요소:**

- SkippableItemReader: 오류 발생 시 해당 Item을 건너뛸 수 있도록 한다.
- ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 한다.
- SaveStateCallback: 잡의 중단 시 현재 상태를 저장하여 재시작 시 이어서 처리할 수 있도록 한다.

# **MyBatisItemWriter**

- Spring Batch에서 제공하는 ItemWriter 인터페이스를 구현하는 클래스이다.
- 데이터를 MyBatis를 통해 데이터베이스에 저장하는 데 사용된다.

### **구성 요소**

- SqlSessionTemplate: MyBatis SqlSession 생성 및 관리를 위한 템플릿 객체이다.
- SqlSessionFactory: SqlSessionTemplate 생성을 위한 팩토리 객체이다.
- StatementId: 실행할 MyBatis SQL 맵퍼의 스테이tement ID이다.
- ItemToParameterConverter: 객체를 ParameterMap으로 변경할수 있다.

### **장점**

- ORM 연동: MyBatis를 통해 다양한 데이터베이스에 데이터를 저장할 수 있다.
- SQL 쿼리 분리: SQL 쿼리를 Java 코드로부터 분리하여 관리 및 유지 보수가 용이하다.
- 유연성: 다양한 설정을 통해 원하는 방식으로 데이터를 저장할 수 있다.

### **단점**

- 설정 복잡성: MyBatis 설정 및 SQL 맵퍼 작성이 복잡할 수 있다.
- 데이터베이스 종속: 특정 데이터베이스에 종속적이다.
- 오류 가능성: 설정 오류 시 데이터 손상 가능성이 있다.
