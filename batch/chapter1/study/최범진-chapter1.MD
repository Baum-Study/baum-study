## 배치 기본 설정

```java
/**
 * - @EnableBatchProcessing Spring Batch를 사용하기 위한 설정을 추가한다.
 */
@EnableBatchProcessing 
@SpringBootApplication
public class BatchApplication {

  public static void main(String[] args) {
    SpringApplication.run(BatchApplication.class, args);
  }

}
```

## 3.0 이후

> 이전에는 @EnableBatchProcessing을 사용하여 Spring Boot의 Spring Batch 자동 구성을 활성화할 수 있었습니다. Spring Boot v3.0부터는 이 어노테이션이 더 이상 필요하지 않으므로 Spring Boot의 자동 구성을 사용하려는 애플리케이션에서 이 어노테이션을 제거해야 합니다. 이제 @EnableBatchProcessing으로 주석이 지정되거나 Spring Batch의 DefaultBatchConfiguration을 확장하는 빈을 정의하여 자동 구성을 중단하도록 지시할 수 있으므로 애플리케이션이 Spring Batch의 구성 방식을 완전히 제어할 수 있습니다.
https://spring.io/guides/gs/batch-processing
> 

## 스프링배치 스키마 구조

### BATCH_JOB_INSTANCE 테이블

BATCH_JOB_INSTANCE 테이블은 **JobInstance**와 관련된 모든 정보를 저장하며, 전체 계층 구조의 최상위 역할을 합니다. 

```sql
CREATE TABLE BATCH_JOB_INSTANCE  (
  JOB_INSTANCE_ID BIGINT PRIMARY KEY,
  VERSION BIGINT,
  JOB_NAME VARCHAR(100) NOT NULL,
  JOB_KEY VARCHAR(32) NOT NULL
);
```

- **JOB_INSTANCE_ID**: 인스턴스를 식별하는 고유 ID로, 기본 키 역할을 합니다. 이 열의 값은 **JobInstance**의 `getId` 메서드를 호출하여 얻을 수 있습니다.
- **VERSION**: 버전에 대한 정보입니다.
- **JOB_NAME**: Job 객체에서 가져온 작업 이름입니다. 인스턴스를 식별하는 데 필요하므로 **null** 값이 될 수 없습니다.
- **JOB_KEY**: 동일한 작업의 각 인스턴스를 고유하게 식별하기 위해 **JobParameters**를 직렬화한 값입니다. 동일한 작업 이름을 가진 JobInstance는 각기 다른 JobParameters 값을 가져야 하며, 이에 따라 JOB_KEY 값도 달라야 합니다.

### BATCH_JOB_EXECUTION_PARAMS 테이블

BATCH_JOB_EXECUTION_PARAMS 테이블은 **JobParameters** 객체와 관련된 모든 정보를 저장합니다. 이는 0개 이상의 키/값 쌍을 포함하여 Job에 전달되며, 해당 Job이 실행된 매개변수들을 기록합니다. 각 매개변수가 Job의 ID 생성에 기여하는 경우, **IDENTIFYING** 플래그가 **true**로 설정됩니다. 이 테이블은 정규화되지 않았으며, 매개변수 유형을 나타내는 컬럼이 하나로 통합되었습니다.

```sql
CREATE TABLE BATCH_JOB_EXECUTION_PARAMS  (
	JOB_EXECUTION_ID BIGINT NOT NULL,
	PARAMETER_NAME VARCHAR(100) NOT NULL,
	PARAMETER_TYPE VARCHAR(100) NOT NULL,
	PARAMETER_VALUE VARCHAR(2500),
	IDENTIFYING CHAR(1) NOT NULL,
	constraint JOB_EXEC_PARAMS_FK foreign key (JOB_EXECUTION_ID)
	references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
);
```

- **JOB_EXECUTION_ID**: BATCH_JOB_EXECUTION 테이블의 외래 키로, 해당 매개변수 항목이 속한 작업 실행을 나타냅니다.
- **PARAMETER_NAME**: 매개변수 이름.
- **PARAMETER_TYPE**: 매개변수의 유형을 나타내는 완전한 이름.
- **PARAMETER_VALUE**: 매개변수 값.
- **IDENTIFYING**: 해당 매개변수가 관련된 JobInstance의 ID 생성에 기여했는지를 나타내는 플래그.

### BATCH_JOB_EXECUTION 테이블

BATCH_JOB_EXECUTION 테이블은 **JobExecution** 객체와 관련된 모든 정보를 저장합니다. Job이 실행될 때마다 새로운 JobExecution이 생성되며 테이블에 새로운 행이 추가됩니다.

```sql
CREATE TABLE BATCH_JOB_EXECUTION  (
  JOB_EXECUTION_ID BIGINT PRIMARY KEY,
  VERSION BIGINT,
  JOB_INSTANCE_ID BIGINT NOT NULL,
  CREATE_TIME TIMESTAMP NOT NULL,
  START_TIME TIMESTAMP DEFAULT NULL,
  END_TIME TIMESTAMP DEFAULT NULL,
  STATUS VARCHAR(10),
  EXIT_CODE VARCHAR(20),
  EXIT_MESSAGE VARCHAR(2500),
  LAST_UPDATED TIMESTAMP,
  constraint JOB_INSTANCE_EXECUTION_FK foreign key (JOB_INSTANCE_ID)
  references BATCH_JOB_INSTANCE(JOB_INSTANCE_ID)
);
```

- **JOB_EXECUTION_ID**: 고유한 실행을 식별하는 기본 키.
- **VERSION**: 버전.
- **JOB_INSTANCE_ID**: BATCH_JOB_INSTANCE 테이블의 외래 키로, 실행이 속한 인스턴스를 나타냅니다.
- **CREATE_TIME**: 실행이 생성된 시간.
- **START_TIME**: 실행이 시작된 시간.
- **END_TIME**: 실행이 종료된 시간. 비어 있는 경우 오류를 의미.
- **STATUS**: 실행 상태. (예: COMPLETED, STARTED).
- **EXIT_CODE**: 실행 종료 코드.
- **EXIT_MESSAGE**: 실행 종료에 대한 상세 메시지.
- **LAST_UPDATED**: 마지막으로 갱신된 시간.

### BATCH_STEP_EXECUTION 테이블

BATCH_STEP_EXECUTION 테이블은 **StepExecution** 객체와 관련된 모든 정보를 저장합니다. 각 JobExecution에 대해 적어도 하나의 StepExecution 항목이 생성됩니다.

```sql
CREATE TABLE BATCH_STEP_EXECUTION  (
  STEP_EXECUTION_ID BIGINT NOT NULL PRIMARY KEY,
  VERSION BIGINT NOT NULL,
  STEP_NAME VARCHAR(100) NOT NULL,
  JOB_EXECUTION_ID BIGINT NOT NULL,
  CREATE_TIME TIMESTAMP NOT NULL,
  START_TIME TIMESTAMP DEFAULT NULL,
  END_TIME TIMESTAMP DEFAULT NULL,
  STATUS VARCHAR(10),
  COMMIT_COUNT BIGINT,
  READ_COUNT BIGINT,
  FILTER_COUNT BIGINT,
  WRITE_COUNT BIGINT,
  READ_SKIP_COUNT BIGINT,
  WRITE_SKIP_COUNT BIGINT,
  PROCESS_SKIP_COUNT BIGINT,
  ROLLBACK_COUNT BIGINT,
  EXIT_CODE VARCHAR(20),
  EXIT_MESSAGE VARCHAR(2500),
  LAST_UPDATED TIMESTAMP,
  constraint JOB_EXECUTION_STEP_FK foreign key (JOB_EXECUTION_ID)
  references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
);
```

- **STEP_EXECUTION_ID**: 고유한 실행을 식별하는 기본 키.
- **VERSION**: 버전.
- **STEP_NAME**: 해당 실행이 속한 스텝의 이름.
- **JOB_EXECUTION_ID**: BATCH_JOB_EXECUTION 테이블의 외래 키.
- **START_TIME**: 실행 시작 시간.
- **END_TIME**: 실행 종료 시간.
- **STATUS**: 실행 상태.
- **COMMIT_COUNT**: 커밋된 횟수.
- **READ_COUNT**: 읽은 항목 수.
- **FILTER_COUNT**: 필터링된 항목 수.
- **WRITE_COUNT**: 작성 및 커밋된 항목 수.
- **READ_SKIP_COUNT**: 읽기에서 건너뛴 항목 수.
- **WRITE_SKIP_COUNT**: 작성에서 건너뛴 항목 수.
- **PROCESS_SKIP_COUNT**: 처리 중 건너뛴 항목 수.
- **ROLLBACK_COUNT**: 롤백된 횟수.
- **EXIT_CODE**: 실행 종료 코드.
- **EXIT_MESSAGE**: 실행 종료에 대한 상세 메시지.
- **LAST_UPDATED**: 마지막 갱신 시간.

### BATCH_JOB_EXECUTION_CONTEXT 테이블

BATCH_JOB_EXECUTION_CONTEXT 테이블은 Job의 **ExecutionContext**와 관련된 모든 정보를 저장합니다. 각 JobExecution에 대해 하나의 Job ExecutionContext가 존재하며, 실패 후에 JobInstance가 중단된 위치에서 다시 시작할 수 있도록 하는 상태를 저장합니다.

```sql
CREATE TABLE BATCH_JOB_EXECUTION_CONTEXT  (
  JOB_EXECUTION_ID BIGINT PRIMARY KEY,
  SHORT_CONTEXT VARCHAR(2500) NOT NULL,
  SERIALIZED_CONTEXT CLOB,
  constraint JOB_EXEC_CTX_FK foreign key (JOB_EXECUTION_ID)
  references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
);
```

- **JOB_EXECUTION_ID**: JobExecution을 나타내는 외래 키.
- **SHORT_CONTEXT**: SERIALIZED_CONTEXT의 문자열 버전.
- **SERIALIZED_CONTEXT**: 직렬화된 전체 컨텍스트.

### BATCH_STEP_EXECUTION_CONTEXT 테이블

BATCH_STEP_EXECUTION_CONTEXT 테이블은 Step의 **ExecutionContext**와 관련된 모든 정보를 저장합니다. 각 StepExecution에 대해 하나의 ExecutionContext가 존재하며, 실패 후에 JobInstance가 중단된 위치에서 다시 시작할 수 있도록 하는 상태를 저장합니다.

```sql
CREATE TABLE BATCH_STEP_EXECUTION_CONTEXT  (
  STEP_EXECUTION_ID BIGINT PRIMARY KEY,
  SHORT_CONTEXT VARCHAR(2500) NOT NULL,
  SERIALIZED_CONTEXT CLOB,
  constraint STEP_EXEC_CTX_FK foreign key (STEP_EXECUTION_ID)
  references BATCH_STEP_EXECUTION(STEP_EXECUTION_ID)
);
```

- **STEP_EXECUTION_ID**: StepExecution을 나타내는 외래 키.
- **SHORT_CONTEXT**: SERIALIZED_CONTEXT의 문자열 버전.
- **SERIALIZED_CONTEXT**: 직렬화된 전체 컨텍스트.

### Identity (ID) 관리

**BATCH_JOB_INSTANCE**, **BATCH_JOB_EXECUTION**, **BATCH_STEP_EXECUTION** 테이블에는 각각의 ID 열이 있으며, 이 열들은 해당 테이블의 기본 키로 작동합니다. 그러나 이러한 ID 필드는 데이터베이스에서 자동 생성되는 키가 아니라 별도의 시퀀스를 통해 생성됩니다. 이는 도메인 객체를 데이터베이스에 삽입한 후, 해당 키가 Java 객체에 설정되어야 고유하게 식별할 수 있기 때문입니다. 최신 데이터베이스 드라이버(JDBC 3.0 이상)에서는 데이터베이스 생성 키를 지원하지만, 이를 필수 기능으로 요구하지 않고 시퀀스를 사용합니다. 각 스키마 변형에는 다음과 같은 시퀀스 생성 구문이 포함됩니다:

```sql
CREATE SEQUENCE BATCH_STEP_EXECUTION_SEQ;
CREATE SEQUENCE BATCH_JOB_EXECUTION_SEQ;
CREATE SEQUENCE BATCH_JOB_SEQ;
```

### 시퀀스를 지원하지 않는 데이터베이스에서의 처리

많은 데이터베이스 벤더는 시퀀스를 지원하지 않기 때문에, MySQL과 같은 시스템에서는 다음과 같은 대체 방식을 사용합니다:

```sql
CREATE TABLE BATCH_STEP_EXECUTION_SEQ (ID BIGINT NOT NULL) type=InnoDB;
INSERT INTO BATCH_STEP_EXECUTION_SEQ values(0);
CREATE TABLE BATCH_JOB_EXECUTION_SEQ (ID BIGINT NOT NULL) type=InnoDB;
INSERT INTO BATCH_JOB_EXECUTION_SEQ values(0);
CREATE TABLE BATCH_JOB_SEQ (ID BIGINT NOT NULL) type=InnoDB;
INSERT INTO BATCH_JOB_SEQ values(0);
```

위의 방식에서는 각 시퀀스를 대신하여 테이블을 사용합니다. 이 경우 **Spring**의 핵심 클래스인 **MySQLMaxValueIncrementer**가 이러한 시퀀스 테이블의 단일 열을 증가시켜 시퀀스와 유사한 기능을 제공하게 됩니다.
