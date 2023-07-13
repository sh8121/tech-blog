Java Generic 을 파헤쳐보자 - 심화편
==============================
![Java 로고](https://github.com/sh8121/tech-blog/assets/20632477/cbd53d33-6261-4ed2-9521-26ea826b8644)
 
이번에는 Java Generic 에서 주의해야 하는 심화 개념들을 몇가지 다뤄보겠습니다.   

## 1. Type Erasure
Java Generic 을 관통하는 주요 개념 중에 Type Erasure 라는 개념이 있습니다.   
이론적으로는 Generic 을 운영하기 위해 부가적으로 들어간 소스코드들이 바이트코드 레벨에서는 모두 제거되는 것을 의미하는데요.   
[개념편](https://velog.io/@sh8121/Java-Generic-%EC%9D%84-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-%EA%B0%9C%EB%85%90%ED%8E%B8 "Java Generic 을 파헤쳐보자 - 개념편")에서 다루었던 예제를 기반으로 좀 더 상세히 알아보겠습니다.   

```java
public class Tv {
    private String title;

    public Tv(String title) {
        this.title = title;
    }

    public String getTitle() {
        return title;
    }
}

public class Radio {
    private String name;

    public Radio(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

위처럼 Tv, Radio 객체가 존재할 때   

```java
public class RemoteController<Device> {
    private Device connectedDevice;

    public RemoteController(Device connectedDevice) {
        this.connectedDevice = connectedDevice;
    }

    public Device getConnectedDevice() {
        return connectedDevice;
    }
}
```

제네릭을 사용하여 Tv, Radio 를 제어할 수 있는 하나의 RemoteController 제네릭 타입(Generic Type) 을 만들 수 있다고 했습니다.   
실제 사용은 아래와 같이 할 수 있습니다.   

```java
Tv tv1 = new Tv("티비1");
Radio radio1 = new Radio("라디오1");
RemoteController<Tv> tvRemoteController1 = new RemoteController<Tv>(tv1);
RemoteController<Radio> radioRemoteController1 = new RemoteController<Radio>(radio1);

Tv connectedTv = tvRemoteController1.getConnectedDevice();
System.out.println(connectedTv.getTitle());

Radio connectedRadio = radioRemoteController1.getConnectedDevice();
System.out.println(connectedRadio.getName());
```

근데 위 코드의 바이트코드를 decompile 해보면 아래와 굉장히 유사한 코드를 얻을 수 있습니다.   

```java
Tv tv1 = new Tv("티비1");
Radio radio1 = new Radio("라디오1");
RemoteController tvRemoteController1 = new RemoteController(tv1);
RemoteController radioRemoteController1 = new RemoteController(radio1);

Tv connectedTv = (Tv)tvRemoteController1.getConnectedDevice();
System.out.println(connectedTv.getTitle());

Radio connectedRadio = (Radio)radioRemoteController1.getConnectedDevice();
System.out.println(connectedRadio.getName());
```

마치 제네릭이 없을 때의 사용코드와 유사합니다.   
한가지를 더 살펴보겠습니다.   

```java
Tv tv1 = new Tv("티비1");
Radio radio1 = new Radio("라디오1");
RemoteController<Tv> tvRemoteController1 = new RemoteController<Tv>(tv1);
RemoteController<Radio> radioRemoteController1 = new RemoteController<Radio>(radio1);

System.out.println(tvRemoteController1.getClass());
System.out.println(radioRemoteController1.getClass());
```

위 코드를 실행한 결과는 아래와 같습니다.   

```java
class org.example.generic.RemoteController
class org.example.generic.RemoteController
```

타입 인자를 전달하여 선언한 RemoteController\<Tv\>와 RemoteController\<Radio\> 객체에 대한 런타임 클래스 정보를 보면 차이가 없는 것을 알 수 있습니다. Tv, Radio 라는 타입인자가 클래스 레벨에 전달되는 것이 아니라 객체 단위로 전달되는 것이기 때문에 어찌 보면 당연한 결과입니다.   
이렇듯 제네릭과 관련된 소스 코드 상의 정보들은 컴파일러에 의해 제거되는데 이러한 특징을 Type Erasure 라고 합니다.   
제네릭이 이러한 방식으로 동작하는 가장 큰 이유는 제네릭의 없던 시절에 작성된 코드, 그러니까 JDK5 이전의 코드와의 호환성 이슈 때문입니다.   
결국 제네릭은 런타임 실행 코드에는 영향을 주지 않으면서 컴파일 타임에 개발자에게 Type Safety 를 포함한 편의 기능을 제공하는 방식으로 동작하는데 이러한 특징 때문에 Java Generic 을 syntactic sugar 라고 하기도 합니다.   

한걸음만 더 나아가 보겠습니다.   
위에서 말했듯 제네릭은 객체 단위로 타입 정보가 부여되기 때문에 기본적으로 클래스 정보에 영향을 주지 않는데요, 클래스 정보에 제네릭이 명시적으로 포함되는 경우들이 있습니다.   
주로 제네릭 타입을 상속받거나, 포함하는 경우인데요, 예시를 확장해서 알아보겠습니다.   

```java
public class TvRemoteController extends RemoteController<Tv> {
    public TvRemoteController(Tv connectedDevice) {
        super(connectedDevice);
    }
}
```

위와 같이 RemoteController\<Device\> 를 상속받아서 TvRemoteController 를 정의하거나   

```java
public class RadioRemoteController {
    private RemoteController<Radio> remoteController;

    public RadioRemoteController(Radio connectedDevice) {
        remoteController = new RemoteController<>(connectedDevice);
    }

    public Radio getConnectedDevice() {
        return remoteController.getConnectedDevice();
    }
}
```

위와 같이 RemoteController\<Device\> 객체를 포함하여 RadioRemoteController 를 정의할 수 있습니다.   
이런 경우 TvRemoteController, RadioRemoteController 는 클래스 레벨에서 각각 Tv, Radio 타입 인자에 의존하기 때문에 리플렉션을 통해 타입 인자 정보를 확인할 수 있습니다.   

```java
Tv tv = new Tv("티비");
Radio radio = new Radio("라디오");
TvRemoteController tvRemoteController = new TvRemoteController(tv);
RadioRemoteController radioRemoteController = new RadioRemoteController(radio);

ParameterizedType remoteControllerTvType = (ParameterizedType)tvRemoteController.getClass().getGenericSuperclass();
System.out.println(remoteControllerTvType.getActualTypeArguments()[0]);

ParameterizedType remoteControllerRadioType = (ParameterizedType)radioRemoteController.getClass()
        .getDeclaredField("remoteController").getGenericType();
System.out.println(remoteControllerRadioType.getActualTypeArguments()[0]);
```

위 코드를 실행하면   

```java
class org.example.generic.Tv
class org.example.generic.Radio
```

과 같은 결과를 얻을 수 있습니다.   
이처럼 클래스 정의부가 제네릭 타입과 특정 타입 인자에 의존하는 경우, 타입에 대한 정보가 런타임에도 유지됩니다.   

## 2. PECS
이번에는 Java Generic 의 또다른 중요한 개념인 PECS(Producer Extends Consumer Super) 에 대해서 알아보겠습니다.   
이를 설명하기 위해서는 몇가지 추가적인 배경 설명이 필요합니다.   
Java Generic 은 불공변(invariance)이라는 특징을 갖는데요, 서로 다른 제네릭 타입 간에는 상하위의 관계가 없다는 특징입니다.   

```java
public class Electronics {
}

public class Tv extends Electronics {
    private String title;

    public Tv(String title) {
        this.title = title;
    }

    public String getTitle() {
        return title;
    }
}

public class Radio extends Electronics {
    private String name;

    public Radio(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

위처럼 상속관계로 Electronics, Tv, Radio 클래스를 재구성했을 때   

```java
Tv tv = new Tv("티비");
Radio radio = new Radio("라디오");

RemoteController<Electronics> tvRemoteController = new RemoteController<Tv>(tv); //compile error
RemoteController<Electronics> radioRemoteController = new RemoteController<Radio>(radio); //compile error
```

RemoteController\<Electronics\> 참조변수로 RemoteController\<Tv\>, RemoteController\<Radio\> 객체를 받을 수 없습니다.   
제네릭의 불공변 때문에 RemoteController\<Electronics\> 와 RemoteController\<Tv\> 는 전혀 상관 없는 타입으로 인정되기 때문입니다.   
제네릭 자체가 Type Safety 를 도모하기 위해 탄생한 개념인 만큼 제네릭의 불공변성은 어찌보면 당연한 것 같습니다. 하지만 불공변성 때문에 사용하는 입장에서 유연성이 너무 떨어지는 단점도 있습니다.   
이러한 문제점을 해결하기 위해 Java Generic 에서 추가적으로 지원하는 기능이 와일드카드(?) 라는 개념입니다.   

```java
Tv tv = new Tv("티비");
Radio radio = new Radio("라디오");

RemoteController<?> tvRemoteController = new RemoteController<Tv>(tv);
RemoteController<?> radioRemoteController = new RemoteController<Radio>(radio);
```

위처럼 RemoteController\<?\> 타입의 참조변수는 해당 타입 매개변수 위치에 어떤 타입이 오든 받을 수 있으며, 위 코드는 정상 동작 합니다.   
와일드카드를 쓰면서 동시에 받을 수 있는 타입 인자에 제약을 줄 수도 있는데 이때 extends 와 super 키워드를 사용합니다.   

```java
Tv tv = new Tv("티비");
Radio radio = new Radio("라디오");

RemoteController<? extends Electronics> tvRemoteController = new RemoteController<Tv>(tv);
RemoteController<? extends Electronics> radioRemoteController = new RemoteController<Radio>(radio);
```

\<? extends Electronics\> 는 Electronics 의 하위 타입들(Electronics 포함)을 받을 수 있으며, 이러한 경우를 Upper Bounded Wildcard 라고 합니다.   

```java
Electronics electronics = new Electronics();
RemoteController<? super Tv> remoteController = new RemoteController<Electronics>(electronics);
```

\<? super Tv\> 는 Tv 의 상위 타입들(Tv 포함)을 받을 수 있으며, 이러한 경우를 Lower Bounded Wildcard 라고 합니다.   
PECS 란 Producer, 즉 데이터를 생산해내는(조회 기능으로 이해하면 됩니다.) Component 에서는 extends 를 사용하고 Consumer, 즉 데이터를 소비하는(저장, 수정 등의 기능) Component 에서는 super 를 사용한다는 의미입니다.   

```java
List<Tv> tvs = new ArrayList<>();
tvs.add(new Tv("티비1"));
tvs.add(new Tv("티비2"));
tvs.add(new Tv("티비3"));

List<? extends Electronics> electronics = tvs;
for (Electronics e : electronics) {
	System.out.println(e);
}
```

위의 코드에서 List\<? extends Electronics\> electronics 는 '내가 참조하는 리스트 객체의 element 는 정확한 타입이 먼지는 모르겠지만 아무튼 Electronics 의 하위타입이기는 하다' 라고 해석됩니다. 때문에 for(Electronics e: electronics) 와 같은 코드가 가능합니다. 참조하는 객체가 new ArrayList\<Tv\>() 이던, new ArrayList\<Radio\>() 이던 그 element 들을 Electronics 로 받을 수 있기 때문이죠.   

```java
electronics.add(new Tv("티비4")); //compile error
```

반면 위 코드는 컴파일 에러를 냅니다. electronics 가 참조하는 객체가 new ArrayList\<Tv\>() 일수도 있지만 new ArrayList\<Radio\>() 일수도 있기 때문에 Tv 객체의 등록을 허락하지 않습니다.   

```java
List<? super Electronics> list = new ArrayList<Electronics>();
list.add(new Tv("티비"));
list.add(new Radio("라디오"));
list.add(new Electronics());
```

위 코드에서 List\<? super Electronics\> list 는 '내가 참조하는 리스트 객체의 element 는 먼지는 모르겠지만 Electronics 의 상위타입이기는 하다' 라고 해석됩니다. 따라서 위 코드는 정상 동작합니다. List\<? super Electronics\> list 가 참조하는 객체가 new ArrayList\<Object\>() 이건, new ArrayList\<Electronics\>() 이건 Tv, Radio, Electronics 객체를 담을 수 있기 때문이죠.   

```java
for(Electronics e : list) { //compile error
    System.out.println(e);
}
```

반면 위 코드는 컴파일 에러를 냅니다. list 변수가 참조하는 객체가 new ArrayList\<Electronics\>() 라면 문제가 없겠지만 new ArrayList\<Object\>() 라면 문제가 생기기 때문이죠.   
이렇듯 for(Electronics e : electronics) 처럼 데이터를 제공하는 Producer 에서는 extends 를 사용하고 list.add(...) 처럼 데이터를 소비하는 Consumer 에서는 super 를 사용하는 것을 PECS 라고 합니다.   

지금까지 Java Generic 을 사용하면서 주의해야하는 심화 개념들을 알아봤습니다.     
[개념편](https://velog.io/@sh8121/Java-Generic-%EC%9D%84-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-%EA%B0%9C%EB%85%90%ED%8E%B8) 에서 주로 'Java Generic 의 주된 목적(Type Safety)과 기본적인 사용 방법' 을 다루었다면 이번 '심화편' 에서는 'Java Generic 의 독특한 특성들' 에 대해서 다뤄 보았습니다.     
이 독특한 특성들을 잘 활용하면 'Type Safety' 를 지키면서도 '유연한' 코드를 작성할 수 있습니다. 실제로 많은 제네릭 기반의 라이브러리들이 이 특성을 적절하게 활용하고 있고요.     
다음 포스팅에서는 이러한 특성들을 실제 Application 개발에서 어떻게 사용할 수 있는지에 대한 이야기를 해보겠습니다.      
