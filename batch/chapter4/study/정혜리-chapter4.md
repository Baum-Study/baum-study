# FlatFileItemReader, FlatFileItemWriter

## FlatFileItemReader
- SpringBatch 에서 제공하는 기본적인 ItemReader로, 텍스트 파일로부터 데이터를 읽음
- 고정 길이, 구분자 기반, 멀티라인 등 다양한 형식의 텍스트 파일을 지원

### 장점
- 간단하고 효율적인 구성: 설정 및 사용이 간편하며 대규모 처리에도 효율적
- 다양한 텍스트 파일 형식 지원: 고정길이, 구분자 기반, 멀티라인 등 다양한 형식의 텍스트 파일을 읽을 수 있음
- 확장 가능성: 토크나이저, 필터 등을 통해 기능 확장 가능

### 사용처
- 고정길이나 구분자 기반, 멀티라인 등의 다양한 형식의 텍스트 파일에서 데이터 처리할 경우 적합
- 복잡한 데이터 구조에는 적합하지 않다

### 구성요소
- Resource: 읽을 텍스트 파일을 지정
- LineMapper: 텍스트 파일의 각 라인을 Item으로 변환하는 역할
- LineTokenizer: 텍스트 파일의 각 라인을 토큰으로 분리하는 역할
- FieldSetMapper: 토큰을 Item 속성에 매핑하는 역할
- SkippableLineMapper: 오류 발생 시 해당 라인을 건너뛸 수 있도록 함
- LineCallbackHandler: 라인별로 처리를 수행할 수 있도록 함
- ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 함


## 샘플 코드
### Customer 모델 생성
```kotlin
data class Customer(
    var name: String? = null,
    var age: Int? = null,
    var gender: String? = null,
)
```

### FlatFileItemReader 빈 생성
```kotlin
@Bean
fun flatFileItemReader(): FlatFileItemReader<Customer> {
    return FlatFileItemReaderBuilder<Customer>()
        .name("FlatFileItemReader")
        .resource(ClassPathResource("./customer.csv"))
        .encoding(encoding)
        .delimited().delimiter(",")
        .names("name", "age", "gender")
        .targetType(Customer::class.java)
        .build()
}
```
- resource
  - Resource 타입을 인자로 받으며, 위의 코드에서는 ContextPathResource를 사용해 클래스 패스 내부의 파일을 읽어들인다
- encoding
  - 파일 데이터의 인코딩을 지정한다
- delimited
  - 구분자로 설정되어 있음을 의미한다. lineTokenizer를 DelimitedLineTokenizer로 지정한다
- delimiter
  - 어떤 구분자를 사용할 지 지정한다 ("," 가 기본 값이기 때문에 지정하지 않는 것과 동일함)
- names
  - 구분자로 구분된 데이터의 이름을 지정한다
- targetType
  - 구분된 데이터를 어떤 모델에 넣을지 클래스 타입 지정

<br />

-----

<br />

## FlatFileItemWriter
FlatFileItemWriter 는 Spring Batch에서 제공하는 ItemWriter 인터페이스를 구현하는 클래스이다.
데이터를 텍스트 파일로 출력하는 데 사용된다.

### 구성요소
- Resource: 출력 파일의 경로를 지정한다
- LineAggregator: Item을 문자열로 변환하는 역할
- HeaderCallback: 출력파일 헤더를 작성하는 역할
- FooterCallback: 출력파일 푸터를 작성하는 역할
- Delimiter: 항목 사이 구분자를 지정
- AppendMode: 기존 파일에 추가할지 여부를 지정

### 장점
- 간편성: 텍스트 파일로 데이터를 출력하는 간편한 방법을 제공한다
- 유연성: 다양한 설정을 통해 원하는 형식으로 출력 파일을 만들 수 있다
- 성능: 대량의 데이터를 빠르게 출력할 수 있다

### 단점
- 형식 제약: 텍스트 파일(2차원 표 포함) 형식만 지원한다
- 복잡한 구조: 복잡한 구조의 데이터를 출력할 경우 설정이 복잡해진다 ([예시](#복잡한-구조-예시---patternmatchingcompositelinemapper))
- 오류 가능성: 설정 오류 시 출력 파일이 손상될 수 있다

## 샘플 코드
### FlatFileItemWriter 
```kotlin
@Bean
fun flatFileItemWriter(): FlatFileItemWriter<Customer> {
    return FlatFileItemWriterBuilder<Customer>()
        .name("FlatFileItemWriter")
        .resource(FileSystemResource("./output/cusomer_new.csv"))
        .encoding(encoding)
        .delimited().delimiter("\t")
        .names("Name", "Age", "Gender")
        .build()
}
```

- FlatFileItemWriterBuilder: 파일로 결과를 쓰기 위한 빌더 객체
- name: FlatfileItemWriter의 이름을 지정한다
- resource: 저장할 최종 파일
- encoding: 저장할 파일의 인코딩 타입
- delimited().delimiter: 각 필드를 구분할 delimiter 지정
- append: 기존 파일에 추가로 작성할지, 새로운 파일로 덮어쓸 지 지정한다

### 테스트
아래와 같은 파일을 추가해 테스트할 수 있다.
- customer.csv (input)
```
hyeri, 99, F
john, 100, M
Tom, 1, M
```

- customer_new.csv (output)
```
hyeri	99	F
john	100	M
Tom   1	  M
```

-----

## LineAggregator 구현
LineAggregator는 FlatFile에 저장할 아이템을 String으로 변환하는 방법을 지정한다.

LineAggregator 구현 시 aggregate()를 반드시 정의해야 하는데, 해당 함수는 제네릭으로 지정한 타입의 인자를 받아 String을 반환하는 것을 알 수 있다.

### LineAggregator interface
```java
public interface LineAggregator<T> {
    String aggregate(T item);
}
```

### CustomerLineAggregator 구현 및 적용

- LineAggregator 구현
```kotlin
class CustomLineAggregator : LineAggregator<Customer> {
    override fun aggregate(item: Customer): String {
        return "${item.name}:${item.age}"
    }
}
```

- FlatFileItemWriter 에 lineAggregator 설정
```kotlin
@Bean
fun flatFileItemWriter(): FlatFileItemWriter<Customer> {
    return FlatFileItemWriterBuilder<Customer>()
        .name("FlatFileItemWriter")
        .resource(FileSystemResource("./output/cusomer_new.csv"))
        .encoding(encoding)
        .delimited().delimiter("\t")
        .names("Name", "Age", "Gender")
        .append(false)
        .lineAggregator(CustomLineAggregator()) // 추가됨
        .build()
}
```

- 출력 결과
```
hyeri:99
john:100
Tom:1
```

## FlatFileHeaderCallback 구현
FlatFileHeaderCallback는 출력될 파일에 헤더를 달아주는 역할을 하며, write 함수를 반드시 구현해야 한다.

아래는 필드명 추가 예시이다.

- CustomHeader 작성
```kotlin
class CustomHeader : FlatFileHeaderCallback {
    override fun writeHeader(writer: Writer) {
        writer.write("NAME:AGE")
    }
}
```

- flatFileItemWriter에 설정
```kotlin
@Bean
    fun flatFileItemWriter(): FlatFileItemWriter<Customer> {
        return FlatFileItemWriterBuilder<Customer>()
            .name("FlatFileItemWriter")
            .resource(FileSystemResource("./output/cusomer_new.csv"))
            .encoding(encoding)
            .delimited().delimiter("\t")
            .names("Name", "Age", "Gender")
            .append(false)
            .lineAggregator(CustomLineAggregator())
            .headerCallback(CustomHeader()) // 추가
            .build()
    }
```

- 출력 결과
```
NAME:AGE
hyeri:99
john:100
Tom:1
```

## FlatFileFooter, ItemProcessor 로 집계 작성하기
총 고객 수, 총 고객의 나이를 집계한 결과를 하단에 작성해보자

### FlatFileFooterCallback 구현
```kotlin
class CustomFooter(
  val aggregateCustomers: ConcurrentMap<String, Int>
) : FlatFileFooterCallback {
  override fun writeFooter(writer: Writer) {
    writer.write("총 고객 수: ${aggregateCustomers["TOTAL_CUSTOMERS"]}" )
    writer.write(System.lineSeparator())
    writer.write("총 나이: ${aggregateCustomers["TOTAL_AGES"]}")
  }
}
```

### itemProcessor 추가 및 footerCallback 설정 추가
```kotlin
@Bean
fun flatFileStep(jobRepository: JobRepository, transactionManager: PlatformTransactionManager): Step {
    log.info { "------------------ init flatFileStep --------------------------" }

    return StepBuilder("flatFileStep", jobRepository)
        .chunk<Customer, Customer>(chunkSize, transactionManager)
        .reader(flatFileItemReader())
        .processor { item ->
            aggregateCustomers.putIfAbsent("TOTAL_CUSTOMERS", 0)
            aggregateCustomers.putIfAbsent("TOTAL_AGES", 0)

            aggregateCustomers["TOTAL_CUSTOMERS"] = aggregateCustomers["TOTAL_CUSTOMERS"]!! + 1
            aggregateCustomers["TOTAL_AGES"] = aggregateCustomers["TOTAL_AGES"]!! + item.age!!
            item
        }
        .writer(flatFileItemWriter())
        .build()
}

@Bean
fun flatFileItemWriter(): FlatFileItemWriter<Customer> {
  return FlatFileItemWriterBuilder<Customer>()
    .name("FlatFileItemWriter")
    .resource(FileSystemResource("./output/cusomer_new.csv"))
    .encoding(encoding)
    .delimited().delimiter("\t")
    .names("Name", "Age", "Gender")
    .append(false)
    .lineAggregator(CustomLineAggregator())
    .headerCallback(CustomHeader())
    .footerCallback(CustomFooter(aggregateCustomers)) // 추가
    .build()
}
```

### 결과
```
NAME:AGE
hyeri:99
john:100
Tom:1
총 고객 수: 3
총 나이: 200
```

-------

### 복잡한 구조 예시 - PatternMatchingCompositeLineMapper
한 파일에 여러 유형의 레코드가 있는 경우의 예시

참고: https://docs.spring.io/spring-batch/reference/readers-and-writers/flat-files/file-item-reader.html

```java
@Bean
public PatternMatchingCompositeLineMapper orderFileLineMapper() {
	PatternMatchingCompositeLineMapper lineMapper =
		new PatternMatchingCompositeLineMapper();

	Map<String, LineTokenizer> tokenizers = new HashMap<>(3);
	tokenizers.put("USER*", userTokenizer());
	tokenizers.put("LINEA*", lineATokenizer());
	tokenizers.put("LINEB*", lineBTokenizer());

	lineMapper.setTokenizers(tokenizers);

	Map<String, FieldSetMapper> mappers = new HashMap<>(2);
	mappers.put("USER*", userFieldSetMapper());
	mappers.put("LINE*", lineFieldSetMapper());

	lineMapper.setFieldSetMappers(mappers);

	return lineMapper;
}
```

