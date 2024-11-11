## 📌 Spring Batch

- **대용량 데이터**를 처리할 수 있도록 설계된 프레임워크로, DB나 파일에서 데이터를 읽고 가공하여 저장하는 작업을 효율적으로 수행할 수 있다.
- 스케줄러와 연동하여 특정 시간에 자동으로 실행할 수 있어, 주기적으로 반복되는 작업(ex. 월말 정산)을 효율적으로 처리할 수 있다.
- 트랜잭션 관리, 재시도, 스킵, 재시작 등의 기능을 제공한다.

<br/>

## 📌 Spring Batch 핵심 용어

- Job: 전체 배치 처리 과정을 추상화한 개념으로, Spring Batch 계층에서 최상위에 위치한다.
- JobInstance: 특정 Job의 실제 실행 인스턴스를 의미하며, 매일 처리되는 Job을 구성한다고 가정하면 매일 실행될 때마다 새로운 JobInstance가 생성된다.
- JobParameters: Job이 실행될 때 필요한 파라미터를 제공하며, JobInstance를 구별하는 역할을 한다. Spring Batch는 4가지 타입(String, Double, Long, Date)의
  파라미터를 지원한다.
- JobExecution: JobInstance 한 번의 실행 시도를 나타낸다.
- Step: Job의 하위 단계로, 실제 배치 처리 작업이 이루어지는 단위이다. 한 개 이상의 Step으로 Job이 구성되고 각 Step은 순차적으로 처리된다.
- StepExecution, ExecutionContext, JobRepository, ...

<br/>

## 📌 Spring Batch 스키마 구조

![img.png](../image/방재영.png)

- Spring Batch는 Job과 Step의 실행 상태를 관리하기 위해 메타데이터 스키마를 사용한다.
- 이 스키마는 배치 작업의 상태 추적, 재시작, 트랜잭션 관리 등을 지원한다.

### [ 테이블 목록 ]

1. **BATCH_JOB_INSTANCE**
2. **BATCH_JOB_EXECUTION_PARAMS**
3. **BATCH_JOB_EXECUTION**
4. **BATCH_STEP_EXECUTION**
5. **BATCH_JOB_EXECUTION_CONTEXT**
6. **BATCH_STEP_EXECUTION_CONTEXT**

---

### 1. BATCH_JOB_INSTANCE

**Job 인스턴스 정보**를 저장하는 테이블이다.

| 컬럼명               | 설명                      |
|-------------------|-------------------------|
| `JOB_INSTANCE_ID` | Job 인스턴스의 고유 ID (PK)    |
| `VERSION`         | 버전 정보                   |
| `JOB_NAME`        | Job의 이름                 |
| `JOB_KEY`         | Job Parameter로 생성된 고유 키 |

---

### 2. BATCH_JOB_EXECUTION_PARAMS

Job 실행 시 사용된 **파라미터 정보**를 저장한다.

| 컬럼명                | 설명                |
|--------------------|-------------------|
| `JOB_EXECUTION_ID` | Job 실행 ID (FK)    |
| `PARAMETER_NAME`   | 파라미터 이름           |
| `PARAMETER_TYPE`   | 파라미터 타입           |
| `PARAMETER_VALUE`  | 파라미터 값            |
| `IDENTIFYING`      | Job의 동일성 체크를 위한 값 |

---

### 3. BATCH_JOB_EXECUTION

Job의 **실행 정보**를 저장한다.

| 컬럼명                | 설명                                            |
|--------------------|-----------------------------------------------|
| `JOB_EXECUTION_ID` | JobExecution의 고유 ID (PK)                      |
| `JOB_INSTANCE_ID`  | Job 인스턴스 ID (FK)                              |
| `VERSION`          | 버전 정보                                         |
| `CREATE_TIME`      | JobExecution 생성 시간                            |
| `START_TIME`       | JobExecution 시작 시간                            |
| `END_TIME`         | JobExecution 종료 시간                            |
| `STATUS`           | JobExecution 실행 상태 (`COMPLETED`, `STARTED` 등) |
| `EXIT_CODE`        | JobExecution 종료 코드                            |
| `EXIT_MESSAGE`     | JobExecution 종료 메시지                           |
| `LAST_UPDATED`     | JobExecution 수정된 시간                           |

---

### 4. BATCH_STEP_EXECUTION

각 Step의 **실행 정보**를 저장한다.

| 컬럼명                  | 설명                                             |
|----------------------|------------------------------------------------|
| `STEP_EXECUTION_ID`  | StepExecution의 고유 ID (PK)                      |
| `JOB_EXECUTION_ID`   | JobExecution ID (FK)                           |
| `VERSION`            | 버전 정보                                          |
| `STEP_NAME`          | 실행된 StepExecution의 Step의 이름                    |
| `START_TIME`         | StepExecution 시작 시간                            |
| `END_TIME`           | StepExecution 종료 시간                            |
| `STATUS`             | StepExecution 실행 상태 (`COMPLETED`, `STARTED` 등) |
| `COMMIT_COUNT`       | StepExecution 동안 커밋한 횟수                        |
| `READ_COUNT`         | StepExecution 동안 읽기 아이템 수                      |
| `WRITE_COUNT`        | StepExecution 동안 쓰기 아이템 수                      |
| `FILTER_COUNT`       | StepExecution 동안 팔터링된 아이템 수                    |
| `READ_SKIP_COUNT`    | StepExecution 동안 읽기 스킵된 아이템 수                  |
| `WRITE_SKIP_COUNT`   | StepExecution 동안 쓰기 스킵된 아이템 수                  |
| `PROCESS_SKIP_COUNT` | StepExecution 동안 프로세스가 스킵된 아이템 수               |
| `ROLLBACK_COUNT`     | StepExecution 동안 롤백된 아이템 수                     |
| `EXIT_CODE`          | StepExecution 종료 코드                            |
| `EXIT_MESSAGE`       | StepExecution 종료 메시지                           |
| `LAST_UPDATED`       | StepExecution 수정된 시간                           |

---

### 5. BATCH_JOB_EXECUTION_CONTEXT

JobExecution의 **컨텍스트 정보**를 저장한다.

| 컬럼명                  | 설명             |
|----------------------|----------------|
| `JOB_EXECUTION_ID`   | Job 실행 ID (FK) |
| `SHORT_CONTEXT`      | 간단한 컨텍스트 정보    |
| `SERIALIZED_CONTEXT` | 직렬화된 컨텍스트 데이터  |

---

### 6. BATCH_STEP_EXECUTION_CONTEXT

StepExecution의 **컨텍스트 정보**를 저장한다.

| 컬럼명                  | 설명              |
|----------------------|-----------------|
| `STEP_EXECUTION_ID`  | Step 실행 ID (FK) |
| `SHORT_CONTEXT`      | 간단한 컨텍스트 정보     |
| `SERIALIZED_CONTEXT` | 직렬화된 컨텍스트 데이터   |

---

<br/>

## 📌 Spring Batch 시퀀스 테이블 구조

Spring Batch는 **Job**, **Job Execution**, **Step Execution**의 고유 ID 생성을 위해 시퀀스 테이블을 사용한다. 이를 통해 ID의 일관성을 유지하고, 충돌을 방지할 수
있다.

### 1. BATCH_JOB_SEQ

- **Job 인스턴스**의 고유 ID 생성을 담당한다.

| 컬럼명          | 타입      | 설명                         |
|--------------|---------|----------------------------|
| `ID`         | BIGINT  | Job 인스턴스의 고유 ID 값          |
| `UNIQUE_KEY` | CHAR(1) | 시퀀스를 구분하는 유니크 키 (기본값: ' ') |

---

### 2. BATCH_JOB_EXECUTION_SEQ

- **Job Execution**의 고유 ID 생성을 담당한다.

| 컬럼명          | 타입      | 설명                         |
|--------------|---------|----------------------------|
| `ID`         | BIGINT  | Job Execution의 고유 ID 값     |
| `UNIQUE_KEY` | CHAR(1) | 시퀀스를 구분하는 유니크 키 (기본값: ' ') |

---

### 3. BATCH_STEP_EXECUTION_SEQ

- **Step Execution**의 고유 ID 생성을 담당한다.

| 컬럼명          | 타입      | 설명                         |
|--------------|---------|----------------------------|
| `ID`         | BIGINT  | Step Execution의 고유 ID 값    |
| `UNIQUE_KEY` | CHAR(1) | 시퀀스를 구분하는 유니크 키 (기본값: ' ') |

---

### Spring Batch에서 시퀀스 테이블이 중요한 이유

- **중복 방지**: 동일한 Job 또는 Step이 여러 번 실행될 때 고유한 식별자를 부여해 중복 문제를 피할 수 있다.
- **재시작 가능성**: 실패한 작업이 재시작될 때 이전의 실행 기록을 추적하고, 정확한 위치에서 이어서 수행할 수 있다.
- **일관성 유지**: 여러 트랜잭션이 동시에 실행될 때도 일관된 ID 값을 부여할 수 있다.
