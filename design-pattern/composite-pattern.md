# 디자인 패턴 - 복합체 패턴
- composite 와 leaf 를 동일하게 취급하여, 사용자에게 이 둘을 구분하지 않고 동일하게 이용 가능하도록 하는 구조 패턴
	- 더 쉽게 말하면 구현체 자체는 계층이 존재하나, 사용자는 이를 구분하지 못하도록 추상화함
	- 예를 들면 사용자는 현재 파일이 디렉토린지 텍스트 파일인지 구분하지 못하고 '열기', '복사'를 진행할 수 있음.

## 새로운 메뉴를 계속 추가하고 싶을 때
### 선언부
```kotlin
class CoffeeMenu {
    private val menuItems = listOf(
        MenuItem("에스프레소", "뒤지게 씀",   3_000.toBigDecimal()),
        MenuItem("라떼",       "는 말이야",   5_000.toBigDecimal()),
        MenuItem("카푸치노",   "거품이라는 뜻", 500.toBigDecimal()),
        MenuItem("모카",       "멍멍",     1_000_000.toBigDecimal()),
    )

    fun getIterator(): Iterator<MenuItem> = menuItems.iterator()
}
```

```kotlin
class AlcoholMenu {
    private val menuItems = arrayOfNulls<MenuItem>(4).also {
        it[0] = MenuItem("소주",   "한국의 전통 술",      6_000.toBigDecimal())
        it[1] = MenuItem("맥주",   "시원한 탄산 음료",    8_000.toBigDecimal())
        it[2] = MenuItem("와인",   "포도로 만든 술",     30_000.toBigDecimal())
        it[3] = MenuItem("위스키", "오크통에서 숙성된 증류주", 300_000.toBigDecimal())
    }

    fun getIterator(): Iterator<MenuItem> = menuItems.filterNotNull().iterator()
}
```

### 사용처
```kotlin
class Waitress(
    private val coffeeMenu: CoffeeMenu,   // ← CoffeeMenu 를 직접 알아야 함
    private val alcoholMenu: AlcoholMenu, // ← AlcoholMenu 도 직접 알아야 함
) {
    fun printMenu() {
        coffeeMenu.getIterator().forEach { it.print() }
        alcoholMenu.getIterator().forEach { it.print() }
    }
}

```
- 새로운 메뉴(`양꼬치`, `회` ...)가 추가될 때마다 `Waitress`도 같이 수정해야 한다.
- "알콜 메뉴" 가 아닌 "소주" 만 출력하고 싶어도 내부 구현체의 정보를 모두 인지해야한다.
- '어차피 다 같은 메뉴인데?'


## 복합체 패턴으로 합쳐주면?
```kotlin
interface MenuComponent {
    val name: String
    val description: String
    val price: BigDecimal get() = throw UnsupportedOperationException()

    fun print()

    fun add(component: MenuComponent): Unit = throw UnsupportedOperationException()  
fun remove(component: MenuComponent): Unit = throw UnsupportedOperationException()
}
```

```kotlin
class MenuItem(
    override val name: String,
    override val description: String,
    override val price: BigDecimal,
) : MenuComponent {

    override fun print() {
        // ...
    }
}
```

```kotlin
class Menu(
    override val name: String,
    override val description: String,
) : MenuComponent {

    private val values = mutableListOf<MenuComponent>()

    override fun add(component: MenuComponent) { 
	    values.add(component) 
	}
    
    override fun remove(component: MenuComponent) { 
	    values.remove(component)
	}

    override fun print() {
        // ...
    }
}
```

```kotlin
fun main() {  
    val allMenus = Menu("전체 메뉴", "오늘의 메뉴판")  
  
    val coffeeMenu = Menu("커피", "따뜻하고 차갑고")  
    coffeeMenu.add(MenuItem("에스프레소", "뒤지게 씀", 3_000.toBigDecimal()))  
    coffeeMenu.add(MenuItem("라떼", "는 말이야", 5_000.toBigDecimal()))  
    coffeeMenu.add(MenuItem("카푸치노", "거품이라는 뜻", 500.toBigDecimal()))  
    coffeeMenu.add(MenuItem("모카", "멍멍", 1_000_000.toBigDecimal()))  
  
    val alcoholMenu = Menu("주류", "어른의 음료")  
    alcoholMenu.add(MenuItem("소주", "한국의 전통 술", 6_000.toBigDecimal()))  
    alcoholMenu.add(MenuItem("맥주", "시원한 탄산 음료", 8_000.toBigDecimal()))  
    alcoholMenu.add(MenuItem("와인", "포도로 만든 술", 30_000.toBigDecimal()))  
    alcoholMenu.add(MenuItem("위스키", "오크통 숙성 증류주", 300_000.toBigDecimal()))  
  
    val beerMenuTap = Menu("생맥주 종류", "탭 리스트")  
    beerMenuTap.add(MenuItem("카스", "국민 맥주", 5_000.toBigDecimal()))  
    beerMenuTap.add(MenuItem("기네스", "흑맥주 대장", 9_000.toBigDecimal()))  
  
    alcoholMenu.add(beerMenuTap)  
  
    allMenus.add(coffeeMenu)  
    allMenus.add(alcoholMenu)  
  
    allMenus.print()  
}
```

**패턴 장점**

- 범용성 - 다형성 재귀를 통해 복잡한 트리 구조를 보다 편리하게 구성 할 수 있다. 
- 수평적, 수직적 모든 방향으로 객체를 확장할 수 있다.
- 새로운 Leaf 클래스를 추가하더라도 클라이언트는 추상화된 인터페이스 만을 바라봐서 영향 없음

**패턴 단점**

- 범용성 - 추후 관리가 어려움
- 디버깅이 어려움
