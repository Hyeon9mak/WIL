# AI-agent

정해진 Flow 외 LLM 이 독자적으로 판단해야하는 분기 행동이 포함되면 Agent 라고 부를 수 있다.

<img width="772" alt="Image" src="https://github.com/user-attachments/assets/4844b745-2809-41bc-ad60-090a5297c6a0" />

일련의 예시로, Neo4j 에서 발표한 Text2Cypher Agent 의 경우 아래 그림과 같이 우선 Cypher 로 변환 후 질문을 던져보았을 때
오류가 발생하면 Cypher 를 수정하고 성공하면 그제서야 다음 스탭을 진행하는 기본적인 구조를 가지고 있다.

<img width="745" alt="Image" src="https://github.com/user-attachments/assets/96364b95-acff-49d9-8d27-bc74274a987d" />

수정된 Cypher 문을 평가하는 노드까지 추가하면 아래 그림과 같아진다.

<img width="787" alt="Image" src="https://github.com/user-attachments/assets/ee3211d5-1338-421e-8306-0ae35eee4ef2" />
