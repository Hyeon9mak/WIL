# Vector 검색의 기본

## Vector & Embedding

### 지도 학습과 비지도 학습

- 지도 학습
  - 미리 Label 이 지정된 Data Set 을 이용하여 model 을 학습시키는 방법
  - Input Data property 와 Output Label 간의 관계를 학습
  - 흔히 Machine Learning 이라고 부르던 그 방법
- 비지도 학습
  - Label 이 없는 Data Set 을 이용하여 model 을 학습시키는 방법
  - Data 간의 패턴과 구조, 유사성, 군집화 등을 학습
  - 흔히 Deep Learning 이라고 부르던 그 방법

ML/DL 과 마찬가지로, Vector/Embedding 도 지도/비지도 학습을 통해 높은 밀도의 Vector Space 를 만들어낸다.

### Vector 와 Embedding

현실 세계의 복잡한 Data 를 숫자로 변환하여 처리하는 방법.
숫자 값의 유사도를 이용해서 요소 간 semantic(의미) 과 구문적 관계를 파악한다.

- Vector
  - 다차원 Space 에서의 위치를 나타내는 수치 배열
  - 예: [0.1, 0.5, 0.3] (3차원 Vector)
  - 추천 시스템에서 사용자와 item 을 모두 vector 로 표현하여 유사도 기반 추천 가능
- Embedding
  - 자연어 처리에서 단어, 문장, 문서 embedding 은 text data 의 vector 표현이다.
  - 의미적으로 유사한 Data 가 Vector Space 에서 가깝게 위치
  - social network 사용자와 그들 간의 관계는 graph embedding 으로 표현 가능

### 그래서, Vector 는 어디에 쓰이는가?

BM25 는 확률적 정보 검색 이론에 기반하여 사용되는 텍스트 검색 Algorithm 이다.
단어 빈도와 역문서 빈도를 활용하여 문서의 관련성을 평가하고, 전통적인 검색 엔진에서 널리 사용된다.
그러나 완전 일치에 의존하기 때문에 동의어나 오타, Semantic 의미를 포착하는 데 한계가 있다.
또한 단어 간 관계나 문맥을 이해하지 못한다.

반면 Vector 검색은 완전 일치와 [ANN(Approximate Nearest Neighbor) 검색](https://www.elastic.co/kr/blog/understanding-ann)을 결합하여
텍스트의 의미적 유사성을 포착한다. Vector 검색은 단어의 의미적 관계를 이해하고, 유사한 의미를 가진 단어들을 함께 그룹화할 수 있다.
따라서 오타나 동의어가 포함된 질의에도 유리한 모습을 보여준다.

Vector 검색은 다음과 같은 분야에서 널리 활용할 수 있다.

- Ecommerce 검색: 고객이 은어를 사용하거나 오타를 내더라도 관련 제품을 추천
- 문서 검색: 유사한 내용, 문맥 또는 주제를 가진 문서를 검색
- 질의응답: 사용자의 질문과 의미적으로 유사한 답변을 제공
- image 인식 및 검색: image 를 고차원 Vector 로 표현하여 image 인식과 검색을 가능케 함.
- 음악 추천: 음악 트랙을 Vector 로 표현하여 유사한 음악 추천
- 보안과 사용자 해옹 분석 등..
