# CompositeItemProcessor 를 통한 단계별 Processor 처리

## CompositeItemProcessor
CompositeItemProcessor는 Spring Batch에서 제공하는 ItemProcessor 구현 클래스.

여러 개의 ItemProcessor를 단계별로 처리할 수 있도록 한다.

### 구성요소
- Delegates: 처리를 수행할 ItemProcessor 목록
- TransactionAttribute: 트랜잭션 속성 설정

### 장점
- 단계별 처리: 여러 단계로 나누어 처리를 수행해 코드를 명확하고 이해하기 쉽게 만들 수 있다.
- 재사용 가능성: Processor를 다른 Job에서도 재사용할 수 있다.
- 유연성: 다양한 ItemProcessorㄹ를 조합해 원하는 처리 과정을 구현할 수 있다.

### 단점
- 설정 복잡성: 여러 개의 Processor를 설정 & 관리해야 하므로 설정이 복잡할 수 있음
- 성능 저하: 여러 단계의 처리과정을 거치므로 성능이 저하될 수 있음

### 샘플코드 
- 프로세서1: 이름과 성별을 소문자로 변환
- 프로세서2: 나이에 20 더하기

```kotlin

```