# Invalidations를 이용한 CloudFront 캐싱 컨트롤

## summary
3차 데모데이에 맞춰 프론트엔드 쪽에 새로운 요구사항이 등장했다.

> • AWS로 이전  
> • 시맨틱 버저닝 추가  
> **• 사용자가 배포된 기준으로 항상 최신 버전을 봐야됨**

나는 **사용자가 배포된 기준으로 항상 최신 버전을 봐야됨** 요구사항을 CloudFront 가 갖고 있는
캐싱 정책을 변경해서, 항시 원본 저장소(S3 버킷)를 참조하라는 것으로 해석했다.
때문에 `CloudFront - Edit behavior - Cache key and origin requests - Cache policy` 설정에서 
기본 설정인 `CachingOptimized`를 `CachingDisabled`로 바꿔 CloudFront가 캐싱을 진행하지 않도록 하고자 했다.

그런데 [루트](https://github.com/junroot)가 `CloudFront - Invalidations` 설정을 이용해야 하지 않겠냐는 의견을 제시했다. 

> Invalidations 설정은 캐시가 아직 만료되지 않고 남아 있어도 강제로 캐시를 삭제하는 기능이다. 
> 삭제 후 요청이 들어오면 캐시를 진행하는 엣지 로케이션이 오리진(S3 버킷)에서 새 파일을 가져오므로 캐시가 갱신되는 효과를 가진다. 

> Cloudfront로 배포되는 파일은 기본 설정을 변경하지 않았다면 [24시간동안 캐시가 유지](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/Expiration.html)된다.


곰곰히 생각해보니 루트의 말이 맞는거 같았다. CloudFront는 주로 S3 버킷에 대한 조회권한 및 도메인 설정, 캐싱을 위해서 사용된다. 그런데 캐싱 정책을 `CachingDisabled`로 바꿔 캐싱을 막는다는 건, CloudFront의 주요 기능 중 하나를 사용하지 않는 것과 다름 없었다.

<br>

## CloudFront Invalidations - Web console
![image](https://user-images.githubusercontent.com/37354145/128022287-deb8b9a6-e6cd-49e3-a7ac-37e77211b67b.png)

`CloudFront - Invalidations` 메뉴로 이동 후 `Create Invalidation` 버튼을 누른다.

![image](https://user-images.githubusercontent.com/37354145/128022660-45953592-9823-4a49-93b8-e5722de0490f.png)

빈 칸에 **캐시를 지우고자 하는 파일의 절대경로(`/example.txt`)** 를 입력한다. 하단의 `Add item` 버튼을 통해 한 번에 여러 파일의 캐시를 지울 수도 있다.
캐시를 지우고 싶은 파일명을 모두 적었다면 `Create invalidation` 버튼을 클릭해주면 끝난다.

![image](https://user-images.githubusercontent.com/37354145/128023639-104e1154-f998-45d9-bf19-56b307dc50b7.png)

Invalidation 요청이 만들어지면 Status가 In progress 상태로 보인다. 곳곳의 엣지 로케이션에 퍼져있는 캐시 파일을 제거하는 것이므로, 다소 시간이 소요될 수 있다. (약 15~30분)
Invalidation 요청이 모든 엣지 로케이션에 적용이 완료되면 Status가 Completed로 바뀐다.

![image](https://user-images.githubusercontent.com/37354145/128026314-2b2a252f-2fb8-43db-aebd-9b26a0e99765.png)

<br>

그러나 웹 콘솔을 이용하는 방법은 굉장히 귀찮다! 프론트엔드 배포가 끝나면 매번 CloudFront 콘솔에 로그인하고, 배포용 CloudFront distribution을 찾고, Invalidation 요청하는 단계를 거쳐야한다. 

'CI/CD로 구성해둔 배포 자동화 마지막 단계에 Invalidation 요청도 포함시킬 수 없을까?' 고민을 했는데, 다행히 aws cli를 이용한 CloudFront Invalidation 요청 방법이 있었다!

<br>

## CloudFront Invalidations - aws cli
우선 aws cli를 통해 Invalidation을 요청하는 방법은 아래와 같다.

```
$ aws cloudfront create-invalidation --distribution-id [CloudFront distribution 아이디] --paths "[파일 절대경로명1]" "[파일 절대경로명2]"
```

> 이 기능을 처음 이용할 당시 권한 문제(`not authorized to perform:cloudfront:CreateInvalidation`)를 마주쳤다. 
> CU께 상황설명을 설명 드리고 Invalidation 요청 권한을 부여 받았다. 항상 감사합니다 CU! 🙇‍♂️

요청이 성공적으로 생성된 것을 웹 콘솔을 통해 확인했다면, 자동화를 위해 해당 명령어를 CI/CD 스크립트에 추가 해주면 끝난다. 우리 팀의 경우 Github actions를 이용중이기 때문에 프론트엔드 배포용 workflow 파일을 수정했다.

```yml
name: Node.js Package

on:
  push:
    branches: [release]
    
    ...(생략)...

      - name: 배포
        run: aws s3 sync ./dist s3://bucket-babble-front/ --delete

      - name: 캐시 무효화
        run: aws cloudfront create-invalidation --distribution-id ${{ secrets.DISTRIBUTION_ID }} --paths "/index.html" "/bundle.js"
```

이를 통해 평상시 프론트엔드 파일을 엣지 로케이션에 캐싱해두다가 프론트엔드 배포시 자동으로 엣지 로케이션에 캐싱되어 있는 파일들을 지우고 새로 배포된 파일을 다시 캐싱하는 CloudFront가 구성되었다. 캐싱의 이점과 사용자가 배포된 기준으로 항상 최신 버전을 봐야된다는 제한 사항을 모두 자동으로 챙길 수 있다!

<br>

## References
- [Invalidation으로 CloudFront 콘텐츠 갱신하기(Cache Control) - ACLOUD BLOG!](http://blog.a-cloud.co.kr/2020/01/23/invalidation%EC%9C%BC%EB%A1%9C-cloudfront-%EC%BD%98%ED%85%90%EC%B8%A0-%EA%B0%B1%EC%8B%A0%ED%95%98%EA%B8%B0cache-control/)
- [콘텐츠가 캐시에 유지되는 기간(만료) 관리 - Amazon CloudFront](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/Expiration.html)