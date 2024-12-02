## FlatFileItemReader, FlatFileItemWriter 사용해보기

### FlatFileItemReader 개요

- Spring Batch에서 제공하는 기본적인 ItemReader
- 장점
  - 설정 및 사용 간편, 대규모 데이터 처리에 효율적
  - 고정 길이, 구분자 기반, 멀티라인 등 다양한 형식의 텍스트 파일을 읽을 수 있음
  - 토크나이저, 필터 등을 통해 기능 확장 가능
- 단점: 복잡한 데이터 구조 처리에는 부적합

### FlatFileItemReader 주요 구성 요소

- Resource: 읽을 텍스트 파일 지정
- LineMapper: 텍스트 파일의 각 라인을 Item으로 변환
- LineTokenizer: 텍스트 파일의 각 라인을 토큰으로 분리
- FieldSetMapper: 토큰을 Item의 속성에 매핑
- SkippableLineMapper: 오류 발생 시 해당 라인 스킵
- LineCallbackHandler: 라인별로 처리 수행
- ReadListener: 읽기 시작, 종료, 오류 발생 등의 이벤트 처리

### FlatFileItemWriter 개요

- Spring Batch에서 제공하는 ItemWriter 인터페이스 구현 클래스
- 데이터를 텍스트 파일로 출력하는 데 사용됨

### FlatFileItemWriter 구성 요소

- Resource: 출력 파일 경로 지정
- LineAggregator: Item을 문자열로 변환
- HeaderCallback: 출력 파일 헤더 작성
- FooterCallback: 출력 파일 푸터 작성
- Delimiter: 항목 사이 구분자 지정
- AppendMode: 기존 파일에 추가할지 여부 지정
- 장점
    - 간편성: 텍스트 파일로 데이터를 출력하는 간편한 방법 제공
    - 유연성: 다양한 설정을 통해 원하는 형식으로 출력 파일 생성 가능
    - 성능: 대량의 데이터를 빠르게 출력 가능
- 단점
    - 형식 제약: 텍스트 파일 형식만 지원
    - 복잡한 구조: 복잡한 구조의 데이터를 출력할 경우 설정이 복잡해질 수 있음
    - 오류 가능성: 설정 오류 시 출력 파일이 손상될 수 있음
