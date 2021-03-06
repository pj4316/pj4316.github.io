---
layout: authors
bg: "tools.jpg"
title: "[Effective Java 3rd] 1.객체 생성"
crawlertitle: "[Effective Java 3rd] || 1.객체 생성"
summary: "[Effective Java 3rd] || 1.객체 생성"
date: 2019-02-26
slug: post-url
author: "Max Lee"
permalink: /Authors/
---

## 핵심 내용
- 객체를 만들어야 할 때와 만들지 말아야 할때를 구분하는 법
- 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법

---

### \[Item 1] 생성자 대신 정적 팩토리 메소드를 고려하라 (Static Factory)
pros.
  1. 이름을 가질 수 있다 (생성자가 어떤 역할을 하는지 구분이 가능하다)
   - 매개변수와 생성자 자체로 반환될 객체의 특성을 설명하기 힘들다.
  2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
   - 불변 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
   - 생성 비용이 큰 객체가 자주 요청되는 상황에서 성능을 크게 올려준다. (Flyweight Pattern)
  3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
   - 반환 할 타입을 설정할 수 있기 떄문에 '엄청난 유연성'을 제공한다.

   > 자바8전에는 인터페이스에 정적메서드를 선언할 수 없었다. 현재는 가능
  4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다. (EnumSet)
  5. 정적 팩토리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
  
cons.
  1. 상속하려면 public이나 protected 생성자가 필요하기에 정적 팩토리 메소드만 제공하면 하위 클래스를 만들 수 없다.
  2. 정적 팩토리 메소드는 프로그래머가 찾기 어렵다.
  
요약 : 정적 팩토리 메소드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이용하고 사용하는 것이 좋다.

---
### \[Item 2] 생성자에 매개변수가 많다면 빌더를 고려하라 (Builder)

정적 팩토리와 생성자는 선택적 매개변수가 많을때 적절히 대응하기 어렵다.
이를 보완한 것이 바로 빌더이다. 원하는 매개변수를 하나씩 선택적으로 추가하여 객체를 생성할 수 있다.

\[추상 클래스 - 피자]
```
public abstract class Pizza{
  public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}
  final Set<Topping> toppings;
  
  abstract static class Builder<T extends Builder<T>>{
    EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
    public T addTopping(Topping topping){
      toppings.add(Objects.requireNonNull(topping));
      return self();
    }
    
    abstract Pizza build();
    
    protected abstract T self();
  
  }
  
  Pizza(Builder<?> builder){
    toppings = builder.toppings.clone();
  }
}
```
\[클래스 - 뉴욕 피자]
```
public class NyPizza extends Pizza{
  public enum Size {SMALL, MEDIUM, LARGE};
  private final Size size;
  
  public static class Builder extends Pizza.Builder<Builder>{
    private final Size size;
    
    public Builder(Size size){
      this.size = Objects.requiredNonNull(size);
    }
    
    @Override
    public NyPizza build(){
      return new NzPizza(this);
    }
    
    @Override
    protected Builder self(){
      return this;
    }
  }
  
  private NyPizza(Builder builder){
    super(builder);
    size = builder.size;
  }
}
```
\[실제 적용]
```
NyPizza pizza = new NyPizza.Builder(SMALL)  // SMALL 사이즈
     .addTopping(SAUSAGE)                   // SAUSAGE 토핑 추가
     .addTopping(ONION)                     // ONION 토핑 추가
     .build();                              // 피자 만들기
```

---
### \[Item 3] private 생성자나 열거 타입으로 싱글턴임을 보증하라 (SingleTone)
### \[Item 4] 인스턴스화를 막으려거든 private 생성자를 사용하라 (private Constructor)
싱글톤이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
싱글톤의 전형적인 예로는 **무상태(Stateless) 객체**나 설계상 유일해야하는 **시스템 컴포넌트**를 들 수 있다.
클래스를 싱글톤으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워진다.

```
public class Elvis{
  private static final Elvis INSTANCE = new Elvis();    // Elvis의 SingleTone 객체
  private Elvis();
  
  public Elvis getInstance(){             //getInstance를 제외하고, Elvis객체를 얻을 수 없다.
    return INSTANCE;
  }
}
```
추상 클래스로는 객체의 인스턴스화를 막을 수 없다. 생성자를 private하게 만드는 것만이 생성을 막을 수 있다.

---
### \[Item 5] 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 (Dependency Injection)
많은 클래스가 하나 이상의 자원에 의존한다. 예를 들어, 맞춤법 검사기는 사전(Dictionary)에 의존한다. 하지만 이를 사전 하나에만 의존하도록 싱글톤으로 선언하면 매우 유연하지 않다. 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글톤 방식이 적합하지 않다.
인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이, 클라이언트가 원하는 자원을 사용하도록 할 수 있는 방법이다.

이러한 방식(의존 객체 주입)은 생성자, 정적 팩토리, 빌더에 응용 가능하다.
생성자에 자원 팩토리를 넘겨주는 방식이 있다. 자바8에서 Supplier<T> 인터페이스가 팩터리를 완벽하게 표현한 예이다.
  
`클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글톤과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.`
