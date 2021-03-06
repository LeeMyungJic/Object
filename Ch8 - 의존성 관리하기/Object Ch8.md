## 8장

#### 의존성 이해하기

###### 변경과 의존성

- 시점에 따른 다른 의미

  - 실행 시점
    - 의존하는 객체가 정상적으로 동작하기 위해서는 실행 시에 의존 대상 객체가 반드시 존재해야 함
  - 구현 시점
    - 의존 대상 객체가 변경될 경우 의존하는 객체도 함께 변경됨

  ~~~ java
  public class PeriodCondition implements DiscountCondition {
      private DayOfWeek dayOfWeek;
      private LocalTime startTime;
      private LocalTime endTime;
          
      // screening의 인스턴스가 없거나 getStartTime 메시지를 이해할 수 없다면 이 메소드는 예상대로 동작하지 않음
      public boolean isSatisfiedBy(Screening screening) {
          return screening.getStartTime().getDatOfWeek(). ... ;
      }
  }
  ~~~

  - PeriodCondition과 screening 사이에 의존성이 존재

  - 의존성은 방향을 가지며 항상 단방향

    - Screening 변경 -> PeriodCondition 영향 o
    - PeriodCondition 변경 -> Screening 영향 x

  - PeriodCondition은 DayOfWeek, LocalTime, Screening에 의존성을 가짐

    - DayOfWeek, LocalTime은 인스턴스 변수로 사용됨 (실선 표기)
    - Screening은 메소드 인자로 사용됨 (점선 표기)

    

###### 의존성 전이

- 3단논법처럼 생각
  - A가 B에 의존
  - B가 C에 의존
  - 따라서 A는 C에 잠재적으로 의존
- 의존성은 함께 변경될 수 있는 가능성을 의미
  - 따라서 모든 경우에 의존성이 전이되는 것은 아님
  - 전이 여부는 변경의 방향과 캡슐화 정도에 따라 달라짐
- 의존성의 종류
  - 직접 의존성
    - 한 요소가 다른 요소에 직접 의존
    - PeriodCondition이 screening에 의존
  - 간접 의존성
    - 직접적 관계 존재 x, 의존성 전이에 의해 영향이 전파되는 경우
    - 이 경우 의존성은 PeriodCondition의 코드 안에 명시적으로 드러나지 않음

###### 런타임 의존성과 컴파일타임 의존성

- 런타임 : 애플리케이션이 실행되는 시점
  - 객체 사이 의존성을 다룸
- 컴파일타임 : 작성된 코드를 컴파일하는 시점, 문맥에 따라서는 코드 그 자체를 가리킴
  - 클래스 사이의 의존성을 다룸
- 영화 예매
  - 컴파일타임 의존성 : Movie는 DiscountPolicy에 의존
  - 런타임 의존성 : Movie는 상황에 따라 AmountPolicy 또는 PercentPolicy에 의존
  - 코드만 보면 Movie는 오직 추상 클래스인 DiscountPolicy에만 의존
  - AmountPolicy나 PercentPolicy에 대한 언급은 전혀 없음
  - Movie가 직접 두 클래스에 의존하도록 만들 수도 있지만, 결합도를 높일 뿐만 아니라 새로운 할인 정책 추가의 어려움 발생
- 클래스가 협력할 객체의 클래스를 명시적으로 드러내고 있다면 다른 클래스와 협력할 가능성 사라짐 



###### 컨텍스트 독립성

- 클래스가 사용될 특정한 문맥에 대해 최소한의 가정만으로 이뤄져 있다면 다른 문맥에서 재사용하기 더 수월해짐
- 반대로 클래스가 특정한 문맥에 강하게 결합될수록 다른 문맥에서 사용하기 더 어려워짐
- Movie 안에 PercentDiscountPolicy 명시
  - Movie가 비율 할인 정책이 적용된 영화의 요금을 계산하는 문맥에 사용됨을 암시 가능
- Movie 안에 추상클래스 DiscountPolicy 명시
  - 할인 정책에 따라 계산하지만 어떤 정책을 따르는지 결정하지 않음



###### 의존성 해결하기

- 컴파일 의존성은 구체적인 런타임 의존성으로 대체돼야 함

- 의존성 해결 : 컴파일타임 의존성을 실행 컨텍스트에 맞는 적절한 런타임 의존성으로 교체하는 것

- 의존성 해결을 위한 일반적인 3가지 방법

  - 객체를 생성하는 시점에 생성자를 통해 의존성 해결

    ~~~java
    public class Movie {
        public movie(...) {
            ...
            this.discountPolicy = discountPolicy;
        }
    }
    ~~~

  - 객체 생성 후 setter 메소드를 통해 의존성 해결

    - 객체 생성 이후 의존하고 있는 대상을 변경할 수 있는 가능성을 열어 놓고 싶을 때 유용

    ~~~ java
    Movie avatar = new Movie(...);
    avatar.setDiscountPolicy(new AmountDiscountPolicy(...));
    
    // Movie 인스턴스가 생성된 후에도 DiscountPolicy 설정할 수 있는 setter 제공해야 함
    public class Movie {
        public void setDiscountPolicy(DiscountPolicy discountPolicy) {
            this.discountPolicy = discountPolicy;
        }
    }
    ~~~

    ~~~java
    // setter를 통해 인스턴스 생성이후 정책 변경
    Movie avatar = new Movie(...);
    avatar.setDiscountPolicy(new AmountDiscountPolicy(...));
    ...
    avatar.setDiscountPolicy(new PercentDiscountPolicy(...));
    ~~~

    - 실행 시점에 의존 대상을 변경 가능 -> 유연한 설계

    - 객체 생성 이후 의존 대상을 설정 -> 설정 전까지는 객체의 상태가 불완전할 수 있음

      ~~~java
      Movie avatar = new Movie(...);
      avtar.calculateFee(...);   // NullPointException 발생
      avatar.setDiscountPolicy(new AmountDiscountPolicy(...));
      ~~~

      ~~~java
      // 더 좋은 방법은 setter 방식과 생성자 방식을 혼합
      // 완전한 상태의 객체 생성 후 필요에 따라 setter를 이용하여 변경
      Movie avatar = new Movie(..., new PercentDiscountPolicy);
      ...
      avatar.setDiscountPolicy(new AmountDiscountPolicy(...));
      ~~~

  - 메소드 실행 시 인자를 이용해 의존성 해결

    ~~~ java
    // Movie가 항상 할인 정책을 알 필요까지 없고, 가격을 계산할 때만 일시적으로 알아도 무방할 경우
    public class Movie {
        public Money calculateMovieFee(Screening screening, DiscountPolicy discountPolicy) {
            return fee.munus(...);
        }
    }
    ~~~

    - 메소드가 실행되는 동안만 일시적으로 의존 관계가 존재해도 무방할 경우 유용
    - 메소드가 실행될 때마다 의존 대상이 변경될 경우 유용
    - 하지만 메소드를 호출하는 대부분의 경우에 매번 동일한 객체를 인자로 전달하고 있다면 앞 두 방법이 더 좋음



#### 유연한 설계

###### 의존성과 결합도

- 의존성은 객체들의 협력을 가능하게 만드는 매개체라는 관점에서는 바람직 함

  ~~~java
  // Movie가 직접 PercentDiscountPolicy에 의존한다고 가정
  public class Movie {
      ...
      private PercentDiscountPolicy percentDiscountPolicy;
      
      public Movie(..., PercentDiscountPolicy percentDiscountPolicy) {
          ...
          this.percentDiscountPolicy = percentDiscountPolicy;
      }
  }
  ~~~

  - Movie가 PercentDiscountPolicy에 의존하고 있다는 사실을 명시적으로 드러냄

  - 의존성의 존재 자체는 객체 사이 협력을 가능하게 만들기 때문에 문제 없음

  - 하지만 다른 할인 정책이 필요한 문맥에서 Movie 재사용 가능성을 제거해버림

    -> 의존성의 정도가 문제

  - 해결 방법은 Movie의 메시지를 이해할 수 있는 타입(DiscountPolicy)을 정의하는 것

    - 컴파일타임 의존성을 런타임 의존성으로 대체 가능

  - 바람직한 의존성은 재사용성과 관련

    - 컨텍스트에 독립적인 의존성을 바람직함
    - 특정한 컨텍스트에 강하게 결합된 의존성은 바람직하지 않음
    - 위 예제의 경우
      - Movie를 AmountDiscountPolicy와 협력하게 만드려면?
      - PercentDiscountPolicy 타입을 AmountDiscountPolicy로 변경하는 방법 뿐
      - 수정으로 이번에는 Movie가 AmountDiscountPolicy만 사용할 수 있는 컨텍스트에 강하게 결합됨
      - 결국 바람직하지 못한 의존성을 바람직하지 못한 또 다른 의존성으로 대체한 것...

  - 바람직한 의존성

    - 느슨한 결합도 또는 약한 결합도를 가진다 말함

  - 바람직하지 못한 의존성

    - 단단한 결합도 또는 강한 결합도를 가진다 말함

  >의존성의 관점에서는 "의존성이 존재한다" 또는 "의존성이 존재하지 않는다"라고 표현해야 함
  >
  >결합도의 관점(의존성의 정도를 상대적으로 표현할 경우)에서는 "결합도가 강하다" 또는 "결합도가 느슨하다"라고 표현 함 
  
  

###### 지식이 결합을 낳는다

- 결합도의 정도는 한 요소가 자신이 의존하고 있는 다른 요소에 대해 알고 있는 정보의 양으로 결정
- 서로에 대해 알고 잇는 지식의 양이 결합도를 결정
  - Movie가 직접 AmountDiscountPolicy나 PercentDiscountPolicy에 의존하는 경우
    - Movie는 협력할 객체가 할인 정책에 따라 할인 요금을 계산할 것이라는 사실을 알게 됨
  - Movie가 DiscountPolicy에 의존할 경우
    - 그저 할인 요금을 계산한다는 사실만 알게 됨



###### 추상화에 의존하라

- 대상에 대해 알아야 하는 지식의 양을 줄일 수 있음 -> 느슨한 결합도 유지 가능
- 추상화와 결합도 관점에서 의존 대상 분류
  - 구체 클래스 의존성
  - 추상 클래스 의존성
    - 구체 클래스에 비해 메소드 내부 구현과 자식 클래스 종류에 대한 지식 숨길 수 있음
    - 따라서 구체 클래스보다 결합도가 더 낮음
    - 여전히 협력하는 대상이 속한 클래스 상속 계층이 무엇인지 알고 있어야 함
  - 인터페이스 의존성
    - 상속 계층을 모르더라도 협력 가능
    - 어떤 메시지를 수신할 수 있는지에 대한 지식만 남김
    - 추상 클래스 의존성보다 결합도 낮아짐
- 의존하는 대상이 추상적일수록 결합도는 낮아짐 !!



###### 명시적인 의존성

~~~java
public class Movie {
    ...
    private DiscountPolicy discountPolicy;
    
    public Movie (String title, Duration runningTime, Money fee) {
        ...
        this.discountPolicy = new AmountDiscountPolicy(...);
    }
}
~~~

- discountPolicy는 추상 클래스인 DiscountPolicy으로 선언돼 있음
- 생성자에서 구체 클래스인 AmountDiscountPolicy의 인스턴스를 직접 생성하여 대입
- Movie는 추상 클래스뿐만 아니라 구체 클래스에도 의존하게 됨

~~~java
// 생성자를 이용하여 의존성 제거

public class Movie {
    ...
    private DiscountPolicy discountPolicy;
    
    // 인스턴스 변수와 생성자 타입 모두 추상 클래스인 DiscountPolicy로 선언
    // 런타임에 Amount 또는 Percent의 인스턴스를 선택적으로 전달 가능
    // Movie는 생성자의 인자로 전달된 인스턴스에 의존
    public Movie (String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        ...
        this.discountPolicy = discountPolicy;
    }
}
~~~

- 생성자의 인자로 전달받는 방법 vs 생성자 안에서 직접 생성하는 방법
  - 가장 큰 차이는 퍼블릭 인터페이스를 통해 할인 정책을 설정할 수 있는 방법을 제공하는지 여부
  - 생성자의 인자 전달
    - Movie가 DiscountPolicy에 의존한다는 사실을 Movie의 퍼블릭 인터페이스에 드러냄
    - setter나 메소드 인자를 사용하는 방식에도 동일
    - 명시적인 의존성
  - 생성자 안에서 직접 생성
    - Movie 내부에서 AmountDiscountPolicy의 인스턴스를 직접 생성하면 DiscountPolicy에 의존한다는 사실을 감춤
    - 숨겨진 의존성
  - 의존성은 명시적으로 표현돼야 함
    - 그래야 퍼블릭 인터페이스를 통해 컴파일타임 의존성을 적절한 런타임 의존성으로 교체 가능

###### new는 해롭다

- 결합도 측면에서 new가 해로운 이유
  - 구체 클래스 이름을 직접 기술해야 함. 따라서 추상화가 아닌 구체 클래스에 의존 -> 결합도가 높아짐
  - 구체 클래스뿐만 아니라 어떤 인자를 이용해 클래스의 생성자를 호출해야 하는지도 알아야 함 -> 지식의 양이 늘어남

~~~java
public class Movie {
  
  public Movie (...) {
    // AmountDiscountPolicy를 생성하기 위해 생성자에 전달되는 인자를 알아야 함 -> 지식의 양 증가
    // SequenceCondition, PeriodCondition에도 의존하게 만듦
    this.disCountPolicy = new AmountDiscountPolicy(Money.won(800)),
    											new SequenceCondition(1),
    											new PeriodCondition(DayOfWeek.MONDAY, ...),
    											...
  }
}
~~~

- 인스턴스를 생성하는 로직과 생성된 인스턴스를 사용하는 로직을 분리하여 해결

  ~~~ java
  // Movie 클래스에는 AmountDiscountPolicy의 인스턴스에 메시지를 전송하는 코드만 남아있어야 함
  // Movie는 AmountDiscountPolicy 인스턴스를 사용하는 책임만 남게 됨
  public class Movie {
    ...
    private DiscountCondition discountCondition;
    
    public Movie(...) {
      ...
      this.discountCondition = discountCondition;
    }
  }
  
  // 이제 Movie가 AmountDiscountPolicy 인스턴스를 생성하지 않고 Movie 클라이언트가 생성
  // AmountDiscountPolicy 인스턴스 생성 책임이 Movie 클라이언트로 옮겨짐
  Movie avatar = new Movie("아바타", ...,
                          new AmountDiscountPolicy(...), 
                          new SequenceCondition(1), ...
                          );
  ~~~

  - 생성의 책임을 클라이언트로 옮김
    - Movie는 DiscountCondition의 모든 자식들과 협력 가능하게 됨



###### 가끔은 생성해도 무방하다

- 주로 협력하는 기본 객체를 설정하고 싶은 경우 이에 해당
- Movie가 대부분 AmountDiscountPolicy와 협력하고 가끔 PercentDiscountPolicy와 협력하는 경우
  - 모든 경우에 인스턴스 생성 책임을 클라이언트로 옮길 경우 클라이언트들 사이 중복 코드가 늘어나며 Movie의 사용성도 나빠짐
  - 기본 객체를 생성하는 생성자 추가, 이 생성자에서 DiscountPolicy의 인스턴스를 인자로 받는 생성자를 체이닝하여 해결



###### 표준 클래스에 대한 의존은 해롭지 않다

- 의존성이 불편한 이유는 그것에 항상 변경에 대한 영향을 암시하기 때문
- 자바의 경우 JDK에 포함된 표준 클래스가 이에 속함



###### 컨텍스트 확장하기

- 할인혜택을 제공하지 않는 경우
  - DiscountPolicy를 상속받는 NoneDiscountPolicy를 추가
  - Movie에 특별한 if문 추가 없이 구현이 가능

- 중복 적용이 가능한 할인

  - Movie가 하나 이상의 DiscountPolicy와 협력 가능해야 함

  ~~~ java
  public class OverlappedDiscountPolicy {
    // Movie가 DiscountPolicy의 인스턴스들로 구성된 List를 인스턴스 변수로 갖게 함
    private List<DiscountPolicy> discountPolicies = new ArrayList<>();
    
    public OverlappedDiscountPolicy(DiscountPolicy ... discountPolicies) {
      this.discountPolicies = Arrays.asList(discountPolicies);
    }
    
    @Override
    protected Money getDiscountAmount(...) {
      Money result = Money.ZERO;
      for(DiscountPolicy each : discountPolicies) {
        result = result.plus(...);
      }
      return result;
    }
  }
  
  // OverlappedDiscountPolicy의 인스턴스를 생성하여 Movie에 전달하는 것만으로 중복 할인 적용 가능
  Movie avatar = new Movie("아바타", ... ,
                          new OverlappedDiscountPolicy(new AmountDiscountPolicy(...), 
                                                      new PercentDiscountPolicy(...)));
  ~~~

  - 설계가 유연해진 이유
    - Movie가 DiscountPolicy라는 추상화에 의존
    - 생성자를 통해 DiscountPolicy에 대한 의존성을 명시적으로 드러냄
    - new와 같이 구체 클래스를 직접 다뤄야 하는 책임을 Movie 외부로 옮김
    - Movie가 의존하는 추상화인 DiscountPolicy 클래스에 자식 클래스를 추가하여 간단하게 컨텍스트 확장 가능

- 조합 가능한 행동
  - 다양한 할인 정책이 필요한 컨텍스트에서 Movie를 재사용할 수 있었던 이유
    - 코드 수정없이 협력 대상인 DiscountPolicy 인스턴스를 교체할 수 있기 때문
    - Movie가 비율 할인 정책에 따라 요금 계산을 원한다면
      - PercentDiscountPolicy의 인스턴스를 Movie에 연결
    - Movie가 금액 할인 정책에 따라 요금 계산을 원한다면
      - AmountDiscountPolicy의 인스턴스를 Movie에 연결
    - Movie가 중복 할인을 원한다면
      - OverappedDiscountPolicy의 인스턴스를 Movie에 연결
  - 유연하고 재사용 가능한 설계
    - 객체가 어떻게 하지는지를 나열하지 않고도 객체들의 조합을 통해 무엇을 하는지 표현하는 클래스들로 구성됨
    - 따라서 클래스의 인스턴스를 생성하는 코드를 보는 것만으로도 어떤 일을 하는지 파악 가능

