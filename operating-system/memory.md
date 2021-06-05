# 리눅스 메모리
 Noncontiguous allocation 전까지 설명하는 메모리 프로그램 주소들은 덩어리째(Contiguous allocation)로 올라가서 편리해보이지만, 실제 현대에서는 가상 메모리를 통해 프로그램이 부분적(Noncontiguous allocation)으로 메모리에 올라가고, OS가 이를 관리하는 등 굉장히 복잡하다. 그저 메모리의 역사에 대해 살펴보기 위함이니 너무 간단하게 생각하진 말것.


## 메모리는 주소로 이루어진 매체
또한 주소를 통해서 접근하는 매체. 메모리에는 주소가 매겨진다.
주소를 크게 2가지로 나눌 수 있다.

- **Logical Address (= Virtual Address)**
  - 프로세스마다 독립적으로 가지는 주소공간
  - 각 프로세스마다 0번지부터 시작하는 논리 주소를 각자 가진다.
  - CPU가 보는 주소는 Logical adddress 다.
- **Physical Address**
  - 메모리에 실제 올라가는 위치 (물리적인 주소)
  - 물리적인 메모리 낮은 주소에는 OS Kernel이 올라가있다.
  - 물리적인 메모리 높은 주소에는 여러 프로세스들이 섞여서 올라가있다.

주소 바인딩 : 주소를 결정(변환)하는 것.

> Symbolic Address --> Logical Address --> Physical Address

Symbolic Address 는 개발자 입장에서 사용하는 주소. 변수, 함수 등으로 Logical Address를 담아서 관리할 때 사용한다.

컴파일 단계에서 Symbolic Address가 Logical Address로 변환된다. (즉, Logical, Physical Address 는 모두 숫자로 이루어져있음을 추측할 수 있다.) 

Logical Address를 할당 받은 프로세스가 실제로 실행되기 위해선 메모리의 Physical Address로 찾아가야 한다.

<br>

## CPU가 바라보는 주소는 논리주소일까, 물리주소일까?

CPU는 하드웨어이기 때문에 물리주소일 것 같지만, 논리주소를 바라보고 있다.

>  이는 이전 그림을 잘 살펴보면 알 수 있는데, 로드타임/런타임 메모리 주소를 확인해보면 5XX/3XX/7XX 등으로 물리 주소가 변환되고 있지만, 그 내부코드에 있는 논리주소(0, 10, 20, 30, 40)은 건드리지 못하고 있다. 이 논리 주소를 바꾸려면 아무리 런타임 바인딩이라도 코드 상의 주소라서 재컴파일이 필요하다.

??? 솔직히 무슨 말인지 모르겠다. 
그냥 CPU가 MMU에게 주소 변환을 전적으로 맡기고 논리 주소만 읽는 '관심사의 분리'를 행한다고 생각

<br>

## Logcal Address 가 Physical Address로 바인딩되는 시점
크게 3가지 시점으로 나눌 수 있다.

### Compile time binding

- 물리적 메모리 주소가 컴파일 타임에 알려진다
- 시작 위치 변경시 다시 컴파일이 수행되어야 한다
- 컴파일러는 절대 코드(absolute code)를 생성한다

###  Load time binding - 프로그램이 막 시작되는 단계

- Loader 의 책임하에 물리적 메모리 주소를 부여한다
- 컴파일러가 재배치 가능한 코드(relocatable code)를 생성한 경우 가능하다

### Execution time binding (=Run time binding) - 프로그램 실행 도중

- 수행이 시작된 이후에도 프로세스의 메모리 상 위치를 옮길 수 있다.
- CPU가 주소를 참조할 때마다 binding을 점검한다(Address Mapping Table)
- Run time binding은 하드웨어적인 지원이 필요하다
  (Ex. base and limit registers, MMU)


<img width="612" alt="image-20210517123228246" src="https://user-images.githubusercontent.com/37354145/118755541-043a2980-b8a4-11eb-869c-450c46eed24f.png">

(A에 A + B 연산 결과를 할당하고, C로 이동해서 종료하는 프로그램이다.)  
A, B, C Symbolic address들이 컴파일 타임에 Logical Address로 변경된다.

### **1.Compile time binding** 
Compile time binding은 컴파일 시점에 물리적인 주소가 결정된다고 했다. 
즉, 프로그램을 메모리에 올릴 때는 컴파일 시점에 결정된 주소에만 프로그램을 올려야한다.
물리적 메모리에 다른 주소가 많이 남아돌더라도, 딱딱하게 그 주소에만 올리려고 한다.

수행되는 프로그램 수가 적은 임베디드 등의 환경에서는 컴파일 타임에 주소를 모두 할당하고 사용하는 것이 효율적이겠지만, 수행되는 프로그램 수가 많은 PC 환경에서는 굉장히 비효율적이기 때문에 현재 컴퓨터시스템에서는 크게 사용되지 않는다.

컴파일 타임 바인딩을 사용할 때 컴파일 타임에 주어진 주소는 논리적 주소이기도 하지만 동시에 물리적 주소가 되므로, 이것을 **절대코드(absolute code)**라고 부른다.

메모리에 올라가고 싶은 위치를 바꾸고 싶다면 재컴파일을 해야만 한다. 절대코드라 불리는 이유가 이것이다.

### 2. Load time binding
프로그램이 최초 수행되는 시점에 물리적인 주소가 결정된다.
Compile time에는 논리적인 주소까지만 결정되고, 프로그램이 실행될 때 메모리에서 비어있는 주소를 확인하고 그 쪽에 올린다.

Load time binding과 Runtime binding은 컴파일 타임에 논리적 주소만 결정되었을 뿐, 물리적 주소는 얼마든지 재배치 가능하므로 **재배치 가능 코드(relocatable code)** 를 가지게 된다.

### 3. Run time binding
Load time binding 처럼 프로그램 수행시에 물리적인 주소가 결정되는건 똑같지만, 실행 도중에도 물리적인 주소를 얼마든지 변환시킬 수 있다.

그렇다면 어느 순간에 주소가 이동하는가?
운영체제가(혹은 자발적으로) 프로세스를 일시적으로 인터럽트 시키는 등의 행동으로 메모리에서 쫒아냈다가, 다시 수행상태로 되돌아 올 때 기존 사용중이던 주소의 자리가 차있다면 다른 비어있는 메모리 주소를 찾아서 이동하는 형태를 가진다. 즉 프로세스의 상태(State)가 변화하는 순간.

우리가 사용하는 PC 환경에서는 당연히 Run time binding을 보유하고 있을 것이다.

Compile time binding과 Load time binding은 프로그램이 시작될 때 주소가 결정되고 그 주소의 변화가 없지만(그냥 물리주소를 더해주면 끝), Run time binding은 프로그램 수행 중에도 계속 주소가 바뀌므로, CPU가 메모리 주소를 요청할 때마다 주소 바인딩을 체크해야한다. 그러기 위해선 하드웨어적인 지원이 필요하다. (MMU)

<br>

## MMU(Memory Management Unit)
논리 주소를 물리 주소로 매핑해주는 Hardware device.

### MMU scheme
사용자 프로세스가 CPU에서 수행되며 생성해내는 모든 주값에 대해 **base register (=relocation register)**의 값을 더한다.

<img width="601" alt="image-20210517125630622" src="https://user-images.githubusercontent.com/37354145/118755551-07cdb080-b8a4-11eb-8d81-a83324de1c30.png">

기본적인 MMU에서는 register 2개를 통해서 주소 변환을 수행한다.

### Relocation register (=base register)
CPU가 논리적 주소로 process의 정보를 요청하면, relocation register가 가지고 있던 **"메모리에 올라가있는 프로세스의 시작위치"** 를 더해서 실제 물리적 주소를 만들어낸다. 그 주소로 찾아가서 내용을 읽어서 CPU에게 가져다 준다.

### Limit register
이 프로그램의 최대 크기가 몇인지 (마지막 주소가 어디인지)를 담고 있다. 

이 프로그램이 만약 악의적인 프로그램이라서, 본인의 크기가 3000인데도 불구하고 CPU에게 메모리 4000번지를 달라고 하게끔 유도할 수도 있다. 그렇게 되면 다른 프로그램에 존재하는 메모리 위치를 요청하게 된다. 남의 프로그램 정보를 보려고 하면 안된다. 그래서 base register 주소를 더해주기 전에 limit register로 정보 체크를 먼저한다.

<img width="613" alt="image-20210517134000090" src="https://user-images.githubusercontent.com/37354145/118755552-08fedd80-b8a4-11eb-945e-2a547ab08475.png">

즉, Limit register 확인 작업이 먼저 선행되고, 그 후에 relocation register 작업이 진행된다.

Limit register 확인 작업에서 값을 벗어남을 확인하면 TRAP(software interrupt)에 걸리게 된다.
TRAP에 걸리면 CPU제어권이 프로그램에서 운영체제에게로 넘어가게 된다. 그러면 운영체제는 TRAP이 왜 걸렸는지, 악의적인 이유였음이 밝혀지면 강제 ABORT 시키는 등의 처리를 수행한다.

user program은 논리 주소만을 다룬다. 실제 물리 주소를 볼 수 없으며, 알 필요도 없다. (CPU도 마찬가지)

<br>

## Dynamic Loading
- 프로세스 전체를 메모리에 다 올리는 것이 아니라, 해당 루틴이 불려질 때 메모리에 로드 하는 것.
- Memory utilization 의 향상
- 가끔씩 사용되는 많은 양의 코드의 경우 유용
  - 프로그램이라는 것은 방어적으로 만들어지기 때문에(대부분의 오류처리, 검증로직 등 - 특히나 좋은 소프트웨어일수록 특이한 INPUT도 모두 처리하기 위해) 자주 사용되는 부분은 굉장히 한정적이다. 
  - 예를 들어 오류처리 루틴. 잘 사용되지 않지만 분명히 사용되고, 많은 메모리를 차지한다.
- 운영체제의 특별한 지원 없이 사용자 프로그램 자체에서 구현이 가능하다.
  - 운영체제 라이브러리를 통해 지원 가능하다!

> 그러나 현재 컴퓨팅에서 필요하면 메모리에 올라가고, 필요없으면 쫒겨나는건 운영체제가 가진 페이징 시스템에 의한 것이지, Dynamic Loading을 통해서 진행되는 것은 아니다. 
>
> 페이징 시스템은 운영체제가 해주는 것. Dynamic Loading은 프로그래머가 직접 그렇게 되도록!
>
> (혼용해서 사용하기도 한다. 그렇지만 근본적으로 다른 것임을 인지하면 좋겠다는 듯.)

그렇다면 프로그래머가 직접 수작업으로 메모리에 올리고 내리고를 다 만들어야하나?
다행히도 그건 아님. 운영체제가 라이브러리 형태로 제공해주고 있다. 프로그래머는 라이브러리만 사용하면 된다.

<br>

## Overlays

- 메모리에 프로세스의 부분 중 실제 필요한 정보만 올리는 것
- 프로세스의 크기가 메모리보다 클 때 유용
- 운영체제의 지원없이 사용자에 의해 구현
- 작은 공간의 메모리를 사용하던 초창기 시스템에서 수작업으로 프로그래머가 구현
  - 다른 말로는 Manual Overlay
  - 프로그래밍이 불편하고 매우 복잡하고 어렵다.

> 다이나믹 프로그래밍과 어떻게 다른가?
>
> 오버레이는 "메모리보다 프로세스가 큰 경우"라는 전제조건이 붙는다.
> 오버레이는 운영체제가 라이브러리를 제공해주지 않는다.

<br>

## Swapping

### Swapping
프로세스를 일시적으로 메모리에서 **backing store** 로 쫒아내는 것

### Backing store(=swap area)
디스크. 많은 사용자의 프로세스 이미지를 담을 만큼 충분히 빠르고 큰 저장 공간

### Swap in / Swap out
- 일반적으로 중기 스케줄러(swapper)에 의해 swap out(메모리에서 아웃) 시킬 프로세스 선정

- Priority-based CPU scheduling algorithm
  - 중요도가 낮은 프로세스를 메모레엇 OUT
  - 중요도가 높은 프로세스를 메모리로 IN

- 컴파일 타임 혹은 로드타임 바인딩에서는 원래 메모리 위치로 다시 swap in 해야함
  - 그렇기 때문에 사실 Swapping의 효과를 십분발휘하긴 어렵다

- 런타임 바인딩은 추후 빈 메모리 영역 아무 곳에나 올릴 수 있다
  - Swapping을 효과적으로 사용하기 위해선 런타임 바인딩이 지원되어야한다

- swap time은 대부분 transfer time(swap되는 데이터 양에 비례하는 시간)임
  - 상당히 방대한 양(프로세스 전체)이 통째로 SWAP되기 때문에, 단순한 FILE I/O로 생각하면 안된다
  - 우리가 일반적으로 디스크에 접근하는 시간은 SEEK TIME(디스크 읽기 헤더가 움직이는 시간)이 
    가장 크기 때문에 SEEK TIME을 생각하지만(TRANSFER TIME이 워낙 미미해서), SWAPPING에서는 TRANSFER TIME도 상당부분을 차지한다.


<img width="545" alt="image-20210517135512261" src="https://user-images.githubusercontent.com/37354145/118755555-09977400-b8a4-11eb-8dd5-415b530dc776.png">

> 원래 이야기하는 SWAPPING도 본연의 의미는 이게 맞다. 그러나 최근엔 페이징 시스템에서 페이지를 쫒겨내는 것을 또 SWAP-OUT / SWAP-IN이라는 명칭을 사용한다.

<br>


## Dynamic Linking

Linking : 여러군대 존재하던 컴파일된 파일들을 하나로 묶어서 실행파일로 만드는 것.
	내가 소스파일을 여러개 따로 코딩해서 링킹하기도 하고, 라이브러리를 끌어와서 링킹하기도 한다. 
	(내 코드 안에 라이브러리 코드가 포함되는 등)

- Static Linking
  - 라이브러리가 프로그램의 실행 파일 코드에 포함됨
  - 실행 파일의 크기가 커짐
  - 동일한 라이브러리를 사용하는 각각의 프로세스가 각자 메모리에 올리므로 메모리 낭비
    (예를 들면 printf 함수의 라이브러리 코드)
  - 즉, Static Linking을 하는 순간 외부 코드(라이브러리)라도 메모레 입장에선 프로세스의 일부러 받아들인다.

- Dynamic linking
  - Linking 을 실행시간(Execution-time)까지 미루는 기법
  - 실행중 라이브러리나 다른 코드를 호출하는 시점에 도달하면, 그 때 연결(link)됨.
  - 별도로 존재하는 코드(라이브러리)의 위치를 기억해둔 포인터(stub) 이라는 작은 코드를 둔다.
  - 라이브러리가 이미 메모리에 있으면 그 루틴의 주소로 가고, 없으면 그때 디스크에서 읽어옴
  - 그래서 Dynamic Linking을 지원하는 라이브러리를 Shared-library 라고도 부른다.
  - 리눅스에선 Shared Object, 윈도우에선 DLL(Dynamic Linking Library)파일
  - 운영체제(OS)의 도움이 필요하다.

<br>

## Allocation of Physical Memory

물리적 메모리를 어떻게 관리할 것인가?

<img width="109" alt="image-20210517142535576" src="https://user-images.githubusercontent.com/37354145/118755559-0b613780-b8a4-11eb-91ff-b2727ba1561a.png">

- 메모리는 일반적으로 두 영역으로 나누어 사용
  - OS 상주 영역
    -  intterrupt vector와 함께 낮은 주소 영역 사용
  - 사용자 프로세스 영역
    - 높은 주소 영역 사용

- 사용자 프로세스 영역의 할당 방법

  - **Contiguous allocation (연속 할당)**
    - 각각의 프로세스가 메모리의 연속적인 공간에 적재
    - MMU의 주소 변환도 비교적 간단했다
    - **Fixed Partition allocation (고정 분할)**
      - 물리적 메모리를 몇 개의 영구적 분할(partition)으로 나눔
      - 분할의 크기가 모두 동일한 방식과 서로 다른 방식이 존재
      - 분할당 하나의 프로그램 적재
      - 동시에 메모리에 올릴 수 있는 프로세스의 수가 고정됨
      - 최대 수행 가능한 프로그램의 크기 제한
        - External fragmentation 발생
          - 프로그램 크기보다 분할(partition)크기가 작은 경우
          - 아무 프로그램에도 배정되지 않은 빈 곳인데도 프로그램이 올라갈 수 없는 작은 분할
        - Internal fragmentation 발생
          - 프로그램 크기보다 분할의 크기가 큰 경우
          - 하나의 분할 내부에서 발생하는 사용되지 않는 메모리 조각
          - 프로그램이 배정되었지만 사용하지 못하는 공간
    - **Variable partition allocation (가변 분할)**
      - 프로그램의 크기를 고려해서 할당
      - 분할의 크기, 개수가 동적으로 변한다
      - 기술적 관리 기법이 필요하다
        - 프로그램이 실행될 때마다 순서대로 차곡차곡 메모리에 올리는 방식
        - 프로그램 A, B, C 순서대로 적재되어 수행중이다가, B가 역할을 끝내고 메모리에서 빠져나감
        - 프로그램 D가 메모리에 올라왔으나, B가 빠져나간 공간이 너무 작아서 들어가지 못함
        - 프로그램 D는 자연스럽게 여유공간이 있는 쪽으로 들어갔고, B가 빠져나간 공간은 외부조각으로 만들어짐
        - 프로그램 D가 차지하고 남은 공간도 너무 작아 다른 프로그램이 사용하지 못할 가능성이 높아 External fragmentation 으로 분류할 수 있다.
      - 가변 분할 방식은 Internal fragmentation은 안생긴다!

<img width="598" alt="image-20210517143054940" src="https://user-images.githubusercontent.com/37354145/118755566-0dc39180-b8a4-11eb-8ebc-f26e846097df.png">

  - **Noncontiguous allocation (불연속 할당)**
    - 프로그램을 구성하는 주소공간을 잘게 쪼갠다(Page 단위)
    - 하나의 프로세스가 메모리의 여러 영역에 분산되어 올라갈 수 있음
    - Paging
    - Segmentation
    - Paged Segmentation

<br>

## Contiguous allocation / Hole

- 가용 메모리 공간
- 다양한 크기의 Hole들이 메모리 여러 곳에 흩어져 있다
- 프로세스가 도착하면 수용가능한  Hole을 할당
- 운영체제는 다음의 정보를 유지한다
  1. 할당 공간 - 사용중인 공간
  2. 가용 공간 (Hole) - 유휴공간, 사용되지 않는 공간



그렇다면, 운영체제는 어떤 Hole에 프로세스를 할당해야하는가?

<br>

## Contiguous allocation / Dynamic Storage-Allocation Problem (가변분할 방식 할당 문제)

가변 분할 방식에서 size N인 요청을 만족하는 가장 적절한 Hole을 찾는 문제

- **First-fit**
  - 크기가 N 이상인 것들 중 가장 처음 마주하는 Hole에 할당
  - 현재 순간에 가장 좋은 방법 (빠르니까)
- **Best-fit**
  - 크기가 N 이상인 것들 중에서 가장 작은 Hole에 할당
  - Hole 들의 리스트가 크기순으로 정렬되지 않은 경우 모든 Hole을 탐색해야 한다
  - 최적의 Hole에 할당하지만, 결국 미세하게 작은 Hole들이 생겨나게 된다
  - 미래를 위해 가장 좋은 방법 (공간 이용률을 최대한 확보할 수 있으니까)
- **Worst-fit**
  - 크기가 N 이상인 것들 중에서 가장 큰 Hole에 할당
  - 역시나 모든 Hole 리스트를 탐색해야한다
  - Best fit에 비해서 상대적으로 아주 큰 Hole들에 생성된다.

테스트 결과 First-fit과 best-fit이 worst-fit 보다 속도와 공간 이용률 측면에서 효과적으로 알려졌다.
즉, 굳이 Worst-fit을 사용할 이유는 없다.

<br>

## Contiguous allocation / compaction

- external-fragmentation 문제를 해결하는 방법 중 하나 (디스크 조각 모음)
- 사용 중인 메모리 영역을 한군데로 몰고, Hole들을 다른 한곳으로 몰아 아주 큰 block(Hole)을 만드는 것
- 최소한의 메모리 이동으로 Compaction하기 어렵다 (매우 복잡하고, 비용이 많이 드는 방법)
  - 어떤 프로그램을 이동시킬 것인가도 결정하기 굉장히 어렵고...
- Compaction은 프로세스의 주소가 실행 시간에 동적으로 재배치가 가능한 경우에만 수행할 수 있다
  Run time binding 에서나 수행할 수 있다는 말

<br>

## Noncontiguous allocation / Paging

- Process의 Virtual memory를 동일한 사이즈의 Page 단위로 나눔
- Virtual memory의 내용이 Page 단위로 Non-contiguous 하게 저장된다
- 일부는 backing storage에, 일부는 physical memory에 저장
- Physical memory도 Page 단위로 미리 잘라둔다 - Frame
- Logical memory를 Page 단위로 나눠둔다 - Frame과 같은 크기
- 모든 가용 Frame 들을 관리한다
- Page table을 사용해서 Logical address를 Physical address로 변환한다
- External fragmentation이 발생하지 않는다
- 다만, Internal fragmentation은 발생할 수 있다
  - 프로세스를 잘라둔 크기가 반드시 Page 단위로 맞춰지라는 법은 없으니까
  - 그렇지만 Page 크기가 작다면 Internal fragmentation의 영향력이 있진 않을테니 무시해도 되는 정도

MMU로 주소 변환이 굉장히 복잡해진다. 
잘려진 각각의 Page가 어디로 올라가있는지 일일히 Page table을 참조해서 계산해야한다.

<br>

##  Noncontiguous allocation / Segmentation

프로그램을 구성하는 주소 공간을 의미를 갖는 크기로 자르는 것.

Code Segmentation, Data Segmentation, Stack Segmentation 등으로 잘라서 각각의 세그먼트를 필요시에 물리적 메모리에 올릴 수 있게. (물론 더 잘게 자를 수도 있다. - 메서드 단위 등)

크기가 균일하지 않다. 그래서 프로그램을 통째로 메모리에 올릴 때 Hole 관련 이슈가 발생한다.

<br>

##  Noncontiguous allocation / Paged Segmentation

세그멘테이션과 페이징 기법을 혼합한 것

<br>

## Paging

<img width="895" alt="image-20210519132427347" src="https://user-images.githubusercontent.com/37354145/120880592-834e9200-c606-11eb-8ce3-fc14f683f6b7.png">


프로세스를 구성하는 논리적 주소를 동일한 크기의 Page 단위로 잘라서 비어있는 물리적 메모리 Frame 에 올리는 것.
논리적 주소 - 물리적 주소 변환을 위해서 Page Table이 사용된다.
위 그림을 다르게 표현하면 아래와 같이 볼 수 있다.

<img width="881" alt="image-20210519132728829" src="https://user-images.githubusercontent.com/37354145/120880593-83e72880-c606-11eb-995d-157da131f8ef.png">

CPU가 **논리적인 주소(Page주소 + offset 주소)** 로 요청을 하게 되면 Page 주소와 offset 주소를 분리한다.
분리된 주소에서 Page 주소는 Page table을 참조, offset 주소는 Page table을 참조해서 찾아낸 Frame 주소와 다시 합쳐진다.
합쳐진 주소가 물리적인 주소가 되어 물리 메모리에 위치한 Frame에서 데이터를 가져온다.

그렇다면 이 중요한 Page table은 어디에 위치시켜야 하는가?  
(이전 contiguous allocation에서 사용된 MMU는 별도의 하드웨어였으나, 현대에는 프로세서와 같은 칩에 회로로 삽입된다)

보통 Page의 크기는 4kb다. 프로그램마다 다르지만 Page table 엔트리가 100만개 이상 필요한 경우가 대다수다.   
100만개의 엔트리가 CPU의 register에 들어갈 수 있나? 어림도 없지.  
그리고 Page Table은 "프로그램마다" 필요하다. 프로그램 1개당 1개씩 필요하다.  
메모리 접근을 위한 주소 접근인데 HDD에 저장하기도 그렇고, 캐시 메모리에 넣기도 너무 용량이 크다.  
그럼 어디에? 결국 메모리에 저장한다.

<img width="981" alt="image-20210519133935406" src="https://user-images.githubusercontent.com/37354145/120880594-847fbf00-c606-11eb-9b0a-0fcb6a46a8e7.png">

실제로 메모리에 접근하기 위해선 페이지테이블 접근 1회, 실제 데이터에 접근 1회로 총 2회가 필요하다.

그렇다면 페이징 기법에서 MMU는 어떻게 사용되고 있는가?

- Page-table base register(PTBR) - 시작 위치, 이전 relocation register(base register)
- Page-table length register(PTLR) - 전체 길이, 이전 Limit register

2가지로 사용되고 있다.

데이터 접근을 위해 매번 메모리에 2회 접근하는건 비용이 상당히 크다. 뭘 하든 비용이 2배니까.

그래서 별도의 하드웨어 지원을 또 받는다. - Associative register로 구성되는 Translation Look-aside Buffer(TLB)
일종의 캐시 메모리. 메인 메모리보다 빠른, 메인 메모리와 CPU사이에 존재하는 주소 변환을 위한 하드웨어


<img width="747" alt="image-20210519134414875" src="https://user-images.githubusercontent.com/37354145/120880596-847fbf00-c606-11eb-8bd9-f3e0949588b1.png">

(그림 속 Page table은 사실 물리적 메모리 내부에 존재한다!)

 컴퓨터 구조 시간에 메인 메모리 윗단에 캐시 메모리가 있다는걸 배울텐데, 운영체제 입장에서는 캐시 메모리의 존재를 알지 못한다. (감추어진 계층 - 관심사 분리)

Page table을 통한 주소 변환을 위한 별도의 캐시 메모리를 두고 있는데, 그것이 바로 TLB다. 오로지 주소 변환만을 위해 사용되며, 데이터를 보관하는 캐시 메모리와는 다르다.

TLB는 Page table에서 자주 사용되는 소수의 주소에 대해서 캐싱을 하고 있다. 당연히 메인 메모리보다 접근 속도가 빠른 하드웨어로 구성이 되어있다.

CPU가 주소를 전달하게 되면, Page table에 접근하기 전에 우선 TLB부터 탐색한다. TLB에 저장되어 있는 주소 중에 매칭되는 녀석이 있는 경우 (TLB hit) TLB를 통해 바로 주소 변환을 진행한다. (메인 메모리에 총 1회 접근)
TLB에 주소가 없는 경우 (TLB miss) Page table을 참고해서 주소 변환을 다시 진행한다. (메인 메모리에 총 2회 접근)

자세히 보면 TLB는 Page table과 다른 구조를 가질 수 밖에 없음을 알 수 있는데, Page table은 프로세스가 가진 전체 주소에 대해 매칭되는 물리 주소를 가지고 있기 때문에, CPU가 던지는 Page 주소를 순서(Index)처럼 사용해서 Frame 탐색이 가능해서 별도의 Key 없이 Frame 정보만 가지고 있다.

반면 TLB는 전체 프로세스의 연속되는 주소정보를 가지고 있는게 아니라 자주 사용되는 소수의 주소에 대해서 랜덤하게 가지고 있기 때문에, Map 자료 구조와 같이 Key와 Value 형태로 Page 주소와 Frame 주소를 함께 가지고 있다.
또한 그렇기 때문에 주소를 변환할 때 Page table처럼 특정 항목만 검색할 수 없고, TLB 전체를 탐색해야한다.
전체탐색은 뭐다? 느릴 수 밖에 없다.

<img width="1039" alt="image-20210519135241542" src="https://user-images.githubusercontent.com/37354145/120880597-85185580-c606-11eb-893f-82e53b87294a.png">

그렇기 때문에 TLB는 병렬적으로(동시에) 탐색이 가능한 Associative register를 이용해서 만들어진다.
(반대로 Page table은 해당 인덱스를 찾아가면 바로 주소 변환이 이루어지기 때문에 Associative register 등을 사용할 필요가 없다)

TLB는 프로세스마다 존재하는게 아니라, 컴퓨팅 시스템 1개에 1개만 존재하는 경우가 대부분이기 때문에, Context-switch (CPU 할당 권한이 넘어가면) 발생시 TLB의 정보를 모두 날리고 CPU 권한을 얻는 프로세스의 정보로 채우기 시작한다.

<img width="978" alt="image-20210519135540437" src="https://user-images.githubusercontent.com/37354145/120880598-85b0ec00-c606-11eb-9058-380bc5b76d35.png">

그렇다면 실제로 메모리에 접근하는 시간이 얼마나 향상되었는가?

TLB에 접근하는 시간을 E, 메인 메모리 접근시간 1, TLB로부터 주소변환이 되는 비율(탐색 성공 비율)을 A 라고 뒀을 때

실질적으로 효과를 보는 시간은 EAT =  <메모리 2회 접근시간> + <TLB접근시간> - <탐색 성공 비율>

<탐색 성공 비율> 에 따라 다르긴 하겠지만, 캐시 메모리 동작 특성(성공 확률)을 생각해보면 A 값이 엄청 크다는걸 유추 가능. 결론 - 매우 빠름!

<br>

## Two-Level Page table

<img width="945" alt="image-20210519140324815" src="https://user-images.githubusercontent.com/37354145/120880599-85b0ec00-c606-11eb-88cb-2bc99da02c7c.png">

왜 two-level page table을 사용하는가?

보통 컴퓨터 시스템에서 이짓거리를 하는 이유는 크게 2가지다. 속도 향상을 위하거나, 공간 절약을 위하거나.

Page table을 1회 접근하던걸 2회 접근으로 늘렸으니 속도가 빨라지진 않았을거다. 그렇다면? 공간 절약.

- 현대의 컴퓨터는 Address space가 배우 큰 프로그램 지원
  - 32 bit address 사용시 각 프로그램이 표현 가능한 주소 공간 : 2^32 (4G)의 주소 공간 (최근에는 64 bit address)
    - 2^10 = K, 2^20 = M, 2^30 = G, 2^40 = T... 
    - page size가 4kb일 경우 1M(약 100만)개의 page table entry 필요
    - 각 page entry의 크기가 4Byte일시 프로세스당 4M의 page table 필요
      - 메모리는 주소 단위를 Byte 단위로 메긴다
    - 그러나, 대부분의 프로그램은 4G의 주소 공간 중 지극히 일부분만 사용하므로, page table 공간의 낭비가 심함
    - 그럼 왜 프로세스 전체에 대해 페이지 테이블을 만드는가?
    - 페이지 테이블은 index 형식으로 접근하기 때문에, 전체 주소에 대한 엔트리가 필요할 수 밖에 없다.
- page table 자체를 page로 구성하자!
- 사용되지 않는 주소 공간에 대한 outer page table 의 엔트리 값은 NULL(대응하는 inner page table이 없음)

<img width="1062" alt="image-20210519141612056" src="https://user-images.githubusercontent.com/37354145/120880600-86498280-c606-11eb-8d0c-3b6d0426094b.png">

CPU가 논리 주소(P1, P2, d)를 던졌을 때,바깥쪽 페이지 테이블에서  P1 주소를 찾아서 두번째 조회할 페이지 테이블이 어떤 녀석인지 찾아내고, 찾아낸 두번째 페이지 테이블에서 P2 주소를 찾아서 실제 데이터가 위치한 물리 주소를 받아낸다.

단 여기서 주의해야할 것, 두 번째 테이블(안쪽 페이지 테이블)의 크기가 Page size와 완전히 동일하다. 안쪽 테이블 자체가 Page화 되어 Page Frame에 들어가있는 것이기 때문이다.(보통 4Kb) 테이블 엔트리 하나의 크기가 보통 4Byte 라고 했기 때문에, 테이블 하나당 엔트리의 개수는 1K(1000)개.

<img width="1026" alt="image-20210519141634350" src="https://user-images.githubusercontent.com/37354145/120880602-86498280-c606-11eb-9f13-ab97b6fe4a4b.png">

그렇다면 CPU가 뱉어내는 논리 주소는 몇 bit씩 분리시켜야 하는가?

page offset부터 살펴보면, Page 하나의 크기가 4Kbyte기 때문에, 12bit을 필요로 한다 (2^12 -> 4096)

그렇다면 page-table들은 어떻게 10bit씩? 공평하게 반으로 나눠서? ㅋㅋㅋ 그건 아니고~

Page table 엔트리 하나당 크기는? 4Byte. 테이블의 크기는? 4Kbyte. 엔트리는 총 몇 개? 1K개.

1K(1000)를 구분하기 위해선? (2^10 -> 1024)

재밌네. 이 공식대로 64bit로 올라가면?

<img width="718" alt="image-20210519142844163" src="https://user-images.githubusercontent.com/37354145/120880603-86e21900-c606-11eb-82bf-98016b0ec3f3.png">


그러나 효율이 영 안나온다. 바깥쪽 테이블이 너무 크다. 그래서 3, 4 레벨 혹은 해시 페이지 테이블 기법을 사용한다.



그런데 다 따지고 보면, Two-level page table은 시간적으로도 손해, 공간적으로도 어쨌거나 100만개의 entry가 필요하고, 각각 레벨의 테이블을 저장하는 것도 결국 공간을 차지하기 때문에 공간적으로도 손해인것처럼 보인다. 그럼에도 사용하는 이유는?

Outer table은 전체 프로세스의 크기만큼 만들어지지만, Inner table은 사용되지 않는 Outer table의 경우 만들지 않는다. 즉, 자주 사용되지 않는 Outer table이 Inner table을 만들지도, 가리키지도 않는 (Pointer -> NULL) 상태를 유지한다.

이 기법을 통해서 공간절약을 진행하고 있는 것이다.

<br>

## Multi-level Paging

<img width="1004" alt="image-20210519143355006" src="https://user-images.githubusercontent.com/37354145/120880604-877aaf80-c606-11eb-878f-10f77a3aa7ba.png">

프로그램의 주소 공간이 굉장히 넓어지면(64bit) 점점 더 여러 단계 페이지 테이블을 사용하는게 가능한데, 

테이블을 위한 공간을 더 많이 줄일 수 있지만, 한 번 주소 변환을 하려면 페이지 테이블을 여러번 거쳐야하고,

페이지 테이블이라는게 또 물리적 메모리 상에 존재하기 때문에, 메모리 1회 접근을 위해(예를 들어 4단계 테이블이면) 주소 변환을 위해 메모리에 4회, 실제 데이터에 1회 합해서 총 5회의 접근을 필요로 한다.

메모리 접근 시간이 100ns라고만 가정해도 5회접근에 총 500ns 가 걸린다. 그런데 우리가 이전 시간에 TLB를 배웠다.
(TLB통해 실제 데이터가 존재하는 물리 주소에 접근, 여러 단계의 페이지 테이블을 조회하는 과정이 바로 스킵된다.)

**캐시 적중률이 98%인 경우 <98%> * <메모리 접근 + TLB 접근> + <2%> * <4회 테이블 접근 + TLB 접근>**

결국 TLB 덕분에 다단계 페이지 테이블을 사용해도 시간이 그렇게 길게 추가되진 않는다. 덕분에 많은 공간을 아낄 수 있다.



## Memory protection

<img width="941" alt="image-20210519150336004" src="https://user-images.githubusercontent.com/37354145/120880605-877aaf80-c606-11eb-83d3-777acd4f50c1.png">

좌 프로그램마다 주어지는 논리 메모리, 가운데 페이지 테이블, 오른쪽이 물리 메모리

지금까지 본 페이지 테이블에는 논리 메모리에 있는 Page 개수만큼 엔트리가 존재하고, 주소 변환 정보만 들어있다고 말했는데, 사실은 주소 변환 정보만 들어있는게 아니고 추가적인 1bit (Valid, Invalid)가 하나 더 저장되고 있다.

프로그램이 지금 사용하지 않는 페이지여도, 페이지 테이블은 모든 페이지에 대해서(자료 구조 특성상 인덱싱을 위해서) 엔트리를 가질 수 밖에 없다.

위 그림에서도 6, 7 Page는 존재하지 않지만, 주소 체계를 지키기 위해서 Page table에 6,7번이 존재하는 것을 볼 수 있다.

대신 사용이 되고있지 않기 때문에 valid, invalid bit을 통해 표시를 해놓고 있다.

사실은 valid, invalid bit에서 valid 라는 것은, 해당 Page가 물리적 메모리 Frame에 실제로 적재중이라는 이야기다.
반대로 invalid 라는 것은 해당 Page가 물리적 메모리 Frame에 존재하지 않는, 무의미한 엔트리임을 의미한다.
(아예 사용되지 않거나, 당장은 물리적인 메모리에 올릴 필요 없는 - 자주 사용되지 않는. HDD Backing store에 내려가있는)



### Page table의 각 엔트리마다 아래의 bit를 둔다

- Protection bit
  - page에 대한 접근 권한 (read/write/read-only)
  - 접근 권한이라해서 오해할 수 있는데, 애초에 page table은 자신(프로세스)의 주소만 담고 있기 때문에 애초에 다른 프로세스의 주소를 찾아가서 읽는다거나 등의 행위는 일어나지 않는다.
  - Protection의 의미는 Code / Data / Stack 영역에 대한 접근 권한이다.
    - Code 부분은 절대 바뀌면 안되는 영역이라 Read-only로 두고, Data-Stack 부분은 read/write 권한을 모두 주는 등. 그런걸 표기하기 위한 bit다.
- Valid / Invalid bit
  - valid : 해당 주소의 frame에 그 프로세스를 구성하는 유용한 내용이 있음 - 접근 허용
  - invalid : 해당 주소의 frame에 유효한 내용이 없음 - 접근 불허
    - 프로세스가 그 주소 부분을 사용하지 않거나, 해당 페이지가 swap area(HDD backing store)에 있는 경우



## Inverted Page Table

- Page table이 매우 큰 이유
  - 모든 프로세스별로 그 논리주소에 대응하는 모든 페이지에 대한 페이지 엔트리가 존재
  - 대응하는 페이지가 메모리에 있든 아니든 간에 페이지 테이블에는 엔트리로 존재해야한다 (인덱싱)

<img width="1083" alt="image-20210519151907989" src="https://user-images.githubusercontent.com/37354145/120880606-88134600-c606-11eb-8f50-050e73cde06d.png">

원래는 프로세스마다 존재하는 페이지 테이블이었는데, 시스템 안에 딱 1개의 페이지 테이블만 존재하도록 한 것.  
inverted page table의 엔트리는 프로세스의 page 개수만큼이 아니라 물리적 메모리의 frame 개수만큼 존재한다.  

원래 기존 page table은 인덱싱을 통해 위치를 찾아냈는데, inverted page table은 그게 불가능하다.  
일단 물리 메모리에 찾아가서 원하는 프로세스 page table의 논리 주소를 찾아내는 것이다.  
그래서 inverted page table 전체를 검색해야만 원하는 frame 주소로 접근할 수 있다.

그렇다면 이런걸 왜쓰나? -> 결국 page table을 위한 공간을 절약하자는 취지다.  
대신 공간은 아꼈지만 시간에 대한 overhead가 발생하게 된다.

inverted page table의 엔트리에는 process id도 포함된다! (누구의 논리 주소인지 알아야하기 때문에)  
(즉 위 그림논리주소에서 PID는 고유한 번호지만, P는 동일한 번호가 여러개 존재할 수도 있다.)

- Inverted page table
  - Page frame 하나당 page table 하나의 엔트리를 둔 것 (System-wide)
  - 각 page table entry는 각각의 물리적 메모리의 page frame이 담고 있는 내용 표시 (process-id, logical address of process)
  - 단점
    - 테이블 전체를 탐색해야함
    - TLB처럼 associative register를 사용해서 단점 해결 - expensive



## Shared Page

다른 프로세스들과 공유하는 페이지.

![image-20210519152718731](https://media.vlpt.us/images/infoqoch/post/88e1ae56-2e02-4d0e-a01d-df51ab6ce39f/image.png)

P1, P2, P3가 서로 같은 Code 영역을 가지고 (Data, Stack만 달라지고) 프로그램을 수행한다.

- Shared code
  - Re-entrant Code (=Pure Code)
  - **Read-only**로 하여 프로세스 간 하나의 code만 메모리에 올림
    (텍스트 에디터, 컴파일러, 윈도우 시스템 등)
  - Shared code는 **모든 프로세스의 logical address space에서 동일한 위치**에 있어야한다
  - Read-only와 동일한 logcial address에 위치하지 않는다면 Shared code가 될 수 없다.
- Private code and data
  - 각 프로세스들이 독자적으로 메모리에 올린다
  - private data는 logcial address space 어떤 곳에 와도 무방하지

> 헷갈리지 말 것. IPC(Inter Process Communication)에서 사용하는 Shared-Memory와는 다른 개념이다.
>
> 기법은 동일하지만 용도가 다르다. IPC SharedMemory는 READ/WRITE도 모두 가능하다.

여기까지가 Page 기법에 대한 소개였다!

<br>

## Segmentation

- 프로그램은 의미 단위인 여러 개의 segment 로 구성
  - 작게는 프로그램으 구성하는 함수(메서드) 하나하나를 세그먼트로 정의
  - 크게는 프로그램 전체를 하나의 세그먼트로 정의 가능
  - 일반적으로는 code, data, stack 부분이 하나씩 세그먼트로 정의된다
- Segment는 다음과 같은 logical unit들임.
  - Main()
  - function
  - global variables
  - stack
  - symbol table, arrays

## Segmentation / 주소 변환

- logical address는 <Segment-number / offset > 2가지로 구성된다
- **Segment table**
  - each table enry has
    - base - starting physical address of the segment
    - limit - length of the segment
- **Segment-table base register (STBR)**
  - 물리적 메모리에서의 segment table의 위치
- **Segment-table length register (STLR)**
  - 프로그램이 사용하는 segment의 수(길이)
    - segment number 's' is legal if s < STLR.

<img width="941" alt="image-20210519155724123" src="https://user-images.githubusercontent.com/37354145/120880610-89447300-c606-11eb-8d7f-cdd4f9e15df6.png">


Segment table을 살펴보면 page table과 다르게 limit이라는 정보를 하나 더 갖고 있는데, segment 자체가 의미를 단위로 자르기 때문에 page와 달리 길이가 균일하지 않다. 그래서 현재 segment의 길이가 어느정도인지 같이 저장해두고 있어야 한다.

(그림에는 표시되어있지 않지만) 우선 CPU가 던진 s/d 논리 주소에서 segment 번호를 먼저 확인한다. 만일 프로그램이 전체 가지고 있는 segment 개수(STLR)보다 큰 번호일 경우 trap에 빠진다. 그리고 segment의 길이보다 큰 길이의 offset을 요청하는 경우에도 trap에 빠뜨린다.

2가지 경우를 모두 통과한 경우 base 위치만큼 떨어진 곳에 시작 위치를 잡고, offset만큼의 떨어진 위치로 한 번 더 이동해서 원하는 내용의 주소에 접근한다.

페이징 기법은 어차피 페이지의 크기가 균일하기 때문에, offset의 크기가 page의 크기를 따른다. 그러나 segment 기법은 segment 마다의 offset 크기가 다르다. 그러기 때문에 이전에 살펴본 10/10/12bit 등으로 크기가 잘리지 못하고 offset의 크기만큼 bit가 미리 결정된 후 나머지 남는 bit만큼 segment가 잡힌다. 즉, offset으로 표현하는 bit가 많을수록 segment bit이 적어진다.

페이징 기법에서는 시작주소가 frame 번호로 시작할 수 있었다. 물리적인 메모리도 같은 크기의 frame으로 나누어졌기 대문에. 그러나 segment에서는 그렇게 될 수 없다. 그래서 물리적인 메모리 주소가 정확한 Byte 단위의 주소로 나뉘어져야만 한다. 

그리고 Page 기법을 사용했던 이유 중 하나인, 가변적인 메모리 적재 정보들의 크기로 인해 사용되지 못하는 Hole들이 생성되게 된다. 그게 Segmentation 기법의 단점이다.


- Protection
  - 각 세그먼트 별로 protection bit가 있다
  - Each entry:
    - Valid bit = 0 => illegal segment
    - READ/WRITE/EXECUTE 권한 bit
- Sharing
  - shared segment - 의미 단위로 쪼개기
    Paging 기법과 다르게 (테이블 하나당 엔트리 100만개) 테이블 하나당 개수가 5개? 이런식으로 굉장히 적다.
  - same segment number - 같은 의미 번호
  - segment는 의미 단위기 때문에 공유(Sharing)과 보안(protection)에 있어 Paging기법보다 효과적이다
- Allocation
  - first fit / best fit을 써야한다 (워스트는 ㄲㅈ)
  - External fragmentation 발생
  - 길이가 동일하지 않으므로 가변분할 방식과 동일한 문제점들이 모두 발생한다

Table을 위한 메모리 낭비가 심한 쪽 - Paging. 이것도 같이 생각해보면 좋다.



## Sharing of segments
<img width="949" alt="image-20210519155913106" src="https://user-images.githubusercontent.com/37354145/120880611-89dd0980-c606-11eb-8849-14ade322b33b.png">

세그먼트를 서로다른 2개의 프로세스가 공유하는 그림

editor - Code 영역에 해당되는 부분. 공유 세그먼트로 사용된다. 0번 세그먼트로 같은 logical 주소를 사용한다는 것을 집중해서 보자. 실제 물리 주소와 길이도 동일하다.



세그먼트 끝.

<br>

---

<br>

## Segmentation with Paging (Paged Segmentation)

세그먼트를 여러 개의 Page로 구성하는 기법. 여러개의 Page로 쪼개진 세그먼트가 메모리에 올라간다.

<img width="931" alt="image-20210519160143357" src="https://user-images.githubusercontent.com/37354145/120880613-8a75a000-c606-11eb-92e9-eb62dcd7884a.png">

세그먼트 하나가 여러개의 페이지로 구성되어 있기 때문에, 먼저 Segment에 대한 주소 변환을 진행한다.

각 프로그램이 가지고 있는 논리 주소는 세그먼트 번호와 세그먼트 내부에서 얼마나 떨어져있는지 나타내는 Offset으로 구성.

Segment-table을 찾아서 Segment 하나당 존재하는 Page table을 찾아낸다.
(기존 Paging 기법에서는 프로세스당 이었는데, 여기서는 segment 하나당 page table 하나.)

그렇다면 Page table의 엔트리 개수는? segment length(세그먼트 하나의 크기, 길이)가 된다! 

예외처리는? 세그먼트의 길이보다 offset 값이 더 크다면 잘못된 주소로 접근(페이지 테이블 엔트리 개수를 넘기는 접근)이므로, 예외처리!



장점 - Allocation 문제가 안생긴다.

단점 - 공유/보안 등의 문제를 Segment-table 레벨에서 진행해야한다.

즉, Segment 단위 장점은 Segment-table 레벨에서 챙기고, Segment 단위 단점은 실제 메모리에 Page 단위로 올리면서 해결하려고 시도. 2가지 기법의 장점을 모두 누릴 수 있다.

오리지널 Segmentation 기법을 사용하는 시스템은 사실 거의 존재하지 않는다.

---

여기까지가 Memory Management의 모든 내용이다.

모두 정리한 후, OS가 어떻게 여기 개입하는지 느낀 바가 있는가? 여기까지 모든 내용중에 OS가 개입한 것은 거의 없다.

그렇다. 여기까지 내용은 모두 MMU가 어떻게 메모리 주소를 변환하고 관리해주는지에 대한 내용.
즉, 하드웨어적인 메모리 관리 기법이었다.

운영체제가 개입하게 되면, CPU권한 할당을 매 클럭마다 `process->os->process->os ...` 가 반복되어야 한다.

주소 변환은 무조건 하드웨어적으로 이루어진다. 운영체제가 끼어들어야하는 영역은? I/O 장치를 건드리는 순간. 사용자 프로세스는 I/O장치에 접근 못하기 때문에. 운영체제한테 I/O 작업을 도와달라고 해야한다.

Virtual Memory 부터 운영체제가 중요한 역할을 하게 된다.
