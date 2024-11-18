# Spring Batch 아키텍처

![img.png](../image/방재영1.png)

# Tasklet

- 배치 작업에서 가장 기본적인 실행 단위를 정의
- 일반적으로 Step 내부에서 실행되며, 단일 태스크(작업)을 수행하는 데 사용
    - 간단한 처리 로직에 적합
    - 파일 읽기, 데이터 초기화, 로깅 등 반복 처리가 필요 없는 작업에서 주로 사용

### 주요 메서드

```java
	@Nullable
	RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception;
```

- StepContribution
    - 배치 실행 과정에서 Step의 현재 상태를 추적 및 업데이트할 수 있는 객체
    - 읽은 데이터 개수, 쓴 데이터 개수, 필터링된 데이터 개수 등을 기록
    - Step의 기여(contribution)를 통해 결과를 집계
- ChunkContext
    - Step에서 실행되는 Tasklet의 호출 간에 공유되는 컨텍스트 정보를 제공한다.
    - 실행 중인 Step과 관련된 컨텍스트 데이터를 저장
    - StepExecutionContext와 JobExecutionContext를 통해 데이터 공유.

### 반환값

- RepeatStatus
    - 작업이 완료되었는지 또는 계속 실행되어야 하는지를 나타낸다.
    - RepeatStatus.FINISHED
        - Tasklet 실행이 완료되었음을 나타낸다.
    - RepeatStatus.CONTINUABLE
        - 작업이 아직 완료되지 않았으며, 반복해서 실행해야 함을 나타냅니다.
- null 반환 시 처리
    - null 반환은 RepeatStatus.FINISHED 와 동일하게 간주됩니다.

### 예외처리

- Tasklet 실행 중 오류가 발생하면 Exeption을 던질 수 있습니다. 예외를 던지며 Step을 재시작하거나, 전체 Job을 실패 상태로 마무리할 수 있습니다.

### 유의사항

- 트랜잭션 처리
    - Tasklet 내의 작업은 트랙잭션으로 감싸져 실행되며 실패 시 롤백이 가능하고, 동일한 데이터로 재시도가 가능하다.
- 상태 관리
    - 작업 실행 중 필요한 데이터는 StepContribution 또는 ChunkContext를 통해 관리

# Chunk

- 대량의 데이터를 효율적으로 처리하기 위해 데이터를 일정 크기로 나누어 처리하는 실행 모델입니다.
- Reader-Processor-Writer의 단계로 데이터를 처리하며, 주로 대량 데이터 작업에 적합합니다.
- 트랜잭션 범위가 청크 단위로 제어되며, 실패 시 청크 단위로 롤백 및 재시도가 가능합니다.

### ItemReader

- 데이터를 읽어오는 역할을 수행합니다.
- 데이터 소트: 데이터베이스, 파일, API등
- 청크 단위로 읽을 데이터를 제공

### ItemProcessor

- 읽어온 데이터를 변환하거나 가공하는 역할을 수행
- 데이터 필터링, 데이터 정체, 형식 변환 등 다양한 처리 가능

### ItemWriter

- 처리된 데이터를 저장하거나 출력하는 역할
- 데이터 저장: 데이터베이스, 파일, 메시지 큐 등 다양한 대상.
- 청크 단위로 Writer에 전달된 데이터를 처리

### 트랜잭션 처리

1. 청크시작: ItemReader가 데이터를 읽음
2. 청크 처리: 읽어온 데이터는 ItemProcessor를 통해 가공
3. 청크 저장: ItemWriter가 데이터를 저장
4. 커밋 또는 롤백
    1. 청크 내 모든 작업이 성공하면 트랜잭션을 커밋
    2. 실패 시 전체 청크가 롤백되고 재시도 가능

### 반환값

- 청크 모델은 반복 실행을 관리하며, Chunk 단위로 상태를 기록합니다.
- 데이터가 처리되거나 실패 시 StepExecution 을 통해 상태를 기록합니다.

### 예외처리

- Chunk 처리 중 하나의 데이터가 실패하면, 해당 청크 전체가 롤백됩니다.
- Spring Batch는 Retry 및 Skip 정책을 통해 오류 처리 로직을 제공합니다.

### 유의사항

- 적절한 Chunk 크기 설정
    - 크기가 너무 작으면 트랙잭션 횟수 증가
    - 크기가 너무 크면 메모리 부담 증가
- 데이터 일관성
    - 트랜잭션 범위가 청크 단위로 설정되므로 데이터 손실 방지
- ExecutionContext 활용
    - 작업 중간 상태를 기록하여 재시작 가능

## InitializingBean

- Spring Framework에서 제공하는 인터페이스로, 빈(Bean)이 초기화될 때 특정 작업을 수행할 수 있도록 한다, 빈 초기화 로직을 명시적으로 구현할 때 사용된다.

```java
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}
```

- afterPropertiesSet
    - Spring 컨테이너가 빈의 모든 의존성을 주입한 후 호출
    - 빈 초기화 로직을 정의하는 곳
    - 주로 의존성이 제대로 설정되었는지 검증하거나, 빈 초기화 작업을 수행하는 데 사용

# Spring Batch 흐름

![img.png](../image/방재영2.png)

1. JobScheduler: 배치를 트리거링.
2. JobLauncher: Job을 실행하며, JobExecution과 ExecutionContext 초기화.
3. Step 실행: StepExecution을 생성하고 Step 내 작업을 처리.
4. Chunk 처리: Reader → Processor → Writer 순으로 청크 데이터를 처리.
    - 청크 단위로 트랜잭션 커밋/롤백.
5. 상태 기록: JobExecution 및 StepExecution에 실행 결과와 상태 정보를 저장.
6. Job 완료: JobExecution 상태를 완료로 설정하고 저장.

# Spring Batch 저장 정보

- JobInstance: Job 이름과 파라미터를 기준으로 고유하게 정의된 실행 단위.
- JobExecution: Job의 물리적인 실행을 나타내며, JobInstance와 1:N 관계.
- StepExecution: Step의 실행 정보를 나타내며, 읽기/쓰기 개수, 상태 등을 기록.
- ExecutionContext: 실행 중 공유 데이터를 저장. StepExecutionContext는 Step 내에서만, JobExecutionContext는 Job 간 데이터 공유.
- JobRepository: 실행 상태, 결과 정보를 저장하는 역할.
