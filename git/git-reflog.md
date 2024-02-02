> git rebase 까지 모두 완료 후 당당하게 remote repository 로 force-push 했는데 잘못되었음을 뒤늦게 인지했을 때 어떻게 되돌려야할까?

`rebase`, `reset` 명령어를 이용하다보면 자연스럽게 commit 들이 제거된다. `git branch -D {branch name}` 명령어를 이용하면 branch도 제거된다. 그러던 중 갑작스럽게 복구가 필요하다면 어떻게 해야할까?

다행히 깃은 모든 행동이력을 보관하고 있다. `git reflog` 명령어를 입력해 그간의 이력을 모두 되살펴보고, 복구를 진행하자.

## commit 복구

```
$ git reflog
```

```
6e6c645e (HEAD -> step3, origin/step3) HEAD@{0}: commit (amend): feat: 지하철 노선 구간 삭제 API 구현
c88f72cd HEAD@{1}: commit: feat: 지하철 노선 구간 삭제 API 구현
98bd6ed8 HEAD@{2}: commit: test: 지하철 노선 구간 삭제 인수 테스트 작성
ebcacdbf HEAD@{3}: commit: feat: 지하철 노선 구간 등록 API 구현
cb25a98c HEAD@{4}: reset: moving to HEAD^
92660f09 HEAD@{5}: commit (amend): feat: 지하철 노선 구간 등록 API 구현
c5deee5e HEAD@{6}: reset: moving to HEAD^
59d02804 HEAD@{7}: commit: test: 지하철 노선 구간 삭제 인수 테스트 작성
c5deee5e HEAD@{8}: commit (amend): feat: 지하철 노선 구간 등록 API 구현
eb8213ae HEAD@{9}: commit: feat: 지하철 노선 구간 등록 API 구현
cb25a98c HEAD@{10}: commit: refactor: 구간 추가로 인한 지하철 노선 생성/조회/삭제 로직 리팩터링
a84979ff HEAD@{11}: commit: test: 지하철 노선 구간 등록 테스트 코드 작성
dfbff524 HEAD@{12}: commit: docs: 요구사항 정리
c87d59ac (next-step/hyeon9mak, hyeon9mak) HEAD@{13}: checkout: moving from hyeon9mak to step3
c87d59ac (next-step/hyeon9mak, hyeon9mak) HEAD@{14}: pull next-step hyeon9mak: Fast-forward
0cc4ddae HEAD@{15}: reset: moving to HEAD
0cc4ddae HEAD@{16}: checkout: moving from step2 to hyeon9mak
8c41724d (origin/step2, step2) HEAD@{17}: commit: refactor: 지하철 노선 인수 테스트간 영향을 받지 않도록 초기화 로직 추가
6859f484 HEAD@{18}: commit: feat: 지하철 노선 삭제 API 구현
472f2924 HEAD@{19}: commit: feat: 지하철 노선 수정 API 구현
8c24e1c6 HEAD@{20}: commit: feat: 지하철 노선 조회 API 구현
98db64fd HEAD@{21}: commit: feat: 지하철 노선 목록 조회 API 구현
9b40e11d HEAD@{22}: commit: refactor: 지하철 노선 생성 패키지 정리
684f40fe HEAD@{23}: commit: feat: 지하철 노선 생성 API 구현
86ff6300 HEAD@{24}: commit: test: 지하철 노선 삭제 인수 테스트 작성
8de940ad HEAD@{25}: commit: test: 지하철 노선 수정 인수 테스트 작성
ac76538d HEAD@{26}: commit: test: 지하철 노선 조회 인수 테스트 작성
d258a99a HEAD@{27}: commit: test: 지하철 노선 목록 조회 인수 테스트 작성
2c11b783 HEAD@{28}: commit: test: 지하철 노선 생성 인수 테스트 작성
6530ec75 HEAD@{29}: commit: refactor: 지하철 역 패키지 정리
633957b3 HEAD@{30}: commit: docs: 요구사항 정리
0cc4ddae HEAD@{31}: checkout: moving from hyeon9mak to step2
0cc4ddae HEAD@{32}: rebase (finish): returning to refs/heads/hyeon9mak
0cc4ddae HEAD@{33}: rebase (start): checkout next-step/hyeon9mak
.
.
.
```

내림차순으로, rebase 시작부터 어떤 결정을 내렸는지까지 모두 하나하나 추적할 수 있다. 이동하고 싶은 시점의 Hash ID(`6859f484`) 값이나 HEAD 로부터 얼마나 떨어진 commit(`HEAD@{18}`)인지 확인이 끝난다면 빠져나와서 아래 명령어 중 하나를 입력하자.

```
$ git reset --hard 6859f484
```

```
$ git reset --hard HEAD@{18}
```

로컬 git 상태가 해당 시점으로 되돌아간다!

## branch 복구

마찬가지로 `reflog` 명령어로 복구하고 싶은 branch 이름과 hash id 를 확인한 다음, 아래 명령어를 입력하면 된다. 

```
$ git checkout -b {deleted branch name} {commit hash id}
```

## 꿀팁

`grep` 명령어를 함께 이용하면 찾고 싶은 정보를 훨씬 빠르게 얻어낼 수 있다.

```
$ git reflog | grep {찾고 싶은 정보}
```

### 예시

```
git reflog | grep 'docs: 요구사항 정리'

dfbff524 HEAD@{12}: commit: docs: 요구사항 정리
633957b3 HEAD@{30}: commit: docs: 요구사항 정리
a624af40 HEAD@{47}: commit: docs: 요구사항 정리

git reflog | grep "docs: 요구사항 정리"

dfbff524 HEAD@{12}: commit: docs: 요구사항 정리
633957b3 HEAD@{30}: commit: docs: 요구사항 정리
a624af40 HEAD@{47}: commit: docs: 요구사항 정리
```

