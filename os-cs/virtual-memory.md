# Virtual Memory

물리적인 메모리의 주소 변환은 OS에서 관여하지 않는다. 그러나 가상 메모리 주소 변환은 전적으로 OS에서 관리한다.
(이 파트의 강의는 하드웨어 메모리가 Paging 기법을 사용하는 것으로 가정하고 진행된다. - 실제로 대부분의 시스템에서 Paging 기법을 사용하기 때문에)

가상 메모리를 통해 Page table 전체를 올리는 것이 아닌, 요청되는 부분에 대해서만 올리기 시작한다



## Demand Paging

- 실제로 필요할 때(요청이 있을 때) page를 메모리에 올리는 것
  - I/O 양의 감소 - 실제 프로그램의 대부분은 방어코드가 구성되어있고, 실 기능은 얼마 안된다
  - Memory 사용량 감소
  - 빠른 응답 시간(동시에 올라온 여러프로그램의 실 기능에 대한)
  - 더 많은 사용자 수용 - 더 많은 프로세스를 감당해낼 수 있다
- Valid / Invalid bit의 사용
  - Invalid의 의미
    - 사용되지 않는 주소 영역인 경우
    - 페이지가 물리적 메모리에 없는 경우
  - 처음에는 모든 page entry가 invalid로 초기화
  - address translation 시에 invalid bit이 set 되어 있으면
    - => "Page fault"

<img width="937" alt="image-20210519182814034" src="https://user-images.githubusercontent.com/37354145/120880614-8a75a000-c606-11eb-82ee-7abd5a2875cf.png">


물리 메모리에 올라와있는 것들은 Demand Paging에 의해서 추가된 것, HDD에 저장되어 있는 것들은 backing store에 저장되어 있는 것을 의미한다. 

G, H는 현재 프로세스에서 필요로 하지 않는 영역이지만, 주소 공간에서 총 8개의 Page를 제공해줘서 Page table의 인덱싱을 맞추기 위해 채워넣은 것이다. 그래서 invalid로 표시된다.

B, D, E는 프로세스에서 필요로 하는 영역이지만, 당장 사용되지 않기에 OS에 의해서 Swap Area(Backing store)에 내려가있다. 그래서 invalid로 표시된다.

A, C ,F 는 현재 프로세스에서 사용중이므로 물리 메모리에 올라와있고, Valid로 표시되고 있다.

이러한 특성 때문에 프로그램을 최초로 실행시키면 page table은 전체 invalid로 표시되어있다. 그러다가 호출되는 경우에 valid로 바뀌기 시작한다.

CPU가 logical memory를 통해 페이지에 접근하려했을 때, page table에 invalid로 표시되어 있었다면? Page fault. backing store에서 메모리로 올려야한다. (I/O). 이건 사용자 프로그램(프로세스)가 독단적으로 행할 수 없다. 그래서 운영체제가 CPU를 가지고 fault난 Page를 메모리로 올린다.



## Page Fault

- invalid page를 접근하면 MMU가 trap을 발생시킴 (page fault trap)
- Kernel mode로 들어가서 page fault handler가 invoke 된다
- 다음과 같은 순서로 page fault를 처리한다

<img width="951" alt="image-20210519184525390" src="https://user-images.githubusercontent.com/37354145/120880616-8b0e3680-c606-11eb-99a2-c856d43e611f.png">


1. Invalid reference? (bad address, protection violation) => abort process (trap)
2. CPU가 OS에게로 넘어옴. Get an empty page frame -> (비어있는 frame이 없으면 뺏어온다: replace)
3. 해당 page를 disk에서 memory로 읽어온다
   1. disk I/O가 끝나기까지 해당 프로세스는 CPU를 preempt 당함 (block)
      디스크는 굉장히 느린 장치기 때문에, page를 메모리로 끌어오는 것 자체가 굉장히 비싼 일이다.
      프로세스가 CPU를 가지고 있어봐야 오랜 시간 동안 할게 없기 때문에, CPU를 내놓는다.
   2. Disk read가 끝나면 page tables entry에 기록, valid/invalid bit = "valid"
   3. ready queue에 process를 insert -> dispatch later
4. 이 프로세스가 CPU를 잡고 다시 running
5. 아까 중단되었던 instruction을 재개

<img width="1028" alt="image-20210519184652089" src="https://user-images.githubusercontent.com/37354145/120880617-8ba6cd00-c606-11eb-9e89-8968bb253b30.png">

앞서 말했듯 Page를 디스크에서 메모리로 끌어오는 일은 굉장히 많은 시간을 소모하기 때문에, Page fault률이 얼마냐에 따라서 메모리 접근 시간이 크게 좌우된다.

실제 테스트를 해보면 거의 0.1에 가까운 수치가 나온다. 대부분의 경우 page fault가 나지 않고, 메모리에 있는 내용을 참조해서 해결해낸다.

P = Page fault 시간

(1-P) = Page Fault가 나지 않는 시간

(1-P) * memory access = Page fault가 나지 않는 경우 메모리에 접근하는 시간만 필요로 한다

p(OS & HW page fault overhaed + [swap page out if needed] + swap page in + OS & HW restart overhead)

= page fault가 나는 경우 대충.. 엄청난 시간이 걸린다.

<br>

## Page replacement

빈 frame이 없는 경우 쫒아내는 것

- Page replacement
  - 어떤 frame을 빼앗을지 결정해야함 (OS의 역할)
  - 곧바로 사용되지 않은 Page를 쫒아내는 것이 좋다
  - 동일한 페이지가 여러 번 메모리에서 쫒겨났다가 다시 들어올 수 있음
- Replacement Algorithm
  - page-fault rate를 최소화하는 것이 목표
  - 알고리즘의 평가
    - 주어진 Page reference string에 대해 page fault를 얼마나 내는지 조사
  - reference string의 예시
    - 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5


<img width="970" alt="image-20210519185233905" src="https://user-images.githubusercontent.com/37354145/120880618-8ba6cd00-c606-11eb-8ee8-d77df2bb26e9.png">

그런데 만약 replace 당하는 victim(희생양)이 메모리에 올라와있던 중 내용이 변경되었다면? write가 발생핬다면? (backing store에 적혀있는 내용과 달라졌다면?) -> backing store에도 변경된 내용을 write해주어야 한다.

변경된 내용이 없었다면? 그냥 물리 메모리에서 지워버리기만 하면 된다.

어쨌거나 쫒아내고 새로운 page를 올려두면 된다. Page table에는 기존 있다가 쫒겨난 녀석의 valid-invalid bit은 invalid로, backing store에 있다가 메모리로 올라온 녀석은 valid로 바꾸어주면 된다. 이 역할은 운영체제가 하게 된다.



## Optimal Algorithm - page fault를 가장 적게하는 알고리즘

<img width="1031" alt="image-20210519185627592" src="https://user-images.githubusercontent.com/37354145/120880619-8c3f6380-c606-11eb-96ed-5a29a0b9d207.png">

Optimal algorithm은 MIN(OPT) 가장 먼 미래에 참조되는 page를 쫒아낸다.

그림에서 5번 페이지가 최초로 참조되는 부분을 주목하자.

5번 페이지를 위해 기존 1,2,3,4 중에 하나를 replace 해야하는데, 미래에 어떤 page 가 가장 늦게 참조되는가?

5번 페이지가 최초 참조되는 이후 1,2,3,4,5 순서대로 참조가 일어난다. 즉 현재 frame에서 미래에 가장 늦게 참조되는 녀석은 4번이므로, 4번이 쫒겨나고 5번이 그 자리를 꿰차게 된다.

Optimal algorithm은 최고의 알고리즘이다. 그러나 '미래에 어떤 페이지가 참조될 것인지' 모두 인지하고 있어야한다는 점 때문에, 현실적으로는 적용이 불가능한 알고리즘이다. 그래서 `Offline Optimal Algorithm` 이라고 부른다.

그럼 이걸 왜 생각하냐? 다른 현실적인 알고리즘들의 성능에 대한 upper bound, 상한선, 최고로 좋은 page faults율을 비교하는데 사용된다. 아무리 좋은 알고리즘을 만들어도 이거보다 좋을 순 없다는걸 ㅇㅇ하게.



## FIFO(First In First Out) Algorithm

여기서부턴 미래를 모를 때 사용할 수 있는 알고리즘.


<img width="1068" alt="image-20210519190301740" src="https://user-images.githubusercontent.com/37354145/120880620-8c3f6380-c606-11eb-8459-d8278f002651.png">

미래를 모르니까 과거를 참조해서 사용하는 방식.

메모리에 가장 먼저 들어온 녀석을 가장 먼저 쫒아낸다.

FIFO 알고리즘이 가진 특이한 특징이 있는데, 보통 frame 수를 늘리면 page fault율이 떨어지기 마련인데, FIFO는 frame수가 늘어나면 되려 page fault율이 높아진다. 이런 기이한 현상을 우리가 "FIFO Anomaly(=Belady's Anomaly)"라고 부른다.



## LRU(Least Recently Used) Algorithm
가장 덜 최근에 사용된 (가장 오래전에 사용된 걸) 쫒아낸다. 메모리 관리 등에서 가장 널리 많이 사용된다.

<img width="1066" alt="image-20210519190517497" src="https://user-images.githubusercontent.com/37354145/120880621-8cd7fa00-c606-11eb-8977-42cc0543ad48.png">

FIFO보다 fault 회수가 줄어들었네!

## LFU (Least Frequently Used) Algorithm
<img width="509" alt="image-20210519190803704" src="https://user-images.githubusercontent.com/37354145/120880622-8cd7fa00-c606-11eb-9dda-44a8262a4f52.png">

참조 회수가 가장 적은 페이지를 지운다.

참조 회수가 동률은 페이지가 등장할 수 있다. 이런 경우 "꼭 이래라"하고 명시된 건 없지만, 대부분 성능 향상을 위해서 참조 시점이 가장 오래된 녀석을 replace 하도록 구현할 수 있다.

## LRU vs LFU
<img width="1037" alt="image-20210519190955429" src="https://user-images.githubusercontent.com/37354145/120880623-8d709080-c606-11eb-8933-f5e2922e6b3c.png">

LRU LFU 장단이 워낙 명확해서, 지난 시간 동안 2가지 장점을 취하는 여러가지 알고리즘이 논의된 바가 있다.


## LRU LFU 알고리즘 구현
### LRU
링크드리스트 형태로 페이지들의 참조 시간을 관리한다.

<img width="427" alt="image-20210519191332580" src="https://user-images.githubusercontent.com/37354145/120880624-8d709080-c606-11eb-8947-268b977aeef7.png">

페이지가 참조될 때마다 MRU 쪽으로 페이지 참조 시간을 이동시킨다.  
이렇게 구현할 경우 replace 가 일어날 때 LRU 쪽에 있는 페이지 하나만 건드리면 되므로, 시간 복잡도가 O(1)이 나온다.

### LFU
LRU와 비슷하게 링크드리스트를 사용하는데, 참조 회수를 미리 정렬 시킨다? 라고 생각 할 수 있지만, 

<img width="427" alt="image-20210519191526790" src="https://user-images.githubusercontent.com/37354145/120880625-8e092700-c606-11eb-98ea-1ee7e341d5f7.png">

그렇게 되면 참조회수가 1이 증가했을 때, 그 아래에 있는 녀석들과 한 번씩 비교를 진행해야한다.  
그럼 시간 복잡도가 O(n)이 나와버린다.


<img width="428" alt="image-20210519191614641" src="https://user-images.githubusercontent.com/37354145/120880626-8ea1bd80-c606-11eb-9819-08f7d0aa081e.png">
<img width="409" alt="image-20210519191740763" src="https://user-images.githubusercontent.com/37354145/120880627-8ea1bd80-c606-11eb-95ba-c0fe9b987ee8.png">

그래서 Min Heap 자료구조(complete binary tree)를 써버린다. 이러면 시간복잡도가 O(log n)이 나온다.

> log N에 대해 체감이 안오나? n이 1,000,000 이라고 쳤을 때, log n이면 10^6 = 6이 나온다. 단 6회만에 비교가 끝날 수도 있다는 것.


## 다양한 캐싱 환경

- 캐싱 기법
  - 한정된 빠른 공간(=캐시)에 요청된 데이터를 저장해두었다가 후속 요청시 캐시로부터 직접 서비스하는 방식
  - paging system 외에도 cache memory, buffer caching, web caching 등 다양한 분야에서 사용
    (캐시 메모리의 경우 운영체제 과목에선 다루지 않았지만 - 운영체제가 모르는 계층이므로)
    (실제 하드웨어적으론 CPU가 메인 메모리에 접근하기 전에 cache memory에 먼저 물어보는 식으로 동작한다)
    (주로 컴퓨터 구조 과목에서 이를 다룬다)
- 캐싱 운영의 시간 제약
  - 교체 알고리즘에서 삭제할 항목을 결정하는 일에 지나치게 많은 시간이 걸리는 경우 실제 시스템에서 사용할 수 없다
  - Buffer caching 이나 web caching의 경우
    - O(1)에서 O(log n) 정도까지 허용 - LRU와 LFU가 여기에 만족한다!
  - Paging System의 경우
    - page fault인 경우에만 OS가 관여. 애초에 page fault가 안나면 할 수 있는게 없다.
    - 페이지가 이미 메모리에 존재하는 경우 참조시각 등의 정보를 OS가 알 수 없다
    - O(1)인 LRU의 list조작조차 불가능


<img width="990" alt="image-20210519193456139" src="https://user-images.githubusercontent.com/37354145/120880628-8f3a5400-c606-11eb-9b87-2e41d81d2bd2.png">

Valid bit을 가진 (이미 물리 메모리에 올라온 ) 녀석에 대해 참조를 시도하면, 운영체제가 해주는 일은 아무것도 없다. invalid bit을 가진(물리 메모리에 없는) 녀석에 대해 참조를 시도하면(page fault) 그때서야 운영체제가 권한을 가진다. 운영체제가 disk에 있는 page를 읽어서 메모리로 올리는 (DIsk I/O, replace) 작업을 수행해야한다.

**그런데 과연 운영체제가 LRU, LFU 알고리즘에 필요한 정보 (최근 참조 정보, 최다 참조 정보)를 알아낼 수 있는가?**

답은, 절반만 알 수 있다. 이미 물리 메모리에 올라와있어서 page fault를 회피한 page에 대한 정보는 알 수 없고, page fault가 발생해서 CPU권한이 운영체제로 넘어가는 시점의 page에 대해서만 운영체제가 정보를 기억할 수 있다.

결론적으로, Paging System에선 LRU, LFU 알고리즘을 소화할 수 없다. 못해낸다. 즉 LRU, LFU 알고리즘은 캐시 메모리, 버퍼 캐시 등에나 사용되는 알고리즘이다.



## Clock Algorithm

Paging System에서 사용하는 알고리즘. LRU 알고리즘을 따라한 것.

- Clock algorithm
  - LRU의 근사(approximation) 알고리즘
  - 여러 명칭
    - Second change algorithm
    - NUR(Not Used recently), NRU(Not Recently Used)
  - Reference bit을 사용해서 교체 대상 페이지 선정 (Circular list)
  - Reference bit가 0 인 것을 찾을 때까지 포인트럴 하나씩 앞으로 이동
  - 포인터가 이동하는 중 reference bit1은 모두 0으로 교체
  - refence bit이 0인것을 찾으면 그 페이지를 replace
  - 한 바퀴 되돌아와서도 (=second change) 0이면 그 때는 replace 당함
  - 자주 사용되는 페이지라면 second change가 올 때 1


reference bit을 1로 바꾸는 작업은 하드웨어가 해주는 일. 운영체제 관할은 아님.
reference bit을 0으로 바꾸면서 쫒아낼 page를 찾는 일이 운영체제 관할이다.

즉, 완벽한 LRU는 아니지만 LRU와 어느정도 근사한 효과를 낼 수 있다!

- Clock algorithm의 개선
  - reference bit과 modified bit (dirty bit)을 함께 사용
  - reference bit = 1: 최근에 참조된 페이지
  - modified bit = 1 : 최근에 변경된 페이지 (I/O를 동반하는 페이지 - 쫒아낼 때 backing store도 Write 해줘야하는)
- 조금의 꼼수를 부리자면, reference bit이 0인 녀석들 중에서 modified bit이 1인 녀석보다 0인 녀석을 먼저 쫒아내면 Disk에 Write해주는 회수가 적어지므로, 조금이나마 더 빨라지는 효과를 누릴 수 있다.


<img width="1077" alt="image-20210519194629037" src="https://user-images.githubusercontent.com/37354145/120880629-8f3a5400-c606-11eb-856a-98f329f9f3c4.png">



## Page Frame의 Allocation
지금까지 배운 LRU, LFU, Clock 알고리즘에서는 프로그램 여러개가 물리적인 메모리에 적재되어 있으나, Page를 쫒아낼 때는 어떤 프로세스의 Page인지 무관하게 알고리즘에 맞춰서만 쫒아냈다.

그러나 실제 Page fault율을 낮추기 위해선 일련의 Page들이 함께 (유의미한 - Segment 단위로) 올라와야 한다.

- **Allocation problem : 각 process에 얼마만큼의 page frame을 할당할 것인가?**
- Allocation의 필요성
  - 메모리 참조 명령어 수행시 명령어, 데이터 등 여러 페이지 동시 참조
    (프로그램이 Code만 수행하는게 아니고 Data에 접근하는 등.. 여러가지 작업을 수행하니까)
    - 명령어 수행을 위해 최소한 할당되어야 하는 Frame의 수가 있음
  - Loop를 구성하는 page들은 한 번에 allocate 되는 것이 유리함
    - 최소한의 Allocation이 없으면 매 loop 마다 page fault
  - 그렇다고 일련의 페이지를 모두 올려주려고 하다보면 하나의 프로세스가 메모리 전체를 차지하는 일이 발생할 수 있다
    그래서 공평하게 나눠주는 등 Page frame allocation이 필요하다.
- **Allocation Scheme**
  - Equal allocation: 모든 프로세스에 똑같은 개수 할당 - 프로그램마다 필요한게 다르므로 비효율적
  - Proportional allocation: 프로세스 크기에 비례하여 할당 - 이것도 적절한 방법은 아닐 수 있다
  - Prioirity allocation: 프로세스의 중요도에 따라 다르게 할당

## Global vs Local replacement
- **Global replacement - 미리 process별로  allocation 하지 않는 방법**
  - replace시 다른 process에 할당된 frame을 빼앗아 올 수 있다
  - 자연스럽게 process별 할당량을 조절하는 또 다른 방법
  - FIFO, LRU, LFU 등의 알고리즘을 global replacement로 사용시에 해당한다
  - Working set, PFF 알고리즘을 사용한다
- **Local replacement - 미리 process별로 allocation 하는 방법**
  - 자신에게 할당된 frame 내에서만 replace
  - FIFO, LRU, LFU 등의 알고리즘을 process 별로 운영할 경우


## Thrashing
<img width="956" alt="image-20210519200735327" src="https://user-images.githubusercontent.com/37354145/120880630-8fd2ea80-c606-11eb-9746-08bb42873824.png">

**x - 동시에 메모리에 올라온 프로그램들 / y - CPU 사용률**

메모리 유휴 공간이 너무 많은 경우 CPU 사용률이 떨어지지만, 메모리 유후공간이 줄어들고 CPU가 쉬는 시간이 줄어들면 메모리 사용률이 높아진다 - > 긍정적인 효과

그러나 이것이 과해지면 CPU 사용률이 뚝 떨어진다. 모든 프로세스들이 I/O 처리를 요청하고 CPU는 쉬어버리기 때문. 이게 바로 쓰레싱.

- 프로세스의 원활한 수행에 필요한 최소한의 page frame 수를 할당받지 못한 경우 발생
- page fault rate가 매우 높아짐
- CPU utilization이 낮아짐 - CPU가 할 일이 없음
- OS는 MPD(Multiprogramming degree)를 높여야한다고 판단 - "CPU가 놀고있네?"
- 또 다른 프로세스가 시스템에 추가됨 (Higher MPD)
- 프로세스 당 할당된 frame의 수가 더욱 감소
- 프로세스는 page의 swap in / swap out으로 매우 바쁨
- 대부분의 시간에 CPU는 한가히 놀고있음
- low throughput

thrashing을 막기 위해선? MPD를 조절해주어야 한다. 그걸 해주는 알고리즘이 바로 **Working-set, PFF**다.



## Working-set Model
- **Locality of reference**
  - 프로세스는 특정 시간 동안 일정 장소만을 집중적으로 참조한다
    (루프 동안에 루프 내부의 페이지만 참조하는 등. 메서드의 내부만 참조되는 등.)
  - 집중적으로 참조되는 해당 page들의 집합을 **locality set**이라 한다
    (루프 하나의 집합.) - Working-set 알고리즘에선 locality set을 working-set 이라고 부른다.
- **Working-set Model**
  - Locality 에 기반하여 프로세스가 일정 시간동안 원활하게 수행되기 위해 한꺼번에 메모리에 올라와 있어야하는 page들의 집합을 **Working-set**이라 부른다
  - Working set 모델에서는 process의 working set 전체가 메모리에 올라와 있어야 수행되고, 그렇지 않을 경우 모든 frame을 반납한 후 swap out(suspend)
    (나는 최소 working-set이 5개인데, frame을 3개 밖에 안준다고? 더럽고 치사해서 일 안해. 5개 줄 때까지 일 안해 ㅅㄱ)
  - Thrashing을 방지하고, Multiprogramming degree를 결정함



## Working-set Algorithm
과거 시간을 참고해서 결정한다.
<img width="1073" alt="image-20210519201709645" src="https://user-images.githubusercontent.com/37354145/120880631-8fd2ea80-c606-11eb-98c3-8e5e8117304c.png">

t1 시점에서 보면, 델타(삼각형)크기가 10이라고 가정했을 때, 필요한 페이지가 1,2,5,6,7로 총 5개(Working-set 크기 5)다. 이 때 Frame의 개수를 5개 지급 받을 수 있으면 메모리에 올라오고, 못 받으면 메모리에서 내려온다.

t2 시점에서 보면, 델타(삼각형)크기가 10이라고 가정했을 때, 필요한 페이지가 3,4 로 총 2개(Working-set 크기2)다. 이 때 Frame의 개수를 2개 지급 받을 수 있으면 메모리로 올라가고, 못 받으면 메모리에서 내려온다.

즉, 참조된 후 **델타 시간 동안만 해당 페이지를 메모리에 유지한 후(Working-Set)** 버린다는 말과 동일하다!

과거를 통해서 working-set을 추정한다.

- Working-set Algorhithm
  - Process들의 working set size의 합이 page frame 수보다 큰 경우
    - 일부 process를 swap out 시켜 남은 process의 working set을 우선적으로 충족. (MPD를 줄인다)
  - Working set을 다 할당하고도 page frame이 남는 경우
    - swap out 되었던 프로세스들에게 working set을 할당 (MPD를 키운다)
- Window size - Delta
  - working set을 제대로 탐지하기 위해서는 window size를 잘 결정해야함
  - size가 너무 작으면 locality set을 모두 수용하지 못할 우려
  - size가 너무 크면 여러 규모의 locality set을 수용
  - size가 무한대면 전체 프로그램을 구성하는 page를 working set으로 간주



## PFF(Page-fault-frequency)
<img width="995" alt="image-20210519202442626" src="https://user-images.githubusercontent.com/37354145/120880632-906b8100-c606-11eb-81cd-ed3d72c7fa2d.png">

- 직접 page-fault rate를 참조한다.
  - page fault를 많이 일으키는 프로그램은 page frame을 더 많이 준다.
  - page fault가 적게 일어나면 page frame을 회수한다.

- 여유 frame이 없으면 일부 프로세스를 통째로 swap out 시킨다.



## Page size의 결정

- Page size를 감소시키면?
  - 페이지 수 증가
  - 페이지 테이블의 크기 증가
  - Internal fragmentation 감소
  - Disk transfer의 효율성 감소
    - Seek/rotation vs transfer
      Seek 시간이 워낙 크기 때문에...
  - 필요한 정보만 메모리에 올라와 메모리 이용이 효율적
    - Locality의 활용 측면에서는 좋지 않음
- Trend
  - Larget page size



## Swap-space management
<img width="1078" alt="image-20210519205444091" src="https://user-images.githubusercontent.com/37354145/120880633-906b8100-c606-11eb-85d1-1f2b68b67c8a.png">

- Disk를 사용하는 이유
  - 메모리의 휘발적인 특성 -> file system
  - 프로그램 실행을 위한 메모리 공간 부족 -> swap space(swap area)
- Swap-space
  - virtual memory system 에서는 디스크를 메모리의 연장 공간으로 사용
  - 파일 시스템 내부 둘 수도 있으나, 별도의 partition을 사용하는게 일반적이다
    - 공간 효율성보다는 속도 효율성 우선
    - 일반 파일보다 훨씬 짧은 시간만 존재하고, 자주 참조된다
    - 따라서, block의 크기 및 저장 방식이 일반 파일시스템과 다르다

Swap area는 seek 타임을 줄여서 시간 효율성을 높여야한다. (어차피 금방 사라질 녀석들이기 때문에)

512Kb 등의 대용량의 단위로 빨리 대충 올리고 내린다.