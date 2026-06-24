## System prompts
system prompt 를 활용해서 Claude 가 사용자의 입력에 어떻게 반응할지 커스터마이징이 가능하다.
Claude 의 어조, 스타일, 접근 방식 등을 구체적인 사용 사례에 맞게 튜닝해보자.

가령 수학 선생님 봇을 만든다고 생각해보자. "학생이 `5x + 2 = 3` 에서 `x` 를 어떻게 알아낼까요?" 라고 물어봤을 때 곧바로 답을 뱉어내는게 능사는 아닐 것이다.

- 처음에는 완전한 해답이 아닌 힌트를 주고,
- 문제를 단계별로 천천히 풀어낼 수 있도록 유도한다.
- 그리고 유사한 문제에 대한 해답을 예로 제시한다.
- 절대, 직접적인 답변을 주어선 안된다.
- 계산기를 이용하라는 허튼 대답을 해선 안된다.

이런 방식으로 Claude 가 응답할 스타일과 톤을 지정하는데 사용 되는 것이 system prompt.
이 역시도 Claude API 에서 `system` 파라미터에 주입해서 조정이 가능하다.

```python
system_prompt = """
You are a patient math tutor.
Do not directly answer a student's questions.
Guide them to a solution step by step.
"""

client.messages.create(
	model=model,
	messages=messages,
	max_tokens=1000,
	system=system_prompt
)
```

아래 prompt 를 통해 작성되는 함수 수준의 차이도 꼭 느껴보면 좋다.
prompt 가 얼마나 중요한지 깨닫게 되는 수준.

```python
messages = []

add_user_message(messages, "중복 문자를 검사하는 파이썬 함수를 작성해주세요.")

# answer = chat(messages = messages)
answer = chat(
	messages = messages,
	system="당신은 짧고 간결한 파이썬 코드를 매우 잘 작성하는 파이썬 엔지니어입니다."
)

answer
```


## Temperature
<img width="999" height="542" alt="Image" src="https://github.com/user-attachments/assets/70db3268-1c58-457b-b87a-aad7453dbbe7" />
Temperature 는 Claude 의 응답을 얼마나 명확하게 할지, 창의적이게 할지를 결정지을 수 있는 다이얼이다.
앞서 설명했듯, Claude 는 주어진 input 들을 잘게 token 화 하여, 주변 이웃 token 들을 조합하여 나올 수 있는 응답에 대해 확률을 분석한다.

여기서 0~1 사이 소수점으로 확률을 결정 짓는데 영향을 주는 것이 바로 Temperature 이다.
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/bc29f350-d17c-4ae0-8f58-83cbf32034bf" />
Temperature 를 낮게 주면 매우 결정론적이고, 높게 주면 확률을 고르게 분산시켜 다양하고 창의적인 답변이 나온다.

꼭 염두에 두어야 하는 것은, Temperature 는 확률을 균등화 시키는 것 일뿐, 꼭 "창의적인" 답변 생성을 보장하는 것은 절대 아니다.

```python
def chat(messages, system=None, temperature=1.0): 
	params = { 
		"model": model, 
		"max_tokens": 1000, 
		"messages": messages, 
		"temperature": temperature
	} 
	
	if system: 
		params["system"] = system 
	
	message = client.messages.create(**params)
	return message.content[0].text
```
