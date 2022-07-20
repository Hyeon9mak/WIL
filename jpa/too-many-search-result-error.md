# 조회 결과 수가 너무 많을 경우 발생할 수 있는 에러

> nested exception is java.lang.NumberFormatException: character is neither a decimal digit number decimal point nor e notation exponential mark.

왜 에러나나 했는데 데이터 조회값이 너무 크다보니 메모리에 꽉차서 그런거 같음… 
QueryDSL로 조건절 걸어서 조회되는 데이터 개수 줄였음
