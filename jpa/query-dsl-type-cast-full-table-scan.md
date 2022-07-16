# QueryDSL type cast 발생시 full table scan
## 문제 상황
QueryDSL의 Query JOIN 절에 cast(타입변환)이 포함될 경우 
인덱스를 타지 못하고 거의 풀테이블 스캔이 일어나게 된다.
(모든 row를 형변환 시도)

때문에 아래와 같은 상황에서 type casting이 일어나지 않도록 주의해야함.

![image](https://user-images.githubusercontent.com/37354145/179342355-e23eed7d-cbb1-4c1b-8700-7fd1fa3c5281.png)

*`longValue()`로 인해서 type casting이 계속 발생하는 중*

<br>

## 해결 방법
엔티티 내부에 toLong type을 반환하는 getter 메서드 구현.
결국 "query에서 casting을 하냐", "casting된 값을 불러오느냐"의 차이.
미리 casting 된 값을 가져오는 건 인덱스도 정상적으로 태울 수 있다.
