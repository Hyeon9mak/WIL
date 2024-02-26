GitHub 에 Pull Request 를 올리면 가끔 아래와 같은 금지 마크를 발견할 때가 있다.

![](https://i.imgur.com/vgxK7R2.png)

line number 를 보면 알 수 있듯, 파일 가장 끝에 개행이 포함되지 않았을 때 발생하는 오류 표기다. 실제로 프로그램을 실행시킬 땐 아무런 문제가 되지 않는데, 왜 GitHub 에서는 이런 오류 표기를 항상 해주는 걸까?

![](https://i.imgur.com/bDuYvnN.png)

결론부터 말하면 `파일의 마지막에는 개행이 포함되어야 한다.` 라는 [POSIX(IEEE 에서 책정한 UNIX 인터페이스)](https://ko.wikipedia.org/wiki/POSIX)을 만족 시키기 위함이다. 현재 프로그램이 동작하는 것에는 아무런 문제가 없으나, 잠재적으로 문제가 될 수 있기 때문에 GitHub 에서 미리 경고를 하는 것.

## 파일 끝에 개행을 추가하는 이유

[POSIX](https://ko.wikipedia.org/wiki/POSIX) 에서 정의하는  "끝나지 않은 행" 의 기준은 아래와 같다.

> [3.195 Incomplete Line](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap03.html#tag_03_195)  
> A sequence of one or more non-`<newline>` characters at the end of the file.  
> 
> 불완전한 행  
> 파일의 끝에 `<newline>` 이 아닌 문자가 포함되는 것.

그렇다면 온전히 마무리 짓고 하나의 행으로 인정 받는 기준은 무엇일까?

> [3.206 Line](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap03.html#tag_03_206)  
> A sequence of zero or more non-`<newline>` characters plus a terminating `<newline>` character.  
>
> 행  
> 아무것도 적혀있지 않거나, `<newline>` 이 아닌 문자 이후 마지막에 `<newline>` 문자가 포함되는 것.

즉 행의 가장 마지막에 개행문제(`<newline>`)가 하나 포함되어야 온전히 하나의 행으로 인정된다. 그렇지 않을 경우 여전히 행에 입력이 진행중인 것으로 판별하며, 자연스럽게 현재 지점이 EOF(End Of File)인지 판별할 수 없게 된다. (POSIX 에 근거하여 동작하는 컴파일러 등에서 EOF 를 인지하지 못해 비정상적인 동작 결과를 만들어낼 수 있다.)

GitHub 에서는 이러한 잠재적인 오류를 예방하고자 파일 끝에 개행이 없을 시 경고를 띄우는 것이다.

## IntelliJ 사용 환경에서 마지막 개행 자동 추가하기

`Editor` > `General` > `Ensure every saved file ends with a line break` 를 체크하면, 파일을 생성하거나 저장 단축키(`CMD` + `S`)를 누를 때마다 파일 마지막에 개행이 자동으로 적용된다.
![image](https://github.com/mission-study-to-finish-in-15-days/kotlin-racingcar/assets/37354145/31ea7e1c-648f-4441-bac0-2a8ce86e3573)

## References

- [https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap03.html](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap03.html)
- [POSIX - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/POSIX)
