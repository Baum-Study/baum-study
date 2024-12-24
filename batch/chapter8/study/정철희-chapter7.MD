# CompositeItemProcessor 으로 여러단계에 걸쳐 데이터 Transform하기

CompositeItemProcessor는 Spring Batch에서 제공하는 ItemProcessor 구현체다.  
여러 단계에 걸쳐 데이터를 Processing 할 때 사용한다.  

 - Delegates: 처리를 수행할 ItemProcessor 목록
 - TransactionAttribute: 트랜잭션 속성을 설정

장점으로는 여러 단계로 나누어 처리를 수행하기 때문에 재사용성이 높고(다른 Job에서 사용 가능), 다양한 ItemProcessor를 `조합`해서 사용할 수 있다.  
또 단점으로는 여러 개의 Processor를 설정하고 관리해야 하기 때문에 설정이 복잡해질 수 있고 여러 처리 과정을 호출하므로 성능이 저하될 수 있다.  