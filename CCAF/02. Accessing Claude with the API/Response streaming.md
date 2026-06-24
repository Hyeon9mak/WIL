- text/event-stream 형태로 응답 streaming 을 진행한다.
- 당연하게도, 응답에는 10~30초가 소요될 수 있는데, 이 응답을 기다리는 동안 스피너를 보여주는 것은 좋지 않은 UX 를 남긴다.

<img width="801" height="1053" alt="Image" src="https://github.com/user-attachments/assets/5dc9dce1-32f6-4065-8d19-fcea7e6f2b98" />

기본적으로는 `stream=True` 파라미터를 주는 것으로 streaming 형태의 응답을 받을 수 있다.

```python
messages = [] 
add_user_message(messages, "Write a 1 sentence description of a fake database") 

stream = client.messages.create(
	model=model, 
	max_tokens=1000, 
	messages=messages, 
	stream=True
)

for event in stream: 
	print(event)
```

다만 이 호출 방식은 Anthropic SDK 가 제공하는 별도 값들도 모두 포함하고 있어서, 별개 파싱 과정을 필요로 한다.

만약 Claude 가 제공하는 응답만을 받고 싶다면, 아래 방식을 이용하면 된다.

```python
with client.messages.stream(
	model=model, 
	max_tokens=1000, 
	messages=messages
) as stream:
	for text in stream.text_stream: 
		print(text, end="")
```

사용자 체감에는 좋겠지만, DB 등에 최종 응답을 저장하기 위해 최종 형상을 별개로 받는게 가장 좋을 것이다. 다행히도 Anthropic SDK 에서는 이런 것을 지원한다.

```python
with client.messages.stream(
	model=model, 
	max_tokens=1000, 
	messages=messages
) as stream:
	for text in stream.text_stream: 
		print(text, end="")
		
	final_message = stream.get_final_message()
```

