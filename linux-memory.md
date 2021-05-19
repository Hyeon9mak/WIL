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

![image-20210517123228246](/Users/hyeon9mak/Library/Application Support/typora-user-images/image-20210517123228246.png)

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

![image-20210517125630622](/Users/hyeon9mak/Library/Application Support/typora-user-images/image-20210517125630622.png)

기본적인 MMU에서는 register 2개를 통해서 주소 변환을 수행한다.

### Relocation register (=base register)
CPU가 논리적 주소로 process의 정보를 요청하면, relocation register가 가지고 있던 **"메모리에 올라가있는 프로세스의 시작위치"** 를 더해서 실제 물리적 주소를 만들어낸다. 그 주소로 찾아가서 내용을 읽어서 CPU에게 가져다 준다.

### Limit register
이 프로그램의 최대 크기가 몇인지 (마지막 주소가 어디인지)를 담고 있다. 

이 프로그램이 만약 악의적인 프로그램이라서, 본인의 크기가 3000인데도 불구하고 CPU에게 메모리 4000번지를 달라고 하게끔 유도할 수도 있다. 그렇게 되면 다른 프로그램에 존재하는 메모리 위치를 요청하게 된다. 남의 프로그램 정보를 보려고 하면 안된다. 그래서 base register 주소를 더해주기 전에 limit register로 정보 체크를 먼저한다.

![image-20210517134000090](/Users/hyeon9mak/Library/Application Support/typora-user-images/image-20210517134000090.png)

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


![image-20210517135512261](/Users/hyeon9mak/Library/Application Support/typora-user-images/image-20210517135512261.png)

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

![image-20210517142539843](/Users/hyeon9mak/Library/Application Support/typora-user-images/image-20210517142539843.png)

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

  ![image-20210517143054940](/Users/hyeon9mak/Library/Application Support/typora-user-images/image-20210517143054940.png)

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
