# Cloudfront 도메인 이름 변경하기 (신규등록)

## summary
DNS를 활용하는데 2가지 방법이 있다.
1. Route 53로 DNS를 발급 받아 사용한다. (TLS 인증서 등록 과정이 생략된다.)
2. 외부 DNS를 발급 받은 다음, SSL(TLS) 인증서를 ACM(Amazon Certificate Manager)를 통해 등록해서 사용한다.

1번 방법은 사실 AWS의 울타리 안에서 모든게 이루어지는 방법이므로 크게 신경쓸 것이 없다.
2번 방법은 DNS 사에 따라서 조금씩 방법이 달라진다.
나는 내도메인.한국 DNS를 기준으로 작성할 것이다.

<br>

## 선행 작업 - S3
Cloudfront와 연결되어 있는 S3 버켓 쪽에 정적 웹 사이트 호스팅을 진행 할 것임을 설정해주어야 한다.

![image](https://user-images.githubusercontent.com/37354145/128027830-2dc53071-49c6-4b00-ace2-3d3664a31589.png)
![image](https://user-images.githubusercontent.com/37354145/128027905-e37efe5b-a83f-46cc-88e1-2f984bc82116.png)

정적 웹 사이트 호스팅 활성화 후, 인덱스 문서도 지정해준다.
지정하지 않을 경우 기본 주소(`도메인주소.com`)로 접근했을 때 어떤 자료를 보여줄지 S3가 갈피를 잡지 못한다.
때문에 `index.html`와 같은 파일 지정이 강제된다.
인덱스 문서 지정을 통해 `도메인주소.com/index.html`과 같은 효과를 누리게 해준다.

<br>

## 도메인 발급 - 내도메인.한국
내도메인.한국 사이트에서 마음에 드는 도메인을 발급 받는다.

![image](https://user-images.githubusercontent.com/37354145/128028052-9062712d-82cb-49eb-8e33-490b9374318a.png)

발급 받은 후 내도메인 목록을 통해 도메인 수정 메뉴까지 접근한다.

![image](https://user-images.githubusercontent.com/37354145/128029470-89efc110-0f25-4f88-969a-fe678fdb8dcc.png)

도메인 수정 메뉴에 접근하면 여러가지 옵션이 보이는데, 그 중에서 `별칭(CNAME)` 옵션을 사용할 것이다.

<br>

## 도메인 별칭 등록 및 인증 - Cloudfront, ACM
S3와 연결되어 있는 Cloudfront의 Settings-Edit 메뉴로 접근한다.

![image](https://user-images.githubusercontent.com/37354145/128028052-9062712d-82cb-49eb-8e33-490b9374318a.png)

메뉴에 접근하면 3가지 설정을 진행해야 한다.

1. 도메인 별칭 등록
2. 최상위 URL(/)에 보여질 오브젝트 선택
3. 도메인 SSL 인증

![image](https://user-images.githubusercontent.com/37354145/128028189-bf8b48a8-c774-4227-9c17-061fb1032d05.png)

하나씩 진행해보자.

### 1. 도메인 별칭 등록
Alternate domain name (CNAME) 옵션에 ADD item 버튼을 눌러서 내도메인.한국에서 발급 받은 도메인을 입력하자.

![image](https://user-images.githubusercontent.com/37354145/128028387-d74a35bf-3c5b-406c-8c67-60b8c8f97b1f.png)

### 2. 최상위 URL(/)에 보여질 오브젝트 선택
최상위 URL로 접근했을 때 어떤 오브젝트(페이지)가 보여지길 원하는지 설정한다. 우리는 정적 웹 사이트를 운영할 것이므로 `index.html`을 입력하면 된다.

![image](https://user-images.githubusercontent.com/37354145/128028424-13009f09-9a1f-4e49-9d58-50dc92d7dba3.png)

### 3. 도메인 SSL 인증
Request certificate 버튼을 눌러 ACM(Amazon Certificate Manager) 페이지로 접근하자.

![image](https://user-images.githubusercontent.com/37354145/128028461-ad710fad-5c55-4331-91a6-5e9f4c1c0f53.png)

내도메인.한국에서 발급 받은 도메인 이름을 입력한다. 추후 추가적인 도메인 이름 (ex: dev.babble.o-r.kr)을 이용할 계획이라면 
`이 인증서에 다른 이름 추가` 버튼을 누르고 추가 도메인을 입력해주자. 아래 예시에서는 와일드카드(`*`)를 이용해 등록했다.

![image](https://user-images.githubusercontent.com/37354145/128028643-99c3512f-87e2-44ae-97b1-b12883ef56f9.png)

그 후 인증 방식을 선택해야 한다.

![image](https://user-images.githubusercontent.com/37354145/128028678-8d2e13df-f685-431d-9dd8-95c0b8a341cb.png)

이메일 검증 방식의 경우 DNS(Domain Name Service)를 이용중이라면 사실상 이용이 불가능하다. 직접 해당 도메인의 소유자로서 
도메인의 이메일 주소에 개인 이메일이 등록되어 있어야 하는데, 가비아, 내도메인.한국 등 DNS 업체를 이용한 경우 이메일 검증은 어려울 것이다.
자연스럽게 남은 DNS 검증 방법을 선택하고 넘어간다.

![image](https://user-images.githubusercontent.com/37354145/128028702-5bf37067-44d5-4cae-9271-6d8da7e41c84.png)

태그는 굳이 지정해줄 필요 없지만, 혹시 모를 식별을 위해서 해주었다.

![image](https://user-images.githubusercontent.com/37354145/128028832-18e82ac6-1198-4625-81f7-f7f582f51e4e.png)
![image](https://user-images.githubusercontent.com/37354145/128028865-fca8f7be-65db-4bf9-8ce9-60044146537c.png)

모든 설정을 마치고 나면 ACM에서 검증을 위한 [이름]과 [값]을 제공해준다. 이 [이름]과 [값]을 DNS 업체 사이트에 입력하면 된다.

![image](https://user-images.githubusercontent.com/37354145/128029010-455362b3-c30c-4b33-a6e8-ce634118c62b.png)

내도메인.한국 사이트를 기준으로, 별칭(CNAME) 옵션을 활성화 하고 [이름]과 [값]을 입력한다.
[이름]은 ACM에서 제공한 [이름]에서 **도메인을 제외한 앞부분의 인증해시**까지만 입력하면 되고,
[값]은 ACM에서 제공한 모든 [값]을 입력하면 된다. 

> (단, 내도메인.한국에서는 [값]의 가장 마지막에 붙어있는 `.`을 함께 입력시 유효하지 않은 양식이라고 거절한다. 가장 마지막의 `.`을 제거한 후 입력을 진행하면 정상적으로 잘 등록 될 것이다.)

![image](https://user-images.githubusercontent.com/37354145/128029028-28df28e8-36e2-41d2-a6bb-3e3f90e774d5.png)

잠시 1~5분 가량 기다리면 인증이 완료될 것이다. 인증이 완료된 것을 확인하면 다시 내도메인.한국 설정 페이지로 돌아가 설정을 바꿔준다.

![image](https://user-images.githubusercontent.com/37354145/128029055-d2b4f52d-1c94-4910-8ccc-98f7c02d0439.png)

별칭(CNAME) 설정의 [이름] 부분은 지우고, [값] 부분에 Cloudfront의 기본 도메인을 입력한다.

> (다른 DNS 업체에서는 별칭(CNAME) 인증서 발급 영역과 발급 후 도메인 등록 영역을 분리해두었으나, 내도메인.한국에서는 이를 분리해놓지 않아 
위와 같이 다소 헷갈리는 과정을 거쳐야만 한다.)

![image](https://user-images.githubusercontent.com/37354145/128029083-79debb8f-dbbe-4705-a8e5-0eaa7a311850.png)

이후 다시 Cloudfront 설정 화면으로 돌아와 certificate 새로고침 버튼 클릭 후 목록을 찾아보면 SSL 인증서를 찾을 수 있을 것이다.
해당 인증서를 선택 후 설정을 저장하면 새로운 도메인이 Cloudfront에 잘 매겨진 것을 확인 할 수 있다.

![image](https://user-images.githubusercontent.com/37354145/128029116-c9e5176c-f45c-42bb-9f5e-6d3b7ebd3873.png)

<br>

## 반영 확인

실제 Cloudfront 에 도메인이 반영되는데 5~10분 정도의 시간이 소요될 수 있다. 반영이 완료되면 정상적으로 접근이 되는지 확인하면 되겠다.

![image](https://user-images.githubusercontent.com/37354145/128029138-b427e908-2603-4360-b85b-40e775e097d9.png)
