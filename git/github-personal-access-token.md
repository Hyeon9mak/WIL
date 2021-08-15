# Github Personal Access Token

작업 내용을 커밋 후 원본 저장소로 푸시하려 하니 아래와 같은 에러를 마주했다.

```
$ git push origin main

remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.
fatal: unable to access 'https://github.com/hyeon9mak/WIL.git/': The requested URL returned error: 403
```

Github에서 ID/PASSWORD 기반의 Basic Authentication 을 없애고 
ID/Personal Access Token 기반의 Token Authentication 으로 바꾼다 했었는데, 
그게 드디어 적용된 모양이다.

크게 복잡한게 있을까 걱정했는데, [Github Docs](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token)를 살펴보니 별거 없다. 조금 귀찮을 뿐...

<br>

## Github Personal Access Token 발급
Github 로그인 후 `프로필 이미지` 클릭, `Settings` 메뉴로 접근한다.

![image](https://user-images.githubusercontent.com/37354145/129464988-eb1d2c7b-3bc5-45b7-969a-42fd50879a66.png)

이어서 `Developer settings` 로 이동

![image](https://user-images.githubusercontent.com/37354145/129465015-66f8cc92-5455-4491-8756-3a50b0b93005.png)

`Personal access tokens` 메뉴의 `Generate new token` 버튼을 클릭해서 
토큰 발급을 진행하자.

![image](https://user-images.githubusercontent.com/37354145/129465036-d0c62f9c-365e-4a47-94de-3515096f054b.png)

우선 `Note`를 통해 토큰 식별값(이름)을 설정한다. 그 후 만기일을 설정할 수 있는데, 
`7일`, `30일`, `60일`, `90일`, `커스텀`, `만기되지 않음`이 있다.

![image](https://user-images.githubusercontent.com/37354145/129465174-09c587a2-777b-4f73-8ea7-806b6795f74e.png)

여기서 귀찮다고 `만기되지 않음`을 고르면 아래와 같이 귀엽게 경고를 해준다.
귀여운 경고를 무시할 수 없어 `90일`을 선택했다.

![image](https://user-images.githubusercontent.com/37354145/129465183-0e830c61-39de-4c6e-a2c8-91d7323488dd.png)

다음 권한 범위를 설정한다. 대부분의 경우 `repo` 권한으로 충분하지만, 나는 
`workflow` 파일을 건드리는 경우도 잦아서 `workflow` 까지 선택했다.

나머지 기능들은 대부분 Github 웹에 직접 로그인해서 사용하므로 필요해보이지 않았다.

![image](https://user-images.githubusercontent.com/37354145/129465234-a47bc859-6249-4fc3-b81a-d91fdf5f5f45.png)

Personal Access Token이 발급되면 곧장 토큰 값을 복사해야한다. 
이후 다른 화면으로 이동시 토큰 값을 확인할 수 없고 재발급 받아야 하므로, 바로 사용하자.

![image](https://user-images.githubusercontent.com/37354145/129465503-0c1b087c-c7a4-414c-a0b9-0a4c41bcfe32.png)


```
$ git push origin main
Username for 'https://github.com': [Github 계정명 또는 이메일]
Password for 'https://example@gmail.com@github.com': [Personal Access Token]
```

<br>

## KeyChain 설정 변경
그러나 Mac OS를 사용중이라면 push 명령을 수행해도 계속해서 같은 에러를 마주할 수도 있다.
이는 Mac에서 제공하는 `Keychain Access` 앱에 Github ID/PASSWORD가 등록되어 있어서 그렇다.
Github ID/PASSWORD 를 ID/Personal Access Token으로 바꿔주면 된다.

우선 `/System/Applications/Utilities/Keychian Access.app`을 실행시키자.

![image](https://user-images.githubusercontent.com/37354145/129466321-ceef1ff9-c7eb-426f-801c-dfeb2f895f04.png)

검색 기능을 이용해 `Github`를 검색하고, 종류가 `인터넷 암호`인 키를 더블클릭하자.

![image](https://user-images.githubusercontent.com/37354145/129466370-1fc82d48-2a4f-4977-a5f5-fc57ae7de159.png)

`암호보기`를 클릭해서 암호 입력칸을 활성화한 다음, 발급 받았던 Personal Access Token 값을 
입력해주면 된다.

![image](https://user-images.githubusercontent.com/37354145/129466419-86738f6b-05d0-4f59-a5d4-1bd290a4726e.png)

<br>

## References
- [Token authentication requirements for Git operations - The Github Blog](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/)
- [Creating a personal access token - Github Docs](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token)