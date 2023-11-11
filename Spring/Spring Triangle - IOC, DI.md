Spring Triangle - IOC, DI
=====================
![Spring 로고](https://github.com/sh8121/tech-blog/assets/20632477/6d1be92a-ea3c-436a-80bc-55abb93a9e84)

Spring 을 처음 접하는 개발자들이 가장 많이 마주치는 단어 중 하나는 아마도 IOC, DI 일 것 입니다.   
Spring 이라는 단어는 문맥에 따라서 여러가지 의미로 사용이 되지만(상황에 따라 Spring Core Project 를 의미하기도 하고, Spring 생태계 전체를 의미하기도 합니다.) 가장 좁은 의미로 사용이 되면 보통 Spring Container 를 지칭합니다. Spring Container 를 IOC Container 나 DI Container 로도 부르기 때문에, IOC 나 DI 는 자연스럽게 Spring 을 다루면서 자주 접하게 되는 단어입니다.   
이렇듯 IOC, DI 는 Spring 을 다루는 개발자 들에게 매우 친숙한 표현이지만, 각 용어의 개념이나 맥락에 대해서 여러가지 오해가 있는 것 같습니다.   
이번 글에서는 IOC, DI 의 개념을 짚어보고, 그것들이 객체지향과 어떤 관련이 있는지 그리고 Spring 과는 어떻게 연결이 되는지 살펴보겠습니다.   

## IOC vs DI
IOC 와 DI 에 대해 존재하는 가장 큰 오해는 아마도 'IOC = DI' 일 것 입니다. 이 오해는 Spring Container = IOC Container = DI Container 라는 관례에서 비롯됩니다.   
이 오해를 해결하기 위해 각각의 개념을 좀 더 깊게 알아보겠습니다.   
### IOC
IOC 는 Inversion Of Control, 즉 제어의 역전의 약자입니다. 여기서 제어는 '애플리케이션을 구성하는 객체 간의 협력관계, 의존관계에 대한 제어권' 을 의미합니다.   
전통적인 프로그래밍에서 이 제어권은 각각의 객체가 가지고 있습니다. 즉, 각각의 객체가 자신의 역할을 수행하는 책임을 지는 동시에 자신이 어떤 객체와 협력해야 하는지, 어떤 객체에 의존해야 하는지를 결정하는 책임도 지는 것입니다.   
```java
interface Car {

    void excel();

    void breaks();
}
```
엑셀과 브레이크 기능만 지원하는 단순한 Car 인터페이스 입니다.   
Car 인터페이스 규약을 따르는 두가지 차종이 있다고 가정해보겠습니다.   
```java
static class GV70 implements Car {

    @Override
    public void excel() {
        System.out.println("GV70 Excel");
    }

    @Override
    public void breaks() {
        System.out.println("GV70 Break");
    }
}

static class GLC implements Car {

    @Override
    public void excel() {
        System.out.println("GLC Excel");
    }

    @Override
    public void breaks() {
        System.out.println("GLC Break");
    }
}
```
차를 운전하는 운전자 Class 를 선언해보겠습니다.   
```java
static class Driver {

    private Car car = new GV70();

    public void drive() {
        car.excel();
        car.breaks();
        car.excel();
    }
}
```
이 Driver 는 운전 기능(drive())을 제공하는 동시에 자신이 무슨 차를 운전할 지(new GV70()) 결정합니다. 자신이 Car 인터페이스의 구현체 중 어떤 구현체에 의존할 지에 대한 제어권을 스스로 가지고 있는 것이죠.   
제어의 역전이란 Driver 가 어떤 Car 를 운전할 지를 스스로 제어하는 것이 아니라, 이를 조율해주는 별도의 제어권자(Assembler) 를 두는 방식으로 소프트웨어를 설계하는 일종의 개발 패러다임 입니다.   
구체적인 방법론이 아니라, 추상적인 컨셉 혹은 패러다임이라는 것이 포인트입니다.   
### DI
DI 는 Dependency Injection 의 약자로 의존성 주입 혹은 의존관계 주입으로 해석됩니다.   
DI 는 IOC 를 달성할 수 있는 구체적인 방법론 중의 하나입니다.   
조금 더 구체적으로는 DI 는 IOC 에서 말하는 제어권자(Assembler)가 각 객체가 의존하는 객체를 생성하고 주입해주는 방식으로 제어권을 행사합니다.   
```java
static class Driver {

    private Car car;

    public Driver(Car car) {
        this.car = car;
    }

    public void drive() {
        car.excel();
        car.breaks();
        car.excel();
    }
}
```
이번 운전자 클래스는 어떤 차종을 운전할 지를 스스로 제어하지 않고, 생성자를 통해 주입 받습니다.   
```java
static class Assembler {

    private Driver driver;

    public Assembler() {
        driver = new Driver(new GLC());
    }

    public Driver getDriver() {
        return driver;
    }
}
```
새롭게 등장한 Assembler 클래스는 내부에서 운전자 객체를 생성하면서, 운전자가 운전할 차종까지 생성해서 주입해줍니다.   
```java
Driver driver = new Driver();
driver.drive();
```
첫번째 운전자 클래스를 사용하는 코드는 위처럼 직접 운전자 객체를 생성해서 사용합니다.   
```java
Assembler assembler = new Assembler();
Driver driver = assembler.getDriver();
driver.drive();
```
반면 두번째 운전자 클래스를 사용하는 코드는 위처럼 조립자(Assembler)로부터 운전자 객체를 반환받아서 사용합니다.   
### Service Locator
IOC 와 DI 의 차이를 더 명확하게 드러내기 위해 Service Locator 라는 방법론을 추가로 살펴보겠습니다.   
Service Locator 역시 IOC 를 달성할 수 있는 구체적인 방법론 중의 하나입니다.   
```java
static class ServiceLocator {

    public static Car getCar() {
        int val = (int) (Math.random() * 2);
        return val == 0 ? new GV70() : new GLC();
    }
}

static class Driver {

    private Car car = ServiceLocator.getCar();

    public void drive() {
        car.excel();
        car.breaks();
        car.excel();
    }
}
```
DI 의 제어권자(Assembler)는 애플리케이션의 객체들을 직접 생성하고 객체간의 의존관계를 주입하는 방식으로 동작한다면, Service Locator 의 제어권자는 각 객체가 자신이 의존하고자 하는 객체를 요청했을 때 그것을 적절하게 찾아주는 방식으로 동작합니다.   
위의 세번째 운전자 클래스는 자신이 어떤 차를 운전할 지를 직접 정의하지도, 외부에서 주입받지도 않습니다. 단지 ServiceLocator 에게 운전할 차를 찾아달라고 요청하고, ServiceLocator 가 반환하는 차종에 의존합니다.   

위의 내용을 정리해보면, IOC 는 '객체간의 의존관계를 제어하는 별도의 제어권자를 두는 설계 개념'이라고 할 수 있고, DI 와 Service Locator 는 그 개념을 실현하는 구체적인 방법론 이라고 할 수 있습니다.   

## OOP 와 IOC
지금까지 IOC 와 DI 가 무엇인지 알아봤습니다. 그러면 IOC 는 왜 필요할까요?   
OOP 의 가장 주요한 특성 중의 하나는 '다형성' 입니다. 그리고 객체지향 설계원칙(SOLID) 중에 다형성과 연관이 깊은 것은 OCP(개방폐쇄원칙)와 DIP(의존역전원칙) 입니다.(SOLID 에 대한 자세한 내용은 기회가 된다면 다른 글에서 다루도록 하겠습니다.)   
```java
interface Car {

        void excel();

        void breaks();
    }

static class GV70 implements Car {

    @Override
    public void excel() {
        System.out.println("GV70 Excel");
    }

    @Override
    public void breaks() {
        System.out.println("GV70 Break");
    }
}

static class GLC implements Car {

    @Override
    public void excel() {
        System.out.println("GLC Excel");
    }

    @Override
    public void breaks() {
        System.out.println("GLC Break");
    }
}
```
예제에서 사용한 자동차 클래스 그룹은 Car 인터페이스를 기반으로 그 구현체들을 정의하게 함으로써, 다형성의 특성을 나름 잘 활용하고 있습니다.   
```java
static class Driver {

    private Car car = new GV70();

    public void drive() {
        car.excel();
        car.breaks();
        car.excel();
    }
}
```
하지만 첫번째 운전자 클래스를 보면 Car 인터페이스에 의존하는 동시에 구체적인 구현체(GV70) 에도 의존하고 있습니다.(DIP 위배)   
만약 운전자가 운전하는 차종을 변경하고 싶다면   
```java
static class Driver {

    //        private Car car = new GV70();
    private Car car = new GLC();

    public void drive() {
        car.excel();
        car.breaks();
        car.excel();
    }
}
```
위처럼 운전자 클래스에 변경이 필요합니다.(OCP 위배)   
```java
static class Assembler {

    private Driver driver;

    public Assembler() {
//            driver = new Driver(new GLC());
        driver = new Driver(new GV70());
    }

    public Driver getDriver() {
        return driver;
    }
}

static class Driver {

    private Car car;

    public Driver(Car car) {
        this.car = car;
    }

    public void drive() {
        car.excel();
        car.breaks();
        car.excel();
    }
}
```
DI 가 적용된 Driver 클래스는 Car 인터페이스에만 의존합니다.(DIP 준수) 그래서 스스로가 운전하게 될 차종에 상관없이 구현체를 유지할 수 있습니다.(OCP 준수)   
물론 Driver 객체를 생성하고 의존관계를 주입해주는 Assembler 의 코드에는 변경이 일어나지만, Assembler 는 객체간의 의존관계를 설정하는 책임을 수행하는 클래스이기 때문에 객체 의존관계의 변경으로 구현이 바뀌는 것은 자연스럽습니다. 그리고 Production Level 에서 사용하는 Assembler 는 보통 Reflection 과 Configuration 등을 활용하여 구현하기 때문에 의존관계 변경으로 구현 코드가 바뀌는 경우는 거의 없습니다.   
한가지만 더 보겠습니다. '운전자가 어떤 차종을 운전할 것인가' 와 '운전자의 운전행위' 는 서로 다른 관심사입니다. 운전자는 어떤 차종을 운전하든 같은 운전행위를 유지할 수 있으며, 운전자의 운전행위가 바뀐다면 그것은 모든 차종에 일관되게 반영됩니다. 이것을 객체지향의 용어로 치환하면 각각은 서로 다른 '책임'을 갖고 있는 것이며, '변경의 사이클'이 다르다고도 표현합니다.   
결국 Assembler 를 도입함으로써, '어떤 차종을 운전할 지를 결정하는 책임'은 Assembler가 담당하고, Driver 는 '운전행위' 에만 집중하게 됩니다.   
'변경의 사이클' 관점에서 표현하면, '차종의 변경' 은 Assembler 에만 영향을 끼치고 Driver 는 '운전행위의 변경' 에만 영향을 받게 됩니다.   
이렇게 애플리케이션을 구성하는 각 객체가 명확하게 하나의 책임을 갖도록 개발하는 것을 SRP(단일책임원칙) 라고 합니다. 이런 맥락에서 IOC 는 SRP 를 용이하게 하는 패러다임이라고 볼 수도 있습니다.   

## Spring 과 IOC
그렇다면 Spring 과 IOC 는 어떤 연관관계가 있을까요?   
Spring Core 프로젝트에서 가장 중요한 기능이 Spring Bean Container 입니다.   
Spring Container(BeanFactory, ApplicationContext) 는 익히 아시는 것처럼, 설정파일이나 컴포넌트 스캔 등을 이용해서 Spring Bean 들을 생성하고 의존관계를 주입하는 역할을 수행합니다. 각각의 Bean 들은 자신이 의존하는 구체적인 객체에 대한 제어를 Spring Container 에 위임하고, 객체 본연의 책임에 집중합니다.   
위에서 배운 개념들로 다시 정리해보면, Spring Container 는 Spring Bean 들에 대해서 IOC 를 실현해주는 구현체이며, 그 구현기법으로써 DI 를 선택했다고 할 수 있습니다.(Spring 기반의 샘플 코드는 다른 글들을 통해서 다루도록 하겠습니다.) Spring Container 가 IOC Container 나 DI Container 와 동의어처럼 사용되는 것은 이런 이유 때문입니다.   

## 맺음말
마무리 하겠습니다. 객체지향 언어는 그 가장 중요한 특성으로서 '다형성'을 갖고 있으며, 그 특성을 극대화할 수 있는 설계 원칙으로서 OCP, DIP 와 같은 개념이 존재합니다. IOC 는 객체지향 프로그래밍에서 OCP, DIP 를 지키기 위해서 별도의 제어권자를 운영하자는 추상적인 컨셉이고, DI 는 그 컨셉을 의존관계 주입이라는 방식으로 해결하는 구체적인 방법론입니다.   
굉장히 원론적이고 다소 고리타분하지만, Spring 을 제대로 사용하는데 있어서 이 흐름을 이해하는 것이 중요하다고 생각되어 글로 정리해보았습니다.   
이 글이 Spring 을 이해하는 Insight 를 높여주기를 희망하며 마치겠습니다. 긴 글 읽어주셔서 감사합니다.   

## 참고
https://martinfowler.com/articles/injection.html   
https://howtodoinjava.com/spring-core/spring-ioc-vs-di/   
https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8   
