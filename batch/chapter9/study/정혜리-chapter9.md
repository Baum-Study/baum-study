# Custom ItemReader/ItemWriter 구현하기

스프링 배치에서 지원하는 ItemReader와 ItemWriter를 사용하지 않고
비즈니스 로직에 딱 맞는 ItemReader/ItemWriter 를 커스텀하는 방법이다.

<br />

## QuerydslPagingItemReader
### 구현 예시
QueryDsl을 이용해 Paging 데이터 값을 읽어들이는 QuerydslPagingItemReader 구현하기

### QuerydslPagingItemReader 개요
- Querydsl은 SpringBatch의 공식 ItemReader가 아님
- AbstractPagingITemReader를 이용해 Querydsl을 활용하는 ItemReader 구현

### 장점
- JPA 엔티티 추상화: JPA엔티티에 직접 의존하지 않고 추상화된 쿼리를 작성해 코드 유지 관리성을 높일 수 있음
- 동적 쿼리 지원: 런타임 시 조건에 따른 동적 쿼리 생성 가능


### 예시 코드
```kotlin
class QuerydslPagingItemReader<T>(
    private val name: String = QuerydslPagingItemReader::class.simpleName!!,
    private val entityManagerFactory: EntityManagerFactory,
    private val querySupplier: Function<JPAQueryFactory, JPAQuery<T>>,
    private val alwaysReadFromZero: Boolean = false,
    private val chunkSize: Int = 1000,
) : AbstractPagingItemReader<T>() {

    private val entityManager = entityManagerFactory.createEntityManager()

    init {
        pageSize = chunkSize
        super.setName(name)
    }

    override fun doClose() {
        entityManager.close()
        super.doClose()
    }

    override fun doReadPage() {
        initQueryResult()

        val jpaQueryFactory = JPAQueryFactory(entityManager)
        val offset = if(alwaysReadFromZero) 0 else (page * pageSize).toLong()

        val query: JPAQuery<T> = querySupplier.apply(jpaQueryFactory)
            .offset(offset)
            .limit(pageSize.toLong())

        for(entity: T in query.fetch()) {
            entityManager.detach(entity)
            results.add(entity)
        }
    }

    private fun initQueryResult() {
        if(CollectionUtils.isEmpty(results)) {
            results = CopyOnWriteArrayList()
        } else {
            results.clear()
        }
    }
}
```

AbstractPagingItemReader 를 상속해 구현할 경우, doRead() 함수만 구현하면 된다.
코틀린의 경우 빌더 패턴이 필요하지 않아 구현하지 않음

#### 생성자
- name: itemReader를 구분하기 위한 이름
- entityManagerFactory: JPA를 사용하기 위해 entityManagerFactory를 전달받음
- Function<JpaQueryFactory, JpaQuery>: JPAQuery를 생성하기 위한 Functional Interface
- chunkSize: 한 번에 페이징 처리할 크기
- alwaysReadFromZero: 항상 0부터 페이징을 읽을 지 여부를 지정. 페이징 처리된 데이터를 수정할 경우 누락을 방지하기 위해 사용된다

#### doClose()
- entityManager 자원 해제를 위해 재정의

#### doRead()
- initQueryResult
  - 매 페이징 결과만 반환하기 위한 초기화 작업
- JPAQueryFactory를 통해 queryDSL에 적용
- alwaysReadFromZero 가 아니면 offset을 계산한다
- querySupplier.apply:
  - 생성자로 전달받은 querySupplier에 JPAQueryFactory를 적용해 JPAQuery를 생성
  - 페이징을 위해 계산된 offset/limit 값을 지정
- fetch
  - 조회 결과를 반환
  - 이때 entityManager detch 을 통해 실제 DB에 반영되지 않도록 영속성 객체에서 제외

<br /><br />

### CustomItemWriter 개요
Spring Batch 에서 제공하는 ItemWriter 인터페이스를 구현한다.
기본 ItemWriter로 제공되지 않는 특정 기능을 구현하기 위해 사용

### 예시 코드
- customerService
```kotlin
@Service
class CustomService {

    private val logger = KotlinLogging.logger {}

    fun processToOtherService(customer: Customer): Map<String, String> {
        logger.info { "Call API to OtherService....." }
        return mapOf("code" to "200", "message" to "ok")
    }
}
```

- customItemWriter
```kotlin
@Component
class CustomItemWriter(
    private val customService: CustomService,
) : ItemWriter<Customer> {

    override fun write(chunk: Chunk<out Customer>) {
        for(customer in chunk) {
            customService.processToOtherService(customer)
        }
    }

}
```

- 결과 로깅
```text
2025-01-06T23:37:56.775+09:00  INFO 84717 --- [hyeri-batch] [           main] c.h.hyeribatch.chapter09.CustomService   : Call API to OtherService.....
2025-01-06T23:37:56.775+09:00  INFO 84717 --- [hyeri-batch] [           main] c.h.hyeribatch.chapter09.CustomService   : Call API to OtherService.....
2025-01-06T23:37:56.782+09:00  INFO 84717 --- [hyeri-batch] [           main] o.s.batch.core.step.AbstractStep         : Step: [customerQueryDslPagingStep] executed in 204ms
2025-01-06T23:37:56.796+09:00  INFO 84717 --- [hyeri-batch] [           main] o.s.b.c.l.support.SimpleJobLauncher      : Job: [SimpleJob: [name=customerQueryDslPagingJob]] completed with the following parameters: [{'run.id':'{value=10, type=class java.lang.Long, identifying=true}'}] and the following status: [COMPLETED] in 244ms
```