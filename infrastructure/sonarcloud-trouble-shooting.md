# (구구만 고생하신) SonarCloud 적용기

## summary
우아한테크코스 레벨4 [HTTP 서버 구현하기](https://github.com/woowacourse/jwp-dashboard-http/pull/26) 미션을 진행하던 중이었다.
미션대로 구현을 완료하고 PR(Pull Request)를 작성했는데 정적분석 툴 SonarQube 가 자동으로 동작하고,
분석 결과도 PR 페이지에서 곧장 보여주고 있었다?!

![image](https://user-images.githubusercontent.com/37354145/131992878-5cd3b768-e09b-4d61-a7a7-2229b92dd485.png)

자세히보니 SonarQube가 아니라 SonarCloud 였다!

[SonarQube에서 제공하는 Github 연동 서비스](https://www.sonarqube.org/github-integration/?gads_campaign=Asia-SonarQube&gads_ad_gr%5B)는 유료로 알고 있는데, 
SonarCloud의 경우 무료로 이용할 수 있었다!!

마침 Babble 팀에서도 배포 했을 때가 아닌 PR을 작성 했을 때 정적분석이 이뤄지길 바라고 있었고,
분석 결과를 PR 페이지에서 곧바로 확인할 수 있다는게 너무 멋져서 도입을 고려하게 되었다.

SonarCloud에 대해 조사하다보니, SonarCloud를 이용한 CI-based Analysis랑 Automatic Analysis를 동시에 구성하면 충돌한다는 이야기를 접하게 되었다. SonarQube를 이용한 CI-based Analysis를 유지할지, SonarCloud를 이용한 Automatic Analysis를 새로 적용할지 팀원들과 상의가 필요했다.

> "PR 단계에서 정적 분석을 미리 확인하면 배포시에 굳이 확인 안해도 되지 않을까?"  
> "그래도 배포시에도 확인하고 싶어."  
> "그럼 둘 다 가능한가? 둘 다 해보는 건?"

결국 SonarQube를 이용한 CI-based Analysis를 유지하면서 SonarCloud를 이용한 Automatic Analysis를 적용해보는 것으로 결정되었다.

<br>

## 적용기
우선 [https://sonarcloud.io/](https://sonarcloud.io/)에 접속, 
깃허브 계정으로 로그인 후 `Analyze new project`를 클릭해서 
Babble 팀 저장소를 등록하려 하니 계속해서 SonarCloud 페이지가 먹통이었다.
마침 HTTP 서버 구현하기 미션 진행중이라 한참 들여다보던 크롬 
`개발자 모드`-`네트워크 탭`을 살펴보니 에러가 하나 보였다.

```
errors: [{msg: "User is not admin of the ALM organization"}]
```

Babble 팀 저장소는 **woowacourse-teams/2021-babble** 로, 
팀원 모두 저장소에 대해 관리자 급 권한을 가지고는 있지만 
실제 저장소의 주인은 woowacourse-teams 그룹 관리자다. 
이 때문에 내 깃허브 계정으로는 SonarCloud에 woowacourse-teams/2021-babble 저장소 연결이 더 이상 불가능하다고 판단했고, 
구구 코치님께 DM을 드렸다.

---

![image](https://user-images.githubusercontent.com/37354145/131987782-8b07edcd-d43f-49fa-ae46-df1d859b319e.png)

![image](https://user-images.githubusercontent.com/37354145/131986729-62f1e10c-90ce-4d7a-8165-61dd96402aa0.png)

*'멋져서 하는 건 인정이지'*

납득해주신 구구가 woowacourse-teams/2021-babble 저장소에 SonarCloud 깃허브 앱을 연결해주셨지만, 
여전히 권한이 없다는 에러(`This action must be performed by an organization owner`)와 함께 SonarCloud와 연동이 진행되지 않았다.

![image](https://user-images.githubusercontent.com/37354145/131993842-72a84306-8066-4d7b-8a46-fcdd02e139b2.png)
![image](https://user-images.githubusercontent.com/37354145/131993593-052b7762-8694-4692-b955-cf4f4ca4dd9e.png)

2021-babble 저장소를 SonarCloud 서버 상에 프로젝트를 등록하기 위해선  
woowacourse-teams organization의 관리자임이 증명되어야하는데,
우리 팀 멤버들은 woowacourse-teams organization의 관리자가 아니다. 
때문에 `This action must be performed by an organization owner` 에러가 계속해서 발생한 것이다.

---

이어서 구구께서 `woowacourse-teams_2021-babble` 이름의 SonarCloud 프로젝트를 개설해주셨다. 
구구께서 공유해주신 링크를 통해 프로젝트가 생성된 것을 확인할 수 있었으나, 프로젝트를 이용할 수 있는 권한이 없었다. 
당연히 PR을 작성해도 정적분석이 일어나지 않았다.

![image](https://user-images.githubusercontent.com/37354145/132006377-63084749-dce5-45a8-9d12-a528801ab277.png)

---

그 다음은 구구께서 내 SonarCloud 아이디를 찾아 `woowacourse-teams_2021-babble` 프로젝트의 멤버로 등록해주셨다.
프로젝트의 멤버로 등록된 이후 프로젝트 이용이 가능해졌고, Github-actions를 이용한 정적분석 workflow 파일 등록을 진행했다.

![image](https://user-images.githubusercontent.com/37354145/132012963-f7e29902-97b9-4e1e-a787-4940b32106e7.png)

여러가지 Analysis Method 중 Github-actions를 이용한 방법을 선택한다.

![image](https://user-images.githubusercontent.com/37354145/132012991-19bca8c7-f696-495a-a3c1-dc6ddede7020.png)

Github-actions를 이용하는 가이드에서 깃허브 저장소에 `SONAR_TOKEN` 이름의 Secret을 등록하고 
사용할 것을 권장하는데, 우리 팀은 이미 `SONARQUBE_TOKEN` 이름의 Secret을 사용하고 있어 혼란이 있을거 같았다.
때문에 `SONARCLOUD_TOKEN`으로 이름을 바꿔서 등록했다.

![image](https://user-images.githubusercontent.com/37354145/132013032-2edb9551-1bec-4976-88fa-d36a1dce1c90.png)

프로젝트의 build.gradle에 sonarqube 플러그인과 의존성들을 추가해준다.
이어서 `.github/workflows/` 경로에 SonarCloud 정적분석용 workflow 파일을 작성해주면 된다.
우리 팀에선 아래와 같이 작성했다.

```yml
name: SonarCloud Automatical Analysis Build
on:
  push:
    branches:
      - main
      - release
      - develop
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
      - name: Set up JDK 11
        uses: actions/setup-java@v1
        with:
          java-version: 11
      - name: Cache SonarCloud packages
        uses: actions/cache@v1
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar
      - name: Cache Gradle packages
        uses: actions/cache@v1
        with:
          path: ~/.gradle/caches
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle') }}
          restore-keys: ${{ runner.os }}-gradle

      - name: update gradlew access authorized 
        working-directory: ./back/babble
        run: chmod +x gradlew

      - name: Build and analyze
        working-directory: ./back/babble
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
          SONAR_TOKEN: ${{ secrets.SONARCLOUD_TOKEN }}
        run: ./gradlew build sonarqube --info
```

(`update gradlew access authorized` 부분에서 gradlew 파일의 권한을 수정하는데, 
파일의 권한을 수정하지 않을 경우 `Build and analyze` 단계에서 gradlew 파일 액세스 권한 관련 에러가 발생한다.)

![image](https://user-images.githubusercontent.com/37354145/132013822-c666cc48-feb9-413d-8285-9d8c1d100f85.png)

여기까지 진행하고 PR을 다시 작성해보니, 정적분석이 동작했다!!
그러나 sonarcloud(bot)이 정적분석 결과를 PR에 코멘트로 남겨주지 않았다.
분명 2021-babble 저장소에 sonarcloud가 github app으로 등록되어 있었고, 정적분석도 잘 동작했다.
SonarCloud 프로젝트 상에도 정적분석 결과가 잘 전달되어 있었다. 그런데 왜 sonarcloud(bot)은 동작하지 않았을까?

---

원인 파악을 위해 구글링을 하던 중 SonarCloud 커뮤니티에 ['GitHub Pull Request does not show analysis results'](https://community.sonarsource.com/t/github-pull-request-does-not-show-analysis-results/40349) 게시글을 발견 할 수 있었다.
글의 내용은 "정적분석은 완료되었으나 그 분석결과가 Pull Request에 표기되지 않는다."는, 우리와 완전히 동일한 상황에 처한 글이었다.

답변으로는 `Bind this organization to GitHub` - `Choose the organization on Github` 설정이 되어있는지 확인해보라는 것이었다.

![image](https://user-images.githubusercontent.com/37354145/132014645-bc542a1d-16fa-4292-ad04-2860d90f01a7.png)

실제로 woowacourse-teams 조직과 woowacourse-teams_2021-babble 프로젝트가 바인딩되어 있지 않았다.
2021-babble 저장소의 workflow yml 파일이 sonarcloud의 woowacourse-teams_2021-babble 프로젝트에 분석결과는 전달했지만, 
그 분석결과를 PR에 작성하기 위해선 sonarcloud가 2021-babble까지 접근해야하는데, 2021-babble과 연결되어 있지 않으니 작성이 불가능한 것이었다!

직접 바인딩을 하려하니 다시 한 번 권한 문제를 마주쳤다. 이번에도 구구께 바인딩을 요청드리니...

![image](https://user-images.githubusercontent.com/37354145/132014884-03cace6f-d184-4e1c-be41-7fcdbe5d516d.png)

그토록 고대하던 정적분석 결과 sonarcloud(bot)이 등장했다!!

<br>

## Test Coverage 낮추기

SonarCloud를 이용한 Automatical 정적분석을 등록한지 얼마 되지 않아, 루트의 PR이 분석 기준을 통과하지 못했다는 메일이 날아왔다.
PR을 직접 확인해보니 Test Coverage가 78.2%로, 기준인 80%를 넘지 못해서였다.
Test Coverage를 높게 채울수록 좋겠지만, Test Coverage에 휘둘려 기능개발 시간을 잡아먹히길 원하진 않았다.
팀원 모두가 Test Coverage를 조금 낮추길 원했기 때문에 70%로 변경을 진행했다.

![image](https://user-images.githubusercontent.com/37354145/132016511-d124cf9e-c7f6-4d37-aadf-bebdee5545b9.png)

우선 SonarCloud 프로젝트의 `Quality Gates` 메뉴로 접근한다.

![image](https://user-images.githubusercontent.com/37354145/132016625-9807ce55-0d08-439a-b364-87ef16bb664f.png)

기본으로 제공하는 `Sonar way`의 우측상단 `Copy` 버튼을 눌러 Quailty Gate를 복제한다.

![image](https://user-images.githubusercontent.com/37354145/132016637-86803aea-7640-480a-854c-57edd7cf416f.png)

모든 통과기준이 동일하게 복제된 상태에서, 커버리지에 대해서만 `Edit` 버튼을 통해 수정하고, `Set as Default` 버튼을 통해 기본 Quailty Gate로 등록해주는 방법으로 Test Coverage를 낮출 수 있었다. (따로 Quailty Gate가 등록된 프로젝트가 아니라면 모두 Default Quailty Gate를 사용하게 된다.)

<br>

## SonarQube vs SonarCloud
SonarQube와 SonarCloud의 차이가 무엇인지 궁금해서 살짝 조사해보았다.

SonarQube는 CI-based 기반 정적분석, SonarCloud는 Cloud 기반 Automatical 정적분석. 
CI-based는 말 그대로 빌드/배포시에 정적분석 발생하고 SonarQube 스캐너, SonarQube 서버, SonarQube DB 등이 준비 되어 있어야 한다.

Automatical은 SonarQube서버, SonarQube 스캐너 등을 준비하지 않아도 SonarCloud에 프로젝트로 등록만 해두면 PR 단계에서 자동으로 정적분석. 
CI-based도 함께 설정 할 수 있지만, 아쉽게도 gradle는 SonarCloud CI-based 정적분석을 지원하지 않는다고 한다.

정리해보니 SonarQube의 장점이 거의 없는거 같지만? SonarCloud는 어디까지나 SonarCloud에서 제공하는 서버를 사용하고, SonarQube는 별도의 서버와 데이터베이스를 직접 구축할 수 있다. 그만큼 데이터관리나 확장이 자유롭기 때문에 각각의 메리트가 있는거 같다.

<br>

## 후기
babble 팀이 SonarCloud를 도입한게 아니라, 구구께서 babble 팀에 SonarCloud를 도입시켜주신게 아닐까 싶은 경험이었다.
(항상 감사합니다 구구! 🙇‍♂️)

SonarCloud의 프로젝트 등록이 지나치게 어렵지 않나라고도 생각했지만, 
다른 조직 프로젝트의 코드 취약점을 확인해서 공격하는 방식으로도 악용할 수 있기 때문에 
이렇게까지 프로젝트 등록을 어렵게 막아둔게 아닐까 싶기도 하다.

개인 프로젝트 저장소 정도는 간단하게 SonarCloud 프로젝트로 등록할 수 있으므로,
주변에서 SonarQube와 SonarCloud 중 하나를 추천해달라하면 SonarCloud를 적극 추천할거 같다.

<br>

## References
- [Github Code & Quality Analysis - SonarQube](https://www.sonarqube.org/github-integration/?gads_campaign=Asia-SonarQube&gads_ad_gr%5B)
- [GitHub Pull Request does not show analysis results - sonarsource community](https://community.sonarsource.com/t/github-pull-request-does-not-show-analysis-results/40349)
- [Setting up sonarcloud on gradle project on github - sonarsource community](https://community.sonarsource.com/t/setting-up-sonarcloud-on-gradle-project-on-github/32477)
- [SonarSource/sonarcloud-github-action - Github repository](https://github.com/SonarSource/sonarcloud-github-action)
- [GitHub Actions and SonarCloud - technology.amis.nl](https://technology.amis.nl/software-development/github-actions-and-sonarcloud/)
