## SpringBatch ChunkModel과 TaskletModel

### Chunk Model 
처리할 데이터를 일정단위(청크)로 처리하는 방식이다. <br>
`ChunkOrientedTasklet`은 청크 처리를 지원하는 태스크릿의 구체적인 클래스 역할을 수행한다. <br>
청크에 포함될 데이터의 size는 본 클래스의 `commit-interval`이라는 설정값을 이용해 조정한다. <br>
`ItemReader`, `ItemProcesser`, `ItemWriter`는 청크 단위를 처리하기 위한 인터페이스이다. <br>

`ChunkOrientedTasklet`은 `ItemReader`, `ItemProcesser`, `ItemWriter` 구현체를 각각 호출한다. <br>
이때 `ChunkOrientedTasklet`은 청크 단위에 따라 `ItemReader`, `ItemProcesser`, `ItemWriter`를 반복 실행한다. <br>
청크 크기만큼 `ItemReader`가 데이터를 읽어들인다. <br>
청크 단위를 `ItemProcessor`로 전달하고, 데이터를 처리한다. <br>
`ItemWriter`로 전달되어 데이터를 저장하거나, 처리를 수행한다. <br>

### ItemProcessor
`ItemProcessor`는 구체적인 구현체를 구현하여 스프링 배치에 전달한다. <br>
Reader로 읽어들인 청크 데이터들을 처리한다. (데이터 변환하거나, 데이터에 변경 혹은 외부 인터페이스 호출등을 수행) <br>

```kotlin
class MyItemProcessor : ItemProcessor<MyInputObject, MyOutputObject> { // (1)
    override fun process(item: MyInputObject): MyOutputObject { // (2)

        val processedObject = MyOutputObject() // (3)
        return processedObject // (4)
    }
}
```
(1): `ItemProcessor` 인터페이스를 구현한다. 입력, 출력 데이터 타입을 제네릭 타입으로 받는다. <br>
(2): `process` 메소드 구현체를 구현하며, 입력 파라미터로 받아들인다. <br>
(3): `ItemProcessor` 처리 결과로 반환할 출력 객체를 생성한다. 즉, 변환된 데이터 등이 올 수 있다. <br>
(4): 출력 객체를 반환한다. <br>

### ItemReader, ItemProcesser, ItemWriter의 다양한 구현체들
직접 커스텀 구현을 할 수 있지만, 스프링 배치에서는 이미 구현된 다양한 구현체들을 제공한다. <br>

### ItemReader
`FlatFileItemReader`
- 플랫파일 (구조화되지 않은 파일, CSV파일 등)을 읽어들인다.
- 읽어들인 데이터를 객체로 매핑하기 위해서 구분자를 기준으로 매핑 룰을 이용해 객체로 매핑한다.
- 혹은 입력에 대해서 Resource object를 이용해 커스텀하게 매핑할 수도 있다.

`StaxEventItemReader` 
- XML파일을 STAX 기반으로 읽어들인다.

`JdbcPagingItemReader / JdbcCursorItemReader`
- JDBC를 사용해 SQL을 실행하고 데이터베이스의 레코드를 읽는다.
- `JdbcPagingItemReader`는 `JdbcTemplate`을 이용해 각 페이지에 대한 select sql을 나눠 처리하는 방식으로 구현한다. (페이지네이션)
- `JdbcCursorItemReader`는 JDBC 커서를 이용해 하나의 select sql을 발행하여 구현된다. (무한스크롤)


`MyBatisCursorItemReader / MyBatisPagingItemReader`
- `MyBatis`를 이용해 데이터베이스의 레코드를 읽는다.
- 페이지네이션과 무한스크롤 동작 방식은 JDBCXXXXITEMReader와 동일하다. (MyBatis를 이용한다는 것일뿐)
- 또한 `ItemReaderJPA` 구현이나 Hibernate와 연동하여 데이터베이스의 레코드를 읽어오는 `JpaPagingItemReader`, `HibernatePagingItemReader`, `HibernateCursor`를 제공한다.

`JmsItemReader / AmqpItemReader`
- 메시지를 JMS 혹은 AMQP에서 읽어들인다.

### ItemProcessor
`PassThroughItemProcessor`
- 아무 작업도 수행하지 않는다.
- 입력된 데이터의 변경이나 처리가 필요하지 않는 경우에 사용된다.

`ValidatingItemProcessor`
- 입력된 데이터를 체크한다.
- 입력 규칙을 구현하려면 batch 전용 Validator를 구현해야 한다.

`CompositeItemProcessor`
- 동일한 입력 데이터에 대해 여러 `ItemProcessor`를 순차적으로 실행한다.
- `ValidatingItemProcessor`를 사용해 입력 확인을 수행한 후 비즈니스 로직을 실행하려는 경우 활성화된다.

### ItemWriter
`FlatFileItemWriter`
- 처리된 객체를 CSV파일과 같은 플랫 파일로 작성한다.
- 파일 라인에 대한 매핑 규칙 등은 구분 기호 및 개체에서 사용자 정의로 사용할 수도 있다.

`StaxEventItemWriter`
- XML파일로 객체를 쓰기 할 수 있다.

`JdbcBatchItemWriter`
- JDBC를 이용하여 SQL을 수행하고 객체를 데이터베이스에 쓰기 한다.
- 내부적으로는 JDBCTemplate를 사용하게 된다.

`MyBatisBatchItemWriter`
- MyBatis를 사용해 객체를 데이터베이스로 쓰기 한다.

`MyBatisBatchItemWriter`
- JMS 혹은 AMQO로 객체의 메시지를 전송한다.

### Tasklet Model
앞서 본 `Chunk model`은 큰 데이터를 분할해서 `ItemReader -> ItemProcessor -> ItemWriter` 순으로 처리가 필요한 경우에 매우 유용했다. <br>
반면 청크 단위의 처리가 딱 맞지 않을 경우, `Tasklet model`이 유용하다. (한번에 하나의 레코드만 읽어서 쓰기 해야 하는 경우 등) <br>

### Tasklet 구현 클래스
`SystemCommandTasklet`
- 시스템 명령어를 비동기적으로 실행하는 태스크릿이다.
- 명령 속성에 수행해야 할 명령어를 지정해 사용할 수 있다.
- 시스템 명령은 호출하는 스레드와 다른 스레드에 의해 실행되므로 프로세스 도중 타임아웃을 설정하고, 시스템 명령의 실행 스레드를 취소할 수 있다.
 
`MethodInvokingTaskletAdapter`
- POJO 클래스의 특정 메소드를 실행하기 위한 태스크릿이다.
-  `targetObject` 속성에 대한 클래스의 빈을 지정하고, `targetMethod` 속성에 실행할 메소드 이름을 지정한다.
-  POJO 클래스는 일괄 처리 종료 상태를 반환값으로 반환이 가능하지만, 이 경우 `ExitStatus`를 반환값으로 설정해 줘야 한다.
-  다른 타입의 값이 반환될 경우, 반환값과 상관없이 `정상 종료(ExitStatus:COMPLETED)` 상태로 간주된다.

