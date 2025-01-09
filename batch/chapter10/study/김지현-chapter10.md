## SpringBatch Flow Controller
스프링 배치에서 배치 수행 Flow 컨트롤은 여러 Step을 정의하고 조건에 따라 순서대로 실행하거나 특정 Step을 건너뛸 수 있도록 하는 기능이다. <br>
이는 FlowBuilder API를 사용하여 설정된다. <br>

## Flow 컨트롤 방법
##### `next`
- 현재 Step이 성공적으로 종료되면 다음 Step으로 이동한다.
- 계속해서 추가될 수 있으며, start -> next -> next 순으로 진행되도록 한다.

##### `from`
- 특정 Step에서 현재 Step으로 이동한다.

##### `on`
- 특정 Step의 종료 조건(ExitStatus)에 따라 어떠한 Step으로 이동할 건지를 결정한다.

##### `to`
- 특정 Step으로 이동한다.

##### `stop`
- 특정 Step의 작업 결과 상태를 보고 현재 Flow를 종료할지 결정한다.

##### `end`
- FlowBuilder를 종료한다.

### 요약
start, next를 통해서 배치 job의 여러 step을 이어서 수행할 수 있다. <br>
on과 from을 통해서 특정 조건에 따라 스텝을 분기 처리할 수 있다. <br>
stop을 통해서 특정 조건에 따라 배치 작업을 종료할 수 있다. <br>
