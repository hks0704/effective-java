# 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라
- 정적 팩터리 방식과 생성자 방식의 단점 : 선택적 매개변수가 많을 때 대응하기 어렵다.
- 선택적 매개변수 : 대다수의 경우 값이 0인 선택항목 변수

## 점층적 생성자 패턴(telescoping constructor pattern)
- 선택적 매개변수가 많을 때 프로그래머들이 사용했던 패턴
- 선택 매개변수 개수를 늘려가면서 각각의 생성자를 만든다.
```java
// 코드 2-1
public class NutritionFact {
    private final int servingSize;  // (ml, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)   필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택
    
    public NutritionFact(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }
    
    public NutritionFact(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }
    
    public NutritionFact(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFact(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFact(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize    = servingSize;
        this.servings       = servings;
        this.calories       = calories;
        this.fat            = fat;
        this.sodium         = sodium;
        this.carbohydrate   = carbohydrate;
    }
}
```
- 인스턴스를 만들 때는 원하는 매개변수를 모두 포함한 생성자 중에서 가장 짧은 것을 선택
```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
- **단점 : 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려워진다.**
## 자바빈즈 패턴(JavaBeans pattern)
- 선택 매개변수가 많을 때 쓰는 두 번째 대안
- 매개변수가 없는 생성자로 객체를 만들고 세터(setter) 메서드로 원하는 매개변수의 값을 설정하는 방식
```java
// 코드 2-2
public class NutritionFact {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private final int servingSize = -1;
    private final int servings = -1;
    private final int calories = 0;
    private final int fat = 0;
    private final int sodium = 0;
    private final int carbohydrate = 0;
    
    public NutritionFact() {}
    // 세터 메서드들
    public void setServingSize(int val)  {servingSize = val;}
    public void setServings(int val)     {servings = val;}
    public void setCalories(int val)     {calories = val;}
    public void setFat(int val)          {fat = val;}
    public void setSodium(int val)       {sodium = val;}
    public void setCarbohydrate(int val) {carbohydrate = val;}
}
```
- 장점 : 인스턴스를 만들기 쉬워졌고 읽기도 쉽다. (점층적 생성자 패턴의 단점을 보완)
```java
NutritionFact cocaCola = new NutritionFact();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```
- **단점 : 객체 하나를 만들 때 메서드 여러 개를 호출해야 되고 완전히 생성되기 전까지는 일관성(consistency)이 무너진 상태에 놓인다.**
- 일관성이 무너지면 **불변 객체로 만들 수 없고** 프로그래머가 스레드 안전성을 위한 작업을 추가해야 한다.
- 생성이 끝난 객체를 수동으로 사용하지 못하게 막는 방식(freezing)은 실전에서 잘 쓰지 않는다.
## 빌더 패턴
- 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비한 방식
- 객체를 직접 만들지 않고 필수 매개변수만 가지고 생성자를 호출해 빌더 객체 획득
- 이후에 setter를 사용해서 선택 매개변수의 값을 지정
- 마지막에 build 메서드를 사용해 (보통은 불변 상태인) 객체를 얻음
```java
// 2-3
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    
    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;
        
        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories     = 0;
        private int fat          = 0;
        private int sodium       = 0;
        private int carbohydrate = 0;
        
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }
        
        public Builder calories(int val)
            { calories = val;   return this; }

        public Builder fat(int val)
            { fat = val;   return this; }

        public Builder sodium(int val)
            { sodium = val;   return this; }

        public Builder carbohydrate(int val)
            { carbohydrate = val;   return this; }
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
        
        private NutritionFacts(Builder builder) {
            servingSize  = builder.servingSize;
            servings     = builder.servings;
            calories     = builder.calories;
            fat          = builder.fat;
            sodium       = builder.sodium;
            carbohydrate = builder.carbohydrate;
        }
    }
}
```
- 빌더의 setter 메서드는 자기 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다 : 플루언트 API(fluent API) or 메서드 연쇄(method chaining)
- 빌더 패턴의 유효성 검사 : 빌더 생성자와 메서드에서 입력 매개변수 검사, build 메서드가 호출하는 생성자에서 불변식(invariant)검사
- 잘못된 매개변수를 만나면 알려주는 메시지를 담아 `IllegalArgumentException`을 던진다.

### 불변, 불변식
- 불변(immutable or immutability) : 어떠한 변경도 허용하지 않는 것
- 불변식(invariant) : 프로그램이 실행되는 동안 반드시 만족해야 되는 조건

- **빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.**
- 각 계층의 클래스에 관련 빌더를 멤버로 정의한다. 추상클래스는 추상 빌더, 구체클래스는 구체 빌더를 갖게 한다.

```java
// 2-4
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}

    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }
        
        abstract Pizza build();
        
        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }
}
```
- Pizza.Builder 클래스는 재귀적 타입 한정(아이템 30)을 이용하는 제네릭 타입이고 추상 메서드 self를 더해 하위 클래스 형변환 없이 메서드 연쇄를 지원함
- self 타입이 없는 자바에서 이러한 방식의 구현 방법을 시뮬레이트한 셀프 타입(simulated self-type) 관용구라고 부른다.

```java
// 2-5
public class NyPizza extends Pizza {
    public enum Size {SMALL, MEDIUM, LARGE}

    private final Size size; // 크기 매개변수를 필수로 받는다.

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }
        
        @Override public NyPizza build() {
            return new NyPizza(this);
        }
        
        @Override protected Builder self() { return this; }
    }
    
    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```
```java
// 2-6
public class Calzone extends Pizza {
    private final boolean sauceInside; // 소스선택 매개변수를 필수로 받는다.

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }
        
        @Override public Calzone build() {
            return new Calzone(this);
        }
        
        @Override protected Builder self() { return this; }
    }
    
    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```
- 공변 반환 타이핑(covariant return typing) : 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능
- 공변 변환 타이핑으로 클라이언트가 형변환에 신경쓰지 않고 빌더를 사용할 수 있다.
```java
NyPizza pizza = new NyPizza.Builder(SMALL)
        .addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
        .addTopping(HAM).sauceInside().build();
```
- 빌더를 사용하면 가변인수(varargs) 매개변수를 여러 개 사용할 수 있다(장점)
- 빌더 패턴은 상당히 유연하다. 하나의 빌더로 여러 객체를 순회하며 만들 수 있고 넘기는 매개변수에 따라 다르게 만들 수도 있다. 일련번호 같은 특정 필드는 빌더가 알아서 채우도록 만들수도 있다.
- 빌더 패턴의 단점
  - 성능이 민감한 상황에서는 빌더 생성 비용이 문제가 될 수 있다.
  - 점층적 생성자 패턴보다는 코드가 장황하다. 그래서 매개변수가 4개 이상일 때 효과적이다.
- 하지만 매개변수는 점차 많아지고 확장성을 고려할 때 대체로 빌더 패턴을 쓰는 편이 낫다.
## 핵심 정리
- **생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 쓰는 게 더 낫다.**
- 매개변수 중 다수가 아니거나 같은 타입일 때 특히 더 그렇다.
- 빌더는 점층적 생성자보다 소스코드가 간결하고 자바빈즈보다 안전하다.