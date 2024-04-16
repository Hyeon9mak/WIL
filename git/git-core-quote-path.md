`git status` 명령을 이용하는데 아래와 같은 한글 깨짐 현상이 발생했다.

```bash
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	deleted:    atdd/atdd-week-3.md
	modified:   "spring/web-mvc-test \353\212\224 dispatcher servlet \354\235\230 \354\227\255\355\225\240\354\235\204 \353\214\200\354\213\240\355\225\234\353\213\244..md"

...(생략)...
```

원인을 찾아보니 `core.quotePath` 옵션 때문이었다.

> **core.quotePath**  
> Commands that output paths (e.g. _ls-files_, _diff_), will quote "unusual" characters in the pathname by enclosing the pathname in double-quotes and escaping those characters with backslashes in the same way C escapes control characters (e.g. `\t` for TAB, `\n` for LF, `\\` for backslash) or bytes with values larger than 0x80 (e.g. octal `\302\265` for "micro" in UTF-8). If this variable is set to false, bytes higher than 0x80 are not considered "unusual" any more. Double-quotes, backslash and control characters are always escaped regardless of the setting of this variable. A simple space character is not considered "unusual". Many commands can output pathnames completely verbatim using the `-z` option. The default value is true.
> 
> [https://git-scm.com/docs/git-config#Documentation/git-config.txt-corequotePath](https://git-scm.com/docs/git-config#Documentation/git-config.txt-corequotePath)

> **core.quotePath**  
> 이 명령은 경로를 출력하는 경우 (예: `ls-files`, `diff`), 경로 이름에 "비정상적인" 문자가 포함되어 있으면 해당 경로 이름을 이중 인용부호로 둘러싸고, C 언어에서 제어 문자를 이스케이프 하는 방식과 동일하게 백슬래시로 이스케이프합니다 (예: 탭의 경우 `\t`, 개행의 경우 `\n`, 백슬래시의 경우 `\\`) 또는 UTF-8에서 값이 0x80보다 큰 바이트 (예: "마이크로"의 경우 8진수 `\302\265`). 이 변수가 false로 설정된 경우, 0x80 이상의 바이트는 더 이상 "비정상적"으로 간주되지 않습니다. 이중 인용부호, 백슬래시 및 제어 문자는 항상 이스케이프됩니다. 단순한 공백 문자는 "비정상적"으로 간주되지 않습니다. 많은 명령어는 `-z` 옵션을 사용하여 경로 이름을 완전히 그대로 출력할 수 있습니다. 기본값은 `true` 입니다.

아래와 같은 명령을 입력하면 설정 off 가 가능하다.

```bash
$ git config --global core.quotepath false
$ git config --local core.quotepath false
```

