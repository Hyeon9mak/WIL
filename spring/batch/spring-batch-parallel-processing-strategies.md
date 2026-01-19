## Overview  
Spring Batch 는 주로 대용량 데이터를 효율적으로 자동 처리하기 위해 사용한다.  
batch 작업 특성상 실시간 성이 떨어지는 경우가 많아 성능이 크게 중요하지 않지만, 데이터 양이 너무 많아 처리 시간이 지나치게 오래 걸린다던지, 다른 작업과 연결되어 최소한의 처리 시간이 요구되는 경우도 분명히 존재한다.
이 때 Spring Batch 가 제공하는 병렬 처리 전략을 활용하면 작업 처리 시간을 크게 단축시킬 수 있다.  
  
이번 글에서 Spring Batch 병렬 처리 전략을 알아보기 위한 목차는 아래와 같다.  
  
- Spring Batch Step 에서 데이터를 처리하는 단위  
- Spring Batch 병렬 처리 전략 필요성  
- Spring Batch 병렬 처리 전략들의 특징  
  - Multi-threaded Step  
  - AsyncItemProcessor  
  - Partitioning  
- 시나리오별 권장 전략  
  
<br>  

## Spring Batch Step 에서 데이터를 처리하는 단위  
  
> [Spring Batch Documentation - Chunk-oriented Processing](https://docs.spring.io/spring-batch/reference/step/chunk-oriented-processing.html)  
  
![image](https://docs.spring.io/spring-batch/reference/_images/chunk-oriented-processing-with-item-processor.png)  
  
대부분의 대량 데이터를 처리할 땐 Chunk 기반의 처리 방식(Chunk-oriented Processing)을 사용한다.  
Chunk 기반 처리 방식은 Step 이 데이터를 일정 단위로 읽고(Read), 처리하고(Process), 기록(Write)하는 방식을 의미한다.  
그림을 자세히 살펴보면 `read() - read() - process() - process() - write()` 와 같은 흐름을 갖고 있다.  
즉 Chunk 기반 처리 방식에서 각 작업들은 아래와 같은 특징을 갖는다.  
  
- Reader: Item 단위로 하나씩 데이터를 읽어온다. 이를 Chunk 수 만큼 반복한다.  
- Processor: Reader 가 읽어온 Item 단위 데이터를 가공한다. Processor 가 없는 경우 Reader 가 읽어온 Item 을 그대로 Write 로 전달한다.  
- Writer: Item 들을 모아 Chunk 단위로 한꺼번에 기록한다.  
  
  
### ItemReader Paging 처리  
이 때 Reader 에서 실제 Item(데이터) 단위로 RDB 에 query 를 수행하는 것은 지나치게 비효율적일 것이다.  
그렇다고 모든 데이터를 한꺼번에 읽어오는 것 또한 메모리에 부담이다.  
때문에 Reader 내부적으로는 Paging 처리를 이용해서 적정한 크기의 데이터를 읽어오도록 한다.  
이를 `List<Item>` 단위로 Processor 에게 넘길 수도 있고, Iterator 를 이용해 Item 단위로 하나씩 넘길 수도 있다.  
  
아래와 같은 예시를 들어보자.  
  
- RDB 에 10,000 건의 데이터가 존재한다.  
- Chunk 크기를 1,000 으로 설정한다.  
- Reader 는 Paging 처리를 이용해 한 번에 100 건의 데이터를 읽어온다.  
  
이 때 Iterator 를 이용해 하나씩 넘기는 과정을 살펴보면 다음과 같다.  
  
1. Reader 가 100 건의 데이터를 읽어온다.  
2. Reader 가 읽어온 100 건의 데이터를 Item 단위로 하나씩 Step 에 전달한다.  
3. Step 은 Item 단위로 Processor 를 호출해 데이터를 가공한다.  
4. Step 은 가공된 Item 들을 모아 Chunk 크기(1,000) 가 될 때까지 대기한다.  
5. Step 은 Chunk 크기(1,000) 가 될 때마다 Writer 를 호출해 데이터를 기록한다.  
6. 위 과정을 모든 데이터(10,000 건) 가 처리될 때까지 반복한다.  
  
<br>  
  
## Spring Batch 병렬 처리 전략 필요성  
  
<br>  
  
## Multi-threaded Step  
  
<br>  
  
## AsyncItemProcessor  
  
<br>  
  
## Partitioning  
  
<br>  
  
## 시나리오별 권장 전략  
  
<br>  
  
## References  
 - [Spring Batch Documentation - Chunk-oriented Processing](https://docs.spring.io/spring-batch/reference/step/chunk-oriented-processing.html)