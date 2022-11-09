# Command(커맨드)
- 하나의 객체를 통해 여러 객체들에 명령을 할 때 사용
- 요청을 캡슐화, 커맨드 객체가 명령해야하는 객체들에 대한 의존성이 느슨해짐
- 기존 코드의 변경없이 새로운 리시버, 커맨드가 추가가 가능함 OCP
- 작업을 수행하는 객체와 작업을 요청하는 객체의 분리 SRP
- but, 전체적인 설계구조에 대한 이해가 필요하고 복잡해짐

![Command](https://snowdeer.github.io/assets/design-patterns/command.png)

- Invoker : 기능의 실행을 요청하는 호출 클래스
- Receiver : ConcreteCommand에서 execute 메소드를 구현하는 클래스, 기능을 실행하기 위해 사용하는 수신자
- Command : 실행될 기능에 대한 인터페이스, execute 메소드 선언
- ConcreteCommand : 실제로 실행되는 기능을 구현, execute 메소드 구현

- 예시 : 손님(Client) -> 사장,종업원(Receiver) -> 주방장,쉐프(Invoker) execute -> 조리(ConcreteCommand)