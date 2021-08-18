---
title:  "생성자에 매개변수가 많다면 빌더를 고려하라"
excerpt: "JavaEffective 정리 "
categories:
  - Study
tag:
  - Java
  - Book
  - JavaEffective
toc: true
---

> 정적 팩터리와 생성자에는 똑같은 제약이 하나 있다. 선택적 매개변수가 많을때 적절히 대응하기 어렵다.

#### 점층적 생성자 패턴

- 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 받는 생성자를 늘려가는 방식이다. 
- 인스턴스를 만들려면 원하는 매개 변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다.
- 개수가 많아지면 클라이언트 코드를 작성하거나 읽기가 어려워진다.

``` java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    
    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
```


#### 자바빈즈 패턴

- 매개변수가 없는 생성자로 객체를 만든 후 세터 메서들을 호출해 원하는 매개변수의 값을 설정하는 방식이다. 
- 객체 하나를 마들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다.

``` java
public class NutritionFacts {

    // 기본값 초기화, 필수 = -1
    private int servingSize = -1;
    private int servings = -1;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {};
    
    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}


NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setCSodium(35);
cocaCola.setCarbohydrate(27);
```

#### 빌더 패턴

1. 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다. 
1. 그런 다음 빌더 객체가 제동하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다.
1. 마지막으로  매겨변수가 없는 build메서드를 호출해 필요한(보통은 불변인)객체를 얻는다.

>클래스는 불변이며, 모든 매개변수의 기본값을을 한곳에 모아 뒀다. 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있는데 이런 방식을 플루언트 API혹인 메서드 연쇄라고 한다.

```java
public class NutritionFacts {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
        this.sodium = builder.sodium;
        this.carbohydrate = builder.carbohydrate;
    };

    static class Builder {
        
        private final int servingSize;
        private final int servings;
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
        
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        
        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
}


new NutritionFacts.Builder(240,8)
                  .calories(240)
                  .sodium(35)
                  .carbohydrate(27) 
                  .build();

```

>빌더패턴은(파이썬과 스칼라에 있는) 명명된 선택적 매개변수를 흉내 낸 것이다.빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다. 


**빌더 패턴 예제**

``` java
public abstract class Pizza {
    public enum Topping { HAM, ONION, MUSHROOM }
    private final Set<Topping> toppings;

   

    static abstract class Builder<T extends Builder<T>> {
        private EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();
        protected abstract T self();
    }
     Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```

``` java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MIDDLE, LARGE }
    private final Size size;

    public static class Builder extends Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self(){
            return this;
        }
      }  
    private NyPizza(Builder builder) {
        super(builder);
        size =builder.size;
    }
}
```

``` java
// 클라이언트 코드
NyPizza nyPizza = new NyPizza.Builder(SMALL)
.addTopping(ONION)
.addTopping(MUSHROOM)
.build();
```


**예제 패턴 방식**

1. 부모 추상 클래스를 생성하고 부모의 추상 빌더 클래스를 만든다.
  * 자식 클래스는 부모 추상 클래스를 상속받고 자식 빌더는 부모 추상 빌더를 상속 받는다.
1. 부모 클래스준에서의 빌더는 자기자신의 하위타입을 받도록 해다 <T extends Builder<T>>(재귀적인타입 매개변수)
  * 기본적으로 부모 클래스에서  토핑은 빈갑으로 셋팅 addTopping을 할때마다 추가가 된다.
1. self 와 build 추상메서드를 만든다.
  * 추상 클래스로 new 생성이 불가하기 때문에 build는 하위 타입을 만들어준다.
  * self는 하위타입에서 오버라이딩 되어 호출시 this를 반환하여 메서드 연쇄를 지원한다. 
1. 부모 클래스를 상속받는하위 클래스를 생성한다.
  * 하위 요소의 빌더클래스는 부모 빌더를 상속 받는다.
  * 빌더 생성자는 size 필수로 받도록 셋팅한다.
1. 오버라이딩한 builder는 리턴타입을 자식 객체로 만들어 클라이언트에서 타입 캐스팅을 할 필요가 없게 한다.
  * 빌더 호출시 하위 객체가 생성되며 this를 넘겨줌으로서 빌드 자체를매개 변수로 넘겨준다.
  * 생성자에서 빌더를 받아 super호출하고 빌더의 사이즈를 하위객체의 사이즈에 셋팅해준다.
  * super가 호출됨으로써 토핑(상속을 받으면서 하위객체에도 포함)도 셋팅이 된다(상속을 받으면서 하위 빌더에서 빈 토핑이 셋팅 되어 있음)




**예제 클라이언트 구동 방식**
1. nyPizza의 빌더를생성하고 사이즈를 매개변수로 넘겨준다
  * 사이즈는 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다.
  * 빌더에서 사이즈와 토핑(비어 있음)이 셋팅된다.
1. addTopping() 메서를 호출하여 원하는 선택 매개변수들을 설정한다.
  * 빌더의 토핑이 추가된다.
  * Objects.requireNonNull()를 통해 null 체크가능
  * 마지막에 self()호출하면서 this를 반환, 해당 빌드가 유지된다.
1. build() 호출 하여 new nyPizza()를 생성하고 this(빌더)를 매개변수로 넘겨준다. 
  * supper 호출, 부모 생성자가 호출되고 빌더를 받아 객체 토핑에 빌더 토핑을 셋팅된다.
  * 객체 사이즈에 빌더 사이즈를 셋팅한다.
  * 셋팅된 객체를 리턴한다
1. 빌더를 통해 필수 및 선택 매개변수들 셋팅되어 객체가 생성된다.


#### 핵심
생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 더 낫다. 매개변수 중 다수의 필수가 아니거나 같은 타입이면 특히 더 그렇다. 빌더는 점층적으로 생성자보다 클라이언트 코드를 읽고 쓰기가 횔씬 간결하고, 자바 빈즈보다 횔씬 안전허다.