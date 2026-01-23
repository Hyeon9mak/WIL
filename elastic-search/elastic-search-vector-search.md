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

### 개발자가 인지해야하는 것

- DataSet size, property
- search performance
- resource

결국 다양한 모델에 대해 평가를 수행하며 선택한 모델이 kNN(k-Nearest Neighbors)를 달성하는지 확인해야한다.
즉, ES 를 도입한다고 해서 모든 상황에 대응할 수 있는게 아니라, 여러가지를 고려해야한다.

> kNN: k 값은 알고리즘 예측 수행을 할 때 내의 가장 가까운 이웃의 수. 주로 Vector Search 정확도를 조절하는 변수로 활용된다.

kNN 기반 semantic search 또는 완전 일치 검색이 가능해진다.

- Embedding pipeline = Model + DataSet
- DataSet append API
- Data 품질관리 Process(System)
- VectorDB performance test
- Monitoring System

### Model 기반 VectorDB 구축 pipeline
- Hugging Face
	- Maven Central Repository 와 같은 Model Repository
- Data 를 Model 기반으로 Embedding 시키고, 그걸 Vector DB 에 보관함
- 사용자 질의를 Vector 로 변환하여 검색

### Hugging Face
- 자연어처리 Model 전문 연구소
- Model Hub 제공.
	- like the Apache Maven Central Repository 
- Datasets 와 같은 library 도 제공한다.
	- 자연어 처리 및 머신 러닝 작업을 위한 dataset

### ElasticSearch 외에도 도입해 볼 것들
#### Pinecone
- full managed. 자체 hosting solution
- 그러나 관리형이기 때문에 비용 문제가 있음
####  Vespa.ai
- 사용자 정의 랭킹함수 지원 및 실시간 인덱싱 등
- Vector 검색을 위한 특수 설계 DB 라 설정이 복잡함
#### Milvus
- vector 검색에만 초점을 맞추고 있음
- vector, text, 구조화 된 데이터 등 여러 검색 조합이 필요한 경우 사용이 어려움
#### Weaviate
- Semantic Search 특화
- GraphQL 기반 vector 검색
- DataSet 정제가 중요할 수 밖에 없다.
#### ElasticSearch
- ANN 과 완전 일치 검색을 위한 고밀도 Vector DB 지원
- 계층적 탐색 가능한 소규모 세계(HNSW) 그래프
- 당연히 복잡함. 루씬의 dense_vector 데이터 유형 제약사항이 동일하게 적용

### 사용 사례
#### 개인화 통합
- 개인화는 과거 검색 및 유사한 검색을 기반으로 검색에 맥락을 더해줄 수 있다.
- 과거 검색은 개인이 과거에 검색했던 이력들
- 유사한 검색은 타인이 과거에 검색했던 이력들을 활용 가능 할듯
####  NER(개체명 인식)
- 비정규화 DataSet 에서 Entity 를 추출하는 기술
- Text 내 Pattern 과 relation 을 학습함으로서 Entity 를 분류한다.
- 즉, NER 을 잘 활용하면 자연어에서 GraphDB entity 도 발생시킬 수 있다.
- 대표적으로 PII(Personally Identifiable Information) 개인 식별 정보에 쓰인다.
	- NER 기반 개인정보 마스킹 등
####  감정 분석
- 비정형 Text 에서 감정 식별. SNS 등에서 활용
- Python NLTK library 가 주로 사용된다.
- 매우 간단함
```python
import nltk
nltk.download('vader_lexicon')

from nltk.sentiment import SentimentIntensityAnalyzer

sia = SentimentIntensityAnalyzer()
text = "어쩌구"
scores = sia.polarity_scores(text)
print(scores)
```
#### 텍스트 분류
- 스팸 관리, 이메일 필터링, 태킹 등에 활용
- NER 과 아마 방법은 유사할듯? 사용 사례가 다른 것
```python
from sklearn.datasets import fetch_20newsgroups
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.metrics import accuracy_score

# load dataset
newsgroups_train = fetch_20newsgroups(subset='train')
newsgroups_test = fetch_20newsgroups(subset='test')

# vectorizing to text data
vectorizer = CountVectorizer()
X_train = vectorizer.fit_transform(newsgroups_train.data)
X_test = vectorizer.transform(newsgroups_test.data)

# naive bayes train
clf = MultinomialNB()
clf.fit(X_train, newsgroups_train.target)

# suggest test set
y_pred = clf.predict(X_test)

print(f"Accuray: {accuracy_score(newsgroups_test.target, y_pred)}")
print(f"Predicted classes: {y_pred}")
```

### 질의응답(QA)
- GPT, Gemini 등 요즘 대규모 LLMs 들이 집중하는 영역
- model 부분에 더 많은 model 을 연결하는 것도..
```python
from transformers import pipeline

# 질의응답(QA) 모델 적재
model = pipeline("question-answering", model = "distilbert-base-cased-distilled-squad", tokenizer="distilbert-base-cased")

# 질문과 문맥 구성
question = "What is the capital city of Korea?"
context = "South Korea is located ..."

result = model(question=question, context=context)
print(f"Answer: {result['answer']}")
```

### 텍스트 요약
- 뉴스피드 등
- 역시나 transformer library 를 사용하여 요약해볼 수 있다.
- Hugging Face 의 DistilBert model 과 summarizer 를 사용하여 최대/최소 길이 지정도 가능
```python
from transformers import pipeline

text = "..."
summarizer = pipeline("summarization")

summary = summarizer(text, max_length=100, min_length=30, do_sample=False)[0]["summary_text"]

print(summary)
```