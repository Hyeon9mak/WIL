먼저 예시 prompt 를 하나 작성해보자.
이 prompt 를 활용해서 AWS 사용 사례에 특화된 코드를 작성하는데 도움을 주고자 한다.

```python
promtpt = f"""
Please provide a solution to the following task:

{task}
"""
```

prompt 는 사용자에게 다음 세 가지 특정 출력 유형을 작성하는데 도움을 주어야 한다.

- Python code
- JSON configuration files
- 정규 표현식

핵심 요구사항은 사용자가 도움을 요청할 때, 추가 설명이나 헤더, 푸터 없이 셋 중 하나로만 대답을 해야한다는 것이다.

## Generating test datasets

```json
[
    {
	    "task": "Create a Python function to extract the AWS account ID from an ARN."
    },
    {
	    "task": "Write a JSON policy document that allows read-only access to a specific S3 bucket."
    },
    // ...many more...
]
```

수기로 직접 만들든, Claude 와 같은 LLM 을 이용해서 만들든 상관없다. prompt 에 포함될 형태의 test dataset 을 준비하면 된다. (당연히 이럴 땐 Claude Haiku model 을 이용하는게 좋음)

```python
# Load env variables and create client
from dotenv import load_dotenv
from anthropic import Anthropic

load_dotenv()

client = Anthropic()
model = "claude-haiku-4-5"

# Helper functions
def add_user_message(messages, text):
	user_message = {"role": "user", "content": text}
	messages.append(user_message)

def add_assistant_message(messages, text):
	assistant_message = {"role": "assistant", "content": text}
	messages.append(assistant_message)

def chat(messages, system=None, temperature=1.0, stop_sequences=[]):
	params = {
		"model": model,
		"max_tokens": 1000,
		"messages": messages,
		"temperature": temperature,
		"stop_sequences": stop_sequences,
	}

	if system:
		params["system"] = system
	
	message = client.messages.create(**params)
	return message.content[0].text
```

위 코드를 활용해서 직접적으로 dataset 을 생성해보자.

```python
import json

def generate_dataset():
	prompt = """
	Generate a evaluation dataset for a prompt evaluation. The dataset will be used to evaluate prompts
	that generate Python, JSON, or Regex specifically for AWS-related tasks. Generate an array of JSON objects,
	each representing task that requires Python, JSON, or a Regex to complete.
	
	Example output:
	```json
	[
		{
			"task": "Description of task",
		},
		...additional
	]
	\```
	
	* Focus on tasks that can be solved by writing a single Python function, a single JSON object, or a regular expression.
	* Focus on tasks that do not require writing much code
	
	Please generate 3 objects.
	"""
	  
	messages = []
	add_user_message(messages, prompt)
	add_assistant_message(messages, "```json")
	text = chat(messages, stop_sequences=["```"])
	return json.loads(text)
	
dataset = generate_dataset()

with open("dataset.json", "w") as f:
	json.dump(dataset, f, indent=2)
```

당연히, `assistant_message` 와 `stop_sequences` 를 포함시켜서 잘라내는거 잊지 말자.

## Running the eval
평가 데이터셋이 완성 되었다면, 핵심 평가 파이프라인을 구축할 차례다.
앞으로 데이터셋을 "테스트 케이스" 라고 부르겠다.
테스트 케이스를 prompt 와 결합한 뒤 Claude 에 전달한 후 결과를 채점하는 과정을 포함한다.

```python
def run_prompt(test_case):
	"""Merges the prompt and test case input, then returns the result"""
	prompt = f"""
Please solve the following task: 
{test_case["task"]}
"""

	messages = []
	add_user_message(messages, prompt)
	output = chat(messages)
	return output
	
def run_test_case(test_case): 
	"""Calls run_prompt, then grades the result""" 
	output = run_prompt(test_case) 
	
	# TODO - Grading
	score = 10
	
	return { 
		"output": output, 
		"test_case": test_case, 
		"score": score
	}
	
def run_eval(dataset): 
	"""Loads the dataset and calls run_test_case with each case""" 
	results = []
	
	for test_case in dataset: 
		result = run_test_case(test_case) 
		results.append(result) 
	
	return results
	
	
with open("dataset.json", "r") as f:
	dataset = json.load(f) 

results = run_eval(dataset)
print(json.dumps(results, indent=2))
```

파이프라인을 수행시키면, score 를 뽑아내기 위한 Grading 영역을 제외하고 모든 과정이 동작하는 것을 알 수 있다.
즉, 평가를 제외한다면 단순히 dataset 을 준비하고, 각각을 test_case 로 나누고, 반복 수행 시켜서 점수를 뽑아내는 것이 전부다! 어려울게 전혀 없다.

## Grading (채점기)
관례적으로, 채점기는 1~10 사이의 점수를 통해 평가를 진행한다. 어디까지나 관례적일 뿐, True/False 일 수도 있고, 별의별거 다 할 수 있다.

채점기는 크게 3가지 방식으로 구현할 수 있다.
### code grading
- length 검증
- certain(contain) 검증
- Syntax 검증
### model grading
- 응답 품질
- prompt 지침을 잘 따른 답변인지
- 답변의 완성도(완전성)
- 기타 등등... 엄청난 자유도
### human grading
- 응답 퀄리티 검증
- 그냥 사람이 보고 평가할 수 있는 모든 것.
- 다만 시간이 오래 걸리고 평가자가 매우 지루해 함.

3가지 방법 모두에서 가장 중요한 것은 평가 기준을 미리 수립하는 것.
여러가지 방침이 있겠지만 일단 예시에서는 아래 3가지 기준을 수립했다.

1. 응답 포맷이 Python, JSON, 정규식에 일치하는지
2. Syntax 가 올바른지. Validation
3. 사용자의 요청 사항을 제대로 수행하는지. 논리적 오류 등이 포함되지 않았는지.

3가지 기준을 다시 어떤 grading 을 이용할지 나눌 수 있다. 가령 1,2 번 같은 경우 꼭 model grading 을 이용할 필요는 없다. code grading 으로도 충분히 수행할 수 있고 비용이 저렴하며 빠르다. 3번의 경우 model grading 이 적합하다.

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/0e2e9fdf-fb92-4954-b7a0-b3d38ba245a1" />


## model grading

```python
# Function to grade a test case + output using a model
def grade_by_model(test_case, output):
	eval_prompt = f"""
You are an expert AWS code reviewer. Your task is to evaluate the following AI-generated solution.

Original Task:
<task>
{test_case["task"]}
</task>

Solution to Evaluate:
<solution>
{output}
</solution>

Output Format
Provide your evaluation as a structured JSON object with the following fields, in this specific order:
- "strengths": An array of 1-3 key strengths
- "weaknesses": An array of 1-3 key areas for improvement
- "reasoning": A concise explanation of your overall assessment
- "score": A number between 1-10

Respond with JSON. Keep your response concise and direct.
Example response shape:

{{
	"strengths": string[],
	"weaknesses": string[],
	"reasoning": string,
	"score": number
}}
	"""

	messages = []
	add_user_message(messages, eval_prompt)
	add_assistant_message(messages, "```json")
	eval_text = chat(messages, stop_sequences=["```"])
	return json.loads(eval_text)
	
	
# Passes a test case into Claude
def run_prompt(test_case):
	prompt = f"""
Please solve the following task:
	
{test_case["task"]}
"""

	messages = []
	add_user_message(messages, prompt)
	output = chat(messages)
	return output
	
	
# Function to execute a single test case and grade the output
def run_test_case(test_case):
	"""Calls run_prompt, then grades the result"""
	output = run_prompt(test_case)

	model_grade = grade_by_model(test_case, output)
	score = model_grade["score"]
	reasoning = model_grade["reasoning"]

	return {
		"output": output,
		"test_case": test_case,
		"score": score,
		"reasoning": reasoning,
	}
	
	
from statistics import mean
def run_eval(dataset):
	"""Loads the dataset and calls run_test_case with each case"""
	results = []
	
	for test_case in dataset:
		result = run_test_case(test_case)
		results.append(result)
	
	average_score = mean([result["score"] for result in results])
	print(f"Average score: {average_score}")
	
	return results
```

여기서 핵심은, model grading 을 만들어서 점수를 요청할 때는 강점과 약점, 그리고 이유를 함께 요청하는 것이다. **단순히 점수만 요청할 경우 높은 확률로 체감 중간 점수인 6점을 응답하게 된다.** 이를 방지하기 위해서 명확한 이유를 요청하는 것. 미리 이유를 작성하고 그에 대한 평가에 집중할 수 있게 알아서 강점/약점/이유를 만들게 한다.

## Code grading
```python
def validate_json(text):
    try:
        json.loads(text.strip())
        return 10
    except json.JSONDecodeError:
        return 0


def validate_python(text):
    try:
        ast.parse(text.strip())
        return 10
    except SyntaxError:
        return 0


def validate_regex(text):
    try:
        re.compile(text.strip())
        return 10
    except re.error:
        return 0
```

코드 검사기는 위와 같은 방식들을 이용할 수 있다. 위 코드를 이해하면 알 수 있듯이, 각 함수를 언제, 어느 시점에 호출할지 결정 짓기 위해서는 응답 포맷이 어떤 형태인지 미연에 판별 할 수 있어야한다.

그 응답을 결정짓기 위해서는 dataset 에 기대하는 응답 포맷을 미리 작성해주는 것이 좋겠다.

```json
{
	"task": "...",
	"format": "python"
},
{
	"task": "...",
	"format": "json"
},
// ...
```

더 좋은 것은, dataset 에 일일이 format 을 기입하는게 아니라, dataset 생성 로직에 format 까지 한꺼번에 명세하도록 작성해두는 것이다.

```python
import re
import ast


def validate_json(text):
	try:
		json.loads(text.strip())
		return 10
	except json.JSONDecodeError:
		return 0

def validate_python(text):
	try:
		ast.parse(text.strip())
		return 10
	except SyntaxError:
		return 0

def validate_regex(text):
	try:
		re.compile(text.strip())
		return 10
	except re.error:
		return 0

def grade_syntax(response, test_case):
	format = test_case["format"]
	if format == "json":
		return validate_json(response)
	elif format == "python":
		return validate_python(response)
	else:
		return validate_regex(response)
```

dataset 생성 prompt 도 아래와 같이 구문을 추가해주면 된다.

```python
import json

def generate_dataset():
	prompt = """
	...
	
	Example output:
	```json
	[
		{
			"task": "Description of task",
			"format": "json" or "python" or "regex"
		},
		...additional
	]
	\```
	
	* Focus on tasks that can be solved by writing a single Python function, a single JSON object, or a regular expression.
	* Focus on tasks that do not require writing much code
	
	Please generate 3 objects.
	"""
```

또한 아래 구문을 추가하여 강제시키는 것을 잊지 말자.

```
* Respond only with Python, JSON, or a plain Regex
* Do not add any comments or commentary or explanation
```

prompt 는 강제가 아니라 요구이므로, assistant_message 와 stop_sequence 를 명세해서 더 강하게 강제하는 것을 잊지 말자.

```
add_assistant_message(messages, "```code")
```

마지막으로 점수를 더해주면 된다.

```python
model_grade = grade_by_model(test_case, output)
model_score = model_grade["score"]
syntax_score = grade_syntax(output, test_case)

score = (model_score + syntax_score) / 2
```

