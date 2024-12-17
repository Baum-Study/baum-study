# Spring Batch Study

저희는 Spring Batch 5 version을 기준으로 스터디를 진행합니다.

## 스터디 방식

블로그 첫번째 챕터 : https://devocean.sk.com/experts/techBoardDetail.do?ID=166164

1. 매주 위 블로그 읽는다.
2. chapter 별로 글을 정리한다.
3. 실습을 진행할 배치 프로젝트를 만들어서 블로그 따라서 코드를 작성한다.
4. 실습을 할 때 언어, 데이터베이스와 같은 환경은 자유롭게 선택한다.
5. 실습을 하고 난 뒤 커밋 메시지와 브랜치를 통해서 코드 리뷰를 진행한다.
   1. ex) 브랜치 : `chapter1`
   2. ex) 커밋 메시지 : `chapter1 실습`
   3. workflow : `chapter1` 브랜치 생성 -> `chapter1` 브랜치에서 작업 -> `main` 브랜치로 merge -> 다음주에 master 브랜치에서 `chapter2` 브랜치 생성

## 실습 방식

오프라인 스터디는 매주 화요일 오전 8시 30분부터 9시 25분까지 진행합니다.  
발표는 2명이 제비뽑기를 통해서 발표를 진행합니다.  
몰랐던 부분이나 궁금한 부분을 각자 한명을 지목해서 질문하는 방식입니다.  
실습, 코드 리뷰는 질문이 끝나면 바로 진행합니다.  
그리고 배치 스터디는 대용량의 데이터를 처리하는 것이 목적이기 때문에 실습을 진행할 때는 대용량의 데이터를 사용합니다.  
대용량의 데이터를 테스트 하는 방식은 `철희`가 제공해주는 데이터를 사용합니다.   
  ㄴ-> 추후에 더미 데이터 생성 및 테스트 케이스 작성 방식 공유함  
대략 데이터의 양은 100만 ~ 1000만개 정도로 진행합니다.  

## 실습 프로젝트 URL

철희 : `https://github.com/Baum-Study/cherhy-batch`
재영 : `https://github.com/Baum-Study/jaeyoung-batch`

## 12월 10일 5주차 과제
```markdown
더미 데이터 생성 로직은 @정철희 주임이 올립니다.

판매자 별로 일 정산, 월 정산 만들기. 테스트 코드에서 실행 할 수 있도록 합니다.
간단한 결제 데이터 csv 파일로 만들기. -> pk, 판매자 아이디, 상품 아이디, 상품 명, 금액, 결제 일자
파일을 읽어서 DB에 일 정산, 월 정산 저장하기
배치 실행된거 스크린샷 해서 올리기
```

## 12월 10일 5주차 과제 더미 데이터 생성 로직
```Kotlin
testImplementation("org.jeasy:easy-random-core:5.0.0")

internal fun <T, R> EasyRandomParameters.randomize(
    property: KProperty<T>,
    randomizer: Randomizer<R>,
) =
    randomize(
        { it.name == property.name },
        randomizer,
    )

internal fun <T> Number.mapParallel(
    block: () -> T,
) =
    LongStream.range(0, this.toLong())
        .parallel()
        .mapToObj {
            block()
        }
        .toList()


data class Payment(
    val id: Long,
    val sellerId: Long,
    val productId: Long,
    val productName: String,
    val price: BigDecimal,
    val paymentDate: LocalDateTime,
)

internal object PaymentFactory {
    fun generateRandom(): Payment {
        val parameter = EasyRandomParameters()
            .excludeField {
                it.name == Payment::id.name
            }
            .randomize(Payment::sellerId) {
                (1..100).random()
            }
            .randomize(Payment::productId) {
                (1..100).random()
            }
            .randomize(Payment::productName) {
                StringRandomizer(1, 10, Random.nextLong(1, 100)).randomValue
            }
            .randomize(Payment::price) {
                (100..1_000_000).random().toBigDecimal()
            }
            .randomize(Payment::paymentDate) {
                LocalDateTime.of(
                    Random.nextInt(2020, 2022),
                    Random.nextInt(1, 13),
                    Random.nextInt(1, 29),
                    Random.nextInt(0, 24),
                    Random.nextInt(0, 60),
                    Random.nextInt(0, 60),
                )
            }
        return EasyRandom(parameter).nextObject(Payment::class.java)
    }
}

fun main() {
    val payments = 1_000_000L.mapParallel(PaymentFactory::generateRandom)
}
```

## 12월 17일 6주차 과제
```markdown
JpaPagingItemReader로 DB내용을 읽고, JpaItemWriter로 DB에 쓰기

JpaPagingItemReader로 회원 테이블에서 데이터를 읽어와서, JpaItemWriter로 회원 테이블에 데이터를 삭제하고 CASCADE로 연결된 IMAGE 테이블에 데이터를 삭제한다.
```