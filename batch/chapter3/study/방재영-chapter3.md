# Chunk 및 Tasklet Model 정리

## 1. Chunk Model

- Chunk Model은 데이터를 일정 단위(청크)로 나누어 처리하는 방식이다.
- **ChunkOrientedTasklet**은 청크 처리를 지원하는 구체적인 Tasklet 클래스이다.
- **Chunk Size 조정**: `commit-interval` 설정값을 통해 청크에 포함될 데이터의 최대 레코드 수를 조정할 수 있다.
- **구성요소**:
    - **ItemReader**: 데이터를 읽어오는 역할
    - **ItemProcessor**: 데이터를 처리하는 역할. 선택적으로 사용 가능
    - **ItemWriter**: 데이터를 저장하거나 출력하는 역할

### Chunk Model 동작

1. **ChunkOrientedTasklet**은 `ItemReader`, `ItemProcessor`, `ItemWriter`를 호출한다.
2. 청크 크기만큼 데이터를 `ItemReader`가 읽는다.
3. 읽어들인 데이터를 `ItemProcessor`에 전달하여 처리한다.
4. 처리된 데이터를 `ItemWriter`에 전달하여 저장하거나 파일로 출력한다.

---

## 2. ItemReader

`ItemReader`는 데이터를 읽는 역할을 담당하며, Spring Batch는 다양한 구현체를 제공한다.

### 제공되는 구현체

- **FlatFileItemReader**: CSV와 같은 플랫 파일을 읽는다. `delimiter`로 데이터를 매핑하거나 커스텀 매핑이 가능하다.
- **StaxEventItemReader**: XML 파일을 StAX 기반으로 읽는다.
- **JdbcPagingItemReader / JdbcCursorItemReader**:
    - 데이터베이스에서 많은 양의 데이터를 처리해야 하는 경우, 메모리에 있는 모든 레코드를 읽는 것을 피하고 한 번의 처리에 필요한 데이터만 읽고 폐기하는 것이 필요하다.
    - `JdbcPagingItemReader`: `JdbcTemplate` 기반으로 데이터를 페이지 단위로 읽는다.
    - `JdbcCursorItemReader`: JDBC 커서를 사용해 데이터를 순차적으로 읽는다.
- **MyBatisCursorItemReader / MyBatisPagingItemReader**: MyBatis를 사용해 데이터베이스의 데이터를 읽는다.
- **JpaPagingItemReader / HibernatePagingItemReader**:
    - JPA나 Hibernate를 사용해 데이터를 읽는다.
    - Paging과 Cursor의 차이점은 MyBatis를 구현 방법이 다를 뿐, JdbcXXXItemReader과 동일하다.
- **JmsItemReader / AmqpItemReader**: JMS 또는 AMQP 메시지를 읽는다.

---

## 3. ItemProcessor

`ItemProcessor`는 데이터를 처리하는 역할을 담당한다. 데이터 변환, 데이터 변경, 외부 인터페이스 호출 등의 작업을 수행할 수 있다. 선택적으로 사용 가능하다.

### 주요 구현체

- **PassThroughItemProcessor**: 데이터 처리를 생략한다.
- **ValidatingItemProcessor**:
    - 데이터 검증을 수행한다.
    - 일반적인 org.springframework.validation.Validator의 어댑터인 SpringValidator와 org.springframework.validation의 규칙을 제공한다.
- **CompositeItemProcessor**:
    - 여러 `ItemProcessor`를 순차적으로 실행한다.
    - ValidatingItemProcessor를 사용하여 입력 확인을 수행한 후, 비즈니스 로직을 실행하려는 경우 활성화된다.

---

## 4. ItemWriter

ItemWriter는 데이터를 저장하거나 출력하는 역할을 담당한다.

### 주요 구현체

- **FlatFileItemWriter**:
    - CSV와 같은 플랫 파일로 출력한다.
    - 파일 라인에 대한 매핑 규칙은 구분 기호 및 개체에서 사용자 정의로 사용할 수 있다.
- **StaxEventItemWriter**: XML 파일로 출력한다.
- **JdbcBatchItemWriter**: JDBC를 사용해 데이터를 DB에 저장한다.
- **MyBatisBatchItemWriter**: MyBatis를 사용해 데이터를 DB에 저장한다.
- **JmsItemWriter / AmqpItemWriter**: JMS 또는 AMQP로 메시지 전송한다.

---

## 5. Tasklet Model

- Tasklet Model은 청크 단위 처리가 아닌 특정 작업을 수행하는 데 적합하다.
- Tasklet 인터페이스 구현을 통해 커스텀 작업 수행 가능하다.

### 주요 구현체

- **SystemCommandTasklet**:
    - 시스템 명령어를 비동기적으로 실행한다.
    - 명령 속성에 수행해야할 명령어를 지정하여 사용할 수 있다.
    - 시스템 명령은 호출하는 스레드와 다른 스레드에 의해 실행되므로 프로세스 도중 타임아웃을 설정하고, 시스템 명령의 실행 스레드를 취소할 수 있다.
- **MethodInvokingTaskletAdapter**:
    - POJO 클래스의 특정 메서드를 호출한다.
    - targetObject 속성에 대상 클래스의 빈을 지정하고, targetMethod 속성에 실행할 메소드 이름을 지정한다.
    - 호출된 메서드가 작업이 성공했는지, 실패했는지 등의 상태를 반환하려면 ExitStatus 값을 반환해야 한다.
    - 만약 메서드가 ExitStatus 대신 다른 값을 반환하면, 스프링 배치는 특별히 문제가 없다고 보고 `정상 종료(ExitStatus: COMPLETED)`로 간주한다.
