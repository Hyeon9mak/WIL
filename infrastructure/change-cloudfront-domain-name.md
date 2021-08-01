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

![image](https://user-images.githubusercontent.com/37354145/127763110-ad369c72-47dd-4152-b34f-6d62e60773f2.png)
![image](https://user-images.githubusercontent.com/37354145/127763129-86b3bb24-2465-4719-9053-727b6bd9f9b3.png)

정적 웹 사이트 호스팅 활성화 후, 인덱스 문서도 지정해준다.
지정하지 않을 경우 기본 주소(`도메인주소.com`)로 접근했을 때 어떤 자료를 보여줄지 S3가 갈피를 잡지 못한다.
때문에 `index.html`와 같은 파일 지정이 강제된다.
인덱스 문서 지정을 통해 `도메인주소.com/index.html`과 같은 효과를 누리게 해준다.

<br>

## 도메인 발급 - 내도메인.한국
내도메인.한국 사이트에서 마음에 드는 도메인을 발급 받는다.

![image](https://user-images.githubusercontent.com/37354145/127763606-4ac7c799-8d29-4484-a55e-219201be7083.png)

발급 받은 후 내도메인 목록을 통해 도메인 수정 메뉴까지 접근한다.

![image](https://user-images.githubusercontent.com/37354145/127763661-f8317cac-f505-4c99-9e3b-282d3071e675.png)

도메인 수정 메뉴에 접근하면 여러가지 옵션이 보이는데, 그 중에서 `별칭(CNAME)` 옵션을 사용할 것이다.

<br>

## 도메인 별칭 등록 및 인증 - Cloudfront, ACM
S3와 연결되어 있는 Cloudfront의 Settings-Edit 메뉴로 접근한다.

![image](https://user-images.githubusercontent.com/37354145/127763732-91a5ad3d-ef1b-43d1-89c0-0b9eedf1167c.png)

메뉴에 접근하면 3가지 설정을 진행해야 한다.

1. 도메인 별칭 등록
2. 최상위 URL(/)에 보여질 오브젝트 선택
3. 도메인 SSL 인증

![image](https://user-images.githubusercontent.com/37354145/127763751-37f6937d-ffa9-497a-b33e-aca57202ca56.png)

하나씩 진행해보자.

### 1. 도메인 별칭 등록
Alternate domain name (CNAME) 옵션에 ADD item 버튼을 눌러서 내도메인.한국에서 발급 받은 도메인을 입력하자.

![image](https://user-images.githubusercontent.com/37354145/127763822-ab91e50b-e791-4b76-a550-275f75536e27.png)

### 2. 최상위 URL(/)에 보여질 오브젝트 선택
최상위 URL로 접근했을 때 어떤 오브젝트(페이지)가 보여지길 원하는지 설정한다. 우리는 정적 웹 사이트를 운영할 것이므로 `index.html`을 입력하면 된다.

![image](https://user-images.githubusercontent.com/37354145/127763834-f2c4313f-6485-4f60-b814-7e99c469b733.png)

### 3. 도메인 SSL 인증
Request certificate 버튼을 눌러 ACM(Amazon Certificate Manager) 페이지로 접근하자.

![image](https://user-images.githubusercontent.com/37354145/127763882-9cd4f129-8e32-4c05-8ef5-40c22cc248fa.png)

내도메인.한국에서 발급 받은 도메인 이름을 입력한다. 추후 추가적인 도메인 이름 (ex: dev.babble.o-r.kr)을 이용할 계획이라면 
`이 인증서에 다른 이름 추가` 버튼을 누르고 추가 도메인을 입력해주자. 아래 예시에서는 와일드카드(`*`)를 이용해 등록했다.

![image](https://user-images.githubusercontent.com/37354145/127763895-2c822258-8d0b-4dcb-ba86-cce17a1c5e43.png)

그 후 인증 방식을 선택해야 한다.

![image](https://user-images.githubusercontent.com/37354145/127763942-732a9d2e-e44a-42b8-a482-67191b5ba8f6.png)

이메일 검증 방식의 경우 DNS(Domain Name Service)를 이용중이라면 사실상 이용이 불가능하다. 직접 해당 도메인의 소유자로서 
도메인의 이메일 주소에 개인 이메일이 등록되어 있어야 하는데, 가비아, 내도메인.한국 등 DNS 업체를 이용한 경우 이메일 검증은 어려울 것이다.
자연스럽게 남은 DNS 검증 방법을 선택하고 넘어간다.

![image](https://user-images.githubusercontent.com/37354145/127763996-9072f1b0-761a-44f2-b071-2d1573e1e4d1.png)

태그는 굳이 지정해줄 필요 없지만, 혹시 모를 식별을 위해서 해주었다.

![image](https://user-images.githubusercontent.com/37354145/127764007-fb385f9d-be84-4ece-905c-988e34e4f2af.png)
![image](https://user-images.githubusercontent.com/37354145/127764036-3ea2b9c4-b412-41f4-bf2e-be5a1d324ba1.png)

모든 설정을 마치고 나면 ACM에서 검증을 위한 [이름]과 [값]을 제공해준다. 이 [이름]과 [값]을 DNS 업체 사이트에 입력하면 된다.

![image](https://user-images.githubusercontent.com/37354145/127764081-7a78de41-960e-426e-8d2d-1c022116bdd0.png)

내도메인.한국 사이트를 기준으로, 별칭(CNAME) 옵션을 활성화 하고 [이름]과 [값]을 입력한다.
[이름]은 ACM에서 제공한 [이름]에서 **도메인을 제외한 앞부분의 인증해시**까지만 입력하면 되고,
[값]은 ACM에서 제공한 모든 [값]을 입력하면 된다. 

> (단, 내도메인.한국에서는 [값]의 가장 마지막에 붙어있는 `.`을 함께 입력시 유효하지 않은 양식이라고 거절한다. 가장 마지막의 `.`을 제거한 후 입력을 진행하면 정상적으로 잘 등록 될 것이다.)

![image](https://user-images.githubusercontent.com/37354145/127764181-3a4fb002-020a-4f1d-a928-49c18684c4c5.png)

잠시 1~5분 가량 기다리면 인증이 완료될 것이다. 인증이 완료된 것을 확인하면 다시 내도메인.한국 설정 페이지로 돌아가 설정을 바꿔준다.

![image](https://user-images.githubusercontent.com/37354145/127764232-fd33823e-7a7a-4505-b460-2e37d9c8e89e.png)

별칭(CNAME) 설정의 [이름] 부분은 지우고, [값] 부분에 Cloudfront의 기본 도메인을 입력한다.

> (다른 DNS 업체에서는 별칭(CNAME) 인증서 발급 영역과 발급 후 도메인 등록 영역을 분리해두었으나, 내도메인.한국에서는 이를 분리해놓지 않아 
위와 같이 다소 헷갈리는 과정을 거쳐야만 한다.)

![image](https://user-images.githubusercontent.com/37354145/127764372-6065b8a6-de86-40a6-b0c5-7c54d577fa8b.png)

이후 다시 Cloudfront 설정 화면으로 돌아와 certificate 새로고침 버튼 클릭 후 목록을 찾아보면 SSL 인증서를 찾을 수 있을 것이다.
해당 인증서를 선택 후 설정을 저장하면 새로운 도메인이 Cloudfront에 잘 매겨진 것을 확인 할 수 있다.

![image](https://user-images.githubusercontent.com/37354145/127765190-ff977dab-2de0-4c00-bf63-38c31946f3cb.png)

<br>

## 반영 확인

실제 Cloudfront 에 도메인이 반영되는데 5~10분 정도의 시간이 소요될 수 있다. 반영이 완료되면 정상적으로 접근이 되는지 확인하면 되겠다.

![image](https://user-images.githubusercontent.com/37354145/127765172-f3ec81f8-e8a8-43e4-b611-2bc2cf870a6d.png)
