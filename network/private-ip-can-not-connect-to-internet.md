![image](https://user-images.githubusercontent.com/37354145/125147103-4a08c500-e164-11eb-95f3-029b0d8d1c96.png)

AWS EC2 인스턴스 4개를 이용해 서비스 인프라를 구축하려 할 때, 
팀원 루트가 "WAS랑 DB한테 public IP를 할당할 필요가 있어?" 라는 이야기를 꺼냈다.
어차피 클라이언트와 통신을 주고 받아야하는 Web Server(Reverse Proxy)와 우리가 직접 접속해야 할 
Bastion은 public IP가 필수로 필요하지만, 그 외에 같은 IP 대역을 사용하고 있어서 
private IP로 통신이 가능한 WAS와 DB한테는 public IP를 할당할 필요가 없다는 말이었다.

좋은 의견인 것 같아 결국 WAS, DataBase 인스턴스의 public IP를 할당하지 않았고(private IP만 남긴 상황), 
Bastion을 통해 ssh 연결을 성공했다. 
그런데 몇 가지 설정을 진행하던 중에 `sudo apt update` 명령이 동작하지 않았다. 정확히 말하자면 외부 internet과 연결이 되지 않았다. 처음엔 permission 관련 문제인 줄 알았는데, 여러가지 시도를 해보아도 해결되지 않았다.

결국 CU한테 DM으로 질문을 드렸고, "탄력적 IP를 부여하면 해결이 될 것이다." 라는 답변을 받을 수 있었다. 
이 때 유추 가능했던 것이 'public IP가 없으면 internet을 통해 외부로 요청이 불가능 하다.' 였다.

결국 WAS와 DataBase 쪽 인스턴스에 탄력적 IP를 부여해서 문제를 해결 할 수 있었다.

<br>

## private IP는 internet network에 연결 할 수 없나?
IP 대역이 서로 다른 네트워크가 통신을 주고받으려면 둘 중 하나를 만족해야한다고 한다.

1. 서로 상대방의 IP 대역을 알고 있는 경우.  
   가는 길을 알려줘야 하기 때문에 라우터(라우터 테이블)를 쓴다.  
   (같은 네트워크 안에서 길을 알려주는 기능은 스위치라고 한다.)
2. 어느 한쪽이 반대편의 대역으로 변환해서 통신을 시도하는 경우  
   (상대방 IP로 변환하는 기능을 하는 것이 NAT 이다.)

인터넷 영역에서는 모두가 알고 있는 IP로 통신해야 하기 때문에 이 때 필요한게 Public IP 이다.

WAS, DataBase 인스턴스에게는 public IP가 존재하지 않았기 때문에 `sudo apt update` 명령을 통해 
apt 측에 "최신 버전 apt 파일들을 내려주세요!" 라는 요청을 전송했지만, apt 측에서 답변을 받을 대상이 
누군지 찾아내지 못해서 업데이트가 진행되지 못했던 것이다.

<br>

## Elastic IP
AWS Elastic(탄력적) IP는 본래 우리가 사용한 용도로 탄생한 것이 아니다.  

EC2 인스턴스들의 IP는 고정적이지 않다. IPv4 주소체계에서 IP가 워낙 부족한 상황이라, 새로운 EC2 인스턴스가 생성될 때마다 IP주소를 부여하기 어렵기 때문에 EC2 인스턴스가 동작하는 동안에만 IP를 부여하고 멈추면 IP를 반납시키는 형태의 유동 IP를 사용하고 있다.

이런 상황에서 다른 네트워크와 주기적인 연결이 필요한 EC2 인스턴스에게 편의를 제공하고자 고정적인 public IP를 제공해주는 것이 바로 Elastic IP다. AWS의 Elastic IP는 EC2 인스턴스에게 쉽게 부여할 수 있고, 쉽게 제거할 수도 있다.

우리가 WAS, DataBase 인스턴스에 Elastic IP를 부여해서 internet 연결에 성공한 이유도 Elastic IP를 통해 
IP 대역이 서로 다른 internet network들에게 public IP를 공개함으로서 요청과 답변을 주고 받을 수 있는 길을 텃기 
때문이다.

internet network에 연결이 필요한 작업을 모두 끝낸 후 Elastic IP를 다시 회수하면 
IP 대역이 다른 네트워크에서 접근할 방법이 없는, 오로지 같은 대역폭 내에서 private IP로만 접근할 수 있는 
고립된 인스턴스로 만들 수 있다.
