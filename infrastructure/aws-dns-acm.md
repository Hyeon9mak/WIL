## DNS

- dns 는 계층적인 구조
    - 최상위 구조는 국가, 전세계
        - gTLD, NEW gTLD, ccTLD
        - gTLD
            - .com, .net, .org…
        - NEW gTLD
            - .email, .coffe, ….
        - ccTLD
            - kr, jp 국가 TLD. 아무나 못만듬.
    - 두번째 구조는 최상위 구조 권한이 있으면 자연스럽게 만들 수 있음
    - 하위 도메인 역시도 같은 방식으로 쭉쭉쭉…
- 유니케스트?
	- IP 는 하난데, 여러 개로 분산해서 쓰고 있다는 개념
- dns 종류
    - A: IPv4
    - AAAA: IPv6
    - CNAME: redirect
    - MX: mail
    - NS: name server 하나 더!
- DNS 는 최대한 빠르게 응답해야하므로 평소에 UDP 를 사용한다.
    - 512byte 를 넘어가는 경우에만 TCP 를 사용한다.
    - DNS port 는 53
- local pc 에 dns cache 가 있다.
- `dig` 명령어로 확인해보자. `lookup` 도 자주 해보자.
- private DNS 를 이용하는 이유는 보안을 위해서
    - public IP 대역이 공개되면 해커들이 매우 좋아함
- DNS 는 단순히 이름 뿐만 아니라 라우팅도 해준다.
    - 지역, 장애조치, 지연시간, IP 기반, 다중응답, 가중치 등등..

## ACM(AWS Certificate Manager)

> NGINX let’s encrypt 에서 하던 그거당 ㅎ

## QnA
- let’s encrypt 같은건 무료 인증서인데 인증서를 돈주고 사는 것에는 어떤 메리트가 있는지?
	- 알고리즘이 더 복잡하고 그것 자체를 비용으로 볼 수 있음.
- AWS 에서 이걸 무료로 제공해주는 이유는 무엇인지?
	- 자체적으로 다루기 때문에 관리비용이 거의 없는듯
- 가변 IP 에 대해서도 DNS 에서 관리를 해주는지? 관리가 어렵다는게 이런 건지?
	- ㄴㄴ
- 가변 IP 를 사용하는 이유는 비용 때문에 남는 IP 재활용을 위해서 아닌가?
- IP 하나당 금액일텐데 통신사에서 고정 IP 잡는 걸 지원해주는 이유?
    - 안된다. 알아서 바뀔거임
	- IP 비쌈. 통신사중에는 KT 가 제일 비싸고 LG 가 제일 쌈