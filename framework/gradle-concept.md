## 1. 코어 컨셉

### 1-1. Ant 와 Maven 의 단점을 버리고 장점을 취한 빌드 도구

- Maven  의 장점
    - 개발자가 자유롭게 빌드 단위를 지정할 수 있다.
    - COC(Convention OverConfiguration) 전략에 따라 빌드 과정에 대한 많은 부분이 관례로 이미 정해져있다.
- Maven 의 단점
    - COC 전략은 특수한 상황에 유연하지 못하다.

Gradle 은 관례를 가지면서도 유연한 기능을 제공한다.
또한 XML 관련 이슈도 Groovy 언어로 해결했다.

### 1-2. Gradle 특징

- 고성능
    - Gradle은 입력 또는 출력이 변경되기 때문에 실행해야 하는 작업만 실행함으로써 불필요한 작업을 피한다.
    - 빌드 캐시를 사용하여 이전 실행 또는 다른 시스템의 작업 출력을 재사용할 수도 있다.
- JVM
    - **Gradle은 JVM위에서 동작한다.** 그러므로 JDK가 꼭 설치되어 있어야 한다. JVM의 장단점을 그대로 흡수한다.
    - 커스텀 task type이나 plugin과 같은 빌드 로직에서 표준 Java API를 사용할 수 있다. (Java와 친숙하다.)
- 컨벤션
    - Gradle은 Maven의 장점인 관례를 가져와서 일반적인 유형의 프로젝트를 규약을 구현하여 쉽게 구축할 수 있다. (일정한 컨벤션을 제공한다.)
    - 이를 통해, 적절한 플러그인을 적용하면 많은 프로젝트에 슬림한 빌드 스크립트를 쉽게 사용할 수 있다.
    - 물론, 이는 꼭 해당 관례를 따라야한다고 강제하진 않는다.

### 1-3. Gradle 에 대해 꼭 알아야 할 5가지

1. Gradle 은 Java 뿐만 아니라 모든 소프트웨어 빌드에 사용할 수 있는 범용 빌드 도구다.
    1. 대상이나 수행방법을 특정짓지 않기 때문이다.
    2. `plugins` 라는 기능 레이어에 라이브러리를 추가하여 친숙하게 다룰 수 있다.
    3. 사용자가 `plugin` 과 같은 커스텀 계층을 만들고 캡슐화 하여 다룰 수도 있다.
2. 핵심 모델은 DAG 모델을 기반으로 한다.
    1. 즉, Gradle 빌드는 작업(tasks)를 구성하고, 이들을 연결해서 한다는 것. 이때 작업들간의 의존성을 기반으로 작업 그래프(DAG)를 생성한다.
    2. Task: Actions + Inputs + Outputs 로 구성되어 있다.
    3. 작업 그래프 (task graph)가 생성되면, 어떤 작업을 어떤 순서로 실행해야 하는지 결정한 다음 실행을 진행한다.
    4. 이 특성으로 인해 특정 시점에 원하는 작업만 수행할 수 있다. 대상 작업이 의존하는 후속 작업들도 자연스럽게 실행된다.

![image](https://github.com/user-attachments/assets/f6e727f3-9042-4fc2-81f9-0c0f588fcd36)

1. 고정적인 빌드 단계가 이루어져있다. 3단계로.
    1. Initialization (초기화)
        1. 빌드를 위한 환경을 설정하고, 참여할 프로젝트를 결정.
    2. Configuration (배치, 구성)
        1. 빌드에 대한 작업 그래프를 구성, 순서 결정
    3. Execution (실행)
2. 여러가지 방식으로 확장 가능
    1. Custom task types
    2. Custom task actions
    3. Extra properties on projects and tasks
    4. Custom Convention
    5. A Custom model
3. build scripts operate against an API
    1. Gradle의 빌드 스크립트를 동작하는 코드(executable code)로 보기 쉽다.
    2. 잘 설계된 빌드 스크립트는 소프트웨어를 빌드하는데 어떻게 (`How`) 해당 단계들이 동작하는지를 묘사하지 않고, 어떠한 (`What`)단계가 필요한지 설명한다.
    3. 가장 크게 오해하는 부분이 바로 이부분이다. Gradle의 기능과 유연성이 빌드 스크립트가 코드이기 때문에 온다고 믿는 것이다.하지만 이것은 사실이 아니다.Gradle은 스크립트 코드가아닌 기본 모델과 API에서 그 힘이 발휘된다. Best Practice를 참고하자.모범사례에서 권장하는 것은 빌드 스크립트에 명령형 논리(How)를 가능한 적게 넣는 것이다. 대신 선언형 논리(What)을 사용하자.
    4. 하지만 빌드 스크립트를 동작하는 코드로 보는 것이 유용한 영역도 있다.
    5. 바로 빌드 스크립트의 문법이 어떻게 Gradle API와 매핑되는지를 이해하는 것이다.

   > Gradle은 JVM에서 실행되므로 빌드 스크립트도 표준 Java API를 사용할 수 있다.
>

## 2. gradle, gradle wrapper

Gradle의 설치는 두 가지로 나눈다.

1. Gradle설치 -> 설치 (컴퓨터에 전역적으로 사용할 Gradle을 설치하는 방법)
2. Gradle Wrapper -> 무설치 (프로젝트 상위 디렉토리에 [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html#gradle_wrapper)를 사용하는 방법)

Gradle 보다는 Gradle wrapper 사용이 추천된다.

1. 새로운 환경에서 매번 Gradle을 수동으로 설치할 필요없이 쉽게 프로젝트를 빌드할 수 있다. (종속적이지 않다)
2. 주어진 Gradle 버전에 따라 프로젝트를 표준화하여 더 안정적이고 강력한 빌드를 할 수 있다. (프로젝트를 빌드하는 모든 컴퓨터 환경을 일치하게 할 수 있다)
3. Wrapper의 내용을 수정함으로써 간단히 다른 사용자 및 실행 환경 (IDE, Jenkins등등)에 새 Gradle 버전을 프로비저닝할 수 있다.

![image](https://github.com/user-attachments/assets/a18948f3-2202-4ec5-b3fc-c40781c040e1)
![image](https://github.com/user-attachments/assets/589bc3d0-dfbb-446d-ad33-a7138a91ec2a)

## 3. 빌드 단계

### 3-1. **빌드에 기초가 되는 대표적인 객체 - `Gradle`, `Setting`, `Project`**

- [Gradle](https://github.com/gradle/gradle/blob/ba32027bf0656be5c8a71e6281939ff410a9cf1a/subprojects/core-api/src/main/java/org/gradle/api/invocation/Gradle.java)
    - Gradle 빌드 자체를 표현하는 객체이다.
    - Gradle 버전등 정보, 전체 Project 정보, 전체 TaskGraph (순서 포함)등을 표현한다.
- [Setting](https://github.com/gradle/gradle/blob/fd341b1e7016ff0ba82995b4e3211fb6e6805dd4/subprojects/core-api/src/main/java/org/gradle/api/initialization/Settings.java)
    - `settings.gradle`를 읽어 어떤 Project가 빌드에 참여하고 어떤 계층 구조를 가지는지 선언하는 객체이다.
    - Project 객체를 생성하진 않고, 어떤 객체가 생성되어야하는지 설정하는 객체.
- [Project](https://github.com/gradle/gradle/blob/6121fa83ce4ac07a27ee043d8e69b0f5f99d1c49/subprojects/core-api/src/main/java/org/gradle/api/Project.java)
    - 빌드하려는 각 Project의 `build.gradle`를 읽어 생성되는 Project 객체이다.

### 3-2. Initialization

1. **`settings.gradle` 파일을 확인하여 Single 프로젝트 빌드인지 Multi 프로젝트 빌드인지 파악하며, 그에 맞는 객체들을 초기화 및 생성한다.**
2. Initialization 단계에서 Gradle은 각 프로젝트에 해당하는 [Project](https://github.com/gradle/gradle/blob/6121fa83ce4ac07a27ee043d8e69b0f5f99d1c49/subprojects/core-api/src/main/java/org/gradle/api/Project.java) 인스턴스를 생성한다.
    1. 인스턴스를 생성만 할 뿐, 빌드스크립트(build.gradle) 을 읽어 Configuration 단계를 진행하진 않는다.
    2. 그리고 Gradle Daemon 은 JVM 위에서 동작하기 때문에 모두 JAVA 객체다.

### 3-3. Configuration

이전 단계 (Initialization)에서 식별하고 생성한 각 Project에 정의된 빌드 스크립트 (`build.gradle`)를 실행한다.

**정말 말 그대로 인터프리터로 빌드 스크립트 (`build.gradle`)의 내용을 위에서 아래로 한 줄씩 실행한다.**

```groovy
repositories {
    mavenCentral()
}
```

예를 들어 빌드 스크립트에 위 내용이 존재한다면, `mavenCentral()`이라는 `Closure` 인스턴스 (JAVA의 함수형과 유사)를 생성하고 `Project` 인스턴스 `Project.repositories(Closure)`에 `mavenCentral()`를 전달한다.

이는 해당 Project의 Task를 실행할 때 필요한 내용들을 설정하는 과정이라고 보면 된다.

Initialization 과 마찬가지로 설정만 할 뿐, 아직 Task 를 실행(Execution) 하는 단계는 아니다.

예를 들어 `hello` 라는 Task를 Project에 등록하고 싶다면 아래 내용을 빌드 스크립트에 추가하면 된다.

```groovy
tasks.register("hello") {
    doLast {
        println("Hello World!")
    }
}
```

Configuration 단계에서 위 스크립트를 읽고 Project 의 컨테이너 클래스(TaskContainer) 에 위 작업으 등록하게 된다.

### 3-4. Execution

컴파일, 파일 복사, 빌드 디렉토리 정리, 업로드 등 모든 작업이 이 때 수행된다.

**모든 Task는 실행할 작업들을 코드로 가지고있는다. Task를 실행하면 순서에 따라 해당 코드를 실행시키는 것 뿐.**

## 4. DSL(Domain Specific Language)

### 4-1. Closure/Lambda

`{ ... }` 는 `fun something() { ... }` 과 동일하다.

### 4-2. Chained Method

Groov DSLs는 최상위 문에 대한 메서드 호출시 인수 주위에 괄호를 생략할 수 있다.

예를 들어, `plugins`의 매개변수로 `id "some.plugin" version "0.0.1"`를 매개변수로 넘기는데 이는 `id("some.plugin").version("0.0.1")`과 같은 의미이다.

## 5. build script(build.gradle)

### 5-1. Build Script를 작성할 때 핵심이되는 4가지 개념(**projects, build scripts, tasks, plugins**)

build script 는 인터프리터 언어처럼 한 줄 한 줄 아래로 코드가 실행된다.

![image](https://github.com/user-attachments/assets/1f08468d-7bfc-4545-8b2e-3ba262b27b2c)

`build.gradle` 파일 자체는 `Project` 객체라고 보면 된다. `Project` 객체는 Task 를 수행하기 위한 모든 메서드와 속성들을 모아두는 슈퍼 객체다.

> Spring bean container 와 유사한 느낌..
>

### 5-2. 구조

각 Project는 하나의 build script를 실행시켜 task들을 실행하기위한 데이터 구조와 속성을 설정한다. (**Project 1 : N Task**)

자바코드 컴파일하는 Task나 클래스파일로부터 실행파일을 만드는 Task등 자바 애플리케이션 혹은 기타 다른 애플리케이션의 Task를 매번 작성하는 것은 비효율적이다.

이런경우 Plugin을 apply하면 자동으로 관련된 Task를 추가해준다. (**Plugin 1 : N Task**)

![image](https://github.com/user-attachments/assets/8434ad30-ced8-4a37-b44a-503dd8afb9ce)

### 5-3. 새로운 Task 생성

```groovy
task hello {
    println ("[configuration 단계] Hello World")

    doLast {
        println("[Execution 단계] Hello World")
    }
}

tasks.register("hello") {
    println ("[configuration 단계] Hello World")

    doLast {
        println("[Execution 단계] Hello World")
    }
}
```

### 5-4. 기존 존재하는 Task 설정 override

**Gradle엔 이미 수많은 Task가 정의되어있다.**

Gradle엔 이미 애플리케이션을 빌드하는데 필요한 수많은 Task가 정의되어있으며, 기본적으로 빌드 파일(`build.gradle`)에 import된 상태이다.

> 정확히는 Project객체의 TaskContainer안에 이미 많은 Task가 정의되어있다.
>

[DSL API에서 Task types](https://docs.gradle.org/current/dsl/index.html)에 정의된 다양한 Task를 설정만 바꿔서 활용하는 경우 테스크를 다음처럼 등록할 수 있다.

```groovy
tasks.register('<테스크-이름>', TaskClass) {
    // 테스크 설정
}
```

예시 -  copy

copy도 이미 Project 객체안에 정의되어있으며, 이를 아래와 같이 설정을 오버라이딩하여 `copy` Task가 실행되었을 때의 동작을 컨트롤할 수 있다.

```groovy
task copyFile(type: Copy) {
    from '{file 위치}'
    include '{포함할 파일 확장자'
    into '{file 위치}'
}

// 예시
task copySubmodule(type: Copy) {
    from './security'
    include '*.yml'
    into './src/main/resources'
}

tasks.register('<task-name>', Copy) {
    from '{file 위치}'
    includ '{포함할 파일 확장자}
    into "{file 위치}}"
}
```

**repositories, dependencies, configure 모두 이 방식대로 기존에 Project에 정의된 Task 설정을 오버라이딩하는 것이다!!!**

### 5-5. Task 간 의존 설정 (DAG)

**task의 의존성은 `dependsOn`을 통해 설정할 수 있다.**

```groovy
task a {
    doFirst {
        println "A"
    }
}

// dependsOn #1
task b {
    dependsOn a
    doFirst {
        println "B"
    }
}

// dependsOn #2
task c (dependsOn: 'b') {
    doFirst {
        println "C"
    }
}

task d {
    doFirst {
        println "D"
    }
}
```

`dependsOn ${Task}`에 정의된 Task가 먼저 실행되고나서, dependsOn을 설정한 Task가 실행된다.

- `Task d` 실행시 -> A, B, C 순으로 출력된다.

여러개의 dependsOn이 한 task에 존재한다면 그 순서는 보장되지 않는다. 순서에 보장된 task를 실행하기 위해서는 `mustAfterRun“ 을 통해서 정의해야한다.

## 6. plugin

**6-1. Plugin은 Task의 모음이다. (Plugin 1 : N Task**)

즉, Plugin은 Project에 자동으로 Task 들을 추가해준다.

**6-2. Plugin은 다른 의미로 빌드 스크립트의 외부 종속성을 추가하는 것과 같다고 볼 수 있다.**

즉, 빌드 스크립트에서 사용되는 여러 Task를 정의하기위해선 외부 종속성을 먼저 가져와야한다.

<br>

## Refernces

- https://mark-kim.blog/gradle_basic_4/
