# DefaultListableBeanFactory를 통한 컨테이너 Bean 객체 조회

```java
ApplicationContext ac = 
    new AnnotationConfigApplicationContext(AppConfig.class);
```

`@Configuration` 어노테이션을 붙인 설정 클래스를 직접 작성한 경우, `AnnotationConfigApplicationContext`도 직접 생성하므로 
컨테이너에 등록되어 있는 Bean 객체들을 확인하기 용이하지만, 별도로 설정 클래스를 작성하지 않은 경우 컨테이너 객체를 직접 마주하기 어렵다.

다행히 방법이 없는 것은 아니다!  
`@Autowired` 어노테이션으로 스프링 컨테이너 자체를 주입받으면 된다.

스프링의 컨테이너들은 모두 `BeanFactory` 인터페이스를 구현하고 있다. 
대부분의 스프링 컨테이너들은 `DefaultListableBeanFactory` 로 `BeanFactory` 인터페이스를 구현한다. 
즉, `@Autowired` 어노테이션으로 `DefaultListableBeanFactory`를 주입 받으면 
대부분의 컨테이너를 사용할 수 있다!

```java
@Autowired
    DefaultListableBeanFactory defaultListableBeanFactory;

    @Test
    @DisplayName("Default Listable Bean Factory 확인 테스트")
    void defaultListableBeanFactory() {
        System.out.println("##################################################################### START");
        
        for (String beanName : defaultListableBeanFactory.getBeanDefinitionNames()) {
            System.out.println(beanName + "\t " + defaultListableBeanFactory.getBean(beanName).getClass().getName());
        }
        
        System.out.println("##################################################################### END");
    }
```
```
##################################################################### START
...(생략)
subwayApplication	 wooteco.subway.SubwayApplication$$EnhancerBySpringCGLIB$$12d918f2
org.springframework.boot.autoconfigure.internalCachingMetadataReaderFactory	 org.springframework.boot.type.classreading.ConcurrentReferenceCachingMetadataReaderFactory
pageController	 wooteco.subway.PageController
authenticationPrincipalConfig	 wooteco.subway.auth.AuthenticationPrincipalConfig$$EnhancerBySpringCGLIB$$ad7b84d
authService	 wooteco.subway.auth.application.AuthService$$EnhancerBySpringCGLIB$$44ecfd52
jwtTokenProvider	 wooteco.subway.auth.infrastructure.JwtTokenProvider
authController	 wooteco.subway.auth.ui.AuthController
authenticationInterceptor	 wooteco.subway.auth.ui.AuthenticationInterceptor
exceptionAdvice	 wooteco.subway.exception.ExceptionAdvice
lineService	 wooteco.subway.line.application.LineService$$EnhancerBySpringCGLIB$$1553d8d2
lineDao	 wooteco.subway.line.dao.LineDao$$EnhancerBySpringCGLIB$$a83ffd6d
lineController	 wooteco.subway.line.ui.LineController
memberService	 wooteco.subway.member.application.MemberService$$EnhancerBySpringCGLIB$$d5e55c92
memberDao	 wooteco.subway.member.dao.MemberDao$$EnhancerBySpringCGLIB$$ad3c49ad
memberController	 wooteco.subway.member.ui.MemberController
fareService	 wooteco.subway.path.application.FareService
pathFinder	 wooteco.subway.path.application.PathFinder
pathService	 wooteco.subway.path.application.PathService$$EnhancerBySpringCGLIB$$e164c072
pathController	 wooteco.subway.path.ui.PathController
sectionService	 wooteco.subway.section.application.SectionService
sectionDao	 wooteco.subway.section.dao.SectionDao$$EnhancerBySpringCGLIB$$c4e6c18b
stationService	 wooteco.subway.station.application.StationService$$EnhancerBySpringCGLIB$$12e3f762
stationDao	 wooteco.subway.station.dao.StationDao$$EnhancerBySpringCGLIB$$55999e31
stationController	 wooteco.subway.station.ui.StationController
...(생략)
##################################################################### END
2021-06-11 16:48:15.264  INFO 84538 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
2021-06-11 16:48:15.266  INFO 84538 --- [extShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Shutdown initiated...
2021-06-11 16:48:15.268  INFO 84538 --- [extShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Shutdown completed.
BUILD SUCCESSFUL in 18s
4 actionable tasks: 2 executed, 2 up-to-date
4:48:15 오후: Task execution finished ':test --tests "wooteco.subway.line.LineAcceptanceTest.defaultListableBeanFactory"'.
```

## References
- [테스트에 사용한 코드](https://github.com/Hyeon9mak/playground/blob/master/src/test/java/wooteco/subway/line/LineAcceptanceTest.java)
- [[Spring] 빈 등록 정보 확인하기 - DefaultListableBeanFactory](https://blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kbh3983&logNo=220876632483)