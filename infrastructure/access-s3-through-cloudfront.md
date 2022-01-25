# CloudFront를 통해 S3 액세스 하기

Amazon S3(Simple Storage Service)는 간편한 데이터 관리 및 액세스 제어를 제공해주는 저장소다. 
'무한대로 늘어나는 외부 저장소 + 간편한 액세스 제어' 정도로 생각하면 편할 것 같다.

Amazon CloudFront `.html`, `.css`, `.js` 및 이미지 파일과 같은 정적 및 동적 웹 콘텐츠를 사용자에게 더 빨리 배포하도록 지원하는 웹 서비스다. 전 세계 곳곳에 존재하는 Edge location에 웹 콘텐츠를 캐싱해두고 제공하는 것으로 
빠른 성능을 보장한다고 한다. 
사용자의 요청이 들어왔을 때 지연 시간이 낮은 (가까운) Edge location에 해당 콘텐츠가 존재할 경우 곧장 되돌려 주고, 
없다면 그 때 지정된 Origin 저장소(S3, HTTP 웹서버 등)에서 콘텐츠를 찾아내어 캐싱 후 되돌려 준다.
2가지 상황 모두에서 사용자는 CloudFront가 제공하는 도메인으로 요청을 보내는 것 뿐이므로, 어떤 상호작용을 거쳐서 
본인이 원하는 콘텐츠를 제공받는지 알 수 없다. (알 필요도 없다.)

프로젝트를 진행하면서 아래와 같은 제한사항이 안내 되었다.

> S3 사용과 관련해서 몇가지 공유드릴 사항이 있어요.
> 1. accesskey, secretkey 발급은 해드릴 수 없어요. ec2-s3-deploy 역할을 활용해주세요.  
> ec2 페이지에서 우측 상단에 `작업 > 보안 > IAM 역할 수정` 으로 진행하면 됩니다.
>	
> 2. 퍼블릭 오픈도 불가합니다. 정적 리소스 읽기 작업은 Cloudfront를 통해 접근하도록 구성하시고, 쓰기 작업은
> ec2-s3-deploy 역할이 부여된 인스턴스에서 가능합니다. (참고로 이 역할에 cloudwatch 권한도 부여해두었어요)

개발자, 서비스 제공자가 S3 bucket 쪽에 콘텐츠를 Create/Update/Delete 할 때는 S3 bucket에 연결된 EC2(아마도 Web-Server 역할을 수행하는)를 통해서, 클라이언트가 S3 bucket 쪽 컨텐츠를 Read 할 때는 CloudFront를 통해서 액세스 하게 끔 구성하라는 것으로 해석했다.

<br>

## CloudFront 설정
`CloudFront - Origins` 탭에서 Origin으로 만들어둔 S3 bucket을 연결한다. 

![무제 001](https://user-images.githubusercontent.com/37354145/125394332-821e3b00-e3e4-11eb-9082-1499d5c1d556.png)

그러나 연결만으론 콘텐츠 접근이 불가능하다.

```xml
<Error>
	<Code>AccessDenied</Code>
	<Message>Access Denied</Message>
	<RequestId>대충리퀘스트아이디다아아</RequestId>
	<HostId>
		대충호스트아이디다아아
	</HostId>
</Error>
```

CloudFront를 통하건 말건 S3 bucket은 public access로 판단하고 모두 거절해버리기 때문이다. (S3 bucket에서 public access를 모두 차단하면 AWS 계정 사용자만 웹UI를 통해 접근할 수 있다.)

CloudFront를 통해 S3 bucket의 콘텐츠에 접근하기 위해선 'CloudFront로 접근할 땐 S3 bucket의 public access 설정을 무시하게 해줘'라는 추가 설정을 진행해야 한다.

`CloudFront - Security - Origin access identities` 메뉴에 접근 후, `Create origin access identity` 버튼을 클릭해서 OAI를 생성한다. 

![무제 002](https://user-images.githubusercontent.com/37354145/125394344-85192b80-e3e4-11eb-946e-593b56ddc55e.png)

그 후 `CloudFront - Distributions` 메뉴로 돌아와 자신의 CloudFront를 선택한 다음 `Origins` 탭을 클릭 후 연결했던 S3 bucket을 선택하고 `Edit` 버튼을 클릭한다.

![무제 003](https://user-images.githubusercontent.com/37354145/125394350-864a5880-e3e4-11eb-99ae-2ce38ce1af32.png)

`S3 bucket access` 설정이 2가지 보인다.

- Don't use OAI (bucket must allow public access)
- Yes use OAI (bucket can restrict access to only CloudFront)

![무제 004](https://user-images.githubusercontent.com/37354145/125394353-86e2ef00-e3e4-11eb-8dea-978acb90c490.png)

Don't use OAI 설정은 S3 bucket의 public access 설정을 따르는 것이고, 
Yes use OAI 설정은 S3 bucket의 public access 설정을 CloudFront OAI를 통해 무시하고 접근하게 해준다.

Yes use OAI 설정을 선택하고, Origin access identity를 방금 전 생성한 OAI로 선택해주면 CloudFront 쪽에서의 설정이 끝난다.

> ![image](https://user-images.githubusercontent.com/37354145/151076115-ab36a712-9dd4-4a1b-8993-6ad97b08a2a6.png)
> 
> **html** 파일을 통해서 테스트를 진행하는 경우 기본값 루트 객체 설정을 확인하자.
> 
> - 기본값 루트 객체 설정을 해주지 않을 경우: `https://{example.url}/index.html`
> - 기본값 루트 객체 설정을 index.html로 설정할 경우: `https://{example.url}`

그러나 OAI를 만들 때 별다른 식별 값 없이 이름만으로 OAI가 생성된다. 
'고작 이정도로 S3 bucket의 액세스 권한을 제어해도 되는걸까?' 라는 생각이 드는데, 다행히 S3 bucket 쪽에도 
별도의 설정이 필요하다.

<br>

## S3 설정
자신의 S3 bucket의 `권한` 탭으로 접근한 후, `버킷 정책`에 아래와 같은 JSON을 작성한다.

![image](https://user-images.githubusercontent.com/37354145/125394832-433cb500-e3e5-11eb-8cbb-8d0d33300f2b.png)

```JSON
{
    "Version": "2008-10-17",
    "Id": "PolicyForCloudFrontPrivateContent",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity [CloudFront에서 만들었던 OAI ID]"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::[현재 접속한 s3 bucket명]/*"
        }
    ]
}
```

`[CloudFront에서 만들었떤 OAI ID]`에는 OAI의 ID 값을 적어주면 되고, 
`[현재 접속한 s3 bucket명]`에는 말 그대로 현재 s3 bucket의 이름을 적어주면 된다.

CloudFront와 S3 설정이 모두 완료되었을 경우 정상적으로 콘텐츠 접근이 가능할 것이다.

<br>

## (추가설정) CloudFront - Origin Shield 설정

![무제 005](https://user-images.githubusercontent.com/37354145/125394354-86e2ef00-e3e4-11eb-8ece-41c5cdca6e77.png)

CloudFront 쪽에 Origin Shield 설정은 Origin으로 등록한 스토리지(S3 등)에 전달되는 리퀘스트의 횟수를 줄여 관리비용을 줄인다고 한다. (캐싱 적중률을 높힌다는 이야기)

CloudFront의 `Origins` 탭 - 연결했던 S3 bucket `Edit` 으로 접근해서 
`Enable Origin Shield` 설정을 확인해보자.

Yes 옵션을 선택 후, 자기 서비스 오리진의 지역에 가까운 지역을 선택해서 골라주면 설정이 끝난다. 
실제 캐싱 적중률 상승에 관련해서는 며칠 간의 관찰이 필요할 것으로 보인다.

<br>

## References
- [Amazon S3이란 무엇인가요? - Amazon Simple Storage Service](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/Welcome.html)
- [Amazon CloudFront란 무엇입니까? - Amazon CloudFront](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
- [Amazon CloudFront Origin Shield 사용 - Amazon CloudFront](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/origin-shield.html)
- [Origin Shield - 리눅서의 기술술 블로그](https://linuxer.name/tag/origin-shield/)
