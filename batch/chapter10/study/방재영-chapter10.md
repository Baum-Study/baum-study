## SpringBatch Flow Controller
- SpringBatch Flow Controller는 조건에 따라 Step을 순서대로 실행하거나 건너뛸 수 있도록 하는 기능으로 FlowBuilder API를 사용하여 설정됩니다.

<br/>

### Flow 컨트롤 방법
- next: 현재 Step이 성공적으로 종료되면 다음 Step으로 이동
```java
@Bean
public Job job() {
    return jobBuilderFactory.get("job")
        .start(step1())
        .next(step2())
        .end()
        .build();
}
```
- from: 특정 Step에서 현재 Step으로 이동
- on: 특정 ExitStatus에 따라 다음 Step을 결정
- to: 특정 Step으로 이동
```java
@Bean
public Job job() {
    return jobBuilderFactory.get("job")
        .start(step1())
        .on("FAILED").to(step3())
        .from(step1()).on("COMPLETED").to(step2())
        .end()
        .build();
}
```
- stop: 현재 Flow를 종료
- end: FlowBuilder를 종료
```java
@Bean
public Job job() {
    return jobBuilderFactory.get("job")
        .start(step1())
        .on("FAILED").stop()
        .end()
        .build();
}
```

---

```java
@Bean
public Flow flow1(Step step1, Step step2) {
    return new FlowBuilder<SimpleFlow>("flow1")
            .start(step1)
            .next(step2)
            .build();
}

@Bean
public Flow flow2(Step step3) {
    return new FlowBuilder<SimpleFlow>("flow2")
            .start(step3)
            .build();
}
```

---

```java
@Bean
public Job job(JobRepository jobRepository, Flow flow1, Flow flow2, Step step4) {
    return new JobBuilder("job", jobRepository)
            .start(flow1)
            .split(new SimpleAsyncTaskExecutor())
            .add(flow2)
            .next(step4)
            .end()
            .build();
}

@Bean
public Job job(JobRepository jobRepository, Flow flow1, Step step3) {
    return new JobBuilder("job", jobRepository)
            .start(flow1)
            .next(step3)
            .end()
            .build();
}

@Bean
public Flow flow1(Step step1, Step step2) {
    return new FlowBuilder<SimpleFlow>("flow1")
            .start(step1)
            .next(step2)
            .build();
}
```