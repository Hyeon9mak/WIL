- prompt engineering 과 더불어, prompt evaluation 역시 매우 중요하다.
- prompt 가 얼마나 유의미한지 알아낼 수 있는 객관적인 지표.
- 평가하는 방법을 먼저 알아야 그 다음 엔지니어링을 의미있게 진행할 수 있다.

## Prompt Engineering vs Prompt Evaluation
### Prompt Engineering
prompt engineering 은 prompt 를 잘 만들기 위한 방법, 기법들.
- Multishot prompting
- XML tag 를 이용한 구조화
- 그 외 많은 모범 사례들
### Prompt Evaluation
prompt 작성법이 아닌 자동화된 테스트를 이용해 효과 측정
- 예상 답변과 일치하는지 검증
- 같은 prompt 의 여러 버전 비교
- 오류 검토 및 출력

## 결국 TDD
많은 엔지니어들이 다음과 같은 방법을 이용해서 평가를 수행한다.

1. prompt 를 만들고 해피 케이스만 테스트 후 완료 판단.
2. prompt 를 몇 차례 테스트 하고 한두 가지 엣지 케이스 처리 후 완료 판단.
3. 평가 파이프라인에 통과 시켜 객관적인 지표 획득

1,2 는 많은 엔지니어들이 빠지는 함정이다. 훨씬 더 크고 많은 엣지 케이스를 처리할 방법이 없다.
결국, TDD 와 같이 **Evaluation-First Approach** 방식을 채택해야한다.

## A typical eval workflow
가장 먼저 짚고 넘어가야 할 것.

1. 매우 다양한 방법으로 workflow 를 구성할 수 있다. 산업 표준은 없다.
2. 매우 다양한 오픈 소스 패키지와 옵션들이 존재한다.

이로 인해, **꼭 workflow 를 사용자 커스텀 형태로 만들 필요가 없다.** 잘 만들어진 workflow 를 활용하는 것 또한 좋은 방법이다. 단지 학습을 위해 직접 만들어보는 것.

### 1. Draft a prompt
평가에 활용될 prompt 를 준비한다.

```python
prompt = f"""
사용자 질문에 대답해주세요.
{question}
"""
```

### 2. Create an Eval Dataset
prompt 를 평가할 수 있는 dataset 을 준비한다.

- 2+2 의 결과는 무엇인가요?
- 오트밀은 어떻게 만들죠?
- 지구에서 달까지의 거리는 어느 정도인가요?

이런 dataset 을 직접 만들 수도 있고, 당연히 LLM 을 활용할 수도 있다.

### 3. Feed Through Claude
모든 dataset 을 prompt 와 합쳐서 Claude 에게 직접 던져 응답을 받는다.

```python
prompt = f"""
사용자 질문에 대답해주세요.
2+2 의 결과는 무엇인가요?
"""
```
```
2 + 2 = 4
```

### 4. Feed Through a Grader
채점기를 통해서 답변 품질을 평가한다. 보통 1~10점 사이의 점수를 부여한다.
이 점수들의 평균을 내어서 prompt 수준이 어느 정도인지 잡아낸다.

> 그럼 채점기는 어떤 걸 기준으로 어떻게 점수를 내는가? 이에 대해서는 점차 알아갈 것이다. 일단 평가 파이프라인을 통해 prompt 를 개선하는 과정에 집중하자.

평균을 깎아 먹는 dataset 에 집중하자. 그 dataset 을 위한 prompt 세부 정책을 추가하는 것으로 평가 점수를 더 끌어올릴 수 있다.

### 5. Change Prompt and Repeat
prompt 를 수정하고 1~4 까지의 과정을 다시 반복한다. 그 과정에서 평균 점수가 개선되었는지를 체크한다.

```python
prompt = f"""
사용자 질문에 대답해주세요.

{question}

응답에 더 많은 세부사항을 포함시켜주세요.
"""
```

결국 서로 다른 prompt 버전을 수치로 비교하고, 점수가 가장 좋은 버전의 prompt 를 채택하면 된다.
일련의 파이프라인을 계속해서 반복하는 것이 가장 중요하다.
**이 방식은 엔지니어링에서 "추측" 을 제거하고, "변형" 이 아닌 "개선" 을 진행하는 과정이 된다.**