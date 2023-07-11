Java Generic 을 파헤쳐보자 - 개념편
=================
![Java 이미지](https://github.com/sh8121/tech-blog/assets/20632477/cbd53d33-6261-4ed2-9521-26ea826b8644)

이번 포스팅에서는 Java의 Generic의 개념을 한번 다뤄 보겠습니다.   
"제네릭"은 "구체적인 타입에 대한 정보를 타입 정의 시점이 아닌 타입의 인스턴스화 시점에 전달함으로써 하나의 타입으로 여러 가지 타입을 표현하는 프로그래밍 기법"을 일반적으로 지칭하는 용어입니다.   
Java의 Generic은 이러한 제네릭에 대한 Java의 구현체라고 할 수 있겠습니다.   
Java Generic의 주요 기능은 다양한 타입의 객체를 다루는 메서드나 클래스에 대해서 컴파일 타임 타입 체크를 가능하게 하여 타입 안정성을 높이고 형 변환의 번거로움을 줄여주는 것입니다.    
위에 적은 문장이 Java Generic의 아주 주요한 개념이라고 할 수 있는데요, 이 문장을 제네릭이 없는 상황과 있는 상황 두가지 예시를 통해 상세히 들여다보겠습니다.   

## 제네릭이 없을 때...
### "다양한 타입의 객체를 다루는 메서드나 클래스"

리모컨을 생산해내는 하나의 공정으로 Tv를 컨트롤하는 리모컨과 Radio를 컨트롤하는 리모컨을 모두 만들어낼 수 있는 공장이 있다고 가정해보겠습니다.   

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

와 같이 Tv, Radio 클래스가 있을 때(즉, Tv, Radio 공장이 있을 때), 두가지 버전의 리모컨을 동시에 생산해낼 수 있는 리모컨 공장은   

```java
public class RemoteController {
    private Object connectedDevice;
 
    public RemoteController(Object connectedDevice) {
        this.connectedDevice = connectedDevice;
    }
 
    public Object getConnectedDevice() {
        return connectedDevice;
    }
}
```

와 같이 생겼을 것입니다.   
위의 3개의 공장을   

```java
Tv tv1 = new Tv("티비1");
Radio radio1 = new Radio("라디오1");
RemoteController tvRemoteController1 = new RemoteController(tv1);
RemoteController radioRemoteController1 = new RemoteController(radio1);
```

와 같은 방법으로 가동시킬 수 있습니다.   

### "컴파일 타임 타입 체크를 가능하게 하여 타입 안정성을 높이고 형 변환의 번거로움을 줄여주는 것"

위의 리모컨 공장에서 생산한 리모컨을 사용하려면   

```java
Object connectedDevice1 = tvRemoteController1.getConnectedDevice();
Tv connectedTv = (Tv)connectedDevice1;
System.out.println(connectedTv.getTitle());
 
Object connectedDevice2 = radioRemoteController1.getConnectedDevice();
Radio connectedRadio = (Radio)connectedDevice2;
System.out.println(connectedRadio.getName());
```

와 같은 방식으로 사용해야 합니다.   
이 리모컨을 정확하게 사용하기 위한 개발자의 사고의 흐름은 이렇습니다.   

1.  tvRemoteController1이라는 변수가 가리키는 RemoteController객체는 Tv객체랑 연결되어 있음을 기억한다.
2.  tvRemoteController1.getConnectedDevice()는 Object를 리턴 하지만 개발자는 이 객체가 사실은 Tv객체임을 알고 있다.
3.  자신 있게 Tv 타입으로 명시적 타입 캐스팅을 하여 객체를 사용한다.
4.  radioRemoteController1에 대해서도 같은 과정을 반복한다.

특별히 문제는 없는 것 같습니다.   
하지만 만약 개발자의 기억이 잘못되었다면   

```java
Object connectedDevice1 = tvRemoteController1.getConnectedDevice();
Radio connectedRadio = (Radio)connectedDevice1;
System.out.println(connectedRadio.getName());
 
Object connectedDevice2 = radioRemoteController1.getConnectedDevice();
Tv connectedTv = (Tv)connectedDevice2;
System.out.println(connectedTv.getTitle());
```

와 같은 형태로 사용이 될 수 있으며, 이 경우 Runtime에 ClassCastException이 발생합니다.   
사고의 경위는 이렇습니다.   

1.  개발자의 기억이 왜곡되어(?) tvRemoteController1에 Radio객체가 연결되어 있다고 생각했다.
2.  tvRemoteController1.getConnectedDevice()는 Object를 리턴 하지만 개발자는 이 객체가 Radio객체라고 믿고 있다.
3.  컴파일러는 Object ↔ Radio 간 타입 캐스팅이 문법적으로는 가능하기 때문에 이에 관여하지 않는다.(컴파일러는 실제로 Tv객체가 들어있는지는 모른다.)
4.  결국 런타임에 Tv ↔ Radio 간 타입 캐스팅에 대한 예외가 발생한다.

### 위의 사용 예제를 정리해보면...

1.  RemoteController는 여러 종류의 객체(Tv, Radio...)를 다룰 수 있다.
2.  RemoteController 객체에 어떤 종류의 객체가 연결되어 있는지를 컴파일러는 알 수 없다.(즉 컴파일 타임 체크를 할 수 없다.)
3.  개발자가 조심히 잘 사용해야 하고, 명시적 타입 캐스팅을 해줘야 한다.
4.  결론적으로, 타입 안정성이 떨어지고 번거로운 형변환을 해야 한다.

그럼 제네릭을 사용한 버전으로 위의 예제를 다시 들여다보겠습니다.   

## 제네릭이 있을 때...

### "다양한 타입의 객체를 다루는 메서드나 클래스"

JDK5 버전 이상에서는 리모컨을 다음과 같이 정의할 수 있습니다.   

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

위의 선언에서 Device를 **타입 변수(type variable)이라고** 하며 이것이 Java Generic의 핵심입니다. (타입, 즉 클래스에 대한 정보를 받을 수 있는 변수라는 의미에서 지어진 이름인 것 같습니다.)   
이렇게 정의된 리모컨은   

```java
Tv tv1 = new Tv("티비1");
Radio radio1 = new Radio("라디오1");
RemoteController<Tv> tvRemoteController1 = new RemoteController<Tv>(tv1);
RemoteController<Radio> radioRemoteController1 = new RemoteController<Radio>(radio1);
```

와 같은 형태로 리모컨 객체를 만들어낼 수 있고   
RemoteController\<Tv\> tvRemoteController1 = new RemoteController\<Tv\>(tv1);   
이 리모컨 객체에 한해서 마치 리모컨 클래스가   

```java
public class RemoteController {
    private Tv connectedDevice;
 
    public RemoteController(Tv connectedDevice) {
        this.connectedDevice = connectedDevice;
    }
 
    public Tv getConnectedDevice() {
        return connectedDevice;
    }
}
```

이렇게 정의된 것처럼 동작합니다. 이때 RemoteController\<Tv\>의 Tv를 **parameterized type이라고** 하며 타입 변수 Device에 실제 타입 Tv가 적용됐다 라고 생각하시면 됩니다.   
(JDK가 실제로 이렇게 동작한다는 건 아닙니다 ^^;; 이 부분은 나중에 type erasure를 다루면서 상세히 얘기해 보겠습니다.)   

### "컴파일 타임 타입 체크를 가능하게 하여 타입 안정성을 높이고 형 변환의 번거로움을 줄여주는 것"

위와 같이 제네릭 클래스로 선언된 RemoteController는 아래와 같이 사용할 수 있습니다.   

```java
Tv connectedTv = tvRemoteController1.getConnectedDevice();
System.out.println(connectedTv.getTitle());
 
Radio connectedRadio = radioRemoteController1.getConnectedDevice();
System.out.println(connectedRadio.getName());
```

제네릭 RemoteController를 사용하는 개발자의 사고의 흐름은 이렇습니다.   

1.  tvRemoteController1 참조 변수의 선언부를 확인한다. 이 리모컨에 연결된 디바이스가 Tv임이 명시되어 있다.(RemoteController\<Tv\> tvRemoteController1 = new RemoteController\<Tv\>(tv1);)
2.  번거로운 형 변환 없이 tvRemoteController1.getConnectedDevice()를 Tv 변수로 받는다. 컴파일러도 타입 변수에 Tv가 할당되었음을 알기 때문에 태클을 걸지 않는다.
3.  radioRemoteController1에 대해서도 같은 과정을 반복한다.

물론 이번에도 개발자는 같은 실수를 할 수 있습니다.   

```java
Radio connectedRadio = tvRemoteController1.getConnectedDevice();
System.out.println(connectedRadio.getName());
 
Tv connectedTv = radioRemoteController1.getConnectedDevice();
System.out.println(connectedTv.getTitle());
```

하지만 이 코드는 컴파일되지 않습니다.   
사고를 예방한 경위는 이렇습니다.   

1.  개발자의 기억이 왜곡되어(?) tvRemoteController1에 Radio객체가 연결되어 있다고 생각했다.
2.  컴파일러가 tvRemoteController1.getConnectedDevice()가 Tv를 리턴함을 알고 있다.
3.  Tv ↔ Radio는 문법적으로 타입 변환이 안되기 때문에 컴파일러가 컴파일 오류를 낸다.

### 위의 사용 예제를 정리해보면...

1.  RemoteController는 여러 종류의 객체(Tv, Radio...)를 다룰 수 있다.
2.  RemoteController 객체마다 어떤 타입의 객체가 연결되어 있는지 컴파일러가 알 수 있다.
3.  타입 캐스팅을 할 필요가 없으며, 개발자가 실수를 하더라도 컴파일러가 알려 준다.
4.  결과적으로 좀 더 Type-Safe 하고 간결한 코드를 작성할 수 있다.

덧붙여, Type-Safe하고 사용하기 편리한 형태로 클래스/메서드를 제공할 수 있다는 것은 결국 높은 코드 재사용성이라는 강력한 추가 이점을 발생시키기도 합니다.   
지금까지 Java Generic의 기본적인 개념에 대해 예제를 통해 알아보았습니다. 다음 포스팅에서는 Java Generic과 관련된 좀 더 심도 있는 개념과 응용 방법에 대해서 알아보겠습니다.   