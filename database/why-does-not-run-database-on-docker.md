# 도커 위에 데이터베이스를 운영하지 않는 이유
![image](https://user-images.githubusercontent.com/37354145/135798201-10147e04-4f47-4b4c-95e9-35795177b44d.png)

## 도커 위에 데이터베이스를 운영하지 않는 이유가 무엇인가요?
이전에 [로컬에서 잘 돌아가고 있는 데이터베이스를 굳이 도커 위로 마이그레이션 시킨 경험](https://hyeon9mak.github.io/migrate-local-database-to-docker/)이 있었다. 그 후로 큰 이슈 없이 서비스가 잘 동작했기 때문에 도커 위에 데이터베이스를 운영하는 것에 만족하고 있었는데, 얼마전 브라운이 초청해주신 홍정민 DBA님의 특별 강연에서 DBA님이 도커 위에 데이터베이스를 운영하는 것에 대해 부정적인 말씀을 하셔서, 질문을 드리고 답변을 받게 되었다.

> Q. 도커 위에 데이터베이스를 운영하지 않는 이유가 무엇인가요?
>
> ---
>
> A. 도커를 이용할 경우 데이터베이스를 이전하는 과정에서 
데이터 손실이 발생할 수 있기 때문에 아직까지도 현업에서 컨테이너 위에서 데이터베이스를 운영하지 않습니다.

데이터베이스는 데이터에 영속성을 부여하여 안전하게 관리하는 것이 가장 최우선이고,
도커는 이와 반대로 쉽고 빠르게 컨테이너를 생성하고 제거하는 등의 배포/개발 환경을 구성하기 위해 존재한다.

서로 패러다임이 충돌하는 와중에 데이터 손실 위험도 존재하고, AWS EC2 역시 가상의 OS 공간이기 때문에 그 위에 도커 컨테이너를 올리면 `가상 + 가상` 2중 가상의 형태로 관리 포인트만 늘어난다고 느껴졌다. 
결국 도커 위에 데이터베이스를 운영할 필요가 없다고 결론을 내렸다.

<br>

## 가만, EC2도 가상환경이잖아?
그런데 이렇게 결론을 내리고 나니 추가적인 의문이 생겼다.
흔하게 크루들끼리 '도커 위에 올리지 말고 로컬에 설치해' 라고 이야기할 때 **로컬**은
AWS EC2 인스턴스를 이야기하는데, 'EC2 인스턴스 또한 도커 컨테이너와 동일한 위험 부담을 가진 존재가 아닌가?' 라는 생각이 들었다.
그러다 보니 영속성이라는 단어 자체에도 의문이 생기고...

이 의문의 해답을 얻고 싶어 특별 강연을 진행해주셨던 홍정민 DBA님께 질문 메일을 드렸고, 답변을 받을 수 있었다.

> '로컬' 은 서버나 환경의 기준이 어떤 것인지에 따라 다르게 불릴 수 있을 것 같아요. 
내가 현재 코드를 작성하고, 컴파일하고 사용하는 환경이 내 Mac book 또는 window PC 라면 그게 로컬이 될 수도 있고,
EC2 인스턴스 위에서 뭔가를 작업한다면 EC2 인스턴스가 로컬이라고 불릴 수 있을 것 같네요.
> 
> '온프레미스 (On-premise)' 는 구글링 해보시면 아시겠지만 IDC (데이터 센터) 내에 물리적으로 랙에 설치한 서버에 전원도 연결하고, 네트워크 스위치와 연결해서 데이터 센터에 만들어놓은 환경입니다.
그 서버 위에 가상화 솔루션으로 가상 서버 (VM) 들을 구성할 수도 있고, 해당 서버에 Linux 또는 기타 OS 를 직접 설치해서 사용할 수도 있죠.
클라우드 환경 (EC2 인스턴스를 포함한 - AWS 와 같은 클라우드 사업자가 만들어놓은 기반 환경) 의 상반된 개념이고요. AWS 입장에서는 본인들의 클라우드 환경이, 본인들의 On-premise 환경이라고 볼 수도 있겠네요. 

로컬이나 온프레미스라는 단어를 결정지을 땐, 사용하는 시점이 중요하다는 걸 깨달을 수 있었다.
EC2 인스턴스 위에서 동작하는 애플리케이션 입장에선 EC2 인스턴스가 로컬일 것이고,
클라우드 서비스라고 생각하는 AWS도, AWS사 입장에서는 On-premise 환경이 되는 것이다!

<br>

## 도커는요?
> "도커를 이용해서 데이터베이스를 사용 또는 이전하는 과정에서는 데이터 손실이 100% 발생한다" 고 할 수는 없습니다.
그렇다면 "데이터베이스는 도커 환경에서 절대로 사용할 수 없다" 가 되어야 하는데, 그건 아니거든요.
사용 해보셔서 아시겠지만, 도커 환경에서 볼륨을 생성하고 데이터베이스를 설치/사용하다가 
컨테이너를 재시작하거나, 또는 컨테이너를 새로 생성해서 기존 볼륨을 마운트하면 데이터가 유실되지 않고 잘 남아있죠.
> 
> 제가 특강 중 언급한 도커 환경의 데이터베이스 안정성은
이론적인 관점에서의 접근이 아닌, 실제 현업에서 서비스를 운영하는 관점의 안정성으로 이해해주시면 좋을 것 같아요.

이론적으로, 그리고 간단하게는 도커 컨테이너에 데이터베이스를 운영하면 안정성에 큰 문제가 없었고 얻을 수 있는 장점도 많았다. (데이터베이스에 문제가 느껴질 때마다 쉽게 컨테이너에 올리고 내리는 것으로 관리하기 좋았다.) 
그런데도 도커 컨테이너에 올리면 안되는 이유는 이론보다는 현업에서 서버스를 운영하는 관점의 안정성이라고 하신다. 
대게 현업에서의 안정성이라고 하면 **'예측 할 수 없는 돌발상황'** 을 대체할 수 있는 능력일 것이다. 
도커 컨테이너도, 로컬에서도 돌발상황은 비슷하지 않을까? 어떤 차이를 이야기하신걸까?

> 도커 또는 클라우드 환경이 아닌 On-premise 환경의 머신에서도 데이터는 유실될 수 있고, 100% 안정성을 가진다고 할 수 없습니다.
> 이론적으로 동작하는 방식대로 동작이 되어야 하는데, 예외 사항은 언제나 발생할 수 있는 것 처럼요. 
그렇기 때문에 현업에서는 이런 안정성 (서비스 가용률) 을 높이기 위해서 database application layer 에서 코드를 수정하거나, 외부적으로는 이중화나 복제 기능을 구성하는 등 다양한 방법으로 개선이 되어있습니다.
> 
> 각 프로덕션 트래픽의 특징들이 다 다르고, 데이터베이스를 사용하는 방식이 다르기 때문에 고려해야되는 것들이 다양한데 (단순 select, dml 쿼리 외에도 다양한 패턴들이 사용됩니다)
도커 환경에서의 데이터베이스는, 아직까지는 안정성을 높이기 위한 다양한 검증과 개선이 활발하게 진행 중인 상태라고 생각합니다. (제 개인적인 견해입니다.)

가장 중요한 차이는 **'안정성에 대한 검증과 개선'** 이었다. 도커나 클라우드 환경이 아닌 On-premise, local 환경에서도 데이터는 유실될 수 있고, 100% 안전하지 않다. 때문에 여러가지 방법을 통해 안정성과 서비스 가용률을 높이려고 노력중이고, 도커 컨테이너는 이런 부분에서 검증과 개선을 진행하는 단계여서 현업 서비스에서 사용하기 불안한 상태였다.

<br>

## 안정성에 차이가 생기는 이유(...는 아직 모르겠다!)
> 여기서 한 스텝 더 나아가, 혹시 현구님이 이론적인 관점에서도 관심이 있으시다면
EC2 와 도커 컨테이너에서 I/O 동작 방식의 차이점이 무엇인지부터 접근해보시면 어떨까 싶어요. 
이 차이로 발생될 수 있는 문제점은 무엇인지, 그리고 좀 더 근본적인 개념으로 OS 위 application 에서 disk I/O 가 발생할 때, OS 시스템에서 (Linux) 어떻게 동작하는지도 함께 생각해보면 좋을 것 같네요.
기본적으로 EC2 에 도커 컨테이너를 올린다면 OS 기준으로는 어떻게 동작 하는 것 일까요?

크루들과 이야기를 나누고 여러가지 의견을 공유해보았으나 마땅한 답을 찾기 어려워 DBA님께 재질문을 드렸었다! 

> OS 위에 동작하는 애플리케이션과 OS 위 컨테이너에서 동작하는 애플리케이션의 I/O 차이에 대해서는
https://storageswiss.com/2015/08/31/docker-and-storage-understanding-docker-io/
요 링크와 운영체제 책에서 이야기하는 IO 서브시스템을 참고해보고, 크루들끼리도 이야기를 나눠봤는데 접근을 잘못한 것인지 조금 어렵게 느껴지네요... 😭
> 
> - 운영체제의 커널을 어떻게 사용하는지에 차이가 있을거 같다.
>     - OS 위에서 동작하는 애플리케이션이 I/O 동작을 요청하면 커널의 I/O 서브시스템이 이전에 동일한 요청을 받은적이 있는지 우선 확인하고, 
>       없다면 디바이스 드라이버에게 보내서 결과를 받는 등 캐싱도 가능하고, 버퍼링, 스케쥴링 등...
>     - 반면 도커 컨테이너가 생성하는 I/O 요청은 매번 동적으로 생성되고 사라지며, 모든 컨테이너가 하나의 커널을 공유하기 때문에
>       I/O 요청 하나가 커널을 공유하는 모든 컨테이너에 영향을 미치게 되지 않을까?
> - 운영체제의 표준 입출력과 컨테이너의 표준 입출력을 동기화하려면 추가적인 동작이 필요할거 같다.
>     - 이 때문에 운영체제 위에서 곧장 동작하는 애플리케이션과 다르게
>       도커 컨테이너 위의 애플리케이션은 추가 동작에 의존하게 될거 같다.
> - 애플리케이션 -> 도커 컨테이너 -> 운영체제 커널 3단계를 거치기 때문에 I/O 동작이 추가로 일어나서 성능에 이슈가 발생할거 같다.
> 
> 여러가지 의견이 나왔는데 '이거다!' 싶은 확신이 생기지 않고… 
> 잘못된 의견도 있을거 같아 섣부르게 생각하기 어려운 상태입니다.. 😭
> 이 부분에 대해서 재질문 드려도 될까요?!

그리고 아래와 같은 답변을 받을 수 있었다.

> 현구님과 크루분들이 고민해보신 방향대로 조금 더 알아보시면 어떨까 싶어요. 접근방식이 잘못되지 않았기 때문에 좀 더 연구해보시면 답을 찾으실 것 같네요!
> 
> 제가 어떤 방향으로 고민해볼지를 잡아드릴 수 있지만, 이 주제는 한 문장이나 한 단어로 답변이 나올 수 있는 주제가 아니기도하고, 
> 또 제가 해답을 가지고 정답을 맞추는 문제가 아니기 때문에 메일로 장문의 답변을 드리는데에는 한계가 있을 것 같아요.
> 함께 공부하시는 크루님들, 코치님들이 계신다면 같이 이야기 나눠보는건 어떨까요? 필요하면 테스트도 해보면 좋을 것 같고요. 

쉽사리 결론을 내놓을 수 없는 부분이기도 하고, 직접 알아내는 것이 더 가치가 있다고 생각해주신 것 같다!

최근 인덱싱, 리플리케이션, 클러스터링 등 데이터베이스의 여러가지 개념을 접하면서 '정말 공부할게 많구나.', '정말 내가 데이터베이스에 대해 모르는게 많구나.'를 새삼 느끼고 있다.
우선 데이터베이스에 대한 지식을 더 많이 쌓은 다음, 안정성에 차이가 발생하는 이유를 다시 고민해보아야겠다.

어쩌면 현업에서 개발을 진행하면서 그런 상황을 직접 겪게 될지도?

<br>

특별 강연 시간을 마련해주신 브라운 코치님, 함께 고민해준 크루들, 
특히 부족한 질문에도 소중한 시간과 지식을 나눠주신 홍정민 DBA님께 감사드린다! 🙇‍♂️🙇‍♂️🙇‍♂️