## 생성자에 인자가 많은 경우 정적 팩터리 메서드를 고려 할 것

특히나 DTO 등 어쩔 수 없이 필드 데이터가 한꺼번에 많이 오가는 경우 사용하기 편하다.
브라운이 수업 중에 말씀하셨던 '도메인이 풍부해질수록 서비스가 간결해진다.'의 확장선이라고 볼 수 있을거같다.
DTO 생성자를 풍부하게 만들수록 컨트롤러, 서비스, DAO 코드가 간결해진다!

그렇지만 꼭 장점만 있는것은 아니다.

- 일반적인 생성자처럼 API 설명에 명확히 드러나지 않는다.
- 인스턴스를 생성해주는 메서드를 직접 찾아내야 한다.
- 그렇기 때문에 널리 알려진 규약에 따라 메서드를 명명해야 한다.

### 널리, 많이 알려진 명명법

|       메서드명        | 내용                                                         | 예시                                                        |
| :-------------------: | :----------------------------------------------------------- | :---------------------------------------------------------- |
|         from          | 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드 | Date d = Date.from(instant);                                |
|          of           | 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드 | Set faceCards - EnumSet.of(JACK, QUEEN, KING);              |
|        valueOf        | from과 of의 더 자세한 버전                                   | BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);   |
| instance, getInstance | (매개변수를 받는다면)명시한 인스턴스를 반환, 같은 인스턴스임을 보장하지는 않는다. | StackWalker luke = StackWalker.getInstance(options);        |
|  create, newInstance  | instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장 | Object newArray = Array.newInstance(classObject, arrayLen); |
|        getType        | getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. Type은 팩터리 메서드가 반환할 객체의 타입이다. | FileStore fs = Files.getFileStore(path);                    |
|        newType        | newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. Type은 팩터리 메서드가 반환할 객체의 타입이다. | BufferedReader br = Files.newBufferedReader(path);          |
|         type          | getType과 newType 축소버전                                   | List litany = Collections.list(legacyLitany);               |

https://hyeon9mak.github.io/Effective-Java-item1/
