# Git rebase merge 트러블 슈팅

babble 팀에선 프로젝트 초기 단계에서 아래와 같은 Git branch merge 전략을 사용하고 있었다.

### 기존 babble 팀에서 사용하던 Merge 전략
![image](https://user-images.githubusercontent.com/37354145/133914395-0362d137-51e7-4f30-a9ff-1fbf2da68bdf.png)

신규 기능 개발시 `Develop`를 기준으로 새로운 브랜치를 생성하고 작업을 진행한다.

![image](https://user-images.githubusercontent.com/37354145/133914396-ccb5a282-3e77-420a-a037-12281e6f44ac.png)

개발이 완료되었을 때 작업된 커밋 내용들을 Squarsh Merge를 이용해서 병합한다.

![image](https://user-images.githubusercontent.com/37354145/133956318-5ca17b3f-a969-4e32-a139-50e986c154e0.png)

Squarsh Merge를 통해 `Develop` 브랜치에 병합된 내용들을 `Release`, `Main` 브랜치에 동기화 시킬 땐 
일반적인 Merge를 이용한다.

정리하자면 아래와 같다.

- `신규기능 브랜치` --Squarsh Merge--> `Develop`
- `Develop` --(Default) Merge--> `Release`
- `Release` --(Default) Merge--> `Main`

이와 같은 Merge 전략을 사용하고 있었지만, 일반 Merge 특성상 Merge시 완전히 새로운 커밋을 작성하기 때문에 `Develop` 브랜치와 커밋 내용이 일치하지 않은 경우가 많았고, 이 때문에 `Release`, `Main` 브랜치의 커밋기록으로 어떤 기능이 추가되었는지 한 눈에 파악이 어려웠다.

그러다 (Default) Merge를 사용하는 단계에서 Rebase Merge를 사용해보면 어떻겠냐는 의견이 나오게 되었다.
Rebase Merge의 경우 병합되기 전 브랜치가 가지고 있는 모든 커밋 정보를 그대로 복사하기 때문에 
작업 내용을 추적하기 용이하다는 점과, 기능 개발이 어떤 단위로 이루어졌는지 한 눈에 파악하기 좋다는 장점이 있을거 같았다.

때문에 우리는 아래와 같은 가설을 세우면서 Rebase Merge를 적극 도입하게 되었다.

<br>

## babble 팀의 가설
![image](https://user-images.githubusercontent.com/37354145/133914400-fa753f8f-221b-4686-b208-5f6f67f56c57.png)

`Develop` 브랜치에서 `Release` 브랜치 쪽으로 Rebase Merge 진행하면 동일한 해시값을 가진 커밋이 
그대로 복제되어 생성되고,

![image](https://user-images.githubusercontent.com/37354145/133914403-876238dd-4f96-48bf-9a99-36be654773a6.png)

`Main` 브랜치까지도 이 커밋을 그대로 복제해서 옮길 수 있을거라 생각했다!

![image](https://user-images.githubusercontent.com/37354145/133914406-eafa8186-9249-439c-951a-87cdc54980d1.png)

작업이 반복되면, `Develop`, `Release`, `Main` 브랜치간 커밋이 완전히 동기화된 상태로 예쁘게 관리될 것 같았다!

그렇게 꿈에 부푼채로 Rebase Merge를 도입했지만, 얼마 안가 문제가 발생했다.

<br>

## Rebase Merge의 실제 동작와 충돌
![image](https://user-images.githubusercontent.com/37354145/133914417-c6868940-70cb-4c15-a688-8bb8cb6e2757.png)

Rebase Merge는 같은 해시코드를 가진 커밋을 복제해서 만드는게 아니라, 내용만 동일한 완전히 새로운 커밋을 새로 만드는 방식이었다.

![image](https://user-images.githubusercontent.com/37354145/133914422-04d3a2e7-b3f7-431c-b2e0-5605eec8975a.png)

때문에 첫 `Develop` 브랜치에서 `Main` 브랜치까지의 Rebase Merge는 성공적으로 수행할 수 있었지만, 곧이은 2번째 
Rebase Merge 작업에서는 `Develop` 브랜치와 `Release` 브랜치간 서로 브랜치 내용이 다르다고 판단되어 작업이 정상적으로 이루어지지 않았다.

<br>

## 그래서 결국엔...
![image](https://user-images.githubusercontent.com/37354145/133931052-c168f0a1-4b8b-4b7c-b78d-c4e61a402e80.png)

아쉽게도 기존에 사용중이던 Merge 전략을 다시 사용하고 있다.
