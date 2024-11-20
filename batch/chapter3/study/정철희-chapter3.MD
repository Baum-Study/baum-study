## SpringBatch ChunkModel과 TaskletModel

### Chunk Model
Chunk Model은 `ItemReader`, `ItemProcessor`, `ItemWriter`를 이용하여 데이터를 처리하는 모델입니다.  
데이터를 처리할 때 `Chunk`라는 단위로 데이터의 수를 지정하여 처리합니다.  
ChunkOrientedTasklet는 청크 처리를 지원하는 Tasklet의 구현체입니다.  
또한 commit-interval이라는 설정값을 통해 Chunk의 크기를 지정할 수 있습니다.  

#### ItemReader
ItemReader는 데이터를 읽어오는 역할을 하고 직접 구현이 가능합니다.  
하지만 SpringBatch에서는 다양한 ItemReader를 제공하고 있기 때문에 굳이 직접 구현할 필요는 없습니다.  

FlatFileItemReader는 csv 파일과 같이 구조화되지 않은 파일들을 읽어오는 ItemReader입니다.  
읽어들인 데이터를 객체로 매핑하기 위해서 delimeter를 기준으로 매핑 룰을 이용하여 객체로 매핑합니다.  
예를 들자면 csv 파일에서 ','를 기준으로 데이터를 읽어와서 객체로 매핑하는 것입니다.  
또한 Resource object를 이용하여 커스텀하게 HTTP, FTP, Amazon S3와 같은 다양한 위치에서 파일을 읽어올 수 있습니다.  

StaxEventItemReader는 xml 파일을 읽어오는 ItemReader입니다.  

JdbcPagingItemReader, JdbcCursorItemReader는 데이터베이스에서 데이터를 읽어오는 ItemReader입니다.  
일반적으로 데이터베이스에서 데이터를 한번에 읽어온다면 메모리 부족으로 인한 문제가 발생하기에 이 두 ItemReader를 사용해서 데이터를 읽어옵니다.  
JdbcPagingItemReader는 JdbcTemplate을 사용해서 Pagination 방식으로 데이터를 읽어옵니다.  
JdbcCursorItemReader는 무한 스크롤 방식으로 데이터를 읽어옵니다.  

MyBatisCursorItemReader, MyBatisPagingItemReader는 MyBatis를 사용해서 데이터를 읽어오는 ItemReader입니다.  

JpaPagingItemReader, HibernatePagingItemReader, HibernateCursorItemReader는 JPA, Hibernate를 사용해서 데이터를 읽어오는 ItemReader입니다.  

JmsItemReader, AmqpItemReader는 JMS, AMQP와 같은 메시징 시스템을 사용해서 데이터를 읽어오는 ItemReader입니다.    
여기서 JMS는 Java Message Service, AMQP는 Advanced Message Queuing Protocol의 약자입니다.

#### ItemProcessor

ItemProcessor는 ItemReader에서 읽어온 데이터를 가공하는 역할을 합니다.  
꼭 필수로 구현해야 하는 것은 아니지만 데이터를 가공할 필요가 있을 때 사용합니다.  

PassThroughItemProcessor는 데이터를 가공하지 않고 그대로 전달하는 ItemProcessor입니다.  

ValidatingItemProcessor는 데이터를 검증하는 ItemProcessor입니다.  

CompositeItemProcessor는 여러개의 ItemProcessor를 사용할 때 사용하는 ItemProcessor입니다.  
CompositeItemProcessorBuilder를 사용해서 여러개의 ItemProcessor를 등록할 수 있고 이 때 ValidatingItemProcessor를 등록해서 데이터를 검증할 수도 있습니다.

#### ItemWriter

ItemWriter는 ItemProcessor에서 가공한 데이터를 저장하는 역할을 합니다.  
ItemReader와 마찬가지로 직접 구현이 가능하지만 SpringBatch에서 제공하는 ItemWriter를 사용할 수 있습니다.  

FlatFileItemWriter는 csv 파일과 같이 구조화되지 않은 파일들을 저장하는 ItemWriter입니다.  

StaxEventItemWriter는 xml 파일을 저장하는 ItemWriter입니다.  

JdbcBatchItemWriter는 데이터베이스에 데이터를 저장하는 ItemWriter입니다.  

MyBatisBatchItemWriter는 MyBatis를 사용해서 데이터를 저장하는 ItemWriter입니다.

JpaItemWriter, HibernateItemWriter는 JPA, Hibernate를 사용해서 데이터를 저장하는 ItemWriter입니다.

JmsItemWriter, AmqpItemWriter는 JMS, AMQP와 같은 메시징 시스템을 사용해서 데이터를 저장하는 ItemWriter입니다.

### Tasklet Model

데이터의 양이 많아 분할해서 처리해야하는 경우에는 Chunk Model을 사용하지만 그렇지 않은 경우에는 Tasklet Model을 사용합니다.  
예를들어 한번에 하나의 레코드만 읽어서 쓰기해야하는 경우 Tasklet Model이 적합합니다.  
Tasklet Model도 역시 Tasklet 인터페이스를 구현해야합니다.  

SystemCommandTasklet은 시스템 명령을 비동기적으로 실행하는 Tasklet입니다.  
시스템 명령을 실행하는 스레드와 SpringBatch의 스레드가 별도로 동작하기 때문에 프로세스 도중 타임아웃을 설정하고 시스템 명령의 실행 스레드를 취소할 수 있습니다.  

MethodInvokingTaskletAdapter는 POJO클래스의 특정 메소드를 실행하는 Tasklet입니다.  
POJO 클래스는 반환 값으로 ExitStatus를 설정해줘야합니다.  
만약 설정을 하지 않는다면 ExitStatus.COMPLETED로 설정됩니다.  
POJO 클래스 설정이 되었자면 targetObject 속성에 빈을 지정하고 targetMethod속성에 실행할 메소드 이름을 지정하면 됩니다.  

아래와 같은 POJO 클래스를 작성합니다.  

```kotlin
@Component
class CustomTask {
    fun performTask(): ExitStatus {
        println("작업 수행 중")
        return ExitStatus.COMPLETED
    }
}
```

그리고 아래와 같이 Tasklet을 설정합니다.  

```kotlin
@Bean
fun methodInvokingTaskletAdapter(customTask: CustomTask) =
    MethodInvokingTaskletAdapter().apply {
        setTargetObject(customTask)
        setTargetMethod("performTask")
    }
```