# Custom ItemReader/ItemWriter 구현 방법 

Spring Batch에서 제공하는 ItemReader/ItemWriter를 사용하면 대부분의 경우 요구사항을 충족할 수 있지만 customizing이 필요한 경우가 있다.  
예를 들어 ItemReader 중에 LinkedList를 사용하고 싶다거나, Querydsl을 사용하고 싶은 경우가 있다.  
이런 경우에는 ItemReader/ItemWriter를 상속받아 구현하면 된다.  

QuerydslPagingItemReader는 실제로 존재하는 ItemReader가 아니기 때문에 AbstractPagingItemReader를 상속받아 구현해야 한다.  
AbstractPagingItemReader는 어댑터 패턴을 사용하여 PagingItemReader를 구현하고 있어서 doReadPage 만 구현하면 된다.  

지금부터 구현할 코드들은 배치 실습 레포지토리인 https://github.com/Baum-Study/cherhy-batch 경로에서 확인할 수 있다.