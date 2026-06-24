## Python 으로 간단한 요청 보내기
### python 기준 SDK import
```python
from dotenv import load_dotenv
load_dotenv()

from anthropic import Anthropic
client = Anthropic()
```

### message 를 보내는 함수
```python
model = "claude-sonnet-4-0"

client.messages.create(
	model = model,
	max_tokens = 1000,
	messages = [
		# ...
	]
)
```

- LLM model 이 `max_tokens` 를 꼭 맞추려고 하지 않는다.
	- 어느 정도 멈추게 만드는 안전 장치 정도, max_tokens 을 넘어서 소비할 수도 있다.
- messages 가 list 형태인 이유.
	- 사용자가 작성한 user message 와 Claude 가 생성했던 assistant message 를 파라미터로 추가할 수 있다.

```python
{
	"role": "user", 
	"content": "What is quantum computing? Answer in one sentence"
},
{
	"role": "assistant", 
	"content": "..."
}
```

대략 이런 느낌.
응답 객체에는 많은 정보가 담겨져 있지만 아래와 같은 형태로 응답만 발라 낼 수도 있다.

```python
message.content[0].text
```

### 최종 형상

```python
from dotenv import load_dotenv
load_dotenv()

from anthropic import Anthropic
client = Anthropic()

model = "claude-sonnet-4-0"

message = client.messages.create(
	model = model,
	max_tokens = 1000,
	messages = [
		{
			"role": "user",
			"content": "양자 컴퓨터에 대해 간략하게 설명해주세요. 한 문장으로."
		}
	]
)

message.content[0].text
```
```
'양자 컴퓨터는 0과 1을 동시에 존재할 수 있는 양자비트(큐비트)를 이용해 기존 컴퓨터보다 특정 문제를 기하급수적으로 빠르게 처리할 수 있는 컴퓨터입니다.'
```

## Python 으로 멀티턴 대화 이어 나가기
Anthropic API 와 Claude 를 사용할 때 반드시 이해해야 할 중요한 개념이 있다.
**Claude 는 대화 기록을 저장하지 않는다. 요청하는 모든 것은 완전히 독립적이며, 이전 내용은 전혀 기억하지 않는다.**

단순하게 Claude 에게 비슷한 결의 질문을 하면, Claude 는 이를 인지하지 못하고 완전히 새로운 답변을 응답한다.

```
user: 양자 컴퓨터에 대해 설명해주세요.
assistant: 양자 컴퓨터는...

user: 다른 문장으로 변환해주세요.
assistant: 어떤 문장을 변환해드릴까요?
```

아마도 Claude Code 또한 `.claude/` 하위 메모리 공간에 압축, 적재해서 context 유지중일 것.

고로 멀티턴 대화를 이어 나가기 위해선 사용자가 직접 메모리 상에서 관리를 진행해야한다.

```python
def add_user_message(messages, text):
	user_message = {"role": "user", "content": text}
	messages.append(user_message)

def add_assistant_message(messages, text):
	assistant_message = {"role": "assistant", "content": text} 
	messages.append(assistant_message) 
	
def chat(messages): 
	message = client.messages.create( 
		model=model, 
		max_tokens=1000, 
		messages=messages,
	) 
	return message.content[0].text
```

아래와 같은 방식으로 대화를 진행하면서 `messages` list 에 계속 append 를 더해간다.

```python
# Start with an empty message list 
messages = [] 

# Add the initial user question 
add_user_message(messages, "Define quantum computing in one sentence") 

# Get Claude's response 
answer = chat(messages) 

# Add Claude's response to the conversation history
add_assistant_message(messages, answer) 

# Add a follow-up question 
add_user_message(messages, "Write another sentence") 

# Get the follow-up response with full context 
final_answer = chat(messages)
```

이러한 구조를 이해하고 보면, Claude 가 session 중간에 계속 context 압축이 필요하다고 하는 이유를 자연스럽게 이해할 수 있겠다.

```python
messages = []

def add_user_message(messages, text):
	user_message = {"role": "user", "content": text}
	messages.append(user_message)

def add_assistant_message(messages, text):
	assistant_message = {"role": "assistant", "content": text}
	messages.append(assistant_message)

def chat(messages):
	message = client.messages.create(
		model=model,
		max_tokens=1000,
		messages=messages,
	)
	return message.content[0].text

while True:
	user_input = input("> ")
	print(">", user_input)
	
	add_user_message(messages = messages, text = user_input)
	answer = chat(messages = messages)
	add_assistant_message(messages = messages, text = answer)

	print("---")
	print(answer)
	print("---")
```

