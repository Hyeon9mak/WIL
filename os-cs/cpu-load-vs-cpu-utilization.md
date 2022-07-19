# CPU Load 와 CPU Utilization 은 어떻게 다른가

## 문제 상황
![image](https://user-images.githubusercontent.com/37354145/179343318-b5c5a901-bcbd-4f1e-bca2-e4f341eae291.png)
어느 날 현태님께서 분석계 DB 상태 알람이 동작하지 않는 걸 체크해달라고 하셨다.
CPU의 사용량이 70%를 넘어가면 관리자에게 알림이 오도록 설정해두신 것 같았다.

![image](https://user-images.githubusercontent.com/37354145/179343322-6ddac00f-c60b-4072-bff3-0439804d87c2.png)

CloudWatch 지표를 살펴보니 CPU Load가 70%를 넘기면 알림이 오도록 셋팅되어 있었다.
그러나 CPU Load는 평균적으로 0~1% 사이를 오갈 뿐이었다.
얼핏 보았을 땐 '거의 사용되지 않고 여유로운 상태인건가?' 라고 오해할 수 있었으나
분석계 DB는 전사에서 끊임없이 사용하므로 CPU 사용량이 1% 대에서 그칠리 없었다.

혹시나 CPU Load가 CPU 사용량을 의미하는게 아닌지 조사를 해보았다.

<br>

## CPU Load
**CPU Load 는 CPU에 실행중이거나 대기중인 작업의 개수를 평균적으로 보여주는 값**이다.
예를 들어 싱글코어 CPU에서 실행중이거나 대기중인 작업이 있는지 10번 확인했을 때 3번 발견된다면 CPU Load는 0.3이 되고,
10번 중 10번이 발견되면 1이 된다.
또한 CPU Load는 코어에 비례하기 때문에 듀얼코어 CPU에서 실행중인 작업 2개가 10번 중 10번 발견된다면 2가 된다.

<img width="1394" alt="image" src="https://user-images.githubusercontent.com/37354145/179740601-dfeddd44-3201-4911-89af-da9b0245f7c7.png">

싱글코어 환경에서 모든 작업이 실행중인 상태로 대기열이 꽉차 있다면 1, 반 밖에 없다면 0.5가 될 것이다.

<img width="1317" alt="image" src="https://user-images.githubusercontent.com/37354145/179741249-283f1d42-8f16-43d8-b5f9-5fbd350929e5.png">

듀얼 코어 환경에서는 실행중인 작업의 총량이 전체 환경의 절반이라면 1, 모든 환경을 꽉 채운다면 2가 될 것이다.

<br>

## CPU Utilization
CPU Utilization은 시스템(OS)과 사용자(App)의 CPU 사용량 합계를 의미한다. 
시스템 사용률이 높다면 시스템의 사양을 높여야하며,
사용자의 사용률이 높다면 애플리케이션 종료, 스케줄링 등을 고민해보아야 한다.
CPU 가 사용중인 양에 대해 한 눈에 파악하기 용이하다는 장점이 있다.

<br> 

## 해결
![image](https://user-images.githubusercontent.com/37354145/179343325-f997ecaa-41bd-4cab-ad78-0e024a1a66e9.png)

결국 CPU Load 대신 CPU Utilization이 70%를 넘기면 알림이 가도록 설정하니, 정상적으로 알림이 동작하는걸 확인할 수 있었다.
그러나 분석계 CPU의 Utilization이 100%에 도달한 상태를 오랜 시간 유지할 경우 추가적인 정보를 취득할 수 없을 것이므로,
분석계 DB의 CPU 코어 개수를 확인하고 다시 적합한 CPU Load 값으로 변경하는 것을 계획했다.

<img width="750" alt="image" src="https://user-images.githubusercontent.com/37354145/179742022-461c5908-71de-4359-ba2a-21b1b2a99b20.png">

<br>

## 정리
배경지식 없이는 CPU Load 지표가 의미하는 바를 파악이 어려우나, 
배경지식을 익히게 된다면 CPU Utilization이 제공해주는 것보다 더 많은 정보를 획득할 수 있다.
예를 들어 아래 그림과 같이 CPU Utilization이 100%인 A, B 서버가 있다고 가정해보자.
A 서버는 10초만에 작업이 끝났고, B 서버는 60초가 되어서야 작업이 끝나게 된다.

<img width="381" alt="image" src="https://user-images.githubusercontent.com/37354145/179736253-761e9fe9-5af4-4227-82ba-06d10ae6d13b.png">

10초가 되는 시점에 CPU Utilization를 측정하면 A, B 서버 모두 100%를 나타내지만, 
CPU Load는 A보다 B 서버의 값이 더 높게 나타나게 된다.
CPU Load는 남아있는 (대기중인) 작업까지 표시해주는 지표이기 때문이다.

일반적으로 CPU 코어가 1개인 경우, CPU Load 임계값을 0.7(일반론적인 값, 세밀한 값은 제각기 다름)로 잡는다고 한다.
코어당 CPU Load 값이 0.7이 넘어가는 경우 추가 CPU 자원을 구성하는 등 해결방법을 고민할 시간이 다가온 것이다.
안정적인 IT 서비스 운영을 위해 평소 CPU Load를 살펴보는 습관을 갖추면 좋을거 같다.

<br>

## References
- [CPU 지표 정리 - 이동인님 Brunch](https://brunch.co.kr/@leedongins/75)
