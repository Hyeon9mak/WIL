# Github actions를 이용한 ECS 배포 자동화

이미 ECS상에 배포환경(클러스터, 서비스, 태스크 정의)이 모두 준비된 상태에서, 
애플리케이션의 변화가 일어났을 때 Github actions를 이용해 
자동으로 배포를 진행하는(태스크 내부 애플리케이션 컨테이너를 교체하는) 자동화 방법을 경험해보았다.

<br>

## ECS 수동 배포
자동화 과정에 대한 더 높은 이해를 위해 ECS 수동 배포를 먼저 진행해보았다.

### ECR에 도커 이미지 업로드하기
<img width="1792" alt="image" src="https://user-images.githubusercontent.com/37354145/152668917-71a558ae-c26b-42fa-ac44-4ed2a45eb72d.png">

**ECS > Amazon ECR > Repositories** 메뉴 접근 후
**레퍼지토리 생성** 기능을 통해 도커 이미지를 업로드할 레퍼지토리를 생성한다.
(기존에 레퍼지토리를 생성해두었다면 생략)

<img width="1599" alt="image" src="https://user-images.githubusercontent.com/37354145/152668918-6f7bafa7-bcc2-4ad5-b4d3-5271dc8e712d.png">

도커 이미지를 업로드할 레퍼지토리를 생성하고, 이미지를 푸시할 수 있는 명령을 조회한다.

<img width="1178" alt="image" src="https://user-images.githubusercontent.com/37354145/152668923-084ac31b-0cc7-4e9b-aa03-7ff6553e3593.png">

제시되는 명령어를 통해 로그인에 성공하면 아래와 같은 결과를 확인할 수 있다.

```
$ aws ecr get-login-password {... 로그인 명령 ...}
Login Succeeded

Logging in with your password grants your terminal complete access to your account.
For better security, log in with a limited-privilege personal access token. Learn more at https://docs.docker.com/go/access-tokens/
```

로그인에 성공한 이후 푸시 명령 설명에 따라 차례대로 도커 이미지를 빌드하고, 
ECR에 푸시할 수 있다.

<img width="1599" alt="image" src="https://user-images.githubusercontent.com/37354145/152668927-6cc36dc6-2eee-4572-94fd-72d6a5f59323.png">
<img width="1478" alt="image" src="https://user-images.githubusercontent.com/37354145/152668929-887a44f0-a0a4-4f5e-a03c-4257e6223bc9.png">
<img width="1502" alt="image" src="https://user-images.githubusercontent.com/37354145/152707996-566eb9cc-d87c-4511-8b6c-18acf6aa515c.png">

이미지가 성공적으로 푸시되었을 경우 위와 같은 방법으로 조회할 수 있었다.
이미지 URI를 새로운 태스크 정의에 사용할 것이므로 복사해둔다.

### 새로운 태스크 정의하기
<img width="1629" alt="image" src="https://user-images.githubusercontent.com/37354145/152708104-dbc68778-0149-4df2-afd6-fc4f2f9923d9.png">
<img width="1372" alt="image" src="https://user-images.githubusercontent.com/37354145/152708121-ea060b4f-b5ac-46f7-82ba-9e9a842d635f.png">

**AWS > 태스트 정의** 메뉴로 접근 후 이미지를 교체할 태스크를 선택한 후,
**새 개정 생성** 버튼을 클릭한다.

<img width="1463" alt="image" src="https://user-images.githubusercontent.com/37354145/152708735-137c4e93-ca7e-4b4b-8ac2-9df13e5ef447.png">

태스크의 정의된 컨테이너들 중 이미지 교체를 원하는 애플리케이션 컨테이너를 확인하고,
복사해두었던 이미지의 URI를 붙여넣는 것으로 컨테이너 내부를 교체한다.
**생성** 버튼을 클릭하면 새로운 개정(버전)의 태스크 정의가 생성된다.

### ECS에 새로운 태스크 띄우기
<img width="1632" alt="image" src="https://user-images.githubusercontent.com/37354145/152668931-b92249ca-f0e7-464e-a987-7db0ac61078d.png">

**AWS > 클러스터 > {클러스터 이름} > 서비스 {서비스이름} > 구성** 메뉴로 접근 후 **편집** 버튼을 클릭한다.

<img width="986" alt="image" src="https://user-images.githubusercontent.com/37354145/152668932-2311056b-7f55-494d-8d6a-ba22609bb561.png">

배포를 진행할 애플리케이션이 담긴 개정(버전)의 태스크를 선택하고 **업데이트**를 클릭한다.
잠시 시간이 지난 후 기존 태스크가 종료되고 새로운 태스크가 실행되면서 배포가 완료된다.

사실상 태스크 컨테이너 내부의 이미지만 교체해서 배포를 진행한다기보다, 
완전히 새로운 태스크를 생성하고 교체하는 방식이다.

<br>

## ECS 수동 배포 - 트러블 슈팅
### 태스크 강제 종료
기존 ECS > 클러스터 > 서비스에서 동작중이던 태스크를 수동으로 종료시킨 후 새로운 태스크를 배포하면
종료시켰던 기존 태스크가 다시 살아나고 새로운 태스크는 배포에 실패하는 문제가 발생했었다.
원인이 뭘까 한참을 헤매이다가 '서비스가 특정 상태에 이르면 자동으로 배포를 일으키나?'라는 의심을 하게 되었고,
실제 테스트로 태스크를 삭제만 해보니 서비스가 1개정(버전) 태스크 정의를 곧바로 불러와서 태스크 개수를 유지하고 있었다.

> [태스크들의 Life cycle 을 관리하는 부분을 서비스라고 한다. 태스크를 클러스터에 몇 개나 배포할 것인지 결정하고, 실제 태스크들을 외부에 서비스 하기 위해 ELB 에 연동 되는 부분을 관리하게된다. 만약 실행 중인 태스크가 어떤 이유로 작동이 중지 되면 이것을 자동으로 감지해 새로운 Task를 클러스터에 배포 하는 고가용성에 대한 정책도 서비스에서 관리한다.](https://bluese05.tistory.com/52)

현재 ECS의 서비스에서도 항시 1개 이상 연결포인트를 유지하려는(고가용성 유지를 위한) 정책을 가지고 있었고, 
이로 인한 현상으로 결론지을 수 있었다.
때문에 서비스에서 동작중이던 태스크를 종료시키고 새로운 태스크를 실행시키는 방법이 아니라,
기존 서비스에서 실행중이던 태스크의 개정(버전)을 교체하는 방법을 선택하게 되었다.

### M1 이슈
태스크의 개정(버전)을 교체하는 방법을 선택했음에도 수동 배포에 계속해서 실패했다.
태스크가 새롭게 준비될 때의 과정을 살펴보니 Datadog agent 컨테이너와 log router 컨테이너는 
성공적으로 실행되었으나, 애플리케이션 컨테이너가 계속 실행에 실패하는 모습을 보여주었다.

로컬에서는 정상적으로 동작하는 것을 분명 확인했으므로, 애플리케이션 문제라기보단 구동환경 차이로 예상을 해보았다.
처음에는 경수님께서 말씀해주신 [컨테이너에서 실행시키는 스크립트 에러](https://mns010.tistory.com/30)로 생각하고 접근했다. 
컨테이너가 어떤 프로그램을 이용해서 실행해야하는지 명확하게 게시해주지 않았을 경우 발생할 수 있는 에러인데, 
Dockerfile 내부에는 간단한 java 빌드만 진행할 뿐이었고, java 컴파일러 또한 명확하게 게시되어 있었으므로 
가능성이 낮아보였다.

다음으로 접근했던 방법은 아키텍쳐 차이였다. ECS 배포 실패에 관련해서 검색을 하던 중 M1 프로세스가 
관련이 있을 수 있다는 글을 stackoverflow에서 접했는데, 마침 노트북이 M1 프로세스였다.
경수님과 재찬님께 ECS 태스크 내부 컨테이너가 동작하는 환경의 프로세스가 amd64인지 질문을 드렸고,
'정확한 프로세스는 모르지만 M1 프로세스에서 빌드된 애플리케이션은 동작하지 않는다.' 라는 힌트를 얻을 수 있었다.

결국 해결 방법은 **amd64 프로세스를 가진 별도의 서버에서 빌드를 진행**하거나, 
**다른 프로세스에서 동작할 수 있도록 빌드를 하는 방법을 찾는 것** 2가지 였다.
나는 후자에 대해 조사를 진행했고, `$ docker buildx`라는 명령어를 찾을 수 있었다.

`$ docker buildx`는 비교적 최근에 등장한 신규 기능으로, 
현재 로컬 프로세스 외의 다른 프로세스로의 빌드를 가능하게끔 지원하는 기능이다.
직접 사용해보면 아래와 같은 정보를 확인할 수 있다.

```
$ docker buildx

Usage:  docker buildx [OPTIONS] COMMAND

Extended build capabilities with BuildKit

Options:
      --builder string   Override the configured builder instance

Management Commands:
  imagetools  Commands to work on images in registry

Commands:
  bake        Build from a file
  build       Start a build
  create      Create a new builder instance
  du          Disk usage
  inspect     Inspect current builder instance
  ls          List builder instances
  prune       Remove build cache
  rm          Remove a builder instance
  stop        Stop builder instance
  use         Set the current builder instance
  version     Show buildx version information

Run 'docker buildx COMMAND --help' for more information on a command.
```

`$docker buildx ls` 명령을 사용해보면 buildx 기능을 통해 빌드 가능한 프로세스들의 종류를 확인할 수 있다.

```
$ docker buildx ls
NAME/NODE       DRIVER/ENDPOINT STATUS  PLATFORMS
desktop-linux   docker
  desktop-linux desktop-linux   running linux/arm64, linux/amd64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
default *       docker
  default       default         running linux/arm64, linux/amd64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
```

ECS 태스크의 컨테이너는 amd64 프로세스일 것이라고 가정하고 buildx를 이용한 빌드를 진행해보았다.
우선 빌드를 진행할 빌더를 생성하고, 해당 빌더를 사용하도록 명령을 입력한다.

```
$ docker buildx create --name {빌더 이름}
$ docker buildx use {빌더 이름}
```

그 후 일반 `$ docker build` 명령을 사용하는 것과 같이 Dockerfile을 준비하고 아래 명령으로 빌드를 진행해본다.
`--platform` 명령은 어떤 프로세스를 위한 빌드를 진행할 것인지 명시하는 것이며, 
`-t` 옵션은 빌드된 도커 이미지의 태그를 정하는 것이다.

```
$ docker buildx build --platform linux/amd64 -t {이미지 태그명} {Dockerfile 경로}

WARN[0000] No output specified for docker-container driver. Build result will only remain in the build cache. To push result image into registry use --push or to load image into docker use --load
```

그러나 빌드를 진행해보면 위와 같은 경고 메세지를 내보내면서 빌드가 진행되지 않는걸 확인할 수 있는데,
`--push` 옵션을 함께 포함해서 도커 허브에 빌드된 이미지를 업로드한 후 사용할 것을 제안하고 있었다.
일반 `$ docker build` 명령과 다르게 `$ docker buildx build` 명령은 도커 이미지 로컬저장을 
지원하지 않는 것 같았다.

```
$ docker buildx build --platform linux/amd64 -t hyeon9mak/hyeon9mak-dev --push .
```

도커허브에 로그인한 후, 다시 `--push` 옵션을 포함해서 빌드를 진행시키면 성공하는 것을 확인할 수 있다.
도커허브에 저장된 이미지는 `$ docker pull` 명령을 통해 불러와서 로컬에 저장시킬 수 있었다.

```
$ docker pull hyeon9mak/hyeon9mak-dev
```

이후 ECR에서 확인할 수 있는 푸시명령을 순서대로 다시 수행해서 amd64 프로세스에서 동작하는 
이미지를 ECR에 저장했고, 배포에 성공할 수 있었다!

<br>

## Github actions를 이용한 ECS 자동 배포
```yml
name: Deploy hyeon9mak assignment
on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest

    env:
      AWS_REGION: ap-northeast-2
      ECS_CLUSTER_NAME: backend
      ECR_REPOSITORY_NAME: hyeon9mak-dev
      ECS_CONTAINER_NAME: hyeon9mak-container
      ECS_SERVICE_NAME: hyeon9mak-service-dev
      TASK_DEFINITION_NAME: hyeon9mak-task-dev
      # Github actions 자동배포 과정에서 사용할 환경변수들
      # AWS ECS 콘솔에서 이름이 모두 일치하는지 확인이 필요하다.

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Configure AWS credentials For Devzone
        uses: aws-actions/configure-aws-credentials@v1
        with:
          # Github Repository Secrets에 값 등록 필요!
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Install JDK 11
        uses: actions/setup-java@v2
        with:
          distribution: 'temurin'
          java-version: 11
          cache: 'gradle' # https://github.com/actions/setup-java#caching-packages-dependencies

      - name: build gradle
        run: ./gradlew clean build

      - name: Download dd-java-agent
        run: wget -O assignment-api/dd-java-agent.jar https://dtdg.co/latest-java-tracer
        # wget 명령의 -O 옵션을 통해 파일의 
        # 경로 및 이름을 지정하여 datadog agent 다운로드

      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          SERVICE_TAG: . # Dockerfile의 경로
          IMAGE_TAG: ${{ github.sha }} # Github가 제공하는 SHA 암호화 사용
        run: |
          # Build a docker container and
          # push it to ECR so that it can
          # be deployed to ECS.
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY_NAME:$IMAGE_TAG $SERVICE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY_NAME:$IMAGE_TAG
          echo "::set-output name=image::$ECR_REGISTRY/$ECR_REPOSITORY_NAME:$IMAGE_TAG"

      - name: Download Task Definition Template
        run: |
          aws ecs describe-task-definition \
            --task-definition ${{ env.TASK_DEFINITION_NAME }} \
            --query taskDefinition \
            > task-definition.json

      - name: Fill in the new image ID in the Amazon ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ env.ECS_CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}

      - name: Deploy Amazon ECS task definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE_NAME }}
          cluster: ${{ env.ECS_CLUSTER_NAME }}
          wait-for-service-stability: true

```

AWS 액세스 키와 보안 키의 경우 yml 파일에 노출되기 민감한 정보이므로, 
Github Secrets를 통해서 관리를 한다.
**애플리케이션 Repository > Settings > Secrets > Actions** 메뉴로 접근해서 
Repository secret으로 등록을 해준다.

<img width="1242" alt="image" src="https://user-images.githubusercontent.com/37354145/152668936-3b84444c-893e-4844-9785-3420988d77a0.png">

해당 키의 값들은 아래 명령을 통해서 확인할 수 있다.

```
$ ~/.aws/credentials

[default]
aws_access_key_id = {액세스 키}
aws_secret_access_key = {보안 키}
```

애플리케이션 Repository에 자동배포용 yml 파일이 준비되면, 
이후 브랜치에 커밋내용이 푸시가 될 때마다 자동으로 ECS 수동배포 과정이 반복될 것이다.

<br>

## References
- [github action을 활용한 ecs 배포자동화 - Woosik Kim](https://well-balanced.medium.com/github-action을-활용한-ecs-배포자동화-dd359c259910)
- [How to compile OpenJDK 11 on an M1 Macbook? - StackOverflow](https://stackoverflow.com/questions/68358505/how-to-compile-openjdk-11-on-an-m1-macbook)
- [Docker Buildx - docker docs](https://docs.docker.com/buildx/working-with-buildx/)
