사용자가 텍스트를 입력해서 질문을 던지면, 마법처럼 질문에 응답을 해주는 Chat app 예시를 생각해보자. 그 내부에서는 무슨 일이 일어날까? 아마 아래 그림과 같을 것이다.

<img width="2329" height="1215" alt="Image" src="https://github.com/user-attachments/assets/ad69fbf2-80f2-44d3-b0ed-c1c985fb0e4d" />

### Client side
- Client(Web, App) 에서 Anthropic API 에 직접적으로 액세스 하려고 시도해선 안된다.
	- API 요청 시마다 secret API key 가 필요하다.
	- 이 key 를 관리하는 것은 당연히 Client 보다 Server 측이 안전하니까.
### Server side
- Server 는 Client 로부터 요청을 받으면 Anthropic API 를 찌른다.
	- Anthropic SDK 를 이용하는 방식이 일반적이다.
	- 물론 SDK 없이 HTTP request 를 직접 보내는 방식도 가능
		- API key, model, messages, max tokens 4가지 정보를 포함해서 보내면 된다.
		- **max tokens** 은 Anthropic LLM model 이 맞추려고 노력하는 안전 장치일 뿐, 꼭 max tokens 만큼의 길이를 맞춰주는 건 아니다!!
### Anthropic API side
Anthropic API 내부에서는 어떤 일이 벌어질까?
1. 사용자의 질의를 더 작은 단위로 나눈다. 이 텍스트 청크를 'token' 이라고 부른다.
   token 은 완성된 단어나 단어의 일부, 공백, 기호 등 무엇이든 될 수 있다.
2. 각 token 은 embedding 으로 변환된다. (유사도 측정을 위해)
   embedding 은 하나의 의미를 나타내는게 아니다. 여러 의미를 포함하고 있다.
   가령, "양자" 단어를 떠올려보자. 최소 2개 이상의 의미를 유추할 수 있다. 그 어느 것이라고 단정 지을 수 없기 때문에 **주변에 있는 다른 단어의 embedding 과 조합해서 유추**하는 과정이 필요하다.
   즉, **정확한 정의로 다듬기 위해 맥락화(Contextualization)** 과정이 필요하다.
<img width="2184" height="870" alt="Image" src="https://github.com/user-attachments/assets/8ea6d3a5-8dd1-4510-8239-2060ec52378e" />
3. 맥락화(Contextualization) 동안에는 주변 단어의 embedding 값과 비교하며 가장 '말이 되는' 단어의 의미를 결정 짓는 것에 집중한다.
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/1a70f77d-5faa-4e07-87f2-9058fd95ba61" />
4. 마지막은 응답 생성(Generation) 단계. 맥락화 과정에서 이웃 embedding 과 비교를 통해 많은 정보를 습득한 후 응답 생성에 필요한 다음 단어의 '확률'을 생성한다.
   이 과정에서 LLM model 은 가장 높은 확률을 자동으로 선택하는게 아니다. 대신 확률과 무작위성의 조합을 사용하여 단어를 선택한다. 이러면 자연스럽고 다양한 응답이 형성된다. (기계적인 답변 회피)
5. 3, 4를 무한 반복하며 응답을 생성하는 것. 출력 응답을 만들어낼 때마다 응답 생성을 끝까지 다 했는지 여부를 결정하기 위해 max token 수를 비교한다던지, 특수 종료 시퀀스를 호출하는 등.
6. 5 가 끝나면 Anthropic API 를 호출한 Server 로 응답을 보낸다. 그리고 생성된 답변을 Server 가 Client 한테 보내면 된다.

응답에는 생성된 답변과 사용량 및 중지 사유가 포함된다.