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

### 2. 클래스 로더와 관련한 문제점?

### 3. 전역 변수 vs 싱글톤

### 4. 싱글톤은 2가지의 책임을 진다?

### 5. 싱글톤의 느슨한 결합 위배

### 6. 싱글톤의 서브클래스