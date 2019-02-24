---
layout: post
title: "Effective Java 3e Ch2"
description: "Effective Java"
date: 2019-02-24
tags: [java,pattern]
comments: true
share: true
---

## 객체 생성과 파괴

### 1. 생성자 대신 정적 팩터리 메소드 고려
- 생성자와 별도로 정적 팩터리 메소드 제공 가능
  - 디자인 패턴의 팩터리 메소드와 다름

```java
public static Boolean valueOf(boolean b) {
  return b ? Boolean.TRUE : Boolean.FALSE;
}
```
#### 장점
1. 이름을 가질 수 있음
   - BigInteger (int, int, Randbom)
   - BigInteger.probablePrime(int, int, Randbom)
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 됨
   - 불변 클래스를 미리 만들어 놓거나, 생성된 인스턴스를 캐싱하여 재활용 가능
   - ```java Boolean.valueOf(boolean); ```
   - Flyweight pattern 과 비슷한 기법
   - instance-controlled 클래스
      - 인스턴스 통제 가능
      - 싱글턴, 인스턴스 불가 아이템, 불변 값 인스턴스 지원
      - flyweight 패턴의 근간
      - 열거타입의 경우 한개의 인스턴스만 생성 보장
3. 반환 타입의 하위 타입 객체를 반환 가능
    - 반환할 객체의 클래스를 자유롭게 선택 가능
4. 입력 매개 변수에 따라 매번 다른 클래스 객체를 반환할 수 있음
    - 반환 타입의 하위 타입일 경우 제한 없음
5. 정적 팩터리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 됨

#### 단점
1. 상속을 하기 위해 public이나 protected 생성자가 필요, 정적 팩터리 메소드만 제공시 하위 클래스 만들 수 없음
2. 정적 팩터리 매소드는 프로그래머가 찾기 어려움
    - 생성자처럼 API 설명에 명확히 드러나지 않음

#### 주로 사용되는 이름
- from
  - ```java Date d = Date.from(instnace); ```
- of
  - ```java Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);```
- valueOf
  - from, of의 자세한 버전
  - ```java BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE); ```
- instance / getInstance
  - 매개 변수를 받을 경우, 명시한 인스턴스 반환 / 하지만 같은 이스턴스임을 보장하지 않음
  - ```java StackWalker luck = StackWaler.getInstance(options); ```
- create / newInstance
  - 매번 새로운 인스턴스 반환 보장
  - ```java Object newArray = Array.newInstance(classObject, arrayLen); ```
- getType
  - getInstnance와 같지만, 생성할 클래스가 아닌, 다른 클래스에 팩터리 메소드를 정의할 때 씀
  - ```java FileStore fs = Files.getFileStore(path); ```
- newType
  - newInstance와 같지만, 다른 클래스에 팩터리 메소드를 정의할 때 슴
  - ```java BufferedReader bf = files.newBufferedReader(path); ```
- type
  - getType, newType의 간견한 버전
  - ```java List<Complaint> litany = Collections.list(legacyLitany); ```

### 2. 생성자에 매개변수가 많은 경우 빌더 고려
- 정적팩터리와 생성자의 경우 선택적 매개변수가 많은 경우 적절히 대응하기 어려움

#### 1. 점층적 생성자 패턴 사용
- telescoping constructor pattern
- 단점
  - 이 경우, 사용자가 설정하기 원하지 않는 매개변수도 포함될 수 밖에 없음
  - 매개변수가 많아질 수록 클라이언트 코드를 작성하기 어려워짐
- [link](https://github.com/dec7/study/blob/master/books/effective-java-3e/project/src/main/java/com/thebudding/book/effectivejava/item2/a/NutritionFacts.java)


#### 2. 자바빈즈 패턴
- 매개변수가 없는 생성자로 객체를 만들고, 세터를 호출해 값을 설정
- 단점
  - 객체를 하나 만들기 위해 메서드를 여러개 호출 해야함
  - 객체가 완전히 생성되기 전까지 일관성이 무서진 상태로 놓여짐
  - 클래스를 불변으로 만들 수 없음, 안전성을 확보하기 위해 추가 작업 필요
- [link](https://github.com/dec7/study/blob/master/books/effective-java-3e/project/src/main/java/com/thebudding/book/effectivejava/item2/b/NutritionFacts.java)

#### 3. 빌더 패턴
- 필요한 객체를 직접 생성하지 않고, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻음
- 빌더가 제공하는 방식으로 매개변수를 설정하고 build() 메소드를 호출하여 객체를 확보
  - [link](https://github.com/dec7/study/blob/master/books/effective-java-3e/project/src/main/java/com/thebudding/book/effectivejava/item2/c/NutritionFacts.java)


- 계층형 형태에 적합합
  - [link](https://github.com/dec7/study/commit/e6b787945274b0c238192dbbd0725c4b60557f45)
  - 공변환 타이핑 (covariant return typing)
    - 각 하위 클래스의 빌더가 정의한 build 메소드는 구체 하위 클래스를 반환하도록 선언
  - 빌더 생성 비용이 크지는 않지만, 성능이 민감한 상황에서는 문제가 될 수 있음
  - 또한, 코드가 장황하므로 매개변수가 4개 이상은 되어야 가치가 있음


### 3. private 생성자나, 열거타입으로 싱글턴임을 보장하라
- 싱글턴은 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말함
  - 예
    - 함수 같은 무상태나, 설계상 유일해야하는 시스템 컴포넌트
  - 단점
    - 클래스를 싱글턴으로 만들면, 이를 테스트하기 어려울 수 있음

#### 1. 인스턴스에 접근하는 유일한 수단 추가

```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() {..}
  public void leaveTheBuildeing() {..}
}
```
- 시스템에서 하나뿐임이 보장
- 리플렉션 API를 사용해 접근할 수 있으나, 방어하기 위해 두번째 객체 생성이 예외를 던지면 됨

#### 2. 정적 팩터리 방식

```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() {..}
  public static Elvis getInstance() { return INSTNACE; }
  public void leaveTheBuildeing() {..}
}
```
- 리플렉션 예외 존재
- 해당 클래스가 싱글턴임을 API에서 알 수 있음
- 코드의 간결함
- 직렬화 (serializable) 구현시 주의
  - 모든 인스턴스를 transient로 선언하고 realResolve 메소드를 제공해야함
  - 그렇지 않으면, 역직렬화시마다 새로운 인스턴스 생성됨


### 4. 인스턴스화를 막기 위해 private 생성자 사용
- 정적 메소드, 정적 필드만 가진 메소드를 만들고 싶을때
- 생성자를 명시하지 않은 경우 컴파일러가 기본 생성자를 추가, private 생성자를 명시하여 클래스의 인스턴스화 막음

### 5. 자원을 직접 명시하지 말고, 의존객체 주입 사용
- 많은 클래스가 하나 이상의 자원에 의존
- 아래 잘못 사용된 예
```java 
// 정적 유틸리티를 잘못 사용한 예
public class SpellChecker {
  private static final Lexicon dictionary = ...;
  private SpellChecker() {}

  public static boolean isValid(String word) {...}
  public static List<String> suggestions(String typo) {...}
}
```

```java 
// 싱글턴을 잘못 사용한 예
public class SpellChecker {
  private final Lexicon dictionary = ...;

  private SpellChecker() {}
  public static SpellChecker INSTANCE = new SpellChecker(..);
  
  ...
}
```

- 다양한 사전에 대해서 사용이 가능하도록 유연한 형태로 개선
```java
public class SpellChecker {
  private final Lexicon dictionary;

  public SpellChecker(Lexicon dictionary) {
    this.dictionary = Objects.requireNonNull(dictionary);
  }

  ...
}
```

- 사용하는 자원에 따라 동작이 달리지는 클래스는 정적유틸리티나 싱글턴방식이 적합하지 않음
- 클래스가 여러 자원 인스턴스를 지원해야하며 클라이언트가 원하는 자원을 사용해야함
  - 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식

- 이 패턴의 변형: 생성자에 자원 팩터리를 넘겨주는 방식
  - 팩터리 메서드 패턴
  - ```java Supplier<T> 인터페이스 ```
  - ```java Mosaic create(Supplier<? extends Tile) tileFactory) {...} x```


### 6. 불필요한 객체 생성을 피하라
- 재사용은 빠르고 세련되므로 되도록 하나의 객체를 재사용하는게 좋음
- ```String s = new String("hello"); ==> String s = "hello";```
  - 같은 문자열은 같은 JVM내에서 같은 객체임을 보장함
- ```Boolean(String) ==> Boolean.valueOf(String)```
  - 팩터리 메소드를 사용하는게 좋음
  - 생성자는 호출할때마다 새로운 객체를 만들지만, 팩터리 메소드는 그렇지 않음
- 정규표현식의 경우
  - ```String.matches()``` 보다, 불변인 Pattern 인스턴스를 생성 후 재사용하는게 좋음
  - 성능을 더 높기이 위해 '지연초기화' 를 적용할 수도 있지만, 향상된 성능에 비해 코드의 복잡성이 높을 수 있으므로 추천하지 않음 
- 어댑터
  - 객체가 불변일 경우 재사용해도 안전함
  - 어댑터의 경우 제2의 인터페이스 역할을 해주는 객체이나, 불명확함
- 오토박싱
  - 프로그래머가 기본타입과 박싱된 기본타입을 섞어 쓸때 자동으로 상호변화해주는 기술
  - 박싱된 타입보다 기본타입을 사용하고, 의도치 않은 오토박싱이 들어있지 않도록 유의
- JVM
  - 객체 생성이 비싸니 피하라는 말이 아님, 요즘의 JVM 엔 큰 부담 아님
  - 프로그램의 명확성, 간결성 위해서 필요
  - 단순히 객체 생성을 피하기 위해 풀을 사용하지 않는게 좋음
    - 커넥션 같은 아주 비싼 객체 제외

### 7. 다 쓴 객체 참조를 해제하라
- 다 쓴 객체의 참조를 끊는게 중요
  - 객체 참조를 null로 처리하는 것은 예외적인 경우
  - 자기 메모리를 직접 관리하는 클래스인 경우 메모리 누수의 주의
  - 캐시 역시 메모리 누수의 주범
  - 리스너와 콜백
  - WeakHeapMap에 키로 저장시 사용 후 즉시 수거

### 8. finalizer와 cleaner의 사용을 피하라
- finalizer
  - 예측할 수 없고, 상황에 따라 위험할 수 있으므로 일반적으로 불필요
  - 오작동, 작성성능, 이식성 문제
  - 몇가지 쓰임새가 있으나 기본적으로 쓰지 말아야 함 
  - java9에서 deprecated 됨
- cleaner
  - finalizer보다 덜 위험, 예측 불가, 느리고, 일반적으로 불필요
- 단점
  - 위 기능은 즉시 수행된다는 보장 없음
  - 수행은 전적으로 gc 알고리즘에 달렸고, 구현마다 다 다름
  - 아래 메소드는 실행보장을 하나, 심각한 결함이 있음
    - ```System.runFinalizerOnExit```
    - ```Runtime.runFinalizerOnExit``` 
    - ThreadStop
    - 위 처리과정에서 발생한 예외는 무시됨, 처리할 작업이 남았더라도 종료됨
- 심각한 성능 문제
  - finalizer가 gc의 효율을 떨어뜨림
- 보안 취약
  - 외부 공격에 취약
  - 공격원리
    - 생성자나 직렬화 과정(readObject, readResolve)에서 예외 발생시 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 함
- 사용처
  - 자원 소유자가 close를 호출하지 않은 것에 대한 대비책
    - 호출되리란 보장은 없으나, 안하는 것보단 나으니
    - 그럴만한 가치가 있는지 고민은 필요
  - 네이티브 피어와 연결된 객체
    - 네이티브 피어: 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체
    - gc는 네이티브 피어의 존재를 알지 못하며, cleaner나 finalizer에서 처리하기 적당한 작업 
    - 성능저하를 감당할 수 있고, 네이티브 피어가 심각한 자원을 가지지 않은 경우에 적절
    - 네이티브 피어라도 즉시 자원회수가 필요한 경우 close메소드 사용

#### clenaer 예제
- [link](https://github.com/dec7/study/commit/514a5e9826d4ee32aa524f7a580e7b8185663c4e)

### 9. try-with-resources 를 사용하라
- 자원 회수에 전통적으로 try-finally가 쓰임
  - 유일한 방안이 아님
  - 둘 이상 자원인 경우 너무 복잡해짐
- try-with-resources
  - AutoClosable을 구현해야함

```java
static void copy(String src, String dst) throws IOException {

  try (InputStream in = new FileInputStream(src); OutputStream out = new FileOutputStream(dst)) {
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0) {
      out.write(buf, 0, n);
    }
  }

}

```
- 짧고, 명확함

