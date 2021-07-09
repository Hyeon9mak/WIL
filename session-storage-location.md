# 쿠키와 세션에서 세션은 어디에 저장되는가?

### 1. 스프링 Bean 객체로?
그렇다고 보기엔 스프링 이전부터 쿠키/세션 개념이 사용되고 있었다.

### 2.(디스패쳐)서블릿?
[토니의 테코톡 - 인증과 인가](https://www.youtube.com/watch?v=y0xMXlOAfss&t=716s) 영상에서 
세션은 각기 다른 서버가 하나의 세션을 동기화하여 갖지 못하는 문제점이 있다고 했는데, 
로드밸런서 역할을 수행하는 디스패쳐 서블릿이 세션을 저장하고 있다면 애초에 문제가 없었을 것이다.

### 3. 데이터베이스?
CPU가 저장장치(HDD, SSD) 접근을 줄이려고 노력하듯 서버에서도 데이터베이스의 커넥션을 줄이기 위해 안간힘을 쓰는데, 
세션을 데이터베이스에 저장해서 커넥션을 2배로 늘리는게 좋은 방향은 아니라는 걸 예측할 수 있다.

### 4. 서버 메모리?
가장 가능성이 높아보인다. 그러나 안그래도 부족한 메모리 공간을 세션으로 더 채운다는게 영 탐탁치 않다.

### 5. 하드 디스크?
메모리보단 느리지만, 여유 공간도 넉넉해 보이고 데이터베이스 커넥션보다 비용이 훨씬 저렴하므로 하드 디스크일 가능성이 가장 높아 보였다.

## 일단 정답은 메모리
결국 정답은 메모리라고 한다. 

정확히 이야기하면 아파치 톰캣의 세션 저장소에 세션이 저장 된다. 
톰캣의 세션 저장소는 톰캣이 점유중인 메모리 공간 내부에 있고, 그 톰캣은 결국 물리적인 서버 메모리에서 동작되므로 
결국 서버 메모리에 저장된다고 보는게 맞을 것이다.

**'그렇다면 아파치 톰캣의 세션 저장소가 쿠키/세션 개념보다 먼저 등장한 것인가?'** 라는 의문이 생긴다. 
위키백과를 통해 정리해보니, 최초 HTTP 쿠키의 등장 1994년이었고 최초 톰캣의 등장은 1998년이라고 한다. 
최초 세션은... 찾기가 어려웠다.

톰캣과 쿠키의 등장 연도가 그리 크게 차이나지 않는걸 보면, 세션도 그즈음에 등장했을 가능성이 높다.
즉, 우리가 아는 세션과 세션 저장소가 거의 동시에 등장했을 것이다!

<br>

## 하드 디스크와 데이터베이스도 맞다
하드 디스크와 데이터베이스도 틀린 대답은 아니다. 
메모리의 휘발성 문제를 해결하는 과정을 생각해보면 자연스럽게 데이터베이스까지 사용했을 것을 예상할 수 있다.

메모리에 저장되는 세션 저장소는 서버가 종료되면 모든 세션이 날아간다는 문제점을 갖고 있었고, 
이를 보완하기 위해 하드 디스크에 세션을 저장하기 시작했을 것이다.

그러다가 서버간 세션 동기화를 위해 하드 디스크가 아닌 세션만을 위한 별도 데이터베이스를 사용하기 시작했고, 
그러다가 세션만을 위한 별도 데이터베이스에 부하가 너무 커지는 것을 해결하기 위해 
토큰 개념이 등장하는 것으로 정리해본다.