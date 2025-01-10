## SpringBatch Flow Controller 개요
Spring Batch에서는 Job Flow를 제어하기 위한 Flow Controller를 제공한다.  
Flow Controller는 여러 Step을 정의하고 조건에 따라 순서대로 실행하거나 특정 Step을 건너뛸 수 있도록 하는 기능이다.  

## Flow 컨트롤 방법
- next: 현재 Step이 성공적으로 종료되면 다음 Step으로 이동한다.
- from: 현재 Step으로 이동한다.
- to: 특정 Step으로 이동한다.
- on: ExitStatus에 따라 다음 Step으로 이동한다.
- stop: 현재 Flow를 종료한다.
- build: Flow를 생성한다.
- end: FlowBuilder를 종료한다.
- split: 병렬로 Step을 실행한다.
- listener: Step 실행 전, 후에 리스너를 등록한다.
- decider: Step 실행 전에 조건을 검사한다.
- fail: 특정 ExitStatus에 따라 Flow를 종료한다.