# @ModelAttribute을 setter 없이 사용할 수 있는 이유

많은 블로그에서 `@ModelAttribute` 어노테이션을 통해 데이터를 바인딩 할 때 setter 메서드가 필요하다고 했다.
그러나 실제 테스트를 진행해보니 setter 메서드 없이도 `@ModelAttribute`를 통한 데이터 바인딩이 성공됨을 확인할 수 있었다.

```java
// DTO
public class TestRequestDto {

    private String name;

    public TestRequestDto(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```
```java
// Controller Method
  @PostMapping("/test/{name}")
  public ResponseEntity<Void> test(@ModelAttribute TestRequestDto testRequestDto) {
      System.out.println(testRequestDto.getName());

      return ResponseEntity.ok().build();
  }
```
```java
// 테스트 코드
    @Test
  void 테스트() {
      RestAssured
          .given().log().all()
          .contentType(MediaType.TEXT_PLAIN_VALUE)
          .when().post("/test/현구막")
          .then().log().all()
          .extract();

  }
```
```
// 결과
현구막
```

심지어 DTO에 기본 생성자도 필요로 하지 않았다. 
오히려 기본 생성자를 구현했을 때 setter 메서드를 함께 구현하지 않으면 바인딩이 이루어지지 않음을 확인할 수 있었다.

```java
// DTO
public class TestRequestDto {

    private String name;

    public TestRequestDto() {
    }

    public TestRequestDto(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```
```
// 결과
null
```

setter 까지 함께 제공해야 정상적으로 동작된다.  
이 부분에 대해 [나봄 선생님](https://github.com/qhals321)께 달려가 질문했다.

처음엔 `@RequestBody, @ResponseBody`와 같이 Jackson 라이브러리를 주입 받아 동작할 것이라고 예상하고, Jackson 버전이 업됨에 따라 기본생성자와 setter 메서드 없이도 바인딩이 가능해진 것으로 접근했다.  
실제로 `@ConstructorProperties`이 [2.7.0 버전을 기점으로 사용 가능하게](https://www.logicbig.com/tutorials/misc/jackson/constructor-properties.html)되었고, 2.9.* 버전을 기점으로 `@ModelAttribute` 관련해서 사용 가능하게되었다는(정확하지 않다) 문서를 확인하고, Jackson 버전을 바꿔가며 테스트를 진행해보았으나 원하는 결과를 얻을 수 없었다.

결국 외부 라이브러리가 아닌 Spring 내부 구현체를 찾던 중 `ModelAttributeMethodProcessor` 를 찾게 되었다.

```java
package org.springframework.web.method.annotation;

public class ModelAttributeMethodProcessor implements HandlerMethodArgumentResolver, HandlerMethodReturnValueHandler {
    ...
```

`ModelAttributeMethodProcessor` 클래스의 `constructAttribute` 메서드에 해답이 있었다.

```java
protected Object constructAttribute(Constructor<?> ctor, String attributeName, MethodParameter parameter, WebDataBinderFactory binderFactory, NativeWebRequest webRequest) throws Exception {

    if (ctor.getParameterCount() == 0) {
        // A single default constructor -> clearly a standard JavaBeans arrangement.
        return BeanUtils.instantiateClass(ctor);
    }

    // A single data class constructor -> resolve constructor arguments from request parameters.
    String[] paramNames = BeanUtils.getParameterNames(ctor);
    Class<?>[] paramTypes = ctor.getParameterTypes();
    Object[] args = new Object[paramTypes.length];
    WebDataBinder binder = binderFactory.createBinder(webRequest, null, attributeName);
    String fieldDefaultPrefix = binder.getFieldDefaultPrefix();
    String fieldMarkerPrefix = binder.getFieldMarkerPrefix();
    boolean bindingFailure = false;
    Set<String> failedParams = new HashSet<>(4);

    ... 생략
```

`if (ctor.getParameterCount() == 0)` 분기에 의해서 파라미터 개수가 0개인 기본 생성자가 확인되면 우선 인스턴스(객체)를 생성하고, setter 메서드를 통한 바인딩을 시도한다.

그렇지 않을 경우 필드에 맞는 파라미터를 가진 생성자를 찾아 바인딩을 시도했다.

<br>

결국 스프링 내부에 구현되어 있는 `ModelAttributeMethodProcessor.constructAttribute()`에 의해서 
DTO에 기본 생성자와 setter 없이도 `@ModelAttribute`를 통한 데이터 바인딩이 가능하다.
