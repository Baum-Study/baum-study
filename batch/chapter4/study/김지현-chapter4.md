### FlatFileItemReader로 단순 파일 읽고, FlatFileItemWriter로 파일에 쓰기
---

## FlatFileItemReader 개요
`FlatFileItemReader`는 배치에서 제공하는 기본적인 `ItemReader`로, 텍스트 파일로부터 데이터를 읽는다. <br>
고정 길이, 구분자 기반, 멀티라인 등 다양한 형식의 텍스트 파일을 지원한다. <br>
간단하고 효율적인 구현: 설정 및 사용이 간편하며, 대규모 데이터 처리에 효율적이다 <br>
확장 가능성: 토크나이저, 필터 등을 통해 기능 확장이 가능하다. <br>
사용처: 고정 길이, 구분자 기반, 멀티라인 등 다양한 형식의 텍스트 파일 데이터 처리 <br>
장점: 간단하고 효율적인 구현, 다양한 텍스트 파일 형식 지원 <br>
단점: 복잡한 데이터 구조 처리에는 적합하지 않음 <br>

## FlatFileItemReader 주요 구성 요소
Resource: 읽을 파일을 지정한다. <br>
LineMapper: 텍스트 파일의 각 라인을 Item으로 변환하는 역할을 한다. <br>
LineToken: 텍스트 파일의 각 라인을 토큰으로 분리하는 역할을 한다. <br>
FieldSetMapper: 토큰을 Item의 속성에 매핑하는 역할을 한다. <br>
SkippableLineMapper: 오류 발생 시 해당 라인을 건너뛸 수 있도록 한다. <br>
LineCallbackHandler: 라인별로 처리를 수행할 수 있도록 한다. <br>
ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 한다. <br>

## Customer 모델 생성하기
```kotlin
data class Customer(
    val name: String,
    val age: Int,
    val gender: String,
)
```
읽어들인 정보를 Customer 객체에 매핑할 수 있도록 객체를 정의한다. <br>

## FlatFileItemReader 빈 생성
```kotlin
    @Bean
    fun flatFileItemReader(): FlatFileItemReader<Customer> {
        return FlatFileItemReaderBuilder<Customer>()
            .name("FlatFileItemReader")
            .resource(ClassPathResource("./customer.csv"))
            .encoding(ENCODING)
            .delimited().delimiter(",")
            .names("name", "age", "gender")
            .targetType(Customer::class.java)
            .build()
    }
```
resource: 클래스 패스 내부에 존재하는 csv파일을 읽어들인다. <br>
encoding: 파일 데이터의 인코딩을 추가한다. <br>
delimited: 구분자로 설정되어 있음을 뜻한다. <br>
delimiter: 구분자를 무엇으로 할지 정한다. <br>
names: 구분자로 구분된 데이터의 이름을 정한다. <br>
targetType: 구분된 데이터를 어느 모델에 넣을지 클래스 타입을 정한다. <br>

## FlatFileItemWriter 개요
`FlatFileItemWriter`는 배치에서 제공하는 `ItemWriter` 인터페이스를 구현하는 클래스이다. <br>
데이터를 텍스트 파일로 출력하는 데 사용된다. <br>

## FlatFileItemWriter 주요 구성 요소
Resource: 출력 파일 경로를 지정한다. <br>
LineAggregator: Item을 문자열로 변환하는 역할을 한다. <br>
HeaderCallback: 출력 파일 헤더를 작성한다. <br>
FooterCallback: 출력 파일 푸터를 작성한다. <br>
Delimiter: 항목 사이 구분자를 지정한다. <br>
AppendMode: 기존 파일에 추가할건지 여부를 결정한다. <br>
장점: 대량의 데이터를 빠르게 출력할 수 있다. <br>
텍스트 파일로 데이터를 출력하는 간편한 방법을 제공하고, 다양한 설정을 통해 원하는 형식으로 출력 파일을 만들 수 있다. <br>
단점: 텍스트 파일 형식만 지원한다. <br>
복잡한 구조의 데이터를 출력할 경우 설정이 복잡해질 수 있고, 설정 오류 시 출력 파일이 손상될 수 있다. <br>

## FlatFileItemWriter 작성하기
```kotlin
    @Bean
    fun flatFileItemWriter(): FlatFileItemWriter<Customer> {
        return FlatFileItemWriterBuilder<Customer>()
            .name("flatFileItemWriter")
            .resource(FileSystemResource("./output/customer_new.csv"))
            .encoding(ENCODING)
            .delimited().delimiter("\t")
            .names("name", "age", "gender")
            .append(false)
            .lineAggregator(CustomerLineAggregator())
            .headerCallback(CustomerHeader())
            .footerCallback(CustomerFooter(aggregateInfos))
            .build()
    }
```
FaltFileItemWriterBuilder: 파일로 결과를 쓰게 하기 위한 빌더 객체 <br>
name: FlatFileItemWriter의 이름을 지정한다. <br>
resource: 저장할 최종 파일 이름 <br>
encoding: 저장할 파일의 인코딩 타입 <br>
delimited().delimiter: 각 필드를 구분할 구분자를 지정한다. <br>
append: true일 경우 기존 파일에 첨부, false일 경우 새로운 파일 <br>
lineAggregator: 라인 구분자를 지정한다. <br>
headerCallback: 출력 파일의 헤더를 지정할 수 있다. <br>
footerCallback: 출력 파일의 푸터를 지정할 수 있다. <br>

## CustomerLineAggregator 작성하기
```kotlin
class CustomerLineAggregator : LineAggregator<Customer> {
    override fun aggregate(item: Customer): String {
        return item.name + "," + item.age
    }
}
```
`LineAggregator`은 FlatFile에 저장할 아이템들을 스트링으로 변환하는 방법을 지정하는 것이다. <br>
위와 같이 aggregate()를 구현하여 아이템을 스트링 문자로 변환하는 것을 확인할 수 있다. <br>

## CustomerHeader 작성하기
```kotlin
class CustomerHeader : FlatFileHeaderCallback {
    override fun writeHeader(writer: Writer) {
        writer.write("ID,AGE")
    }
}
```
`FlatFileHeaderCallback`은 writeHeader()를 구현하고, 출력될 파일의 헤더를 달아주는 역할을 한다. <br>

## CustomerFooter 작성하기
```kotlin
class CustomerFooter(
    private val aggregateCustomers: ConcurrentHashMap<String, Int>,
) : FlatFileFooterCallback {
    override fun writeFooter(writer: Writer) {
        writer.write("총 고객 수: " + aggregateCustomers["TOTAL_CUSTOMERS"])
        writer.write(System.lineSeparator())
        writer.write("총 나이: " + aggregateCustomers["TOTAL_AGES"])
    }
}
```
`FlatFileFooterCallback`은 푸터를 작성할 때 사용된다. <br>
위의 예제에서는 결과를 집계하여 총 고객수와 총 나이를 출력하고 있다. <br>
이때 전달된 HashMap을 전달하여 결과를 출력하고 있음을 확인하자. <br>

## AggregateCustomerProcessor 작성하기
```kotlin
class AggregateCustomerProcessor(
    private val aggregateCustomers: ConcurrentHashMap<String, Int>,
) : ItemProcessor<Customer, Customer> {
    override fun process(item: Customer): Customer {
        aggregateCustomers.putIfAbsent("TOTAL_CUSTOMER", 0)
        aggregateCustomers.putIfAbsent("TOTAL_AGES", 0)
        aggregateCustomers["TOTAL_CUSTOMER"] = aggregateCustomers["TOTAL_CUSTOMER"]!! + 1
        aggregateCustomers["TOTAL_AGES"] = aggregateCustomers["TOTAL_AGES"]!! + item.age
        return item
    }
}
```
`ItemProcessor`는 process()를 구현하였으며, 각 아이템을 하나씩 읽고 아이템의 내용을 집계하고 있다. <br>

## FlatFileItemJobConfig 전체 소스
```kotlin
@Configuration
class FlatFileItemJobConfig {
    private val log = KotlinLogging.logger {}
    private val aggregateInfos = ConcurrentHashMap<String, Int>()
    private val itemProcessor = AggregateCustomerProcessor(aggregateInfos)

    @Bean
    fun flatFileItemReader(): FlatFileItemReader<Customer> {
        return FlatFileItemReaderBuilder<Customer>()
            .name("FlatFileItemReader")
            .resource(ClassPathResource("./customer.csv"))
            .encoding(ENCODING)
            .delimited().delimiter(",")
            .names("name", "age", "gender")
            .targetType(Customer::class.java)
            .build()
    }

    @Bean
    fun flatFileItemWriter(): FlatFileItemWriter<Customer> {
        return FlatFileItemWriterBuilder<Customer>()
            .name("flatFileItemWriter")
            .resource(FileSystemResource("./output/customer_new.csv"))
            .encoding(ENCODING)
            .delimited().delimiter("\t")
            .names("name", "age", "gender")
            .append(false)
            .lineAggregator(CustomerLineAggregator())
            .headerCallback(CustomerHeader())
            .footerCallback(CustomerFooter(aggregateInfos))
            .build()
    }

    @Bean
    fun flatFileStep(
        jobRepository: JobRepository,
        transactionManager: PlatformTransactionManager,
    ): Step {
        log.info { "------------------ Init flatFileStep -----------------" }
        return StepBuilder("flatFileStep", jobRepository)
            .chunk<Customer, Customer>(CHUNK_SIZE, transactionManager)
            .reader(flatFileItemReader())
            .processor(itemProcessor)
            .writer(flatFileItemWriter())
            .build()
    }

    @Bean
    fun flatFileJob(
        flatFileStep: Step,
        jobRepository: JobRepository,
    ): Job {
        log.info("------------------ Init flatFileJob -----------------");
        return JobBuilder(FLAT_FILE_CHUNK_JOB, jobRepository)
            .incrementer(RunIdIncrementer())
            .start(flatFileStep)
            .build()
    }

    companion object {
        const val CHUNK_SIZE = 100
        const val ENCODING = "UTF-8"
        const val FLAT_FILE_CHUNK_JOB = "FLAT_FILE_CHUNK_JOB"
    }
}
```




