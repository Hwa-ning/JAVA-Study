# Decorator (데코레이터)
![Decorator](https://kymr.github.io/files/design-pattern/decorator-pattern/decorator-pattern.png)
![Java BufferedReader](https://cdn.codegym.cc/images/article/d0cf56c5-a505-40de-832c-21a075110e77/800.webp)
데코레이터 패턴이 적용된 java.io 클래스

- 입력 스트림, 출력 스트림을 보면 데코레이터의 단점을 확인할 수 있음.
- 잡다한 클래스가 너무 많아짐, 직관적이지 못함 
- 클래스가 어떤식으로 구성되어있는지 파악이 필요함
``` Java
        InputStream input = getClass().getClassLoader().getResourceAsStream(templateFilePath);
        BufferedReader in = new BufferedReader(new InputStreamReader(input));
```
BD중 소스 코드 중 

``` Java
public class InputStreamReader extends Reader {

    private final StreamDecoder sd;

    /**
     * Creates an InputStreamReader that uses the default charset.
     *
     * @param  in   An InputStream
     */
    public InputStreamReader(InputStream in) {
        super(in);
        try {
            sd = StreamDecoder.forInputStreamReader(in, this, (String)null); // ## check lock object
        } catch (UnsupportedEncodingException e) {
            // The default encoding should always be available
            throw new Error(e);
        }
    }
    ...
```

``` Java
public class BufferedReader extends Reader {
    ...
    public BufferedReader(Reader in, int sz) {
        super(in);
        if (sz <= 0)
            throw new IllegalArgumentException("Buffer size <= 0");
        this.in = in;
        cb = new char[sz];
        nextChar = nChars = 0;
    }

    /**
     * Creates a buffering character-input stream that uses a default-sized
     * input buffer.
     *
     * @param  in   A Reader
     */
    public BufferedReader(Reader in) {
        this(in, defaultCharBufferSize);
    }
    ...
}
```

``` Java
class LowerCaseInputStream extends FilterInputStream{

    protected LowerCaseInputStream(InputStream in) {
        super(in);
    }

    public int read() throws IOException{
        int c = in.read();
        return (c == -1 ? c : Character.toLowerCase((char)c));
    }

    public int read(byte[] b, int offset, int len) throws IOException {
        int result = in.read(b,offset,len);
        for(int i= offset; i < offset + result; i++){
            b[i] = (byte)Character.toLowerCase((char)b[i]);
        }
        return result;
    }
}

public class Main{
    public static void main(String[] args) {
        int c;
        try(InputStream in = new LowerCaseInputStream(
                new BufferedInputStream(
                        new FileInputStream("/Users/hwaning1/IdeaProjects/Algo_Java/src/test.txt")))) {
            while((c = in.read()) >= 0){
                System.out.print((char)c);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
### 데코레이터 패턴을 사용한 자동차 Class
``` Java
interface Car {
    void drive();
}

class KiaCar implements Car {
    @Override
    public void drive() {
        System.out.print("기아차 운행중");
    }
}

class HyundaiCar implements Car {
    @Override
    public void drive() {
        System.out.print("현대차 운행중");
    }
}

class CarDecorator implements Car {
    private final Car car;

    public CarDecorator(Car car) {
        this.car = car;
    }

    @Override
    public void drive() {
        car.drive();
    }
}

class HybridDecorator extends CarDecorator {

    public HybridDecorator(Car car) {
        super(car);
    }

    @Override
    public void drive() {
        super.drive();
        System.out.print(" + 하이브리드"); // +++++
    }
}

class HudDecorator extends CarDecorator {

    public HudDecorator(Car car) {
        super(car);
    }

    @Override
    public void drive() {
        super.drive();
        System.out.print(" + HUD");
    }
}

class NavigationDecorator extends CarDecorator {

    public NavigationDecorator(Car car) {
        super(car);
    }

    @Override
    public void drive() {
        super.drive();
        System.out.print(" + 네비게이션");
    }
}


public class Decorator {
    public static void main(String[] args) {
        Car sportage = new HybridDecorator(new KiaCar());
        sportage.drive();
        System.out.println("\n====================");

        sportage = new HybridDecorator(sportage);
        sportage.drive();
        System.out.println("\n====================");

        sportage = new NavigationDecorator(sportage);
        sportage.drive();
        System.out.println("\n====================");

        sportage = new HudDecorator(sportage);
        sportage.drive();

        Car tucson = new HyundaiCar();
        tucson.drive();
        System.out.println("\n====================");

        tucson = new HybridDecorator(tucson);
        tucson.drive();
        System.out.println("\n====================");

        tucson = new NavigationDecorator(tucson);
        tucson.drive();
        System.out.println("\n====================");

        tucson = new HudDecorator(tucson);
        tucson.drive();
    }
}

```
