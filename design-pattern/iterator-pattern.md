# 디자인 패턴 - 반복자 패턴
- 결국 Iterator override 해서 Array 든 Collection 이든 같은 형태로 사용할 수 있도록 하자.
- 근데, Kotlin 을 이용하면 그 모든게 해결되어 있다.

## Java 코드로 List 와 Array 를 다룰 때
### 선언부
```java
public class CoffeeMenu {

    private final List<MenuItem> menuItems;

    public CoffeeMenu() {
        menuItems = List.of(  
			new MenuItem("에스프레소", "뒤지게 씀", BigDecimal.valueOf(3_000)),  
			new MenuItem("라떼", "는 말이야", BigDecimal.valueOf(5_000)),  
			new MenuItem("카푸치노", "거품이라는 뜻", BigDecimal.valueOf(500)),  
			new MenuItem("모카", "멍멍", BigDecimal.valueOf(1_000_000))  
		);
    }

    public List<MenuItem> getMenuItems() {
        return menuItems;
    }
}
```

```java
public class AlcoholMenu {

    private static final int MAX_ITEMS = 4;

    private final MenuItem[] menuItems;

    public AlcoholMenu() {
        menuItems = new MenuItem[MAX_ITEMS];

        menuItems[0] = new MenuItem(
            "소주", "한국의 전통 술",
            BigDecimal.valueOf(6_000)
        );

        menuItems[1] = new MenuItem(
            "맥주", "시원한 탄산 음료",
            BigDecimal.valueOf(8_000)
        );

        menuItems[2] = new MenuItem(
            "와인", "포도로 만든 술",
            BigDecimal.valueOf(30_000)
        );

        menuItems[3] = new MenuItem(
            "위스키", "오크통에서 숙성된 증류주",
            BigDecimal.valueOf(300_000)
        );
    }

    public MenuItem[] getMenuItems() {
        return menuItems;
    }
}
```

### List 와 Array 를 사용할 때
```java
public static void printWithIndex() {
	System.out.println("커피 메뉴");
	CoffeeMenu coffeeMenu = new CoffeeMenu();
	for (int i = 0; i < coffeeMenu.getMenuItems().size(); i++) {
		MenuItem menuItem = coffeeMenu.getMenuItems().get(i);
		System.out.println(
			menuItem.getName() + " - " +
			menuItem.getDescription() + ": " +
			menuItem.getPrice()
		);
	}

	System.out.println("술 메뉴");
	AlcoholMenu alcoholMenu = new AlcoholMenu();
	for (int i = 0; i < alcoholMenu.getMenuItems().length; i++) {
		MenuItem menuItem = alcoholMenu.getMenuItems()[i];
		System.out.println(
			menuItem.getName() + " - " +
			menuItem.getDescription() + ": " +
			menuItem.getPrice()
		);
	}
}
```

- 결국 같은 `MenuItem` 원소를 이용하지만, 내부 집합체를 무얼 사용하느냐에 따라 외부 사용법에 영향을 준다.
- 어떤 구현 방식을 이용했건 사용처에서는 영향을 받지 않도록 할 방법이 없을까?
	- 반복을 캡슐화하면 된다. `Iterator` 로!

### Iterator 로 추상화
```java
public class CoffeeMenuIterator implements Iterator<MenuItem> {  
  
	private final List<MenuItem> menuItems;  
	private int position = 0;  
	  
	public CoffeeMenuIterator(List<MenuItem> menuItems) {  
		this.menuItems = menuItems;  
	}  
	  
	@Override  
	public boolean hasNext() {  
		return position < menuItems.size();  
	}  
	  
	@Override  
	public MenuItem next() {  
		if (!hasNext()) {  
			throw new NoSuchElementException();  
		}  
		return menuItems.get(position++);  
	}  
}
```

```java
public final class CoffeeMenu {  
  
	private final List<MenuItem> menuItems;  
  
	public CoffeeMenu() {  
		menuItems = List.of(  
			new MenuItem("에스프레소", "뒤지게 씀", BigDecimal.valueOf(3_000)),  
			new MenuItem("라떼", "는 말이야", BigDecimal.valueOf(5_000)),  
			new MenuItem("카푸치노", "거품이라는 뜻", BigDecimal.valueOf(500)),  
			new MenuItem("모카", "멍멍", BigDecimal.valueOf(1_000_000))  
		);
	}  
  
	public Iterator<MenuItem> getIterator() {  
		return new CoffeeMenuIterator(menuItems);  
	}  
}
```

```java
public class AlcoholMenu {

    private static final int MAX_ITEMS = 4;

    private final MenuItem[] menuItems;

    public AlcoholMenu() {
        menuItems = new MenuItem[MAX_ITEMS];

        menuItems[0] = new MenuItem(
            "소주", "한국의 전통 술",
            BigDecimal.valueOf(6_000)
        );

        menuItems[1] = new MenuItem(
            "맥주", "시원한 탄산 음료",
            BigDecimal.valueOf(8_000)
        );

        menuItems[2] = new MenuItem(
            "와인", "포도로 만든 술",
            BigDecimal.valueOf(30_000)
        );

        menuItems[3] = new MenuItem(
            "위스키", "오크통에서 숙성된 증류주",
            BigDecimal.valueOf(300_000)
        );
    }

    public Iterator<MenuItem> getIterator() {  
		return new AlcoholMenuIterator(menuItems);  
	}
}
```

```java
import java.util.Iterator;

public final class MenuPrinter {

    public static void printMenus() {
        CoffeeMenu coffeeMenu = new CoffeeMenu();
        AlcoholMenu alcoholMenu = new AlcoholMenu();

        System.out.println("커피 메뉴");
        printMenu(coffeeMenu.getIterator());

        System.out.println("술 메뉴");
        printMenu(alcoholMenu.getIterator());
    }

    private static void printMenu(Iterator<MenuItem> iterator) {
        while (iterator.hasNext()) {
            MenuItem menuItem = iterator.next();
            System.out.println(
                menuItem.getName() + " - " +
                menuItem.getDescription() + ": " +
                menuItem.getPrice()
            );
        }
    }
}
```


## 이걸 다시 Kotlin 코드로 구현해보면 어떻게 될까?
```kotlin
class CoffeeMenu {  
    val menuItems: List<MenuItem> = listOf(  
        MenuItem(  
            name = "에스프레소",  
            description = "뒤지게 씀",  
            price = 3_000.toBigDecimal(),  
        ),  
        MenuItem(  
            name = "라떼",  
            description = "는 말이야",  
            price = 5_000.toBigDecimal(),  
        ),  
        MenuItem(  
            name = "카푸치노",  
            description = "거품이라는 뜻",  
            price = 5_00.toBigDecimal()  
        ),  
        MenuItem(  
            name = "모카",  
            description = "멍멍",  
            price = 1_000_000.toBigDecimal(),  
        ),  
    )  
}
```

```kotlin
class AlcoholMenu {  
    val menuItems: Array<MenuItem> = arrayOf(  
        MenuItem(  
            name = "소주",  
            description = "한국의 전통 술",  
            price = 6_000.toBigDecimal(),  
        ),  
        MenuItem(  
            name = "맥주",  
            description = "시원한 탄산 음료",  
            price = 8_000.toBigDecimal(),  
        ),  
        MenuItem(  
            name = "와인",  
            description = "포도로 만든 술",  
            price = 30_000.toBigDecimal(),  
        ),  
        MenuItem(  
            name = "위스키",  
            description = "오크통에서 숙성된 증류주",  
            price = 300_000.toBigDecimal(),  
        ),  
    )  
}
```

```kotlin
/**  
 * Kotlin 의 위대함을 맛보아라. */
fun `인덱스 활용`() {  
    val coffeeMenu = CoffeeMenu()  
    val alcoholMenu = AlcoholMenu()  
  
    println("커피 메뉴")  
    for (i in coffeeMenu.menuItems.indices) {  
        val menuItem = coffeeMenu.menuItems[i]  
        println("${menuItem.name} - ${menuItem.description}: ${menuItem.price}")  
    }  
  
    println("술 메뉴")  
    for (i in alcoholMenu.menuItems.indices) {  
        val menuItem = alcoholMenu.menuItems[i]  
        println("${menuItem.name} - ${menuItem.description}: ${menuItem.price}")  
    }  
}  
  
fun `iterator 활용`() {  
    val coffeeMenu = CoffeeMenu()  
    val alcoholMenu = AlcoholMenu()  
  
    println("커피 메뉴")  
    for (menuItem in coffeeMenu.menuItems) {  
        println("${menuItem.name} - ${menuItem.description}: ${menuItem.price}")  
    }  
  
    println("술 메뉴")  
    for (menuItem in alcoholMenu.menuItems) {  
        println("${menuItem.name} - ${menuItem.description}: ${menuItem.price}")  
    }  
}
```

- 알아서 다 된다.
- java byte code 로 변환하면 내부에서 어떻게 되고 있을까?
	- IntelliJ `Tools`-`Kotlin`-`Show Kotlin Bytecode` 메뉴를 이용해보자.