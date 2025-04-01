# Graph RAG

## 개념

### RAG(Retrieval-Augmented Generation)?

- LLM 모델이 뱉는 걸 그대로 활용하기보다 독자적인 지식 베이스를 구축하고 그걸 토대로 응답토록 하는 것. 
- 할루시네이션 방지를 위해.

### Graph RAG

- Graph DB 기반 RAG
- 단순한 벡터 기반의 유사도 검색을 넘어, 노드와 관계까지 다뤄서 세분화된 지식 검색

### 왜 Graph RAG 인가?

기본적으로 RAG 는 결국 '질의문'에 '컨텍스트'와 '예시'를 담아 더 높은 수준의 답변을 기대하는데, 여기서 '컨텍스트'에 넣는 값이 DB 에 말아둔 임베딩 벡터를 retriever 해서 나온 결과 값이다.

단, 여기서 맹점은 임베딩해서 말아둔 값에 '질의문'과 같은 키워드가 포함되어 있지 않다면 유사도가 낮아 탈락되기 때문에 원하는 양질의 결과를 얻기 어려워진다. 이 때문에 Graph DB 의 관계를 이용하는 Graph RAG 를 활용하게 되는 것.
결국 Graph RAG 방식은 아래와 같은 절차를 거친다.

1. 사용자가 질의문을 던진다.
2. 질의문에서 키워드를 뽑아 Cyper(쿼리) 생성
3. GraphDB 에서 노드, 관계를 찾아 결과 반환
4. 쿼리 결과 기반 LLM 에게 질의, 답변 생성
5. 사용자에게 응답

<br>

### Graph RAG 구현 방법

1. 싸이퍼를 통한 노드 및 프로퍼티 질의
2. LLM 모델을 이용한 프로퍼티 임베딩 - 벡터화
3. 벡터 결과물을 노드 프로퍼티로 추가 저장
4. 벡터 프로퍼티에 접근 속도를 개선해주기 위해 인덱스 생성

벡터에 프로퍼티를 더더욱 빠르게 접근할 수 있도록 무얼 기준(코싸인 등)으로 접근할지, 어떤 값 이상으로 접근할지 등 옵션도 설정해줄 수 있음.
이를 통해 retriever 가 더 빠르게 검색을 수행할 수 있다.

<br>

## 활용 프레임워크(Retriever 포함)

### Neo4j

- Neo4j: Graph DBMS, Cypher 를 통한 데이터 관리 
- Cyper: Graph DB query language (e.g. `MATCH (n1)-[r]->(n2) RETURN r, n1, n2 LIMIT 25`)
- node 뿐만 아니라 relationship 에도 property 를 담을 수 있다.

### neo4j_genai

- 생성형 AI 랑 Graph DB 연결을 돕는 파이썬 패키지. 
- 벡터 검색, 그래프 구축 등 가능

Neo4j_genai 코드를 까보면 좋다.
내부 구현 방식 (간단한 Python 코드)를 살펴보면 어떤 형태의 질의를 통해 graph RAG 를 진행하는지 자연스럽게 이해할 수 있다.

### neo4j_genai.VectorRetriever

- VectorRetriever: 벡터 검색을 위한 클래스
- 벡터 검색을 위한 인터페이스를 제공한다.
  - 4가지 정보를 필요로 한다.
    - DB 드라이버 정보, 벡터 인덱스 이름(벡터화/임베딩화 된 데이터 위치), 임베더(질의문을 어떤 임베딩 값으로 변환시킬 모델), 검색 결과 중 어떤 프로퍼티를 원하는지


### neo4j_genai.Text2CypherRetriever

- Text2CypherRetriever: 질의문을 Cypher 쿼리로 변환해주는 클래스
- 2가지 정보를 필요로 한다. Neo4j DB Schema, Input/Output 예시
  - 그래야 Retriever 가 Schema 와 예시를 참고해서 Cypher 쿼리를 생성할 수 있다.

역시나 Text2Cypher retriever 코드를 까보면 좋다.
내부에서 Schema 외 다른 정보를 추가한다던가, Cypher 외 다른 문자를 추가하지 말라는 등의 요구사항이 포함되어 있음을 확인할 수 있다.

### Gradio
