# Singleton (싱글톤)
- 인스턴스를 하나만 생성하여 사용하는 패턴, 생성자의 호출이 반복적으로 발생해도 실제로 생성된 객체는 최초 생성된 객체를 반환
- vs 전역변수 : 전역변수라면 Application 시작시 객체가 생성될 것이나 해당 객체가 사용되지 않을 때, 해당 객체는 자원만 차지하는 쓸모없는 객체가 되어버림
- 객체가 Application 내에서 여러 리소스간에서 공유되는 객체로 사용되는 경우 사용됨. 이때 이 객체는 Application에 단 하나만 존재함.
- ex) 스레드 풀 , 캐시 , 대화상자 , 사용자 설정 , 커넥션 풀 , 디바이스 드라이버 등 객체가 전체 프로그램에서 오직 하나만 생성되어야 하는 경우
- Spring 에서 Java Beans의 명시적인 scope에 대한 설정이 없는 경우 싱글톤 패턴을 적용하여 생성이 이뤄짐.

### 특징
- 생성자를 private 으로 가짐
- 객체를 반환하기 위한 getInstance 메소드를 static 으로 가짐.

```Java 
class SingleTon{
    public static SingleTon instance; // 하나뿐인 인스턴스
    private SingleTon(){}
    static SingleTon getInstance(){
        if(instance == null) { // instance가 null인 경우 아직 생성되지 않았음
            instance = new SingleTon();
            /*
            아직 생성되지 않은 경우 해당 메소드에서는 private 접근제한자에 접근이 가능하므로 생성자를 통해
            SingleTone 객체를 생성 후 멤버 변수인 instance에 대입해줌.
            필요한 상황까지 인스턴스를 생성하지 않게 됨 -> lazy init
            */
        }
        return instance; // null이 아니라면 이미 인스턴스가 생성되었으므로 해당 인스턴스를 반환
    }
}
```
### 정리
 - 생성자가 private이므로 객체 외부에서 해당 싱글톤 객체를 생성할 수가 없음
 - getInstance 메소드를 사용하여 객체를 생성하는데 이때 instance는 초기에 생성되지 않았다면 null로 존재함.
 - 반드시 생성자의 호출은 단 한 번 발생해야 함

### 초콜릿 보일러
``` Java
public class ChocolateBoiler{
    private boolean empty;
    private boolean boiled;

    public ChocolateBoiler(){
        this.empty = true;
        this.boiled = false;
    }

    public void fill(){ // 보일러가 비어있을 때만 재료를 넣음
        if(isEmpty()){
            this.empty = false;
            this.boiled = false;
        }
    }

    public void boil(){ // 재료를 끓여냄
        if(!isEmpty() && !isBoiled()){
            this.boiled = true;
        }
    }

    public void drain(){ // 끓인 재료를 비워냄
        if(!isEmpty() && isBoiled()){
            this.empty = true;
        }
    }

    public boolean isEmpty(){
        return this.empty;
    }

    public boolean isBoiled(){
        return this.boiled;
    }
}
```
해당 코드는 메소드를 호출하기전 해당 작업이 수행가능한지 체크하는 로직이 존재하여 상황을 대비한 코드가 작성되어 있음<br>
but, 런타임에서 2개이상의 인스턴스가 존재한다면 동작이 예상한대로 돌아가지 않음
-> 싱글톤 적용

``` Java
public class ChocolateBoiler{
    // private static 으로 선언된 instance 추가
    private static ChocolateBoiler instance;
    private boolean empty;
    private boolean boiled;

    // 생성자 private 제한
    private ChocolateBoiler(){
        this.empty = true;
        this.boiled = false;
    }

    // getInstacne 추가
    public static ChocolateBoiler getInstance(){
        if(instance == null){
            instance = new ChocolateBoiler();
        }
        return instance;
    }

    public void fill(){ // 보일러가 비어있을 때만 재료를 넣음
        if(isEmpty()){
            this.empty = false;
            this.boiled = false;
        }
    }

    public void boil(){ // 재료를 끓여냄
        if(!isEmpty() && !isBoiled()){
            this.boiled = true;
        }
    }

    public void drain(){ // 끓인 재료를 비워냄
        if(!isEmpty() && isBoiled()){
            this.empty = true;
        }
    }

    public boolean isEmpty(){
        return this.empty;
    }

    public boolean isBoiled(){
        return this.boiled;
    }
}

```

문제점 : 멀티스레딩 문제 발생<br>

|1번 스레드|2번 스레드|instance 값|
|---|---|---|
|getInstance 호출|-|null|
|-|getInstance 호출|null|
|if(instance==null)|-|null|
|-|if(instance==null)|null|
|instance = new ChocolateBoiler();|-|\<object1\>|
|return instance;|-|\<object1\>|
|-|instance = new ChocolateBoiler();|\<object2\>|
|-|return instance;|\<object2\>|

### 해결방법 1.synchronized를 통한 동기화
``` Java
...
public static synchronized SingleTon getInstance(){
...
}
```
동기화로 인한 성능 이슈 발생

### 해결방법 2.인스턴스가 필요할 때 생성이 아닌 초기부터 생성

```Java
public class SingleTon {
    private static SingleTon instance = new SingleTon();

    private SingleTon(){};

    public static SingleTon getInstance(){
        return instance;
    }
}
```
초기 런타임시 JVM에서 클래스가 로딩시 해당 instance 생성 후 해당 객체를 return 하는 방법

### 해결방법 3.DCL(Double Checked Locking) 사용
```Java
public class SingleTon {
    private volatile static SingleTon instance = new SingleTon(); // 변수를 Main Memory에 저장

    private SingleTon(){};

    public static SingleTon getInstance(){
        if(instance == null){
            synchronized (SingleTon.class){ // SingleTon 클래스에 대한 동기화 블록
                if(instance == null){ // 한번 더 null인지를 체크하고 인스턴스 생성
                    instance = new SingleTon();
                }
            }
        }
        return instance;
    }
}
```
1.4이하에서는 사용 불가능한 문법

## 고민해볼것

### 1. 모든 메서드와 변수가 static인 클래스와의 싱글톤의 차이점
- 결과적으로는 같음
- 정적 초기화 순서로 인한 문제가 발생할 수 있음

### 2. 클래스 로더와 관련한 문제점?
- 클래스 로더마다 서로 다른 네임스페이스를 정의하여 클래스 로더가 2개 이상인 경우 같은 클래스에 여러번 로딩하여 인스턴스가 여러 개 생성되는 문제가 발생할 수 있음.

### 3. 전역 변수 vs 싱글톤
- 전역 변수는 Lazy init을 통한 인스턴스 생성이 불가능함 + 처음부터 끝까지 인스턴스를 생성된 채로 가지고 있어야 함.
- 싱글톤은 Lazy init이 가능함 + getInstance()가 호출되는 경우 인스턴스를 생성
- 전역 변수로 객체의 전역 레퍼런스를 계속 만들게되어 네임스페이스가 지저분해짐
- 싱글톤은 남용될 순 있으나 싱글톤은 그럴일이 잘 없음

### 4. 싱글톤은 2가지의 책임을 진다?
- 자신의 인스턴스를 관리
- 본래의 인스턴스를 사용하려는 목적

### 5. 싱글톤의 느슨한 결합 위배
- 싱글톤에 의존하는 객체는 전부 결합됨.
- 싱글톤에 변화가 생기면 연결된 모든 객체를 바꿔야할 가능성이 존재함.

### 6. 싱글톤의 서브클래스
- 싱글톤은 생성자가 private으로 확장(다른 클래스에서 상속)이 불가능함.
- 생성자를 protected나 public으로 하여 어찌저찌 구현했다고 하더라도 모든 서브클래스에서 같은 인스턴스를 멤버로 가지게 됨.

### 싱글톤 사용시 주의점
- 상태를 가진 객체를 싱글톤으로 만들면 안됨<br>
-> Application 내의 단 한개의 인스턴스가 존재할 때, 전역에서 접근할 수 있다면 각기 다른 스레드에서 객체의 상태를 마구잡이로 변경시킬 수 있음
- 상태가 공유된다는 것은 매우 위험한 일이기 때문에 무상태 객체 혹은 설계상 유일해야하는 시스템에 관련된 컴포넌트를 싱글톤으로 작성

### Enum을 통한 싱글톤
``` Java
public enum EnumSingleton {
    
    INSTANCE("Initial class info"); 
 
    private String info;
 
    private EnumSingleton(String info) {
        this.info = info;
    }
 
    public EnumSingleton getInstance() {
        return INSTANCE;
    }
    // getters and setters
}
```