Job 및 Step의 시작과 종료 시, 특정 작업(로그 기록, 통계 정보 리포팅 등) 수행이 필요할 경우 Job, StepListener를 이용할 수 있다.

### Job Listener

- Job의 시작과 종료 때마다 특정 이벤트를 수행할 수 있다.
- JobExecutionListener의 beforeJob(), afterJob() 메서드를 정의하여 사용한다.
- 보통 최종 수행한 배치 Job의 통계정보를 리포팅할 때 사용된다.

### Step Listener

- Step의 시작과 종료 때마다 특정 이벤트를 수행할 수 있다.
- StepExecutionListener의 beforeStep(), afterStep() 메서드를 정의하여 사용한다.
- 보통 최종 수행한 스텝의 로깅 및 중간 집계 리포팅에 사용된다.