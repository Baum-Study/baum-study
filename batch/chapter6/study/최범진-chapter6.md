# **JpaPagingItemReader**

- Spring Batch에서 제공하는 ItemReader로, JPA를 사용하여 데이터베이스로부터 데이터를 페이지 단위로 묶습니다.
- JPA 기능 활용: JPA 엔티티 기반 데이터 처리, 객체 매핑 자동화 등 JPA의 다양한 기능을 활용할 수 있습니다.
- 쿼리 최적화: JPA 쿼리 기능을 사용하여 최적화된 데이터 읽기가 가능합니다.
- 커서 제어: JPA Criteria API를 사용하여 데이터 순회를 제어할 수 있습니다.

## 주요 구성 요소

- **EntityManagerFactory**: JPA 엔티티 매니저 팩토리를 설정합니다.
- **JpaQueryProvider**: 데이터를 읽을 JPA 쿼리를 제공합니다.
- **PageSize**: 페이지 크기를 설정합니다.
- **SkippableItemReader**: 오류 발생 시 해당 Item을 건너뛸 수 있도록 합니다.
- **ReadListener**: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 합니다.
- **SaveStateCallback**: 작업 중단 시 현재 상태를 저장하여 재시작 시 이어서 처리할 수 있도록 합니다.

# **JpaItemWriter**

- Spring Batch에서 제공하는 ItemWriter 인터페이스를 구현한 클래스입니다.
- JPA를 통해 데이터를 데이터베이스에 저장하는 데 사용됩니다.

## 구성 요소

- **EntityManagerFactory**: JPA EntityManager 생성을 위한 팩토리 객체입니다.
- **JpaQueryProvider**: 저장할 엔티티를 위한 JPA 쿼리를 생성하는 역할을 합니다.

### 장점

- **ORM 연동**: JPA를 통해 다양한 데이터베이스에 데이터를 저장할 수 있습니다.
- **객체 매핑**: 엔티티 객체를 직접 저장하여 코드 간결성을 높일 수 있습니다.
- **유연성**: 다양한 설정을 통해 원하는 방식으로 데이터를 저장할 수 있습니다.

### 단점

- **설정 복잡성**: JPA 설정이 복잡할 수 있습니다.
- **오류 가능성**: 설정 오류 시 데이터 손상 가능성이 있습니다.
