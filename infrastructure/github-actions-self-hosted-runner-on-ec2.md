# Github actions self-hosted runner를 EC2에서 동작시키기
![image](https://user-images.githubusercontent.com/4648244/125721279-fabf57c7-99ef-4010-b900-67621f108361.png)

## Summary
최초 Github actions를 통해 프로젝트를 빌드하고 WAS EC2 인스턴스에 빌드된 파일을 전송하는 구조를 구상했다.
그러나 프로젝트 제약사항으로 `등록되지 않은 외부 IP를 통해 AWS EC2 SSH 접근이 불가능하다`가 있었고, 
Github actions 측에서 제공하는 서버로 빌드/배포 과정을 진행하는 방식으로는 완성된 빌드 파일을 보낼 수 있는 방법이 없었다.
코치님께서도 이 제약사항을 풀어줄 수 없다고 완강하게 말씀하셨다.

> 굳이 Github actions를 써야 겠다면, 몇가지 대안이 생각나긴 하네요.
> 
> 1. Github actions의 빌드 결과물을 도커로 빌드한후 도커 허브에 올린다. 서버에서는 이 이미지를 활용한다.
> 2. github action를 self-hosted runner 방식으로 활용한다. 
> 별도로 생성한 빌드서버(EC2, 혹은 로컬 서버)에서 운영서버로 ssh 연결이 가능하도록 구성한다.

어차피 WAS 프로그램도 도커 위에 구동시킬 것이라 1번 방법에도 메리트가 느껴졌으나, 얼마전 팀원 포츈이 
Github actions 측에서 제공하는 서버로 빌드를 시도했을 때 알 수 없는 이유로 빌드가 수행되지 않아서(일시적으로 Github actions 측 서버가 멈추었던 것으로 추측) 2번 방법에 대한 메리트가 더 강하게 느껴졌다.

<br>

## 젠킨스 대비 메리트
2번 방법을 통해 EC2상에 빌드서버를 구성할 경우 젠킨스를 이용하는 것 대비 장점이 무엇일까 고민해보았다.

- 학습 곡선이 젠킨스에 비해 완만하다. 
    - 젠킨스의 설정, 플러그인 지옥
    - Github actions는 yml 파일만 잘 작성해주면 된다.
- EC2서버로 구성해도 익숙한 Github UI를 활용할 수 있다.

아직 프로젝트의 규모도 크지 않고 팀원들도 Github actions의 간단함과 UI를 마음에 들어했기 때문에 
쉽게 결정을 내릴 수 있었다. 이번 기회에 EC2서버 Github actions를 구성하면 추후 젠킨스로 교체하기도 
용이할 터였다.

<br>

## self-hosted actions runner 구성
![image](https://user-images.githubusercontent.com/37354145/126052327-3c183236-898b-4811-996e-6df56bb2907c.png)

위 도식화에서 빨간 박스로 강조된 부분만 다룰 것이다. ([다른 부분들이 궁금할 경우 여기를 참고할 것](https://github.com/Hyeon9mak/WIL/blob/main/infrastructure/reverse-proxy-was-mysql.md))

### self-hosted 서버용 인스턴스 구성
github actions self-hosted 서버를 구성하기 위해 EC2 인스턴스를 생성하자. 
[이 때 인스턴스의 public IP는 활성화로 고정한다.](https://github.com/Hyeon9mak/WIL/blob/main/network/private-ip-can-not-connect-to-internet.md) self-hosted runner가 CI/CD 작업중 저장소의 Actions 탭으로 진행상황과 결과를 지속적으로 전달하고, 저장소 Actions 탭에서 CI/CD 작업을 직접 요청할 수도 있기 때문에 인터넷 연결을 필요로 한다. 

### self-hosted runner 설치 및 연결
![image](https://user-images.githubusercontent.com/37354145/126052643-c6f2746c-2eef-486d-acc0-e8a04bed2bb2.png)

github actions를 통한 CI/CD가 진행되기 원하는 저장소에서 `Settings - Actions - Runners - Add runner` 메뉴로 접근한다. 

![image](https://user-images.githubusercontent.com/37354145/126052777-a7d6978a-c713-4920-937b-ed04abc7596b.png)

self-hosted runner가 동작할 운영환경을 설정하고 해당 운영환경의 터미널로 접속, 
actions runner 압축 파일을 다운로드 후 압축 해제해준다.

이어서 CI/CD가 진행되기 원하는 저장소 쪽에서 발급된 토큰 등록을 통해서 actions runner와 
저장소간 연결을 진행한다. 

![image](https://user-images.githubusercontent.com/37354145/126053180-7a0a4004-4248-49c8-9d17-6c06bcc90350.png)

`name of runner` 설정은 `Settings - Actions - Runners` 메뉴에서 runner를 식별하기 위한 설정이다. 간단하게 Runner의 이름이라고 생각하면 된다.

`additional lables` 설정은 Runner의 추가 라벨이다. 잠시 후 CI/CD 작업을 위한 yml 파일작성시 runner를 식별하기 위해 
이름이 아닌 라벨을 사용하므로, 공들여서 라벨을 지정하자.

연결이 성공적으로 완료되었다면 `./run.sh` 명령어 입력시 
actions runner가 실행되면서 CI/CD 작업 요청을 기다리는 대기상태로 진입된 것을 확인할 수 있다.
(`nohup ./run.sh &` 등의 명령어로 항시 runner가 동작하도록 설정하자.)

![image](https://user-images.githubusercontent.com/37354145/126052884-2052d947-b957-49e4-a2af-b3f3cf0dabb9.png)

<br>

## CI/CD 작업 지정을 위한 yml 파일 작성
![image](https://user-images.githubusercontent.com/37354145/126053276-2ce8d69d-1eba-4cbd-8b56-487a64305280.png)

저장소에서 Actions 탭으로 접근하면 많은 yml 파일 예시(workflow)를 볼 수 있다. 저장소 프로젝트 성격에 맞는 workflow를 선택한다.

```yml
name: Gradle Package

on:
  release:
    types: [created]

jobs:
  build:

    runs-on: ubuntu-latest   # 이 부분에서 runner의 Label이 기입된다.
    permissions:
      contents: read
      packages: write

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      ... (생략)

```

[공식 문서를 참고해서](https://docs.github.com/en/actions/learn-github-actions) 프로젝트의 성격에 맞는 
yml 내용을 작성하면 된다. 이 때 주의할 점은 `runs-on` 설정에 self-hosted runner에 기입했던 라벨을 기입해줘야 
해당 runner를 식별하고 거기서 작업을 진행하게 된다. 그렇지 않을 경우 Github actions에서 제공하는 default runner가 작업을 수행한다.

yml 파일 작성을 완료하고 커밋을 진행하면 yml 파일이 프로젝트의 `/.github/workflows/*.yml` 형태로 저장되게 된다.
지정한 `on:` 설정에 따라 자동으로 EC2에 설치된 actions runner가 작업을 받아들이고 수행하게 될 것이다.
수행되는 작업 내용은 Actions 탭에서 실시간으로 확인할 수 있다.

![image](https://user-images.githubusercontent.com/37354145/126053408-31a5237a-13ae-4ca8-9472-3e2e1ea726fd.png)

<br>

## References
- [Github Actions의 Self-hosted runner 사용기 - Wayne's World](https://mildwhale.github.io/2021-04-24-build-machine-with-m1-macmini/)
- [About self-hosted runners - GitHub Docs](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)
