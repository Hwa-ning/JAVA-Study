# Abstract Factory (추상 팩토리)
- 구체적인 인스턴스를 만드는 팩토리를 추상화하여 관련된 객체들의 조합을 만드는 인터페이스를 제공하는 패턴
- 객체 생성을 서브 클래스로 분리 및 캡슐화하여 확장성 있는 코드가 됨

```Java

public interface Wheel {
    int getWheelSize();
}
```

```Java
public class MorningWheel implements Wheel{
    @Override
    int getWheelSize(){
        return 15;
    }
}
```

```Java
public class EV6Wheel implements Wheel{
    @Override
    int getWheelSize(){
        return 19;
    }
}
```


```Java
public interface Engine {
    String getFuel();
}
```

```Java
public class MorningEngine implements Engine{
    @Override
       String getFuel(){
        return "Gasoline";
    }
}
```

```Java
public class EV6Engine implements Engine{
    @Override
       String getFuel(){
        return "Electric";
    }
}
```

```Java
public abstract class CarFactory{
    abstract Wheel makeWheel();
    abstract Engine makeEngine();
}
```

```Java
public class MorningFactory extends CarFactory{
    @Override
    public Wheel makeWheel(){
        return new MorningWheel();
    }
    
    @Override
    public Engine makeEngine(){
        return new MorningEngine();
    }
}
```
```Java
public class EV6Factory extends CarFactory{
    @Override
    public Wheel makeWheel(){
        return new EV6Wheel();
    }
    
    @Override
    public Engine makeEngine(){
        return new EV6Engine();
    }
}
```

``` Java
public enum CarType{
    MORNING, EV6
}
```

```Java
public class Kia{
    public static CarFactory getFactory(CarType carType){
        switch(carType){
            case CarType.MORNING :
                return new MorningFactory();
            case CarType.EV6 :
                return new EV6Factory();
        }
    }
}
```
``` Java
public class Car{
    Wheel wheel;
    Engine engine;

    Car(Wheel wheel, Engine engine){
        this.wheel = wheel;
        this.engine = engine;
    }
}
```

``` Java
public class Main{
    public static void main(String[] args){
        CarFactory morningFactory = Kia.getFactory(CarType.MORNING);
        Car morning = new Car(morningFactory.makeWheel(), morningFactory.makeEngine());

        CarFactory ev6Factory = new EV6Factory(CarType.EV6);
        Car ev6 = new Car(ev6Factory.makeWheel(), ev6Factory.makeEngine());
    } 
}
```

## 새로운 부품 HeadLight가 나온 경우
### 새로 추가되는 클래스
```Java
public interface HeadLight {
    String getHeadLight();
}
```

```Java
public class MorningHeadLight implements HeadLight{
    @Override
       String getHeadLight(){
        return "cheap HeadLight";
    }
}
```

```Java
public class EV6HeadLight implements HeadLight{
    @Override
       String getHeadLight(){
        return "expensive HeadLight";
    }
}
```
### 수정되는 클래스
```Java
public abstract class CarFactory{
    abstract Wheel makeWheel();
    abstract Engine makeEngine();
    abstract HeadLigth makeHeadLight(); // +++
}
```

```Java
public class MorningFactory extends CarFactory{
    @Override
    public Wheel makeWheel(){
        return new MorningWheel();
    }
    
    @Override
    public Engine makeEngine(){
        return new MorningEngine();
    }

    @Override
    public HeadLigth makeHeadLight(){ // +++
        return new MorningHeadLight();
    }
}
```
```Java
public class EV6Factory extends CarFactory{
    @Override
    public Wheel makeWheel(){
        return new EV6Wheel();
    }
    
    @Override
    public Engine makeEngine(){
        return new EV6Engine();
    }
    
    @Override
    public HeadLigth makeHeadLight(){ // +++
        return new EV6HeadLight();
    }
}
```
``` Java
public class Car{
    Wheel wheel;
    Engine engine;
    HeadLigth headLight; // +++

    Car(Wheel wheel, Engine engine, HeadLigth headLight){ // +++
        this.wheel = wheel;
        this.engine = engine;
        this.headLight = headLight; // +++
    }
}
```
## 신차 Niro가 나온 경우

### 새로 추가되는 클래스
``` Java
public class NiroEngine implements Engine{
    @Override
       String getFuel(){
        return "Hybrid";
    }
}
```
``` Java
public class NiroWheel implements Wheel{
    @Override
    int getWheelSize(){
        return 17;
    }
}
```
``` Java
public class NiroFactory extends CarFactory{
    @Override
    public Wheel makeWheel(){
        return new NiroWheel();
    }
    
    @Override
    public Engine makeEngine(){
        return new NiroEngine();
    }
}
```

### 수정되는 클래스
``` Java
public enum CarType{
    MORNING, EV6, NIRO
}
```

```Java
public class Kia{
    public static CarFactory getFactory(CarType carType){
        switch(carType){
            case CarType.MORNING :
                return new MorningFactory();
            case CarType.EV6 :
                return new EV6Factory();
            case CarType.NIRO :
                return new NiroFactory();
        }
    }
}

