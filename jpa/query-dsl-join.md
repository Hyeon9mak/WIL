# QueryDSL Join 관련

##  QueryDSL type cast 발생시 full table scan
### 문제 상황
QueryDSL의 Query JOIN 절에 cast(타입변환)이 포함될 경우 
인덱스를 타지 못하고 거의 풀테이블 스캔이 일어나게 된다.
(모든 row를 형변환 시도)

때문에 아래와 같은 상황에서 type casting이 일어나지 않도록 주의해야함.

![image](https://user-images.githubusercontent.com/37354145/179342355-e23eed7d-cbb1-4c1b-8700-7fd1fa3c5281.png)

*`longValue()`로 인해서 type casting이 계속 발생하는 중*

### 해결 방법
엔티티 내부에 toLong type을 반환하는 getter 메서드 구현.
결국 "query에서 casting을 하냐", "casting된 값을 불러오느냐"의 차이.
미리 casting 된 값을 가져오는 건 인덱스도 정상적으로 태울 수 있다.

<br>

## QueryDSL에서 하나의 테이블에 각각 나눠서 2번 조인을 걸고 싶을 땐 어떻게 하는가?

Q객체를 2가지 변수로 나누어 할당한 후, 조인을 걸면 된다!

[동일 entity , table 다중 leftJoin시 방법 - 인프런 | 질문 & 답변](https://www.inflearn.com/questions/56400)

![image](https://user-images.githubusercontent.com/37354145/179342748-22067972-25c9-4e35-839e-edb301039755.png)

<br>

## QueryDSL에서 LeftJoin 대상이 되는 Q객체는 nullable 하다.
QueryDSL 조회 결과를 DTO에 바인딩할 때,
LeftJoin Q객체를 non-nullable하게 바인딩하려하면
null이 아니더라도 NullPointerException을 발생시킨다.
(사전에 차단하는 듯)

innerJoion 대상이 되는 Q객체는 non-nullable하므로, 이쪽을 이용해야한다.
