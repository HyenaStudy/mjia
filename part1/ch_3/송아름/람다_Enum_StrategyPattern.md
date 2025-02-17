> 람다 이전의 코드

###### 인터페이스 기반_ 하나의 타입으로 다양한 동작을 구현할 수 있다. 
```JAVA
interface Payment {
    void process(double amount);
}
// Payment의 구현체들은 다양한 동작을 정의할 수 있다.
```

  
###### 익명클래스_반복되는 클래스 정의를 줄일 수 있다. 
```JAVA
Payment payment = new Payment() {
            @Override
            public void process(double amount) {
                System.out.println("신용카드로 " + amount + "원 결제 완료");
            }
        };

```
<br>

### 람다의 장점을 극대화할수 있는 예시_ Enum / 전략패턴


#### 1. Enum
##### :::: 람다를 활용하면 enum class를 직관적으로 작성할 수 있다 :::: 

> enum은 열거형 타입이다.
>
> 아래의 코드를 깊게 읽어보지않아도 람다로 작성된 코드가 어떤것들이 열거된 클래스인지 한눈에 들어온다.
>
> 또한 새로운 결제 방식 추가 시, Enum 요소 + 람다 만 추가하면 된다.
>
> (타입안정성/enum은 잘못된 값이들어오면 컴파일타임에 에러를 잡아낼 수 있다.)
>


```
가능한 결제 리스트를 DB에서 읽어오는 방법도 있겠지만

 Enum 으로 작성

(response 타입별 동작, 에러타입별 동작 등 다른 예시에도 적용가능)
```

<br>

```JAVA
// 1. 실행
public class Main {
    public void main(String[] args) {
        //  PaymentType , PaymentTypeDevlop 둘다 됨
        String selectedPayment = "CreditCard";  // 사용자가 선택한 결제 방식
        double amount = 50000;

        PaymentType.fromString(selectedPayment)
                .ifPresentOrElse(
                        type -> type.process(amount),  // 일치
                        () -> System.out.println(" 지원되지 않는 결제 방식입니다.") // 에러 메시지
                );

    }
}

// 2. 기존 Enum class
/*
* abstract 메서드를 정의하면
* 모든 enum 각 요소에서 반드시 구현해야함
*
* 생성자 private final
* */
public enum PaymentType {
    /* enum 요소정리 */
    CREDIT_CARD("CreditCard") {
        @Override
        public void process(double amount) {
            System.out.println("신용카드로 " + amount + "원 결제");
        }
    },
    PAYPAL("PayPal") {
        @Override
        public void process(double amount) {
            System.out.println("PayPal로 " + amount + "원 결제");
        }
    },
    CRYPTO("Crypto") {
        @Override
        public void process(double amount) {
            System.out.println("암호화폐로 " + amount + "원 결제");
        }
    };

    private final String name;

    PaymentType(String name) {
        this.name = name;
    }

    public abstract void process(double amount);

    // from 문자열 to enum  변환  메서드
    public static Optional<PaymentType> fromString(String name) {
        return Arrays.stream(values())
                .filter(type -> type.name.equalsIgnoreCase(name))
                .findFirst();
    }
}

// 3. 람다로 작성한 Enum class
/*
* @Override 를사용하지않고 람다식으로 대체
*
* 기존 abstract void process 와같이 추상메서드로 선언하지않아도 같은 기능
 * */
public enum PaymentTypeDevlop {
    CREDIT_CARD("CreditCard", amount -> System.out.println("신용카드로 " + amount + "원 결제")),
    PAYPAL("PayPal", amount -> System.out.println("PayPal로 " + amount + "원 결제")),
    CRYPTO("Crypto", amount -> System.out.println("암호화폐로 " + amount + "원 결제"));

    private final String name;
    private final Consumer<Double> paymentAction;  // 람다식저장 필드

    PaymentTypeDevlop(String name, Consumer<Double> paymentAction) {
        this.name = name;
        this.paymentAction = paymentAction;
    }

    public void process(double amount) {
        paymentAction.accept(amount);  // 람다 실행
    }

    public static Optional<PaymentTypeDevlop> fromString(String name) {
        return Arrays.stream(values())
                .filter(type -> type.name.equalsIgnoreCase(name))
                .findFirst();
    }
}


```



<br><br>

> 람다를활용한 Enum class 사용예시 ++

```JAVA
import java.util.function.Consumer;

public enum HttpStatusHandler {
    OK(200, message -> System.out.println("성공: " + message)),
    BAD_REQUEST(400, message -> System.out.println("잘못된 요청: " + message)),
    UNAUTHORIZED(401, message -> System.out.println("인증 필요: " + message)),
    FORBIDDEN(403, message -> System.out.println("접근 금지: " + message)),
    NOT_FOUND(404, message -> System.out.println("페이지 없음: " + message)),
    SERVER_ERROR(500, message -> System.out.println("서버 오류: " + message));

    private final int code;
    private final Consumer<String> action;

    HttpStatusHandler(int code, Consumer<String> action) {
        this.code = code;
        this.action = action;
    }

    public void handle(String message) {
        action.accept(message);
    }

    public static HttpStatusHandler fromCode(int code) {
        for (HttpStatusHandler handler : values()) {
            if (handler.code == code) {
                return handler;
            }
        }
        throw new IllegalArgumentException("Unknown status code: " + code);
    }
}

// 사용 예시
public class Main {
    public static void main(String[] args) {
        int responseCode = 400;
        HttpStatusHandler.fromCode(responseCode).handle("잘못된 요청 발생!");
    }
}

```

<br>

> 아래 코드는 json,xml등의 여러 데이터타입과 encoding type, 여러파라미터 타입으로
>
> 실행되는 로직이나 처리방법이 다른 경우에 사용될 수 있을거 같아
>
> 나중에 참고하려고 가져왔다. 

```JAVA
public enum DataConverter {
    UPPERCASE(str -> str.toUpperCase()),
    LOWERCASE(str -> str.toLowerCase()),
    TRIM(str -> str.trim()),
    REVERSE(str -> new StringBuilder(str).reverse().toString());

    private final Function<String, String> converter;

    DataConverter(Function<String, String> converter) {
        this.converter = converter;
    }

    public String convert(String input) {
        return converter.apply(input);
    }
}

// 사용 예시
public class Main {
    public static void main(String[] args) {
        String text = "  Hello World  ";
        System.out.println(DataConverter.UPPERCASE.convert(text));  // "  HELLO WORLD  "
        System.out.println(DataConverter.TRIM.convert(text));      // "Hello World"
        System.out.println(DataConverter.REVERSE.convert(text));   // "  dlroW olleH  "
    }
}

```

<br><br>

> 권한별로 거쳐야하는 로직과 리턴타입이 다를 수 있다.
>
> 컨트롤러에서 권한정보를 받아 enum으로 실행한다면,
>
> 불필요한 switch문이나, if문이 필요없을것이다.
>
> 리턴되는 값이다르다면 apiWrapper와 같은 reponsEntity를 활용한다.
> 
```JAVA
import java.util.function.Predicate;

public enum UserRole {
    ADMIN(resource -> true),  // 모든 리소스 접근 가능
    MANAGER(resource -> resource.startsWith("MANAGE_")),  // 특정 리소스만 접근 가능
    USER(resource -> resource.equals("VIEW_DASHBOARD"));  // 대시보드만 접근 가능

    private final Predicate<String> permissionCheck;

    UserRole(Predicate<String> permissionCheck) {
        this.permissionCheck = permissionCheck;
    }

    public boolean hasPermission(String resource) {
        return permissionCheck.test(resource);
    }
}

// 사용 예시
public class Main {
    public static void main(String[] args) {
        System.out.println(UserRole.ADMIN.hasPermission("DELETE_USER")); // true
        System.out.println(UserRole.MANAGER.hasPermission("MANAGE_USER")); // true
        System.out.println(UserRole.USER.hasPermission("DELETE_USER")); // false
    }
}
```

<br>


#### 2. 전략패턴

>
>
###### enum은 어떤것들이 있는지 코드에서 정의해놓은 정적
###### 전략패턴은 동적으로 전략을 추가하여 기존 코드 수정 없이 enum보다 유연한 코드 작성이 가능
> 동적으로 적용된다는것은 런타임에 결정된다는 것이다.

```JAVA
import java.util.HashMap;
import java.util.Map;
import java.util.Optional;
import java.util.function.Function;
import java.util.function.Supplier;

public class EventStrategy {
    private final Map<String, Function<Double, Double>> strategies = new HashMap<>();
    private final Map<String, Supplier<Double>> suppliers = new HashMap<>();

    public EventStrategy() {
        strategies.put("DISCOUNT", price -> price * 0.7); // 30% 할인
        strategies.put("COUPON", courseTicket -> courseTicket + 10); // 수강권 추가 10
        suppliers.put("GIFT", () -> (1 + Math.random() * 10)); // 1~10 랜덤 선물 번호 반환
    }

    public void addStrategy(String name, Function<Double, Double> eventFunction) {
        strategies.put(name, eventFunction);
    }

    public void addSupplier(String name, Supplier<Double> supplierFunction) {
        suppliers.put(name, supplierFunction);
    }

    // if-else 없이 `Optional`과 `Map::getOrDefault` 활용
    public double applyEvent(String event, double value) {
        return Optional.ofNullable(strategies.get(event))
                .map(f -> f.apply(value))
                .orElseGet(() -> Optional.ofNullable(suppliers.get(event))
                        .map(Supplier::get)
                        .orElseThrow(() -> new IllegalArgumentException("❌ 잘못된 이벤트 유형입니다: " + event)));
    }
}

public class Main {
    public static void main(String[] args) {
        EventStrategy eventStrategy = new EventStrategy();

        double price = 100.0;
        System.out.println("DISCOUNT: " + eventStrategy.applyEvent("DISCOUNT", price)); // 70.0
        System.out.println("COUPON: " + eventStrategy.applyEvent("COUPON", 5)); // 15.0
        System.out.println("GIFT: " + eventStrategy.applyEvent("GIFT", 0)); // 랜덤 값 (1~10)

        // 실행 중 새로운 전략 추가
        eventStrategy.addStrategy("SUMMER_SALE", priceValue -> priceValue * 0.85);
        System.out.println("☀️ SUMMER_SALE 적용: " + eventStrategy.applyEvent("SUMMER_SALE", price)); // 85.0

        System.out.println(eventStrategy.applyEvent("Free", price)); // 예외 
    }
}




```
