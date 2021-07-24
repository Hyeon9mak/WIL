# S3에 프론트 엔드 배포하기
S3 public access를 모두 막아두었을 경우 상황에 맞게 S3 access 방법을 모색해야 한다.

1. S3에서 모든 public access 허용하기 (비추천)
2. IAM role 설정을 통해 EC2-S3 간 연결 허용하기
    - 이후 EC2에서 awscli를 통해 업로드
3. Github actions를 이용하는 경우 IAM role 새로 생성
    - S3 Full access 권한이 있는 role 생성 후 발급 받은 키를 Github secrets에 등록

2, 3번은 Github secrets에 키를 등록한다는 것 외에 아래 절차를 비슷하게 거친다.

<br>

## amazon cli 설치
```
$ sudo apt install awscli
```

<br>

## curl 설치
```
$ sudo apt install curl
```

<br>

## node 특정 버전 강제 다운로드 및 설치
### definition
```
$ sudo curl -sL https://deb.nodesource.com/setup_[버전].x | sudo bash -
$ sudo apt install -y nodejs
```

### example
```
$ sudo curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -
$ sudo apt install -y nodejs
```

<br>

## 프론트 빌드에 필요한 패키지(의존성) 설치
```
$ npm install
```

<br>

## 빌드
### definition
```
$ npm run [빌드설정명]
```

### example
```
$ npm run prod
```

<br>

## S3로 배포
### definition
```
$ aws s3 sync [배포된 파일 디렉토리명] [S3 버킷 주소]
```

- `sync`: 업데이트가 감지된 파일만 복사 후 덮어쓰기
- `cp`: 모든 파일 복사 후 덮어쓰기

### example
```
$ aws s3 sync ./dist s3://bucket-babble-front/
```

<br>

## References
- [GitHub Action으로 AWS S3에 배포 자동화 - velog](https://velog.io/@1nthek/GitHub-Action%EC%9C%BC%EB%A1%9C-AWS-S3%EC%97%90-%EB%B0%B0%ED%8F%AC-%EC%9E%90%EB%8F%99%ED%99%94)