# Spring Batch 코드 설명 및 아키텍처 알아보기

### Tasklet
스프링 배치에서 Job을 실행할 때 필요한 실제 작업을 정의하는 인터페이스이다. <br>
그래서 Tasklet의 구현체는 반드시 `execute()` 메소드를 구현해야 한다. <br>

`execute()`: `stepContribution`과  `chunkcontext`를 파라미터로 받으며, 최종적으로 `RepeatStatus`를 반환한다. <br>
이 `RepeatStatus`는 `FINISHIEd(태스크릿이 종료됨)`이나 `CONTINUBALE(계속해서 태스크 수행)`을 반환한다. <br>
그리고 `continueIf`라는 조건에 따라 이 태스크릿을 지속할지, 종료할지를 결정할 수 있다. <br>

`afterPropertiesSet()`: `Bean`이 생성된 후에 초기화 작업을 해 준다. 이를 통해 배치 스텝이 진행되기 전 필요한 초기화 로직 등을 수행할 수 있다.<br>

### PlatformTransactionManager
스프링 배치에서 트랜잭션을 관리하는 인터페이스이다. <br>
`DataSourceTransactionManager`: JDBC를 사용하는 경우. <br>
`JpaTransactionManager`: JPA를 사용하는 경우. <br>
`HibernateTransactionManager`: Hibernate와 연동된 경우. <br>

### JobBuilder
Job을 생성하는 빌더 클래스이다. <br>
Job의 이름, 사용될 Step 등을 정할 수 있다. <br>
`RunIdIncrementer`를 통해서 Job이 실행될 때마다 실행 Id를 증가시켜, Job의 유니크성을 보장한다. <br>


### 스프링 배치 모델
스프링 배치는 `DI`와 `AOP`를 지원하는 프레임워크이다.
`Tasklet model`: 로직 자체가 단순한 경우에 사용, 다양한 데이터소스나 파일을 한번에 처리해야 하는 경우 유연함이 있다. <br>
`Chunk model`: 데이터량이 큰 경우에 사용, `Reader/Processor/Writer` 플로우 방식으로 처리됨. <br>

![img.png](../image/김지현-img1.png)
<br>

`Job`: 스프링 배치의 단일 실행 단위, `Step`으로 구성되어 있다. <br>
`Step`: `Job`을 구성하는 처리 단위, `Tasklet` 또는 `Chunk`로 구성되어 있다. <br>
`Job Launcher`: `Job`을 실행하는 인터페이스이며, `Java command`로 `CommandLineJobRunner`를 실행시켜서 `Job`을 실행한다 <br>
`ItemReader`, `ItemProcessor`, `ItemWriter`: 청크 모델에서 사용된다. 데이터를 읽고, 가공하고, 쓰는 역할을 한다. <br>
`Job Repository`: `Job`과 `Step`의 상태를 관리하는 시스템. <br>


# Spring Batch 흐름

![img_1.png](../image/김지현-img2.png)

### 배치 처리 순서
1. `JobScheduler`가 `JobLauncher`를 실행시킨다.
2. `JobLauncher`는 `Job`을 실행시키고 `JobExecution`을 생성한 뒤 `ExecutionContext`에 `Job`에 대한 실행 정보를 저장한다.
3. `Job`은 자신이 가지고 있는 `Step`들을 실행하고 `StepExecution`을 생성한 뒤 `ExecutionContext`에 `Step`에 대한 실행 정보를 저장한다.
4. `Step`은 `Tasklet`이나 `Chunk` 모델로 작업을 수행한다. (여기서는 `Chunk` 모델)
5. `Chunk` 모델을 사용해서 `ItemReader`, `ItemProcessor`, `ItemWriter`를 수행한다.

### Job 정보의 흐름
1. `JobLauncher`는 `JobRepository`를 통해서 `Job Instance` 정보를 데이터베이스에 저장한다.
2. `JobExecution`을 통해 `Job`을 실행하고 실행 정보를 데이터베이스에 저장한다.
3. `Step`은 `JobRepository`를 통해서 처리 결과와 상태정보를 저장한다.
4. `Job`이 완료되면 `JobRepository`를 통해 완료 정보를 저장한다.

# 배치 저장 정보

### Job Instance
잡의 이름과 전달 파라미터가 있다. <br>
`Job`이 중단되는 경우 다음 실행할 때 중단된 곳 이후부터 실행하도록 지원한다. <br>
재실행을 지원하지 않거나 성공적으로 끝난 `Job`이라면 `Job`을 종료한다. <br>

### JobExecution / ExecutionContext
`JobExecution`은 `Job`의 물리적인 실행을 나타낸다. <br>
`Job Instance`는 1번만 실행되지만 `JobExecution`은 여러번 실행될 수 있다. <br>
_즉, `Job Instance`와 `JobExecution`은 1:N 관계이다_

`JobExecution`의 `ExecutionContext`는 `Job`의 실행 정보와 같은 메타데이터를 저장한다. <br>
`ExecutionContext`에는 Json 형태로 데이터가 저장되기 때문에 `Serializable`을 구현한 객체만 저장할 수 있다. <br>

### StepExecution / ExecutionContext
`StepExecution`은 `Step`의 물리적인 실행을 나타낸다. <br>
`Job`은 여러 `Step`을 수행하므로 1:N 관계가 된다. <br>

`StepExecution`의 `ExecutionContext`는 `Step`의 실행 정보와 같은 메타데이터를 저장한다. <br>
`ExecutionContext`에는 Json 형태로 데이터가 저장되기 때문에 `Serializable`을 구현한 객체만 저장할 수 있다. <br>

### JobRepository
`JobExecution`과 `StepExecution` 등과 같이 배치 실행정보나 상태, 결과 정보들이 데이터베이스에 저장된다. <br>
즉 스프링 배치를 수행하기 위해서는 이를 저장할 데이터베이스가 필요하다. <br>
이렇게 저장된 정보를 활용하여 스프링 배치는 배치 잡을 재실행하거나, 정지된 후부터 실행할 수 있게 된다. <br>

