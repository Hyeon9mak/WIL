- 요약. 잘 모르면 쓰지 마라.
- 반대로 말하면 잘 알고 써라.

## 새로운 프로젝트가 시작되면 무얼 해야할까?
- 프로젝트 팀은 사용자 수를 예측
	- 서버 사양과 수를 계산하여 구매요청
- 인프라팀은 서버를 구매하고 IDC 에 꽂음
	- OS, WEB, WAS 등을 설치하고 프로젝트 팀에 제공한다.

어느정도 규모가 있고, 서비스 규모가 작으면 VM 으로 빠르게 해주는 곳도 있음


- 사용자 수를 예측한다.
	- 상황에 따라 곗곡 변동되는데... 어떻게 예측할지 두루뭉실
- 필요한 서버 사양과 수를 계산하여 구매한다.
	- ??

- 각 서버로 애플리케이션을 배포한다.
	- 서버 수가 많을 수록 배포 시간이 오래 걸림.
	- 배포 과정에서 다른 버전의 서비스가 제공되기도 함
	- 배포가 실패하면 큰 손실

그 외에도...
웹 서버 설정 배포는? 웹 서버 인증서는 누가? 자바 버전 올리는 건 어렵고, 톰캣 로그는 어찌 봐야하지? DB 정보는 어떻게 숨겨야하지??

그래서 등장한게 CLOUD!

![](https://i.imgur.com/6TwjiSS.png)

- 서버가 죽으면 어디로 달려가야할까?
	- 달리긴 뭘 달려. 죽이고 새로 띄우면 된다.
	- EC2 는 서버가 아니라 인스턴스임
	- 문제를 찾고 고치는 것보다 죽이고 새로 띄우는게 더 빠르다.
- OS, WEB, WAS 설정은 어떻게?
	- 온갖 서버 설정을 모두 포함해서 AMI(Amazon Machine Image) 로 만들어주면 이미지를 배포한다.
- 서버가 계속 바뀌는데 배포는 어떻게?
	- CodeDeploy 를 사용
	- EC2 를 원하는 전략으로 배포 가능
- EC2 만으로도 이렇게 좋은데, kubernetes 는 왜 쓸까?
	- 서버 자원이 남는데, 한 서버에 서비스 2개 띄우긴 좀 그래...
	- 앱을 AMI 엔 넣기엔 배포가 너무 오래 걸리고, 안넣기엔 배포가 너무 오래 걸리고...
	- 서버 설정 히스토리 관리가 안되고...
	- MSA
- 컨테이너는 인식과 다르게 2001년부터 한땀한땀 발전해온 기술
	- 리눅스의 루트를 바꾸는 기술부터, 스냅샷, namespaces cgroups, docker ...
- 그럼 왜 갑자기 컨테이너를 쓸까?
	- EC2 는 OS 가 필요하다.
	- public cloud 가 보급되면서 컨테이너를 쓰기 좋은 환경이 됐다.
- K8S
	-  컨테이너 오케스트레이션
	- 10년도 안됐는데 벌써 1.27 버전
- 2015년에 나왔는데 왜 이제서야 쓸까?
	- 초기에 구성이 어려웠음
	- kubeadm 으로 서버를 설치하고
	- 6개월마다 인증서 갱신하고
	- 버전업이 빨라서 계속 바뀌고
	- 무엇보다 거부감이 컸음 (누가 운영할건데? 검증됐어?)

![](https://i.imgur.com/AXkNPRZ.png)

## k8s 하면 자주 들리는 단어사전
### Ingress
![](https://i.imgur.com/4scT0Ms.png)

path, header 에 따라 원하는 서비스를 노출시킬 수 있음

### Service
- K8S 에 있는 애플리케이션을 노출하기 위한 객체 (클러스터 내부에서)
  여러 개의 Pod 를 논리적으로 묶어서 트래픽을 전달
- selector 에 맞는 Pod 로 트래픽을 전달.
- 이걸 잘 사용하면 blue/green, canary 등 배포 전략을 활용 가능
- 여러가지 타입이 있다. 상황에 따라 선택
	- ClusterIP
	- NodePort
	- LoadBalancer
	- ExternalName

### Deployment
- 애플리케이션의 원하는 상태를 명시한 추상화된 객체
- 생성되기 원하는 Pod 의 spec 을 yaml 형태로 정의

### Pod
- context 가 공유되는 container 들의 set
	- 1pod != 1container
		- 하나의 pod 에 여러 container 가 존재할 수 있음
	- ip 를 공유한다? container 끼리 localhost 로 통신 가능
	- pod 내부 conatiner 끼리 volume 을 공유한다.

### Namespace
- 리소스 그룹을 격리하는 논리적인 단위
- Namespace 단위로 우리가 아는 자원들이 공유의 이름을 갖고 생성된다.
	- Ingress, Service, Deployment, Pod 등 모두 namespace 에 종속

![](https://i.imgur.com/kLuDb75.png)


VM 의 특정 포트로 들어왔다고 해서 특정 Pod 로 가는게 아니라, Service 가 어디로 밸런싱 하느냐에 따라 여기저기로 다르게 이동할 수 있다.

![](https://i.imgur.com/eehljqQ.png)

얘는 LB 뒤에 pod 가 직접 붙는다. 서비스를 통하지 않는 상태. 그래서 K8S 한다고 하면 듣는 잔소리가 있음

![](https://i.imgur.com/YVem3Qi.png)

![](https://i.imgur.com/n1CHqEs.png)
![](https://i.imgur.com/fu2sA9i.png)
![](https://i.imgur.com/Vb03MP5.png)

k8s 내부적으로도 결국 특정 노드에 묶는 다던지, pod 가 죽었다가 다시 뜰 때도 특정 스토리지를 이용해서 본인의 내용을 원복한다던지 여러가지 효과를 누릴 수 있다.
