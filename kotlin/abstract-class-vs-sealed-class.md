# abstract class 와 sealed class 는 각각 언제 사용하는가?

흔히 kotlin에서 sealed class를 "완전한(제한적인) 계층 구조를 표현할 때", "안전한 계층 구조를 갖고 싶을 때" 사용한다고 한다. 
abstract class와 동일한 기능을 제공하면서, 더욱 뛰어난 안정성(아무나 서브 클래싱하여 확장할 수 있는 가능성을 없앰)을 제공하기 때문에
sealed class를 사용하지 않을 이유가 없었다. 그렇다면 도대체 kotlin에서 abstract class는 언제 사용되는 것일까?

실제로 sealed class 와 abstract class의 가장 큰 차이는 구현체가 동일 패키지 안에 있어야하는지, 아닌지의 여부 뿐이고
그 차이로 인해 라이브러리나 프레임워크를 개발할 때는 abstract class를 활용하는 경우가 많으며(구현체를 클라이언트의 패키지에 두기 때문에),
실무에서도 같은 이유에서 abstract class를 활용할 수 있다. 그러나 그 외의 경우네는 sealed class를 사용하는게 더 좋다고 결론 지었다.

<br>

## References

- [abstract class와 sealed class는 어느경우에 나눠 사용하나요?](https://www.inflearn.com/questions/545726)
