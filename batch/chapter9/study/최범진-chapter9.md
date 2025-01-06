# **QuerydslPagingItemReader**

- Querydsl은 SpringBatch의 공식 ItemReader가 아니므로 AbstractPagingItemReader를 이용하여 Querydsl을 활용할 수 있도록 ItemReader를 만들 수 있습니다.
- Querydsl의 강력하고 유연한 쿼리 기능을 사용하여 데이터를 효율적으로 읽을 수 있습니다.
- JPA 엔티티에 직접 의존하지 않고 추상화된 쿼리를 작성하여 코드 유지 관리성을 높일 수 있습니다.
- 런타임 시 조건에 따라 동적으로 쿼리를 생성할 수 있습니다.
- 일반적인 QueryDSL 페이징은 쉽게 구현할 수 있으며, 필요하다면 doReadPage()의 내용을 조금 수정하면 원하는 방향으로 개발할 수 있습니다.

---

# **CustomItemWriter 개요**

- Spring Batch에서 제공하는 기본 ItemWriter 인터페이스를 구현하여 직접 작성한 ItemWriter 클래스입니다.
- 기본 ItemWriter 클래스로는 제공되지 않는 특정 기능을 구현할 때 사용됩니다.

## **구성 요소**

- ItemWriter 인터페이스를 구현하여 write() 메소드를 작성하고 원하는 처리를 수행합니다.
- 필요한 라이브러리와 객체를 선언하여 사용할 준비를 합니다.
- write() 메소드에서 데이터 처리 로직을 구현합니다.

## **장점**

- CustomItemWriter는 기본 ItemWriter로 제공되지 않는 특정 기능을 유연하게 구현할 수 있는 장점이 있습니다.
- 다양한 방식으로 데이터 처리를 확장할 수 있어 확장성이 뛰어납니다.
- 데이터 처리 과정을 완벽하게 제어할 수 있습니다.

## **단점**

- 개발 과정이 기본 ItemWriter 클래스보다 복잡하여 개발 난이도가 높아질 수 있습니다.
- 테스트 작성이 더 어렵거나 시간이 걸릴 수 있습니다
- 문제 발생 시 디버깅이 더 어려운 경우가 있을 수 있습니다.
