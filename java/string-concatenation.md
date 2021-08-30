# StringJoiner

문자열 중간에 REGEX를 이용해서 문자열들을 이어줘야하는 경우가 있다.
보통 `String.join`이나 `StringBuilder`를 많이 사용해왔는데, 
`StringJoiner`를 통해서도 문자열을 이을 수 있었다.

## String.join
`String.join`을 사용하는 경우는 대게 **문자열이 이미 파싱되어 있는 경우** 사용하게 된다. 
즉, 이미 준비된 상태에서 사용하기 좋다.

```java
String A = "A";
String B = "B";
String C = "C";

String result = String.join(" ", A, B, C);
System.out.println(result);
```
```
A B C
```

<br>

## StringBuilder, StringBuffer
`StringBuilder`는 `String`에 가변성을 부여하는 클래스로, 원래 불변성을 가지는 `String`의 
특성 때문에 일어나는 성능상 문제점을 해결해준다. `StringBuilder`를 이용하면 
동일한 객체 내에서 문자열을 변경하는 작업이 가능해진다.

`String.join`과 비교하자면 `String`의 개수를 정확하게 부여할 수 없을 때(루프) 사용하기 좋다.

```java
StringBuilder stringBuilder = new StringBuilder("A B ");
stringBuilder.append("C");

String result = stringBuilder.toString();
System.out.println(result);
```
```
A B C
```

비슷하게 사용할 수 있는 `StringBuffer`도 존재한다. `StringBuilder`와 `StringBuffer`의 차이는 무엇일까?
가장 큰 차이점은 멀티쓰레드 환경에서의 안정성이다. 
`StringBuffer`는 동기화 키워드를통해 멀티쓰레드 환경에서 안정적이지만, `StringBuilder`는 동기화를 고려하지 않기 때문에 단일 쓰레드에서 성능이 뛰어나다.

정리하자면 아래와 같다.

- 문자열 연산이 적은 멀티쓰레드(동기화가 필요한) 환경: `String`
- 문자열 연산이 많은 멀티쓰레드(동기화가 필요한) 환경: `StringBuffer`
- 문자열 연산이 많은 단일쓰레드(동기화가 불필요한) 환경: `StringBuilder`

<br>

## StringJoiner
`StringBuilder`를 사용할 때 아래와 같이 사용하는 경우가 있었을 것이다.

```java
StringBuilder stringBuilder = new StringBuilder();
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));

while(true) {
    stringBuilder.append(bufferedReader.readLine())
        .append("\n");
}
```

문자열을 이어주는 REGEX를 사용하고자 `.append`를 더하는 방식인데, `StringJoiner`를 이용하면 
이를 깔끔하게 바꿔줄 수 있다!

```java
StringJoiner stringJoiner = new StringJoiner("\n");
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));

while(true) {
    stringJoiner.add(bufferedReader.readLine());
}
```

훨씬 깔끔하지 않은가?! Prefix, Suffix를 지정해주는 것도 가능하다.

```java
StringJoiner stringJoiner = new StringJoiner("\n", "[", "]");
```

사실 streamAPI를 이용하다보면 StringJoiner를 마주쳤을 가능성이 높다.

```java
 List<Integer> numbers = Arrays.asList(1, 2, 3, 4);

 String commaSeparatedNumbers = numbers.stream()
     .map(i -> i.toString())
     .collect(Collectors.joining(", "));
```

<br>

## References
- [Class StringJoiner (Java SE 10 & JDK 10)](https://docs.oracle.com/javase/10/docs/api/java/util/StringJoiner.html)
- [[Java] String, StringBuffer, StringBuilder 차이 및 장단점](https://ifuwanna.tistory.com/221)
