# FlatFileItemReader로 단순 파일 읽고, FlatFileItemWriter로 파일에 쓰기

## FlatFileItemReader 개요

FlatFileItemReader는 Spring Batch에서 제공하는 ItemReader 구현체 중 하나로 텍스트 파일로부터 데이터를 읽어옵니다.  
텍스트 파일에서 길이, 구분자, 라인 별로 데이터를 읽어올 수 있습니다.  
간단하며 성능이 좋고 다양한 형식의 텍스트 파일을 지원하지만 복잡한 데이터 구조 처리에는 적합하지 않습니다.   
토크나이저나 필터 등을 통해 기능을 확장할 수 있습니다.
여기서 토크나이저는 구분자로 텍스트 파일의 라인을 토큰으로 분리하는 역할을 하고 필터는 토큰을 Item의 속성에 매핑하는 역할을 합니다.  

우리가 자주 사용하는 flat file reader를 구성하는 요소들을 살펴보겠습니다.

 - Resource: 읽을 텍스트 파일의 위치를 지정해서 읽어옵니다.
 - LineMapper: 텍스트 파일의 각 라인을 Item으로 변환하는 역할을 합니다.
 - LineTokenizer: 텍스트 파일의 각 라인을 토큰으로 분리하는 역할을 합니다.
 - FieldSetMapper: 토큰을 Item의 속성에 매핑하는 역할을 합니다.
 - SkippableLineMapper: 오류 발생 시 해당 라인을 건너뛸 수 있도록 합니다.
 - LineCallbackHandler: 라인별로 처리를 수행할 수 있도록 합니다.
 - ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트를 처리할 수 있도록 합니다.

## FlatFileItemWriter 개요

FlatFileItemWriter는 Spring Batch에서 제공하는 ItemWriter 구현체 중 하나로 텍스트 파일로 데이터를 출력합니다.  
빠르고(대용량 처리에 적합) 간단하게 텍스트 파일로 데이터를 출력할 때 사용합니다.  
단점은 텍스트 파일로만 출력이 가능하고 복잡한 데이터 구조 처리에는 적합하지 않습니다.    

우리가 자주 사용하는 flat file writer를 구성하는 요소들을 살펴보겠습니다.

 - Resource: 출력할 파일의 경로를 지정합니다.
 - LineAggregator: Item을 문자열로 변환하는 역할을 합니다.
 - HeaderCallback: 출력 파일 헤더를 작성하는 역할을 합니다.
 - FooterCallback: 출력 파일 푸터를 작성하는 역할을 합니다.
 - Delimiter: 항목 사이 구분자를 지정합니다.
 - AppendMode: 기존 파일에 추가할지 여부를 지정합니다.

TODO: 이번 발표때는 코드에 대한 리뷰도 같이 하면 될거같슴다~