# 10월 17일 강의

> 계층을 고려한 리팩터링이 전체 강의의 핵심

## 계층화

계층화(layering)는 소프트웨어 설계자가 복잡한 소프트웨어 시스템을 분할하는 데 사용하는 가장 일반적인 기법.
계층의 관점에서 시스템을 보면 하위 계층 위에 다음 상위 계층이 있는 일종의 레이어 케이크와 같은 형식의 원리 체계가 머릿속에 떠오릅니다.
이 체계에서 상위 계층은 하위 계층이 정의하는 다양한 서비스를 사용하지만, 하위 계층은 상위 계층을 인식하지 못합니다.

### 클라이언트-서버 시스템에서 3티어 아키텍쳐로

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/21345266-89a3-4ea8-9437-0388606daabd/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221018%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221018T023157Z&X-Amz-Expires=86400&X-Amz-Signature=5faa6241bc9e6bec30b0e8b4c2660841e802affb40905efcd76e1ffa72592347&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

클라이언트-서버 시스템이 많은 비중을 차지하던 시기(옛날 옛적)에는 도메인 로직을 UI 화면에 직접 넣어서 개발을 했습니다.
화면에 삽입한 로직은 코드를 복제하기가 쉽지 않기 때문에 간단한 변경을 수행하더라도 수 많은 비슷한 코드를 찾아야 했습니다.
그래서 도메인 로직을 저장 프로시저로 만들어 데이터베이스에 저장하는 방법으로 해결하려 했습니다.
저장 프로시저는 제한적인 구조화 메커니즘을 가지고 있기 때문에 어색한 코드가 되는 문제는 여전했습니다.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/7ad0bdf0-b550-44b7-89eb-6a2768061e95/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221018%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221018T023216Z&X-Amz-Expires=86400&X-Amz-Signature=1851f6cf26a52fdda54394aea4ee12a5e4b8b0f5cf3d461300ac23a46f1d4fab&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

당시 객체지향에 대한 관심이 늘어나면서 객체 커뮤니티가 활성화되고 있었는데이러한 문제를 객체 커뮤니티에서는 도메인 로직 문제를 3계층 시스템으로 해결하려 했습니다.

- ui 를 위한 프레젠테이션 계층
- 도메인 로직을 위한 도메인 계층
- 데이터 소스를 이용

이 방식을 통해 복잡한 도메인 로직을 ui에서 분리해 별도의 계층으로 만들고 객체를 활용해 올바른 구조로 만들 수 있었습니다.
처음엔 당시 시스템이 그만큼 복잡하지 않아서 인기가 없었는데 웹이 등장하면서 판도가 바뀌었습니다.
서버 클라이언트 애플리케이션을 웹 브라우저로 배포하기를 원했습니다.
당시 리치 클라이언트 안에 모든 비즈니스 로직이 포함돼 있었기 때문에 웹 인터페이스를 지원하려면 모든 비즈니스 로직을 전면적으로 수정했어야 했습니다.
당시 3계층으로 설계된 시스템은 프레젠테이션 계층을 추가해 문제를 해결할 수 있었습니다.

### 레이어드 아키텍처

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/3da0b061-787d-44a0-bef2-c7c54d944e0f/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221018%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221018T023303Z&X-Amz-Expires=86400&X-Amz-Signature=e3671299baa43e8d80c9c0774e615e0110d8a0b4fe37142e4fab297e56353794&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

레이어드 아키텍처는 오늘날 가장 일반적인 계층형 아키텍처 패턴입니다.
계층화된 아키텍처 패턴 내의 구성 요소들은 수평 계층으로 구성되며, 각 계층은 애플리케이션 내에서 특정 역할(예: 프레젠테이션 논리 또는 비즈니스 논리)을 수행합니다.
스프링을 처음 접하는 많은 사람들은 **`컨트롤러 - 서비스 - 다오(레포지토리)`**라는 계층으로 접할텐데요,
이는 표현을 담당하는 Presentation Layer에 Controller 객체가 위치하고,비즈니스 규칙을 담당하는 Business Layer에 Service 객체가 위치하고,영속성 로직을 담당하는 Persistence Layer에 Dao 객체가 위치합니다.

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/289dbe1c-ad48-4926-96f6-e8c3e461be58/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221018%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221018T023323Z&X-Amz-Expires=86400&X-Amz-Signature=ab61eff7335a624edfbf8375e048c1f5027b60c105349ceffffd2e2df7e34892&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

### 무조건 따르는게 좋은가?

아키텍처에 대해 어느정도 이해가 있고 숙련도가 있다면처음부터 아키텍처를 설계한 뒤 그대로 구현하면 효율적일 수 있습니다.

```
    public List<Reservation> findAll() {
        return reservationRepository.findAll();
    }

```

그러다 보면 이러한 코드를 경험하시게 될 수 있는데요, 이 코드가 잘못되었다는 이야기는 아니지만 설계에 대해 충분한 이해 없이 관성으로 코드를 작성하다 보면 이게 맞나? 와 같은 의문이 들 수 있습니다.
학습 단계에서는 그 필요성과 장단점을 느끼면서 구현하는것을 추천합니다.

> 직접 생성해서 다루려면 파라미터도 많이 필요하고, 의존성을 들고 돌아다녀야한다. 싱글톤 유지도 어렵다.

<br>

## Spring Core 소개

### Spring Container

```java
@RestController
public class ReservationController {
    private final JdbcTemplate jdbcTemplate;

    public ReservationController(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @PostMapping("/reservations")
    public void save() {

        jdbcTemplate.update(connection -> {

        ...
    }
}

```

컨트롤러에서 디비에 접근하기 위해서는 JdbcTemplate 클래스를 활용합니다.
이 때 JdbcTemplate은 개발자가 직접 생성하지 않고 외부로 부터 받아옵니다.

```java
@RestController
public class ReservationController {

    @PostMapping("/reservations")
    public void save() {
        DataSource datasource = new DataSource(...)
        JdbcTemplate jdbcTemplate = new JdbcTemplate(datasource)
        jdbcTemplate.update(connection -> {

        ...
    }
}

```

그럼 왜 직접 JdbcTemplate을 직접 생성하지 않을까요?

```java
@RestController
public class ReservationController {
    private final JdbcTemplate jdbcTemplate;
    ...
}

```

```java
@RestController
public class ThemeController {
    private final JdbcTemplate jdbcTemplate;
    ...
}

```

디비 접근이 필요한 객체는 많은데 그 때 마다 똑같은 객체를 생성하는 것은 비효율적일 수 있습니다.JdbcTemplate처럼 여러군데서 필요한 객체를 직접 생성하고 관계를 맺어주는 로직을 구현하려다보면 귀찮죠.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/7429414ea1204b4eae9e0a34e00cba29](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/7429414ea1204b4eae9e0a34e00cba29)

그래서 이러한 작업은 스프링이 대신해줍니다. 스프링은 개발자가 지정한 클래스의 객체를 직접 생성하고 만약 객체가 다른 객체에 의존하고 있다면 생성할 때 의존하는 객체와 함께 생성해주어 의존 관계도 관리해줍니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/931bf5eb427245728f85e44edef2df67](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/931bf5eb427245728f85e44edef2df67)

스프링이 관리하는 빈을 스프링빈이라고 합니다.그리고 스프링빈을 관리는 스프링 (IoC) 컨테이너에서 담당합니다.스프링 컨테이너는 여러가지 이름으로 불리우는데 이는 역할에 따라서 부르는 이름이 달라질 수 있습니다.(여기서는 다루지 않겠습니다 ㅎ.ㅎ)

### Component Scan

스프링은 모든 객체를 관리해주나요?아닙니다. 등록한 클래스의 객체만 관리합니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/c6d9603d0b3143e79839f2417c84f5dc](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/c6d9603d0b3143e79839f2417c84f5dc)

몇 가지 방법 중 이 문서에서는 애너테이션을 이용한 등록 방법을 알아봅니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/a38908fc0c9345be80d2978a8b5a427a](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/a38908fc0c9345be80d2978a8b5a427a)

스프링빈으로 등록하고 싶은 객체에 해당 애너테이션을 붙입니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/ade736eef1004077ae104ec4b4548e25](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/ade736eef1004077ae104ec4b4548e25)

@Component를 찾는 것을 Component Scan이라고 하는데 스캔을 하는 시작점을 알려주는 용도로 @ComponentScan을 클래스에 붙입니다.
이렇게 할 경우 @ComponentScan을 붙인 클래스가 위치하고 있는 패키지를 포함하여 하위 패키지에 존재하는 @Component를 찾고 @Component를 가지고 있는 클래스의 객체를 스프링빈으로 만들어서 관리합니다.
뿐만 아니라 스프링빈 대상 클래스들이 서로 의존 관계를 가지고 있으면 의존관계도 고려해서 빈을 생성합니다.

### 어떤 객체를 관리할까요?

앞서 소개한대로 모든 객체를 관리할 필요는 없습니다.
일반적으로 상태가 없는 비즈니스 플로우 객체가 빈 관리의 대상이 된다면
매번 생성하고 주입해주어야 하는 번거로움을 덜 수 있습니다.

### **Spring MVC?**

Spring MVC에서의 MVC는 Model, View, Controller 패턴의 그 MVC가 맞습니다.어떻게 MVC라는 개념이 컴포넌트의 이름이 되었는지를 알아보기 위해서는 옛날얘기를 꺼내야 할 것 같습니다.

Spring Web MVC는 Servlet API를 기반으로 구축된 웹 프레임워크입니다.
스프링 MVC는 다른 많은 웹 프레임워크와 마찬가지로 프론트 컨트롤러 패턴을 중심으로 설계되었습니다. 
중앙 서블릿인 DispatcherServlet은 요청 처리를 위한 공유 알고리즘을 제공하고 실제 작업은 구성 가능한 위임 구성 요소(Handler)에 의해 수행됩니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/5002957548ff465d885cd402499fcd11](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/5002957548ff465d885cd402499fcd11)

MVC는 Model, View, Controller의 약자로, 사용자 인터페이스 상호작용을 세 가지 독립적인 역할로 분할한 패턴입니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/91583d47a0814b918e8e44eaef1a8061](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/91583d47a0814b918e8e44eaef1a8061)


모델은 도메인에 대한 정보를 나타내며 뷰는 UI에서 모델을 표시하는 역할을 합니다.방문 횟수 조회 애플리케이션에서 모델이 방문 횟수 객체라면 뷰는 UI 객체라고 볼 수 있습니다.
정보에 대한 모든 변경 사항은 컨트롤러가 처리합니다.컨트롤러는 사용자로부터 입력을 받고, 모델을 조작하며, 뷰를 적절하게 업데이트 합니다.
MVC에서의 핵심 개념은 **`뷰 <-> 모델`** 분리와 **`컨트롤러 <-> 뷰`** 분리 입니다.

### **뷰 <-> 모델**

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/fa34a33a7910458194cfb13a77c5f1d1](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/fa34a33a7910458194cfb13a77c5f1d1)

뷰와 모델을 분리할 때 여러 장점을 가지고 있습니다.
뷰와 모델의 관심사는 다르기 때문에 이 둘의 개발을 분리하는게 유리합니다. 뷰는 UI의 메커니즘을 주로 고려하고 사용자 인터페이스에 집중하는 반면 모델은 비즈니스 로직이나 데이터베이스 상호작용 등을 고려합니다.
시각적이지 않은 객체는 시각적인 객체보다 테스트하기 쉬운 경우가 많기 때문에 이 둘을 분리하여 개발할 경우 테스트에 용이하다는 장점도 있습니다.
뷰는 모델에 의존하지만 모델은 뷰에 의존하지 않게 구현하면 새로운 뷰를 추가하기 쉽습니다.

### **컨트롤러 <-> 뷰**

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/42a93c53e44542de98199642a2aad920](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/42a93c53e44542de98199642a2aad920)

일반적으로 **`뷰 <-> 모델`**에 비해 **`컨트롤러 <-> 뷰`** 분리는 중요도가 낮아서 컨트롤러와 뷰가 하나로 되어있는 구조가 많았지만, 웹 인터페이스가 확상되면서 다시 관심을 얻었습니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/925fbcd15e2c4820983292336c871dd1](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/925fbcd15e2c4820983292336c871dd1)

관심사에 따라 분리가 될 경우 **`뷰 <-> 모델`** 분리와 마찬가지로 장점을 가지고 있습니다.

### **컨트롤러**

MVC에서 컨트롤러를 구성하는 방식은 페이지 컨트롤러 패턴과 프론트 컨트롤러 패턴이 있습니다.

### **페이지 컨트롤러**

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/cfb5839321804de88f119ce265569d7b](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/cfb5839321804de88f119ce265569d7b)

페이지 컨트롤러는 웹 사이트의 특정 페이지 또는 작업에 대한 요청을 처리하는 개체입니다.쉽게 설명하면 특정 요청마다 처리하는 컨트롤러를 두어 처리하는 형식을 의미합니다.

> 참고 링크 : https://www.baeldung.com/mvc-servlet-jsp

### **모델**

```java
public class Student {
    private int id;
    private String firstName;
    private String lastName;

    // constructors, getters and setters goes here
}

```

### **컨트롤러**

```java
@WebServlet(
  name = "StudentServlet",
  urlPatterns = "/student-record")
public class StudentServlet extends HttpServlet {

    private StudentService studentService = new StudentService();

    private void processRequest(
      HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {

        String studentID = request.getParameter("id");
        if (studentID != null) {
            int id = Integer.parseInt(studentID);
            studentService.getStudent(id)
              .ifPresent(s -> request.setAttribute("studentRecord", s));
        }

        RequestDispatcher dispatcher = request.getRequestDispatcher(
          "/WEB-INF/jsp/student-record.jsp");
        dispatcher.forward(request, response);
    }

    @Override
    protected void doGet(
      HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {

        processRequest(request, response);
    }

    @Override
    protected void doPost(
      HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {

        processRequest(request, response);
    }
}

```

### **뷰**

```java
<html>
    <head>
        <title>Student Record</title>
    </head>
    <body>
    <%
        if (request.getAttribute("studentRecord") != null) {
            Student student = (Student) request.getAttribute("studentRecord");
    %>

    <h1>Student Record</h1>
    <div>ID: <%= student.getId()%></div>
    <div>First Name: <%= student.getFirstName()%></div>
    <div>Last Name: <%= student.getLastName()%></div>

    <%
        } else {
    %>

    <h1>No student record found.</h1>

    <% } %>
    </body>
</html>

```

### **프론트 컨트롤러**

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/2bd69ec1c7f7434598832d405fc2c74a](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/2bd69ec1c7f7434598832d405fc2c74a)

프론트 컨트롤러는 모든 요청을 단일 처리기 객체로 집중하는 방법으로 자주 수행하는 공통적인 작업을 한대 모아 처리할 수 있는 장점이 있습니다.

> 참고 링크 : https://www.baeldung.com/java-front-controller-pattern
>

### **처리기(컨트롤러)**

```java
public class FrontControllerServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request,
      HttpServletResponse response) {
        FrontCommand command = getCommand(request);
        command.init(getServletContext(), request, response);
        command.process();
    }

    private FrontCommand getCommand(HttpServletRequest request) {
        try {
            Class type = Class.forName(String.format(
              "com.baeldung.enterprise.patterns.front."
              + "controller.commands.%sCommand",
              request.getParameter("command")));
            return (FrontCommand) type
              .asSubclass(FrontCommand.class)
              .newInstance();
        } catch (Exception e) {
            return new UnknownCommand();
        }
    }
}

```

### **추상 명령**

```java
public abstract class FrontCommand {
    protected ServletContext context;
    protected HttpServletRequest request;
    protected HttpServletResponse response;

    public void init(
      ServletContext servletContext,
      HttpServletRequest servletRequest,
      HttpServletResponse servletResponse) {
        this.context = servletContext;
        this.request = servletRequest;
        this.response = servletResponse;
    }

    public abstract void process() throws ServletException, IOException;

    protected void forward(String target) throws ServletException, IOException {
        target = String.format("/WEB-INF/jsp/%s.jsp", target);
        RequestDispatcher dispatcher = context.getRequestDispatcher(target);
        dispatcher.forward(request, response);
    }
}

```

### **구현 명령1**

```java
public class SearchCommand extends FrontCommand {
    @Override
    public void process() throws ServletException, IOException {
        Book book = new BookshelfImpl().getInstance()
          .findByTitle(request.getParameter("title"));
        if (book != null) {
            request.setAttribute("book", book);
            forward("book-found");
        } else {
            forward("book-notfound");
        }
    }
}

```

### **구현 명령2**

```java
public class UnknownCommand extends FrontCommand {
    @Override
    public void process() throws ServletException, IOException {
        forward("unknown");
    }
}
```

> 결국 컨트롤러는 웹 요청을 잘 받기 위한 인터페이스(어댑터)다.

<br>

## Dispatcher Servlet

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/af95644d619c45b5aa14b360dc51473e](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/af95644d619c45b5aa14b360dc51473e)

스프링은 **`DispatcherServlet`**이라는 객체를 제공하는데 이 객체는 프론트 컨트롤러의 역할을 합니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/807be13a196f4a8ca0f6d12684eb7a87](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/807be13a196f4a8ca0f6d12684eb7a87)

> https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet-special-bean-types

**`DispatcherServlet`**은 프론트 컨트롤로서 request mapping, view resolution, exception handling 등과 같은 역할을 수행하는데 대신 수행해줄 객체에 해당 작업들을 위임합니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/f399d1bcdb124238aa3608fcc27e50dc](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/f399d1bcdb124238aa3608fcc27e50dc)

> tecoble - DispatcherServlet - Part 1
> 1. DispatcherServlet으로 클라이언트의 웹 요청(HttpServletRequest)가 들어온다.웹 요청을 LocaleResolver, ThemeResolver, MultipartResolver 인터페이스 구현체에서 분석한다.
> 2. 웹 요청을 HandlerMapping에 위임하여 해당 요청을 처리할 Handler(Controller)를 탐색한다.
> 3. 찾은 Handler를 실행할 수 있는 HandlerAdapter를 탐색한다.
> 4. 찾은 HandlerAdapter를 사용해서 Handler의 메소드를 실행한다. 이때, Handler의 반환값은 Model과 View이다.
> 5. View 이름을 ViewResolver에게 전달하고, ViewResolver는 해당하는 View 객체를 반환한다.
> 6. DispatcherServlet은 View에게 Model을 전달하고 화면 표시를 요청한다. 이때, Model이 null이면 View를 그대로 사용한다. 반면, 값이 있으면 View에 Model 데이터를 렌더링한다.
> 7. 최종적으로 DispatcherServlet은 View 결과(HttpServletResponse)를 클라이언트에게 반환한다.

### 인증(세션과 토큰)

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/7707c928c6ce40f893551e493e59b3cb](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/7707c928c6ce40f893551e493e59b3cb)

방탈출 애플리케이션의 예약 관리 기능은 관리자만 사용할 수 있어야 합니다.요청을 받은 서버는 어떻게 관리자인지 알 수 있을까요?

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/0f24d758b288419aa2ead141a08cd452](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/0f24d758b288419aa2ead141a08cd452)

요청을 보낼 때 누군지에 대한 정보를 포함해서 요청해야 합니다.아이디와 패스워드를 입력받고 이 정보를 포함해서 요청을 보내면 됩니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/6b69def114a0427081a4573dc37db7bf](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/6b69def114a0427081a4573dc37db7bf)

하지만 이 방법에는 문제가 있습니다. HTTP는 특성상 매 요청마다 아이디와 패스워드를 포함해서 요청을 보내야 합니다. 이를 위해 매번 사용자로부터 아이디와 패스워드를 입력받아 보내거나 클라이언트에 값을 저장해야합니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/c9fbbcdd0bdc42f2afc8bbbd8737918b](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/c9fbbcdd0bdc42f2afc8bbbd8737918b)

아이디와 패스워드를 매번 보내는 방법 대신 처음 아이디와 패스워드를 서버에 보내고 다음부터는 서버가 발급하는 다른 정보를 포함하여 보내는 방법도 있습니다. 이렇게 할 경우 반복적인 입력과 패스워드 정보를 보호할 수 있습니다.

<br>

## 인증 방법

### Basic Auth

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/021d4fcbfc334755bd14a34d4588d675](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/021d4fcbfc334755bd14a34d4588d675)

기본적으로 아이디와 패스워드를 요청에 포함하는 방법이 있습니다. 이 방법은 Basic Auth라고하며Authorization header에 아이디와 패스워드 정보를 인코딩한 정보를 포함하여 서버에 요청을 보냅니다.

### 세션

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/82083f62328b4c229bd7a7d4ac30fc43](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/82083f62328b4c229bd7a7d4ac30fc43)

최초 서버에 인증하기 위해 아이디와 패스워드를 보냅니다. 서버는 인증을 위한 요청임을 인지하고 세션이라는 공간에 일부를 할당받은 뒤 인식된 사용자의 정보를 담아둡니다. 그런 다음 할당된 공간의 식별값을 헤더에 담아 응답합니다.
응답을 받은 클라이언트(브라우저)는 헤더에서 식별값을 브라우저에 저장해 두고 서버에 다시 요청할 때 마다 식별값을 헤더에 포함시켜 보냅니다.
요청을 받은 서버는 헤더에 식별값을 찾고 있으면 세션에서 식별값으로 저장된 정보를 불러옵니다. 불러온 정보를 통해 누구인지 식별을 할 수 있습니다.

### 토큰

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/2ac44753d63f4e73ac128e671f1a3bf9](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/2ac44753d63f4e73ac128e671f1a3bf9)

세션과 마찬가지로 최초 서버에 인증하기 위해 아이디와 패스워드를 보냅니다. 서버는 토큰을 이용인증을 위한 요청임을 인지하고
응답을 받은 클라이언트는 토큰을 (로컬 스토리지 등)클라이언트 저장공간에 보관합니다. 이 후 요청할 때 마다 토큰을 헤더에 포함시켜 보냅니다.
요청을 받은 서버는 토큰을 찾고 있으면 토큰에서 사용자 식별 정보를 추출합니다. 불러온 정보를 통해 누구인지 식별을 할 수 있습니다.
