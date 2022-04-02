###### tags: `코드 파악`

# Java

## 제네릭

### 명명 규약

T : Type의 약자, 자료형이나 클래스형을 의미, 제네릭에서 기본 자료형을 
타입으로 사용할 수 없어서 여기서 래퍼(Wrapper) 클래스를 의미 
E : Element의 약자, 컬렉션 프레임워크를 사용할 때 각 객체를 지칭하는 의미 
K : Key의 약자, 키와 값이라는 쌍으로 이루어진 형태로 키를 의미 
V : Value의 약자, 키와 값이라는 쌍으로 이루어진 형태로 값을 의미
N : Number의 약자, 수치 계열의 의미로 사용, 여러 개를 사용할 때 각 타입 매개변수 위에 2,3,4 등의 숫자를 붙여 사용 


### 와일드 카드
와일드카드(wild card)란 이름에 제한을 두지 않음을 표현하는 데 사용되는 기호를 의미한다.
자바의 제네릭에서는 물음표(?) 기호를 사용하여 이러한 와일드카드를 사용할 수 있다.

```java=
<?>           // 타입 변수에 모든 타입을 사용할 수 있음.
<? extends T> // T 타입과 T 타입을 상속받는 자손 클래스 타입만을 사용할 수 있음.
<? super T>   // T 타입과 T 타입이 상속받은 조상 클래스 타입만을 사용할 수 있음.
```

https://thecodinglog.github.io/java/2020/12/15/java-generic-wildcard.html


## Stream
스트림(Stream)은 자바 8에서 추가된 기능으로 함수형 인터페이스인 람다(lambda)를 활용할 수 있는 기술이다. 예전에는 배열이나 컬렉션을 반복문을 순회하면서 요소를 하나씩 꺼내 여러가지 코드를(예를 들어 if 조건문 등) 섞어서 작성했다면 스트림과 람다를 이용하여 코드의 양을 대폭 줄이고 조금 더 간결하게 코드를 작성할 수 있다.
스트림은 크게 3가지 단계로 동작한다. 컬렉션이나 배열 등으로부터 스트림을 생성하는 작업(Stream Source), 스트림을 필터링하거나 요소를 알맞게 변환하는 중간 연산(Intermediate Operations), 마지막으로 최종적인 결과를 도출하는 단말 연산(Terminal Operations)으로 나뉜다.

### 스트림 생성
* 컬랙션으로 생성

```java=
List<String> = List.of("hwang","kyeongha");
Stream<String> stream = list.stream();
```
* 배열로 생성

```java=
Stream<String> stream = Arrays.stream(arr);
```

* 병렬 스트림 생성
```java=
Stream<String> stream = list.parallelStream();
```

* Stream.builder()로 생성
```java=
Stream<String> stream = Stream.<String>builder().add("df").build();
```
* Stream.iterate()로 생성
```java=
Stream<Integer> stream = Stream.iterate(0,x -> x+1).limit(3);
```
* 외에 generate(), concat(), empty() 등으로 생성 가능하다.

### stream 중간 연산

 중간 연산의 특징은 반환 값으로 다른 스트림을 반환하기 때문에 이어서 호출하는 메서드 체이닝이 가능하다. 그리고 모든 중간 연산을 합친 다음에 합쳐진 연산을 마지막으로 한 번에 처리한다.
 
* filter 메서드로 필터링

filter 메서드로 스트림 내 요소들을 조건에 맞게 필터링할 수 있다. 메서드의 인자인 Predicate 인터페이스는 test 라는 추상 메서드를 정의하는데, 이는 제네릭 형식의 객체를 인수로 받아 boolean 값을 반환한다.
    
```java=
List<String> list = List.of("Hwang", "KH");
list.stream().filter(s -> s.length() == 5);

// without lambda expression
list.stream().filter(new Predicate<String>() {
    @Override
    public boolean test(String s) {
        return s.length() == 5;
    }
});
    
```
* map으로 변환

```java=

list.stream().map(s -> s.toLowerCase());

// without lambda expression
list.stream().map(new Function<String, String>() {
    @Override
    public String apply(String s) {
        return s.toLowerCase();
    }
});
```

* flatmap 매서드로 단일 스트림 변환
```java=

List<String> list1 = List.of("h", "w");
List<String> list2 = List.of("a", "n");
List<List<String>> combindedList = List.of(list1, list2);

List<String> streamByList = combinedList.stream()
    .flatMap(list-> list.stream())
    .collect(Collectors.toList());

sout(streamByList) // h,w,a,n
```

* distinct 메서드로 중복 제거
```java=
    Foo foo1 = new Foo("123");
    Foo foo2 = new Foo("123");
    List<Foo> list = List.of(foo1, foo2, foo1);
    
    // bar: 123
    // bar: 123
    list.stream().distinct()
        .forEach(System.out::println);
```

* sorted 메소드로 정렬

기본형 특화 스트림의 경우 sorted메서드에 인자를 넘길 수 없다. 따라서 boxed 메서드를 이용해 객체 스트림으로 변환 후 사용해야 한다.
```java=
// 2, 1, 0
IntStream.range(0, 3)
        .boxed() // boxing
        .sorted(Comparator.reverseOrder());
```

* peek 메서드로 각각 요소에 연산 수행
peek 메서드는 스트림 내의 각각의 요소를 대상으로 특정 연산을 수행하게 한다. 원본 스트림에서 요소를 소모하지 않기 때문에 중간 연산 사이의 결과를 확인할 때 유용하다. 주의할 점은 peek 연산은 단말 연산이 수행되지 않으면 실행조차 되지 않는다.

```java=
List<Integer> otherList = new ArrayList<>();
List.of(1, 2, 3).stream()
        .limit(2)
        .peek(i -> {
            otherList.add(i);
        })
        .forEach(System.out::println);

// 1, 2
System.out.println(otherList);

// 단말 연산인 forEach가 없으면 otherList는 비어있다.
```
* skip, limit 메서드로 개수 제한하기
```java=
List<String> list = List.of("a", "b", "c").stream()
        .limit(2).collect(Collectors.toList());

// a, b
System.out.println(list);
```

### 스트림 종료 연산

* forEach : 메서드 순회
* forEachOrdered : 병렬 스트림을 순서대로 순회

* Reduce
reduce 연산을 이용해 모든 스트림 요소를 처리하여 결과를 구할 수 있다.이 메서드는 아래와 같이 세가지 형태로 오버로딩 되어있다.
```java=
// 형태1
Optional<T> reduce(BinaryOperator<T> accumulator); 

// 형태2
T reduce(T identity, BinaryOperator<T> accumulator);

//형태3
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator,
            BinaryOperator<U> combiner);
```

* collect
스트림을 list,Set과 같은 다른 형태의 결과로 반환한다.
```java=
List<String> nameList = list.stream()
        .map(Food::getName) // name 얻기
        .collect(Collectors.toList()); // list로 수집
```

## Functional 인터페이스

전달값을 받아 다른(또는 같은) 타입의 리턴값을 산출하는 abstract method(추상 메서드)를 가지는 인터페이스이다.

Function 클래스를 살펴보면 아래와 같이 되어있다.
```java=
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

* Function<T, R> : 전달자 T 를 받아서 R 타입으로 리턴해준다. (T -> R)

* R apply(T t)
Function 클래스가 가지는 추상메서드이고, Function을 실행하기 위해 호출되는 메서드이다.
T 타입의 전달값을 받고, R 타입의 리턴값을 산출한다.

* identity()
function을 통해서 전달되는 전달자와 리턴되는 값의 타입과 값이 같을 때 사용할 수 있는 메서드이다.
이렇게 전달자와 리턴값의 타입과 값이 같은것을 IdentityFunction 이라고 한다. 

* ex) String-> Integer
```java=
        Function<String, Integer> toInt = new Function<String, Integer>() {
            @Override
            public Integer apply(String value) {
                return Integer.parseInt(value);
            }
        };
        Integer resultNum = toInt.apply("10");

///람다식으로 변환
Function<String, Integer> toInt2 =  (str -> Integer.parseInt(str));
Integer resultNum2 = toInt2.apply("10");

```

## 메소드 레퍼런스
메소드 레퍼런스는 람다 표현식을 더 간단하게 표현하는 방법이다.
다음은 람다식으로 hello를 출력하는 코드다.

```java=
Consumer<String> func = text -> System.out.println(text);
func.accept("Hello");
```

위의 람다식은 다음과 같이 System.out::println과 같은 메소드 레퍼런스로 표현 가능하다. String인자 1개를 받아 void를 리턴하는 함수라는 의미가 생략되어있다.

```java=
Consumer<String> func = System.out::println;
func.accept("Hello");
```

메소드 레퍼런스는 ClassName::MethodName 형식으로 입력한다. 메소드를 호출하는 것이지만 괄호는 써주지 않고 생략한다.

위의 예제처럼 메소드 레퍼런스에는 많은 코드가 생략되어있기 때문에 사용하려는 ***메소드의 인자와 리턴 타입***을 알고 있어야 한다.

메소드 레퍼런서는 사용하는 패턴에 따라 다음과 같이 분류할 수 있다.
static, instance, constructor메소드 레퍼런스

* Static 메소드 레퍼런스
static method를 메소드 레퍼런스로 사용하는 케이스다.
```java=
interface Executable {
    void doSomething(String text);
}

public static class Printer {
    static void printSomething(String text){
        System.out.println(text);
    }
}

public static void main(String args[]){
    Executable exe = text -> Printer.printsomething(text);
    Executable exe2 = Printer::printSomething;

    exe.doSomething("do something");
    exe2.doSomething("do something");
    
    Consumer<String> consumer = Printer::pringSomething;
    
    consumer.accept("do something");
}
```
Consumer<클래스>는 자바에서 제공하는 함수형 인터페이스이다. 위의 예제 Executable과 같은 인터페이스를 매번 만들기 번거롭기 때문에 자바에서 기본적으로 인터페이스를 제공한다. `Consumer<String>` 은 String1개를 인자로 받아 void를 리턴한다.

다음 예제는 Stream과 메소드 레퍼런스를 함께 사용한다.
```java=
List<String> companies = Arrays.asList("h","w","a");
// 1. lambda expression
companies.stream().forEach(company -> System.out.println(company));
// 2. static method reference
companies.stream().forEach(System.out::println);
```

* instance 메소드 레퍼런스
instance 메소드 레퍼런스의 메소드는 static이 아니고 객체의 메소드를 의미한다.

우선 람다식으로 작성
```java=
public static class Company {
    String name;
    public Company(String name) {
        this.name = name;
    }

    public void printName() {
        System.out.println(name);
    }
}

public static void main(String args[]) {
    List<Company> companies = Arrays.asList(new Company("google"),
        new Company("apple"), new Company("samsung"));
    companies.stream().forEach(company -> company.printName());
}
// 실행 결과
// google
// apple
// samsung
```
위의 코드에서 `company->printName()` 은 람다식이다. 이것을 메소드 레퍼런스로 표현하면 Company::printName으로 쓸 수 있다. forEach에서 company객체가 전달될 것을 알기 때문에 이렇게 써주면 위처럼 인스턴스의 메소드를 호출해주는 의미로 이해된다.

```java=
companies.stream().forEach(Company::printName);
```

다음은 `String::length`를 사용하는 예제이다. 이 함수는 스트링 객체의 길이를 리턴해주는 instance메소드이다. mapToInt()가 String을 int로 변환해주길 예상하고 있다.

```java=
companies.stream()
    .mapToInt(String::length) //람다식 : company -> company.length()
    .forEach(System.out::println);
```
instance 메소드 레퍼런스는 Static 메소드 레퍼런스와 헷갈릴 수 있는데, 요구하는 인자와 리턴타입을 알고 있다면 어렵지 않게 사용할 수 있다.

* Constructor 메소드 레퍼런스
Constructor 메소드 레퍼런스는 Constructor를 생성해주는 코드이다.

다음 예제는 람다식으로 생성자를 호출하는 코드이다.
```java=
public static class Company {
    String name;
    public Company(String name) {
        this.name = name;
    }

    public void printName() {
        System.out.println(name);
    }
}

public static void main(String args[]) {
    List<String> companies = Arrays.asList("google", "apple", "google", "apple", "samsung");
    companies.stream()
            .map(name -> new Company(name))
            .forEach(company -> company.printName());
}
// 실행 결과
// google
// apple
// google
// apple
// samsung
```
위 코드를 메소드 레퍼런스를 사용하여 리팩토링한다. `Company::new`의 의미는 name->new Company(name)과 같다.
```java=
List<String> companies = Arrays.asList("google", "apple", "google", "apple", "samsung");
companies.stream()
        .map(Company::new)
        .forEach(Company::printName);
```

* 정리
메소드 레퍼런스는 자주 사용되는 패턴의 람다식을 간단하게 쓸 수 있는 방법이라고 정의할 수 있다. 특정 패턴의 람다식을 줄여서 쓴다고 생각하면 좋을것 같다.


## ImmutableMap
ImmutableMap은 변경 불가능한 Map 유형이다.
이는 Map의 데이터 선언후 고정되거나 일정하다는 것을 의미하며, 읽기 전용이다.
예를들어 맵에 요소를 추가, 삭제 및 업데이트 하려고 하면 UnsupportedOperationException이 발생한다. ImmutableMap은 null요소도 허용하지 않는다.
null 요소로 immutablemap을 만들려고 하면 nullpointerException이 발생한다.

* 초기화 방법
* 인수가 5개 미만일경우
```java=
ImmutableMap<String,String> myMap = ImmutableMap.of(
        "key1", "value1",
        "key2", "value2",
        "key3", "value3");
```
* 인수가 5개 초과

```java=
ImmutableMap<String,String> myMap = ImmutableMap.<String, String>builder()
    .put("key1", "value1")
    .put("key2", "value2")
    .put("key3", "value3")
    .put("key4", "value4")
    .put("key5", "value5")
    .put("key6", "value6")
    .put("key7", "value7")
    .put("key8", "value8")
    .put("key9", "value9")
    .build();
```


## sorted
```java=
@Override public int compareTo(Worker o) { 
    return Integer.compare(this.age, o.getAge()); }
//자신 먼저, 다음 비교대상이면 오름차순
```


## ConcurrentHashMap

Hashtable 클래스의 단점을 보완하면서 Multi-Thread 환경에서 사용할 수 있도록 나온 클래스가 바로 ConcurrentHashMap 이다.(JDK 1.5에서 검색과 업데이트시 동시성 성능을 높이기 위해서 나온 클래스이다.)

HashMap, Hashtable, ConcurrentHashMap 클래스 모두 Map의 기능적으로만 보면 큰 차이는 없다.

https://devlog-wjdrbs96.tistory.com/269


## 쓰레드 로컬

### 쓰레드 로컬 생성
다음은 ThreadLocal 을 생성하는 간단한 예제이다.
```java=
private ThreadLocal myThreadLocal = new ThreadLocal();
```
새로운 ThreadLocal객체를 생성했다. 이 코드는 오직 한 쓰레드 당 한 번의 실행을 허용하여, 다수의 쓰레드에 의한 실행에도 각 쓰레드는 자신들 각자의 ThreadLocal인스턴스를 가지게 된다. 두 쓰레드가 각자의 변수를 ThreadLocal객체에 세팅했다면, 쓰레드들은 서로 다른 ThreadLocal변수값을 가지게 되며, 오직 자신의 변수만 볼 수 있게 된다.

### 쓰레드 로컬 다루기
쓰레드로컬을 생성하면 다음과 같은 방법으로 쓰레드로컬에 값을 저장할 수 있다.
```java=
myThreadLocal.set("A thread local value");
```
다음으로 쓰레들로컬의 값을 읽는 방법은 다음과 같다.
```java=
String ThreadLocalValue = (String) myThreadLocal.get();
```
get() 메소드는 Object 타입 객체를 반환하고, set()메소드 역시 Object 타입을 파라미터로 받는다.

### 쓰레드 로컬 제네릭
쓰레드로컬은 변수 타입을 다루기 쉽도록, 제네릭으로 생성 가능하다.
```java=
private ThreadLocal mythreadLocal = new ThreadLocal<String>();

myThreadLocal.set("hello threadLocal");
String threadLocalValue = myThreadLocal.get();
```

### 쓰레드 로컬 변후 초기화
쓰레드로컬 객체에 세팅한 값은 오직 값을 세팅한 쓰레드만 접근할 수 있기 때문에, 모든 쓰레드가 사용할 수 있는 쓰레드로컬 초기값(디폴트)는 없다. 대신, 쓰레드로컬을 서브클래싱하고 initialValue()메소드를 오버라이딩하는 방법으로 쓰레드로컬의 초기값을 설정할 수 있다.


## 리플랙션

### 리플랙션 사용 설정

Method와 같은 reflection 클래스는 java.lang.reflect에 있다. 이 클래스를 사용하려면 세 단계를 거쳐야 한다. 첫 번째 단계는 조작하려는 java.lang.Class 클래스에 대한 객체를 얻는 것이다. java.lang.Class는 실행 중인 Java 프로그램에서 클래스와 인터페이스를 나타내는 데 사용된다.

Class 객체를 얻는 방법은 다음과 같다.
```java=
Class c = Class.forName("java.lang.String"); //String 에 대한 Class 객체를 가져온다. 
```


또 다른 접근 방식은 기본 유형에 대한 클래스 정보를 얻기 위해 다음을 사용하는 것이다. 
```java=
Class c = int.class; 또는 Class c = Integer.TYPE;  
```


후자의 접근 방식은 원시 타입에 대한 TYPE래퍼(예: Integer)의 미리 정의된 필드에 엑세스한다.

두 번째 단계는 getDeclaredMethods와 같은 메서드를 호출하여 클래스에서 선언한 모든 메서드의 목록을 가져오는 것이다.

이 정보가 있으면 세 번째 단계는 리플렉션 API를 사용하여 정보를 조작하는 것이다. 예를 들면 다음과 같은 시퀀스가 ​​있다.


```java=
Class c = Class.forName("java.lang.String");

Method m[] = c.getDeclaredMethods();

System.out.println(m[0].toString()); //String에 선언된 첫 번째 메서드의 toString()을 출력한다.
```
### instanceOf와 비슷한 isInstance
instanceOf 는 객체가 어떤 클래스인지, 어떤 클래스를 상속받았는지 확인할 수 있는 연산자이다.
리플랙션을 이용해서 다음과 같이 사용할 수 있다.
Object.isInstance(Object);

### 클래스의 메소드 찾기
getDeclatedMethods를 이용해서 Method객체의 리스트를 가져올 수 있다.
getMethods를 사용하면 상속받은 Method들도 가져올 수 있다.

### 함수를 함수 이름으로 호출하기
아래와 같이 add함수를 reflection을 통해 호출할 수 있다.
getMethod함수는 메소드 이름과 매개변수 타입이 일치하는 함수가 있는지 execution time에 찾아서 invoke하게 된다.
```java=
import java.lang.reflect.*;
         
public class method2 {
      public int add(int a, int b)
      {
         return a + b;
      }
         
      public static void main(String args[])
      {
         try {
           Class cls = Class.forName("method2");
           Class partypes[] = new Class[2];
            partypes[0] = Integer.TYPE;
            partypes[1] = Integer.TYPE;
            Method meth = cls.getMethod(
              "add", partypes);
            method2 methobj = new method2();
            Object arglist[] = new Object[2];
            arglist[0] = new Integer(37);
            arglist[1] = new Integer(47);
            Object retobj
              = meth.invoke(methobj, arglist);
            Integer retval = (Integer)retobj;
            System.out.println(retval.intValue());
         }
         catch (Throwable e) {
            System.err.println(e);
         }
      }
}
```
