# State (상태)
- 객체의 내부 사태에 따라 스스로 행동을 변경하는 패턴
- 각 상태에 대응하는 별도의 클래스를 만들고 상태 전이 로직들을 해당 클래스에 이임
- 원래의 호스트 객체 : Context, Context 객체는 상태와 관련된 기능을 스테이트 객체에 위임
- 상태 전이는 Context 객체의 State 객체에서 다른 State 객체로 변경
- 상태를 더 늘리거나 상태 전이에 대한 로직의 수정 가능성이 있을때 적용
- 상태 전이를 위한 조건 로직이 복잡한 경우 이를 해소하기 위해 적용
![image](https://johngrib.github.io/wiki/pattern/state/state.svg)

```Java
import java.util.LinkedList;
import java.util.List;

class Product {
    private String name;
    public static int PRICE = 500;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

interface State {
    int 잔돈반환();

    void 돈투입(int money);

    Product 상품반환();
}

class 판매중 implements State {
    VendingMachine vendingMachine;

    public 판매중(VendingMachine vendingMachine) {
        this.vendingMachine = vendingMachine;
        System.out.println("@@@ 판매중 @@@ ");
    }

    public int 잔돈반환() {
        return vendingMachine.getNowMoney();
    }

    public void 돈투입(int money) {
        System.out.println("=== 돈 " + money + " 원 투입 ===");
        int nowMoney = vendingMachine.getNowMoney();
        vendingMachine.setNowMoney(nowMoney + money);
        if (vendingMachine.getNowMoney() >= Product.PRICE) {
            vendingMachine.state = new 구매가능(vendingMachine);
        }
    }

    public Product 상품반환() {
        System.out.println("=== 현재 금액 : " + vendingMachine.getNowMoney() + " / 상품 가격 : " + Product.PRICE + " ===");
        System.out.println("=== 투입금액이 부족합니다===");
        return null;
    }
}

class 구매가능 implements State {
    VendingMachine vendingMachine;

    public 구매가능(VendingMachine vendingMachine) {
        this.vendingMachine = vendingMachine;
        System.out.println("@@@ 구매 가능 @@@ ");
    }

    public int 잔돈반환() {
        return vendingMachine.getNowMoney();
    }

    public void 돈투입(int money) {
        System.out.println("=== 돈 " + money + " 원 투입 ===");
        int nowMoney = vendingMachine.getNowMoney();
        vendingMachine.setNowMoney(nowMoney + money);
        if (vendingMachine.getNowMoney() >= Product.PRICE) {
            vendingMachine.state = new 구매가능(vendingMachine);
        }
    }

    public Product 상품반환() {
        System.out.println("=== 현재 금액 : " + vendingMachine.getNowMoney() + " / 상품 가격 : " + Product.PRICE + " ===");
        if (vendingMachine.getNowMoney() >= Product.PRICE) {
            vendingMachine.setNowMoney(vendingMachine.getNowMoney() - Product.PRICE);
            vendingMachine.setBalance(vendingMachine.getBalance() + Product.PRICE);

            if (vendingMachine.getProductCount() >= 1) {
                System.out.println("=== 상품을 반환합니다 === ");
                vendingMachine.setProductCount(vendingMachine.getProductCount() - 1);
                if (vendingMachine.getProductCount() == 0) {
                    vendingMachine.state = new 상품매진(vendingMachine);
                }
                return new Product();
            }
        }
        System.out.println("=== 투입금액이 부족합니다 ===");
        return null;
    }
}

class 상품매진 implements State {
    VendingMachine vendingMachine;

    public 상품매진(VendingMachine vendingMachine) {
        this.vendingMachine = vendingMachine;
        System.out.println("@@@ 상품 매진 @@@ ");
    }

    public int 잔돈반환() {
        int nowMoney = vendingMachine.getNowMoney();
        vendingMachine.setNowMoney(0);
        return nowMoney;
    }

    public void 돈투입(int money) {
        System.out.println("=== 돈 " + money + " 원 투입 ===");
        vendingMachine.setNowMoney(money);
    }

    public Product 상품반환() {
        System.out.println("=== 상품이 존재하지 않습니다 ===");
        return null;
    }
}

class VendingMachine {
    public State state = new 판매중(this);

    private int productCount = 10;
    private int nowMoney = 0;
    private int balance = 0;

    public int getNowMoney() {
        return nowMoney;
    }

    public void setNowMoney(int money) {
        this.nowMoney = money;
    }

    public int getBalance() {
        return balance;
    }

    public void setBalance(int balance) {
        this.balance = balance;
    }

    public int getProductCount() {
        return this.productCount;
    }

    public void setProductCount(int count) {
        this.productCount = count;
    }

    public void 돈투입(int money) {
        state.돈투입(money);
    }

    public Product 상품반환() {
        return state.상품반환();
    }

    public int 잔돈반환() {
        return state.잔돈반환();
    }
}

public class Main {
    public static void main(String[] args) {
        VendingMachine vendingMachine = new VendingMachine();
        vendingMachine.돈투입(600);
        vendingMachine.상품반환();
        vendingMachine.잔돈반환();
    }
}
```