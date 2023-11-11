Java Generic - 활용
==============================
![Java 로고](https://github.com/sh8121/tech-blog/assets/20632477/cbd53d33-6261-4ed2-9521-26ea826b8644)

Java Generic 시리즈 마지막 포스팅입니다.   
이번에는 Java Generic 을 활용하는 여러 가지 상황들에 대해 살펴보겠습니다.   
이번 포스팅에서도 [개념편](https://velog.io/@sh8121/Java-Generic-%EC%9D%84-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-%EA%B0%9C%EB%85%90%ED%8E%B8), [심화편](https://velog.io/@sh8121/Java-Generic-%EC%9D%84-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-%EC%8B%AC%ED%99%94%ED%8E%B8) 에서 사용했던 예제를 활용해보겠습니다.   
```java
public class Electronics {
    private String manufacturer;

    public Electronics(String manufacturer) {
        this.manufacturer = manufacturer;
    }

    public String getManufacturer() {
        return manufacturer;
    }
}

public class Tv extends Electronics {
    private String title;

    public Tv(String manufacturer, String title) {
        super(manufacturer);
        this.title = title;
    }

    public String getTitle() {
        return title;
    }
}

public class Radio extends Electronics {
    private String name;

    public Radio(String manufacturer, String name) {
        super(manufacturer);
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

## 1. Collections 
Java Application 을 개발하면서 Generic 을 가장 흔하게 접하는 경우는 아마도 Collection API 를 사용하면서 일 것 입니다.   
Java Collection API 의 최상위 계층이라고 할 수 있는 Collection Interface 와 Map Interface 만 하더라도   
```java
public interface Collection<E> extends Iterable<E> {
    ...
}

public interface Map<K, V> {
    ...
}
```
위처럼 Generic 을 기반으로 정의가 되어 있는 걸 볼 수 있습니다.   
결국 Java Collection API 전체가 Generic 을 기반으로 동작하는데   
```java
List<Tv> tvList = new ArrayList<>();
tvList.add(new Tv("제조사A", "티비A"));
tvList.add(new Tv("제조사A", "티비B"));
tvList.add(new Tv("제조사B", "티비C"));
//tvList.add(new Radio("제조사A", "라디오A")); // Compile Error

List<Radio> radioList = new ArrayList<>();
radioList.add(new Radio("제조사A", "라디오A"));
radioList.add(new Radio("제조사B", "라디오B"));
radioList.add(new Radio("제조사B", "라디오C"));
//radioList.add(new Tv("제조사A", "티비A")); // Compile Error

for (Tv tv : tvList) {
    System.out.println("tv.title = " + tv.getTitle());
}

for (Radio radio : radioList) {
    System.out.println("radio.name = " + radio.getName());
}
```
위처럼 Compile Time 에 Type Check 를 위해 사용하는 것이 가장 기본적인 사용법입니다.   
물론 아래와 같이 개발하는 것도 여전히 가능은 합니다.   
```java
List tvList = new ArrayList();
tvList.add(new Tv("제조사A", "티비A"));
tvList.add(new Tv("제조사A", "티비B"));
tvList.add(new Tv("제조사B", "티비C"));
tvList.add(new Radio("제조사A", "라디오A"));

for (Object o : tvList) {
    Tv tv = (Tv)o; //ClassCastException
    System.out.println("tv.title = " + tv.getTitle());
}
```
Runtime 에 ClassCastException 이 발생할 수 있다는 점을 감수한다면 말이죠.   

여기서 잠시 변성(variance)이라는 개념에 대해서 짚고 넘어가 보겠습니다.   
[심화편](https://velog.io/@sh8121/Java-Generic-%EC%9D%84-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-%EC%8B%AC%ED%99%94%ED%8E%B8) 에서 PECS 를 설명하는 과정에서 Java Generic 은 불공변(invariance) 의 특징을 갖는다고 언급했었는데요, invariance 가 variance 의 종류 중 하나입니다.   
즉, 변성(variance)은 '서로 다른 타입 간에 어떤 관계가 있는가' 에 대한 개념이고 변성의 한 종류로서 불공변(invariance), '서로 다른 타입 간에 어떤 관계도 없는 것'이 있는 거죠. 참고로 변성의 다른 종류로는 공변(covariance), 반공변(contravariance), 이변(bivariance) 등이 있습니다.   
변성의 종류를 다른 방식으로 나누는 방법도 있는데요, 바로 '변성이 언제 정해지는가' 를 기준으로 나누는 방식입니다.   
```java
List<Tv> tvList;
tvList = new ArrayList<Tv>();
//tvList = new ArrayList<Radio>(); //Compile Error
```
위 코드에서 tvList 라는 변수가 Type Parameter 로 Tv 를 사용한다는 것과 때문에 ArrayList&lt;Tv&gt; 객체는 받을 수 있지만, ArrayList&lt;Radio&gt; 객체는 받을 수 없다는 것은 List Interface 를 사용하는 시점에 결정됩니다. List 가 정의되는 시점이 아니고요.   
이러한 변성 결정 방식을 사용지점변성(use-site variance) 라고 합니다.   
반면에 아래의 코드처럼 변성을 결정할 수도 있습니다.   
```java
public class ElectronicsList<E extends Electronics> extends ArrayList<E> {
}
```
위 ElectronicsList 클래스는 Type Parameter 로 Electronics 하위 클래스만을 받을 수 있으며, 이는 ElectronicsList 클래스를 선언하는 시점에 결정됩니다.   
```java
// ElectronicsList<Object> electronicsList; // Compile Error
```
위처럼 클래스를 사용하는 것이 원천적으로 차단되는 거죠.   
이러한 변성 결정 방식을 선언지점변성(declaration-site variance) 이라고 합니다.   

다시 예제로 돌아와서 Collection 에서 Generic 을 좀 더 활용해보겠습니다.   
Electronics 하위 객체 들의 제조사(manufacturer) 를 출력해야 하는 UseCase 가 있다고 가정해보겠습니다.   
가장 쉽게 생각할 수 있는 방법은 아래와 같은 방법일 것 입니다.   
```java
private static void printManufacturer(List<Electronics> electronicsList) {
    for (Electronics electronics : electronicsList) {
        System.out.println("electronics.manufacturer = " + electronics.getManufacturer());
    }
}
```
위 메서드를 아래와 같이 사용할 수 있습니다.   
```java
List<Electronics> electronicsList = new ArrayList<>();
electronicsList.add(new Tv("제조사1", "티비1"));
electronicsList.add(new Tv("제조사2", "티비2"));
electronicsList.add(new Radio("제조사3", "라디오1"));
electronicsList.add(new Radio("제조사4", "라디오2"));

printManufacturer(electronicsList);
```
상속 관계에 의해 electronicsList 는 Tv 객체, Radio 객체를 모두 담을 수 있으며, 각각의 제조사를 출력할 수 있습니다.   
하지만 아래의 코드는 동작하지 않습니다.   
```java
List<Tv> tvList = new ArrayList<>();
List<Radio> radioList = new ArrayList<>();
tvList.add(new Tv("제조사1", "티비1"));
tvList.add(new Tv("제조사2", "티비2"));
radioList.add(new Radio("제조사3", "라디오1"));
radioList.add(new Radio("제조사4", "라디오2"));

//printManufacturer(tvList); // Compile Error
//printManufacturer(radioList); // Compile Error
```
Generic 의 Invariance 때문에 printManufacturer 메서드는 List&lt;Tv&gt; 나 List&lt;Radio&gt; 를 파라미터로 받지 못합니다.   
그럼 위 메서드에 PECS 중 Producer Extends 를 적용해보겠습니다.   
```java
private static void printManufacturer(List<? extends Electronics> electronicsList) {
    for (Electronics electronics : electronicsList) {
        System.out.println("electronics.manufacturer = " + electronics.getManufacturer());
    }
}
```
이번에는 아래의 코드가 정상 동작 합니다.   
```java
List<Tv> tvList = new ArrayList<>();
List<Radio> radioList = new ArrayList<>();
tvList.add(new Tv("제조사1", "티비1"));
tvList.add(new Tv("제조사2", "티비2"));
radioList.add(new Radio("제조사3", "라디오1"));
radioList.add(new Radio("제조사4", "라디오2"));

printManufacturer(tvList);
printManufacturer(radioList);
```
이렇게 와일드카드(?) 를 적절하게 적용하는 것 만으로 Generic 의 활용도를 높일 수 있습니다.   

지금까지 Collection 에서 Generic 을 활용하는 예제를 살펴보았는데요, [개념편](https://velog.io/@sh8121/Java-Generic-%EC%9D%84-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-%EA%B0%9C%EB%85%90%ED%8E%B8)과 [심화편](https://velog.io/@sh8121/Java-Generic-%EC%9D%84-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-%EC%8B%AC%ED%99%94%ED%8E%B8)에서 Generic 을 이해하기 위해 사용했던 코드들과 큰 차이점을 못 느끼셨을 수도 있겠습니다. 이번에는 좀 더 복잡한 활용 예제를 살펴보겠습니다.   

## 2. Type Token
이번에는 Type Parameter 를 통해 객체를 저장하고 조회하는 기능을 구현해 보겠습니다.   
예를 들면 이런 코드를 작성하고 싶은거죠.   
```java
Tv tv = new Tv("제조사1", "티비");
Radio radio = new Radio("제조사2", "라디오");

ElectronicsStore store = new ElectronicsStore();
store.<Tv>save(tv);
store.<Radio>save(radio);

Tv findTv = store.<Tv>find();
Radio findRadio = store.<Radio>find();
```
각 Type 별로 단일 객체를 저장하고 조회하는 코드입니다. 코드 상의 ElectronicsStore 와 같은 Component 는 Singleton 객체 관리 Container 용도나, 특정 타입의 가장 최신 객체 관리 용도로 사용될 수 있을 것 입니다.   
사용하는 입장에서 기대해는 ElectronicsStore 는 아마 이렇게 생겼을 것 입니다.   
```java
class ElectronicsStore {
    public <T extends Electronics> void save(T obj) {
        //Type Parameter T 로 클래스 정보를 조회한 후 
        //(클래스정보, 객체) 로 저장        
    }

    public <T extends Electronics> T find() {
        //Type Parameter T 로 클래스 정보를 조회한 후
        //클래스 정보로 저장된 객체를 반환
    }
}
```
하지만 ElectronicsStore 를 위와 같은 선언부로 제공하는 것은 불가능 합니다. 클래스 정보를 조회한다는 것은 결국 Class&lt;T&gt; 객체를 찾아낸다는 것을 의미하죠. 결국 위 코드를 동작하게 하려면 Type Parameter T 로 Class&lt;T&gt; 객체를 Runtime 에 찾아내야 하는데 이게 불가능 합니다. Type Erasure 때문에 Type Parameter 정보가 Runtime 에는 없어져 버리기 때문이죠.   
대신 아래와 같은 형태로는 제공이 가능합니다.   
```java
class ElectronicsStore {
    private final Map<Class<? extends Electronics>, Object> store = new HashMap<>();

    public <T extends Electronics> void save(T obj, Class<T> clazz) {
        store.put(clazz, obj);
    }

    public <T extends Electronics> T find(Class<T> clazz) {
        return clazz.cast(store.get(clazz));
    }
}
```
단순히 관리하고 싶은 객체의 타입 정보 즉, Class&lt;T&gt; 객체를 파라미터로 직접 받음으로써 문제를 해결할 수 있습니다.   
Class&lt;T&gt; 자체도 제네릭 타입이라는 점을 주목해주세요. 그 점 때문에 위 구현체를 사용하는 입장에서 제네릭의 장점을 살린 코딩을 할 수 있습니다.   
```java
//Type Safety
store.<Tv>save(tv, Tv.class);
store.<Radio>save(radio, Radio.class);

//Type Casting
Tv findTv = store.<Tv>find(Tv.class);
Radio findRadio = store.<Radio>find(Radio.class);

//Type Inference
// store.save(tv, Tv.class);
// store.save(radio, Radio.class);

// Tv findTv = store.find(Tv.class);
// Radio findRadio = store.find(Radio.class);
```
정리해보면 위의 ElectronicsStore 는 Class 객체 즉, 타입 정보를 기반으로 객체를 관리하고 있습니다. 이처럼 타입 정보를 기반으로 코드를 작성하는 기법(혹은 이 때의 타입정보)을 Type Token 이라고 합니다. 그리고 위에서 보셨다시피 Type Token 은 Generic 과 상호보완적으로 사용할 수 있는 대표적인 프로그래밍 기법 중 하나입니다.   

## 3. Super Type Token
위에서 다룬 Type Token 은 매우 유용한 기법이지만 아쉽게도 한계가 있습니다. ElectronicsStore 에 Tv, Radio... 의 리스트를 관리하는 요구사항이 추가되었다고 가정해 보겠습니다.   
우선 ElectronicsStore 가 좀 더 포괄적으로 타입을 담을 수 있도록 변성을 조정하겠습니다.   
```java
class ElectronicsStore {
    private final Map<Class<?>, Object> store = new HashMap<>();

    public <T> void save(T obj, Class<T> clazz) {
        store.put(clazz, obj);
    }

    public <T> T find(Class<T> clazz) {
        return clazz.cast(store.get(clazz));
    }
}
```
사용하는 입장에서는 아마도 이런 식으로 코드를 작성하기를 기대할 것 입니다.   
```java
List<Tv> tvList = new ArrayList<Tv>(){{add(new Tv("제조사1", "티비"));}};
List<Radio> radioList = new ArrayList<Radio>(){{add(new Radio("제조사2", "라디오"));}};

ElectronicsStore store = new ElectronicsStore();
store.save(tvList, List<Tv>.class); //Compile Error
store.save(radioList, List<Radio>.class); //Compile Error

List<Tv> findTvList = store.find(List<Tv>.class); //Compile Error
List<Radio> findRadioList = store.find(List<Radio>.class); //Compile Error
```
하지만 위 코드는 동작하지 않습니다. Type Erasure 에 의해 바이트코드에서 제네릭 관련 정보가 없어지기 때문에 Runtime 에 List&lt;Tv&gt;.class 라는 Class 객체는 존재하지 않습니다. 단지 List.class 만이 존재할 뿐이죠.   
```java
List<Tv> tvList = new ArrayList<Tv>(){{add(new Tv("제조사1", "티비"));}};
List<Radio> radioList = new ArrayList<Radio>(){{add(new Radio("제조사2", "라디오"));}};

ElectronicsStore store = new ElectronicsStore();
store.save(tvList, List.class);
store.save(radioList, List.class);

List<Tv> findTvList = store.find(List.class);
List<Radio> findRadioList = store.find(List.class);
```
위 코드는 Compile 도 성공하고 실행도 정상적으로 됩니다. 하지만 우리가 기대한대로 동작하지는 않습니다.   
```java
System.out.println(findTvList.equals(findRadioList)); // true
```
결과를 보면 findTvList 와 findRadioList 가 같은 객체로 나옵니다. 이게 무슨 상황이냐면 tvList 와 radioList 를 저장할 때 Key로 사용한 List.class 가 같은 객체여서 radioList 가 tvList 를 덮어 버린겁니다.   
```java
for (Tv tv : findTvList) { // ClassCastException
    System.out.println("tv = " + tv);
}
```
당연히 위 코드는 ClassCastException 이 발생합니다.   
리스트를 관리하는 요구사항을 어떻게 해결해야 할까요? 제네릭 타입 정보가 없어지는게 문제이니 Runtime 에도 제네릭 타입 정보가 유지되면 되지 않을까요?   
```java
class TvList extends ArrayList<Tv> {}
```
```java
List<Tv> tvList = new TvList();
ParameterizedType parameterizedType = (ParameterizedType)tvList.getClass().getGenericSuperclass();
System.out.println(parameterizedType); //java.util.ArrayList<org.example.generic.Tv>
System.out.println(parameterizedType.getActualTypeArguments()[0]); //class org.example.generic.Tv
```
위처럼 제네릭 타입을 상속받아서 클래스를 정의한 경우 제네릭 타입 정보가 유지된다고 [심화편](https://velog.io/@sh8121/Java-Generic-%EC%9D%84-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-%EC%8B%AC%ED%99%94%ED%8E%B8) 에서 간단히 다뤘었는데요. 이번 기회에 코드를 좀 더 들여다 보고 넘어가겠습니다.   
코드에서 보시는 것처럼 TvList Class 객체의 getGenericSuperclass() 는 ParameterizedType 이라는 타입의 객체를 반환하는데요, 이게 바로 ArrayList&lt;Tv&gt; 처럼 'Runtime 에도 유지되는 제네릭 타입 정보' 를 위한 인터페이스 입니다. 참고로 TvList 처럼 제네릭 타입을 상속받은게 아니라면 getGenericSuperclass() 는 단순히 부모 타입의 Class 객체를 반환합니다. 그래서 getGenericSuperclass() 의 선언부를 보면 Class&lt;T&gt; 와 ParameterizedType 의 공통 부모인 Type 을 반환합니다.   
위처럼 Reflection 을 활용하는 것은 익명 클래스에서도 동일하게 가능합니다.   
```java
List<Radio> radioList = new ArrayList<Radio>(){};
ParameterizedType parameterizedType = (ParameterizedType)radioList.getClass().getGenericSuperclass();
System.out.println(parameterizedType); //java.util.ArrayList<org.example.generic.Radio>
System.out.println(parameterizedType.getActualTypeArguments()[0]); //class org.example.generic.Radio
```
radioList 객체의 타입은 ArrayList&lt;Radio&gt; 를 상속받은 익명클래스이므로 위 코드가 정상 동작합니다.   
심지어는 제네릭이 중첩된 경우에도 동작하는데,   
```java
List<List<Radio>> radioNestedList = new ArrayList<List<Radio>>(){};
ParameterizedType parameterizedType = (ParameterizedType)radioNestedList.getClass().getGenericSuperclass();
System.out.println(parameterizedType); //java.util.ArrayList<java.util.List<org.example.generic.Radio>>
ParameterizedType nestedParameterizedType = (ParameterizedType)parameterizedType.getActualTypeArguments()[0];
System.out.println(nestedParameterizedType); //java.util.List<org.example.generic.Radio>
```
위 코드가 정상 동작하는 것을 알 수 있습니다.   
그럼 이러한 제네릭의 특성을 이용하여 Type Token 의 한계를 돌파해보겠습니다.   
```java
abstract class TypeReference<T> {
    private Type type;

    public TypeReference() {
        this.type = ((ParameterizedType)getClass().getGenericSuperclass()).getActualTypeArguments()[0];
    }

    public Type getType() {
        return type;
    }
}
```
Class 객체를 대신하여 Type Token 으로 사용할 클래스 입니다. 정확히는 클래스의 멤버변수인 type 이 Type Token 으로 사용이 됩니다. TypeReference&lt;T&gt; 는 추상클래스이기 때문에 TypeReference&lt;T&gt; 를 상속받은 클래스만이 인스턴스화 될 수 있고, 해당 객체의 type 변수는 언제나 자신의 부모클래스 즉, TypeReference&lt;T&gt; 의 'Runtime 제네릭 타입 정보' 를 갖고 있습니다.   
```java
TypeReference<Tv> tvTypeReference = new TypeReference<Tv>() {};
TypeReference<Radio> radioTypeReference = new TypeReference<Radio>() {};
TypeReference<List<Tv>> tvListTypeReference = new TypeReference<List<Tv>>() {};
TypeReference<List<Radio>> radioListTypeReference = new TypeReference<List<Radio>>() {};
System.out.println(tvTypeReference.getType()); //class org.example.generic.Tv (Class 객체)
System.out.println(radioTypeReference.getType()); //class org.example.generic.Radio (Class 객체)
System.out.println(tvListTypeReference.getType()); //java.util.List<org.example.generic.Tv> (ParameterizedType 객체)
System.out.println(radioListTypeReference.getType()); //java.util.List<org.example.generic.Radio> (ParameterizedType 객체)
```
```java
TypeReference<Tv> tvTypeReference1 = new TypeReference<Tv>() {};
TypeReference<Tv> tvTypeReference2 = new TypeReference<Tv>() {};
System.out.println(tvTypeReference1.getType() == tvTypeReference2.getType()); // true

TypeReference<List<Tv>> tvListTypeReference1 = new TypeReference<List<Tv>>() {};
TypeReference<List<Tv>> tvListTypeReference2 = new TypeReference<List<Tv>>() {};
System.out.println(tvListTypeReference1.getType() == tvListTypeReference2.getType()); // false
System.out.println(tvListTypeReference1.getType().equals(tvListTypeReference2.getType())); // true
System.out.println(tvListTypeReference1.getType().hashCode()); //1308278359
System.out.println(tvListTypeReference2.getType().hashCode()); //1308278359
```
getType() 이 Class 객체를 반환하든, ParameterizedType 객체를 반환하든 equals(), hashCode() 메서드는 같은 제네릭 타입 정보에 대해 '같다' 고 결론내리는 점을 주목해주세요.   
그럼 이번에는 TypeReference&lt;T&gt; 를 사용할 수 있게 ElectronicsStore 를 바꿔보겠습니다.   
```java
class ElectronicsStore {
    private final Map<Type, Object> store = new HashMap<>();

    public <T> void save(T obj, TypeReference<T> typeReference) {
        store.put(typeReference.getType(), obj);
    }

    public <T> T find(TypeReference<T> typeReference) {
        return (T)store.get(typeReference.getType());
    }
}
```
TypeReference&lt;T&gt; 를 파라미터로 받아서 내부적으로는 해당 객체의 type 변수를 Key 로 사용하게끔 했습니다.   
```java
ElectronicsStore store = new ElectronicsStore();

Tv tv = new Tv("제조사1", "티비1");
List<Tv> tvList = new ArrayList<>();
tvList.add(new Tv("제조사2", "티비2"));

store.save(tv, new TypeReference<Tv>() {});
store.save(tvList, new TypeReference<List<Tv>>() {});
Tv findTv = store.find(new TypeReference<Tv>() {});
List<Tv> findTvList = store.find(new TypeReference<List<Tv>>() {});
System.out.println(tv == findTv); //true
System.out.println(tvList == findTvList); //true
```
사용하는 입장에서는 목표했던 바를 달성하면서, 여전히 제네릭의 강력한 기능을 지원받을 수 있습니다. save 와 find 호출 시 서로 다른 TypeReference 객체를 전달하더라도 이미 저장한 객체를 잘 찾아오는 것을 확인할 수 있습니다.   
이렇게 Type Token 의 한계를 극복하고 제네릭 타입에 대해서도 타입 정보 기반으로 코드를 작성할 수 있도록 확장한 기법(혹은 이 때 사용하는 타입 즉, TypeReference) 을 Super Type Token 이라고 합니다. Super Type Token 은 이미 많은 프레임워크와 라이브러리에서 사용이 되고 있는데요. [Jackson 의 TypeReference](https://fasterxml.github.io/jackson-core/javadoc/2.2.0/com/fasterxml/jackson/core/type/TypeReference.html) 나 [Spring 의 ResolvableType](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/ResolvableType.html) 등이 대표적인 케이스입니다.   

## 맺음말
긴 글 읽어주셔서 감사합니다. 3편의 포스팅을 통해, Java Generic 의 탄생 배경과 기본 개념부터 주의해야 하는 심화 개념 및 활용 사례에 이르기까지 전반적인 내용을 살펴봤습니다.    
제가 쓴 시리즈는 주로 Java Generic 을 이해하고 사용하기 위해 필요한 주요 맥락을 설명하는데 집중하고 있습니다. 이 글의 통해 큰 맥락을 이해하고, 구체적인 문법을 별도로 공부한다면 제네릭을 한층 더 깊이 이해하고 사용하는 자바 개발자가 될 수 있지 않을까 하는 기대와 바람을 적어 봅니다.   
끝으로 저자 개인적으로도 매우 뜻깊은 시간이었다는 소회를 밝히며 길었던 Java Generic 포스팅을 마무리 하도록 하겠습니다.   

## 참고
https://www.youtube.com/watch?v=01sdXvZSjcI&ab_channel=TobyLee   
https://yangbongsoo.gitbook.io/study/super_type_token   
