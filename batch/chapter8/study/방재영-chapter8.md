## CompositeItemProcessor

- Spring Batch에서 제공하는 ItemProcessor 인터페이스 구현 클래스
- 여러 개의 ItemProcessor를 하나의 Processor로 연결하여 여러 단계의 처리가 가능

### 구성 요소

- Delegates: 처리를 수행할 ItemProcessor 목록
- ~~TransactionAttribute: 트랜잭션 속성 설정~~

### 장점

- 여러 단계로 나누어 처리를 수행, 명확하고 이해하기 쉬운 코드 생성 가능
- 각 단계별 Processor를 재사용, 다른 Job에서도 활용 가능
- 다양한 ItemProcessor를 조합, 원하는 처리 과정 구현 가능

### 단점

- 여러 개의 Processor를 설정하고 관리, 설정 복잡
- 여러 단계의 처리 과정, 성능 저하 가능성

### 예제 코드

- LowerCaseItemProcessor

```java
/**
 * 이름, 성별을 소문자로 변경하는 ItemProcessor
 */
public class LowerCaseItemProcessor implements ItemProcessor<Customer, Customer> {
    @Override
    public Customer process(Customer item) {
        item.setName(item.getName().toLowerCase());
        item.setGender(item.getGender().toLowerCase());
        return item;
    }
}
```

<br/>

- After20YearsItemProcessor

```java
/**
 * 나이에 20년을 더하는 ItemProcessor
 */
public class After20YearsItemProcessor implements ItemProcessor<Customer, Customer> {
    @Override
    public Customer process(Customer item) {
        item.setAge(item.getAge() + 20);
        return item;
    }
}
```

<br/>

- CompositeItemProcess - 위 2개 Processor 적용

```java
@Bean
public CompositeItemProcessor<Customer, Customer> compositeItemProcessor() {
    return new CompositeItemProcessorBuilder<Customer, Customer>()
            .delegates(
                    List.of(
                            new LowerCaseItemProcessor(),
                            new After20YearsItemProcessor()
                    )
            )
            .build();
}
```
