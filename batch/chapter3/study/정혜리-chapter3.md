# Chunk Model 과 Tasklet Model

## Chunk Model

- 처리할 데이터를 일정 단위(청크)로 처리하는 방식
- `ChunkOrientedTasklet`은 청크 처리를 지원하는 Tasklet의 구체적인 클래스 역할을 수행
- 청크 Size는 본 클래스의 commit-interval 설정을 이용해 조정이 가능
- ItemReader, ItemProcessor, ItemWriter는 청크 단위를 처리하기 위한 인터페이스
- `ChunkOrientedTasklet`은 청크 단위에 따라 ItemReader, ItemProcessor, ItemWriter를 반복 실행한다
- 처리 흐름
  - ItemReader에서 데이터 청크 크기만큼 데이터 읽어 ItemProcessor에 전달 
  - ItemProcessor가 처리 후 ItemWriter로 전달
  - ItemWriter는 저장 및 파일 처리 수행

<br />

## ItemReader

스프링 배치에서 다양한 ItemReader 구현체를 제공하고 있고,
직접 커스텀 구현도 가능하다.

### 구현체 종류

#### FlatFileItemReader 
- 플랫파일(구조화 되지 않은 파일)을 읽어들인다.(대표적으로 CSV)
- 객체로 매핑하기 위해 delimeter를 기준으로 매핑 룰을 이용해 객체로 매핑
- 또는 입력에 대해 Resource object를 이용해 커스텀 매핑할 수 있음

#### StaxEventItemReader
- XML 파일 읽는 용
- XML 파일을 StAX(Streaming API for XML) 기반으로 읽는다
- 스트리밍 방식이라 메모리 효율적이다

#### JdbcPagingItemReader / JdbcCursorItemReader
- JDBC를 사용해 SQL을 실행하고 데이터베이스의 레코드를 읽음
- 데이터베이스에서 많은 양의 데이터를 처리해야 하는 경우, 모든 레코드를 읽는 것을 피하고 한 번의 처리에 필요한 데이터만 읽고 폐기하는 방식이 필요
- JdbcPagingItemReader는 JdbcTemplate를 이용해 한 번에 여러 줄 씩 읽어온다
- 반면 JdbcCursorItemReader는 JDBC 커서를 이용해 데이터를 한 줄씩 읽어온다

#### MyBatisCursorItemReader / MyBatisPagingItemReader
- MyBatis를 사용해 데이터베이스의 레코드를 읽음
- MyBatis-Spring 라이브러리를 통해 Spring 과 통합관리한다
- Paging과 Cursor의 차이점은 MyBatis로 구현한다는 점만 다를 뿐, JDBC와 동일

#### JpaPagingItemReader / JpaCursorItemReader
- JPA를 사용해 데이터를 읽어온다
- JPA의 EntityManager를 활용한다

#### HibernatePagingItemReader / HibernateCursorItemReader
- Hibernate를 직접 사용하여 데이터를 읽어온다
- Hibernate의 SessionFactory를 사용한다

#### JmsItemReader / AmqpItemReader:
- 메시지를 JMS혹은 AMQP에서 읽어들인다
- Message Queue 기반 비동기 작업에 사용한다

<br />

## ItemProcessor

기본으로 제공하는 구현체가 없으므로 직접 구현해야 한다.

ItemProcessor는 읽어들인 청크 데이터를 처리한다. (데이터 변환, 변경, 외부 인터페이스 호출 등)

ItemProcessor는 chunk model에서 없어도 되는 옵션

#### 인터페이스
```java
@FunctionalInterface
public interface ItemProcessor<I, O> {
    @Nullable
    O process(@NonNull I item) throws Exception;
}
```

#### 구현체 예시
```kotlin
class MyItemProcessor : ItemProcessor<String, String> {
    override fun process(item: String): String {
        println("Input: ${item}")
      
        val processedItem = item.uppercase()
      
        println("Output: ${processedItem}")
      
        return processedItem
    }
}
```
- 구현 시 입력, 출력 타입을 제네릭 타입으로 받는다
- processor 함수를 구현해 함수 내부에서 변환처리를 수행한다

### 구현체 종류

#### PassThroughItemProcessor
- 아무런 작업도 수행하지 않는다
- 입력된 데이터의 처리가 필요하지 않는 경우 사용

#### ValidatingItemProcessor
- 입력된 데이터를 검증한다
- 입력 확인 규칙을 구현하려면 Spring Batch의 `org.springframework.batch.item.validator.Validator` 를 구현한다.
- 하지만 `org.springframework.validation.Validator` 를 사용할 수 있도록 SpringValidator 라는 어댑터를 제공한다

```kotlin
// 1. org.springframework.validation.Validator 구현
class CustomValidator : Validator {
    override fun supports(clazz: Class<*>): Boolean {
        return InputData::class.java.isAssignableFrom(clazz)
    }

    override fun validate(target: Any, errors: Errors) {
        val data = target as InputData
        if (data.value < 0) {
            errors.rejectValue("value", "negativeValue", "Value must not be negative")
        }
    }
}

// 2. Spring Batch 의 Spring Validator 어댑터 사용
fun createProcessor(): ValidatingItemProcessor<InputData> {
    val springValidator = SpringValidator<InputData>()
    springValidator.setValidator(CustomValidator())

    val processor = ValidatingItemProcessor<InputData>()
    processor.setValidator(springValidator)
    processor.setFilter(true)
    return processor
}
```


#### CompositeItemProcessor
- 동일한 입력 데이터에 대해 여러 ItemProcessor를 순차적으로 실행한다
- ValidatingItemProcessor 로 입력데이터 확인 후 비즈니스 로직을 수행할 때 사용된다
- 실행할 프로세서들을 리스트에 담아 설정하면 된다

<br />

## ItemWriter

### 구현체 종류

#### FlatFileItemWriter
- 처리된 Java 객체를 CSV와 같은 플랫 파일로 작성한다
- 파일 라인 매핑 규칙은 사용자 정의로 사용할 수 있다

#### StaxEventItemWriter
- XML 파일로 자바 객체를 작성할 수 있다

#### JdbcBatchItemWriter
- jdbc를 활용해 sql을 수행하고 자바 객체를 데이터베이스에 쓴다
- 내부적으로 JdbcTemplate 사용

#### MyBatisBatchItemWriter
- mybatis를 사용해 자바 객체를 데이터베이스에 쓰기한다
- MyBatis 에서 제공하는 Spring-MyBatis 라이브러리를 사용한다

#### JmsItemWriter / AmqpItemWriter
- JMS혹은 AMQP로 자바 객체의 메시지를 전송한다

<br /><br />

## Tasklet Model
- 청크 단위 처리가 맞지 않을 때 유용하다.\
  ex) 한 번에 하나의 레코드만 읽어서 쓰기해야하는 경우 등
- Spring Batch의 Tasklet 인터페이스를 구현해야 한다.

### 구현 클래스
#### SystemCommandTasklet
- 시스템 명령어를 비동기적으로 실행하는 Tasklet
- 명령 속성에 수행해야 하는 명령어를 지정해 사용할 수 있다
- 호출하는 스레드와 다른 스레드에 의해 실행되므로, 타임아웃을 통해 스레드를 취소할 수 있다


#### MethodInvokingTaskletAdapter
- POJO 클래스의 특정 메소드를 실행하기 위한 Tasklet
- targetObject 속성에 대상 클래스 빈을 지정, targetMethod 속성에 실행할 메서드 이름을 지정
- 일괄처리 종료 상태를 메서드의 반환 값으로 지정할 경우, ExitStatus를 반환값으로 설정해야 한다(다른 반환값이 반환된 경우 정상종료로 간주)

