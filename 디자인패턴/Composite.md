# Composite (컴포지트)
- 객체를 트리구조로 구성해서 부분-전체의 계층 구조를 표현
- 개별 객체와 복합 객체를 똑같은 방법으로 다룸
- 객체 그룹을 조작하는 것처럼 트리구조를 통해 단일 객체를 조작함
- 전체-부분(whole-part)관계를 표현 ex) 파일구조, 메뉴

![image1](https://t1.daumcdn.net/cfile/tistory/99E9FF455C84AF1E20)

``` Java
public abstract class MenuComponent{
    public void add(MenuComponent menuComponent){
        throw new UnsupportedOperationException();
    }

    public void remove(MenuComponent menuComponent){
        throw new UnsupportedOperationException();
    }

    public MenuComponent getChild(int i){
        throw new UnsupportedOperationException();
    }

    public String getName(){
        throw new UnsupportedOperationException();
    }

    public String getDescription(){
        throw new UnsupportedOperationException();
    }

    public double getPrice(){
        throw new UnsupportedOperationException();
    }

    public boolean isVegetarian(){
        throw new UnsupportedOperationException();
    }

    public void print(){
        throw new UnsupportedOperationException();
    }
}
```
인터페이스가 될 MenuComponent

``` Java
class MenuItem extends MenuComponent {
    String name;
    String description;
    boolean vegetarian;
    double price;

    public MenuItem(String name, String description, boolean vegetarian, double price) {
        this.name = name;
        this.description = description;
        this.vegetarian = vegetarian;
        this.price = price;
    }

    public String getName() {
        return this.name;
    }

    public String getDescription() {
        return this.description;
    }

    @Override
    public double getPrice() {
        return price;
    }

    @Override
    public boolean isVegetarian() {
        return vegetarian;
    }

    @Override
    public void print() {
        System.out.println(getName());

        if (isVegetarian()) {
            System.out.println("(v)");
        }
        System.out.println(getPrice());
        System.out.println(getDescription());
    }
}
```
부분이 될 MenuItem

``` Java

class Menu extends MenuComponent{
    List<MenuComponent> menuComponents = new ArrayList<MenuComponent>();
    String name;
    String description;

    public Menu(String name, String description) {
        this.name = name;
        this.description = description;
    }

    public void add(MenuComponent menuComponent){
        menuComponents.add(menuComponent);
    }

    public void remove(MenuComponent menuComponent){
        menuComponents.remove(menuComponent);
    }

    public MenuComponent getChild(int i){
        return menuComponents.get(i);
    }


    public String getName() {
        return name;
    }

    public String getDescription(){
        return description;
    }

    public void print(){
        System.out.println(getName());
        System.out.println("," + getDescription());
        System.out.println("========================");

        for (MenuComponent menuComponent : menuComponents){
            menuComponent.print();
        }
    }
}
```
전체가 될 Menu

``` Java
public class Main {
    public static void main(String[] args) {
        MenuComponent brunchMenu = new Menu("OO식당 브런치 메뉴", "판매시간 10:00 ~ 14:00");
        MenuComponent dessertMenu = new Menu("OO식당 디저트 메뉴", "판매시간 : all");

        MenuComponent allMenus = new Menu("전체 메뉴", "전체 메뉴");

        allMenus.add(brunchMenu);
        allMenus.add(dessertMenu);

        brunchMenu.add(new MenuItem("파스타", "알리올리오 + 효모빵", true, 5.63));
        dessertMenu.add(new MenuItem("허니브레드 세트", "허니브레드 + 아메리카노", true, 4.5));

        allMenus.print();
    }
}
```
- 비슷한 성격의 객체를 모아서 사용할때 유용
- 코드를 사용하는 클라이언트 측에선 Leaf와 Composite를 일관되게 사용할 수 있음,
    but, Leaf 객체에서 자식을 다루는 메소드를 호출할 수 있어 타입의 안정성을 잃게 됨.
    안정성 보다는 일관성이 중요한 패턴 
