- Claude 는 콘텐츠 주변에 설명 텍스트를 추가하는 걸 좋아한다.
- 대부분의 케이스에는 친절한 설명 방식이 되겠지만, JSON, python code 와 같은 구조화된 데이터를 다룰 땐 문제가 생긴다.
- 그래서 다음과 같은 방식으로 assistant message prefilling 과 stop sequences 를 이용한다.

```python
messages = [] 

add_user_message(messages, "Generate a very short event bridge rule as json")
add_assistant_message(messages, "```json") 

text = chat(messages, stop_sequences=["```"])
```

1. user message 는 Claude 에게 생성할 내용을 알려준다.
2. 미리 채워진 assistant message 는 Claude 에게 이미 마크다운 코드 블록을 시작한 것 같게 만든다.
3. Claude 는 자연스럽게 JSON 콘텐츠를 먼저 채우려고 한다.
4. Claude 가 코드 블록을 '''로 닫으려 하면 stop sequence 가 즉시 생성을 종료해버린다.
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/22fd8dd6-a289-443a-8600-c84f71b54190" />

이를 통해 별도 주석이나 헤더, 푸터 없는 깔끔한 JSON 응답을 기대할 수 있다.
당연히, JSON 뿐만 아니라 그 어느 형태든 이용할 수 있는 강력한 방법이다. 매우 자주 이용하게 될 것.

```python
messages = []

prompt = """
	서로다른 AWS CLI command 3개를 만들어주세요. 각각은 매우 짧은 명령어여야 합니다.
"""

add_user_message(messages, prompt)
add_assistant_message(
	messages = messages,
	text = "모든 응답은 단일 블록으로, 특수 기호 없이. 명령어 하나당 개행(\\n)으로 마무리 ```bash"
)

text = chat(messages, stop_sequences=["```"])
text.strip()
```

assistant message 에 bash 정보만 추가하는 것 외에도, 추가 요구사항(`모든 응답은 단일 블록으로, 특수 기호 없이. 명령어 하나당 개행(\\n)으로 마무리`)을 적어주면 거기에 맞춰서 무언가를 더 진행해준다.