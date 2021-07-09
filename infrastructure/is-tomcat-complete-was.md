# 톰캣은 완전한 WAS일까?
톰캣을 WAS라고 하는 글이 있고, 톰캣은 WAS가 아니라고 하는 글도 있다.

나도 `톰캣 = WAS`라고 생각하던 사람인데,
'톰캣이 WAS가 아니라면 WAS라고 부를 수 있는 나머지 스펙들은 누가 채우고 있을까?' 라는 의문 덕분에 
마크, 케빈과 DM을 주고 받았다.

<br>

## JavaEE가 말하는 WAS
> 웹 서버, 웹 애플리케이션 서버, 웹 컨테이너 라는 3가지 용어를 혼용해서 사용하는데 사실은 제각기 다른 것이다. 

- 웹 서버는 정적인 페이지(HTML, CSS)를 제공하는 서버를 의미한다. 
- 웹 애플리케이션 서버는 서버 사이드 코드를 이용하여 동적인 컨텐츠를 만드는 서버를 의미한다. 

톰캣을 엄연히 정의하면 WAS 보단 서블릿 엔진 혹은 웹 컨테이너 (서블릿 컨테이너로 잘 알려진) 쪽에 가깝다. 
왜냐하면 서블릿과 JSP를 위한 런타임 환경을 제공하지만 JavaEE에서 제안하는 EJB 와 같은 다른 스펙들이 빠져있기 때문이다.

EJB 기술을 대체하는 기술로 등장한 것이 바로 스프링이었다. 스프링 수준에선 EJB뿐만 아니라 JavaEE의 제안 스펙들을 추상화시켜 제공 했으나, 
스프링 부트로 넘어오면서 JavaEE가 제안하는 대부분의 스펙들을 구현체로 제공하기 시작했다.

JavaEE가 말하는 온전한 WAS가 되려면 `EJB, JTA 등 스펙이 포함 + 이걸 서블릿을 관리하는 톰캣에 올림 = WAS`가 되어야 한다.
즉, `톰캣 = WAS` 보단 `톰캣 + 스프링부트 = 스프링 부트 프로젝트 = WAS`가 될 것이다.

<br>

## 러프한 의미의 WAS
그러나 WAS를 단순하게 `클라이언트의 요청에 따라 동적인 결과를 만들어주는 서버측 프로그램` 으로 분류한다면, 
EJB, JTA 등의 스펙이 불필요해지면서 `서블릿 API + JSP실행 및 부가기능`을 제공하는 톰캣을 WAS로 부를 수 
있게 된다.

<br>

## 결론 - 정확히 말하면 톰캣은 WAS가 아니다
WAS를 러프한 의미로 본다면 톰캣은 WAS가 될 수 있다. 그러나 WAS를 JavaEE 제안 스펙에 따라 하나하나 따진다면 
톰캣은 WAS가 될 수 없다. WAS 스펙 중 하나를 충족시키는 프레임워크일 뿐이다.

그럼에도 많은 책과 글에서 톰캣을 WAS로 보는 이유는 스프링&스프링 부트에서 WAS 스펙 대부분을 충족하기도 하고, 
톰캣이 서블릿 컨테이너 역할(서블릿 API) 이상의 기능을 제공해서 사용자의 요청에 따라 동적인 결과를 만들고 응답으로 내보내기 때문에 WAS라고 부르는 것 같다.

<br>

## References
- [[Servlet] 웹서버 vs 애플리케이션 서버 vs 서블릿 컨테이너 ...](https://pjh3749.tistory.com/267)
- [톰캣은 WAS가 아닌가요?](https://okky.kr/article/293917?note=982388)
- [우아한테크코스 백엔드 3기 마크와의 DM](https://github.com/binghe819)
- [우아한테크코스 백엔드 3기 케빈과의 DM](https://github.com/xlffm3)