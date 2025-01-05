## QuerydslPagingItemReader
AbstractPagingItemReader를 이용하여 Querydsl 을 활용할 수 있도록 ItemReader를 만들 수 있다. <br>
Querydsl 기능을 사용하여 데이터를 효율적으로 읽을 수 있다. <br>
JPA 엔티티 추상화: JPA 엔티티에 직접 의존하지 않고 추상화된 쿼리를 작성하여 코드 유지 관리성을 높일 수 있다. <br>
동적 쿼리 지원: 런타임 시 조건에 따라 동적으로 쿼리를 생성할 수 있다. <br>
필요 시 doReadPage()의 내용을 수정해서 원하는 방식으로 개발할 수 있다. <br>

## CustomItemWriter
CustomItemWriter는 Spring Batch에서 제공하는 기본 ItemWriter 인터페이스를 구현하여 직접 작성한 ItemWriter 클래스이다. <br>
기본 ItemWriter 클래스로는 제공되지 않는 특정 기능을 구현할 때 사용된다. <br>

### 구성 요소
ItemWriter 인터페이스 구현: write() 메소드를 구현하여 원하는 처리를 수행한다. <br>
필요한 라이브러리 및 객체 선언: 사용할 라이브러리 및 객체를 선언한다. <br>
데이터 처리 로직 구현: write() 메소드에서 데이터 처리 로직을 구현한다. <br>

### 장점
유연성: 기본 ItemWriter 클래스로는 제공되지 않는 특정 기능을 구현할 수 있다. <br>
확장성: 다양한 방식으로 데이터 처리를 확장할 수 있다. <br>
제어 가능성: 데이터 처리 과정을 완벽하게 제어할 수 있다. <br>

### 단점
개발 복잡성: 기본 ItemWriter 클래스보다 개발 과정이 더 복잡하다. <br>
테스트 어려움: 테스트 작성이 더 어려울 수 있다. <br>
디버깅 어려움: 문제 발생 시 디버깅이 더 어려울 수 있다. <br>
