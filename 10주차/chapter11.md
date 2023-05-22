# 11장 단위 테스트 안티 패턴

### 11.1 비공개 메서드 단위 테스트

단위 테스트를 위해 비공개 메서드를 노출하는 경우에  
식별할 수 있는 동작만 테스트한다는 단위 테스트 기본 원칙을 위반하는 것이다.  
비공개 메서드를 직접 테스트 하는 대신, 포괄적인 식별할 수 있는 동작으로서 간접적으로 테스트하는 것이 좋다.  
때로는 비공개 메서드가 너무 복잡해서 식별할 수 있는 동작으로 테스트하기에 충분한 커버리지를 얻을 수 없는 경우가 있다.
- 테스트에서 벗어난 코드가 어디에도 사용되지 않는 다면 리팩터링 후에도 남아서 관계없는 코드일 수 있다. 
- 비공개 메서드가 너무 복잡하면 별도의 클래스로 도출해야 하는 추상화가 누락됐다는 징후이다.

**비공개 메서드 테스트가 타당한 경우**  
식별할 수 있는 동작이면서 비공개 메서드인 경우,   
메서드가 식별할 수 있는 동작이 되려면 클라이언트 코드에서 사용 되어야 하는데 해당 메서드가 비공개인 경우에는 사용이 불가능하다.  
비공개 메서드를 테스트하는 것 자체는 나쁘지 않다.  
비공개 메서드가 구현 세부 사항의 프록시에 해당하므로 나쁜것이다.  
구현 세부 사항을 테스트 하면 궁극적으로 깨지기 쉽기 때문이다.


예시) 신용조회를 관리하는 시스템   
하루에 한번 데이터베이스에서 직접 대량으로 새로운 조회를 로드한다.  
관리자는 그 조회를 하나씩 검토하고 승인 여부를 결정한다.
```
public class Inquiry
{
    public bool IsApproved { get; private set; }
    public DateTime? TimeApproved { get; private set; }
    
    // 비공개 생성자
    private Inquiry(
        bool isApproved, DateTime? timeApproved)
    {
        if (isApproved && !timeApproved.HasValue)
            throw new Exception();
        IsApproved = isApproved;
        TimeApproved = timeApproved;
    }
    
    public void Approve(DateTime now)
    {
        if (IsApproved)
            return;
        IsApproved = true;
        TimeApproved = now;
    }
}
```
객체를 인스턴스화할 수 없다는 점을 고려해 Inquiry 클래스는 어떻게 테스트 할까?  
승인 로직은 중요하므로 단위 테스트를 거쳐야 한다.  
Inquiry 생성자는 비공개이면서 식별할 수 있는 동작인 메서드의 예이다.  
ORM은 생성자없이 데이터베이스에서 조회를 복원할 수 없기 때문에  
Inquiry 생성자를 공개한다고 해서 테스트가 쉽게 깨지지 않는다.  

### 11.2 비공개 상태 노출
또 다른 안티패턴으로 단위 테스트 목적으로만 비공개 상태를 노출하는 것이다.  
비공개로 지켜야 하는 상태를 노출하지 말고 식별할 수 있는 동작만 테스트하라는 비공개 메서드 지침과 같다.  

고객은 각각 Regular 상태로 생성된 후에 Preferred로 업그레이드 할 수 있으며,  
업그레이드하면 모든 항목에 5% 할인을 받는다.
Promote 메서드를 어떻게 테스트하겠는가?  
이 메서드의 사이드 이팩트는 _status필드의 변경이지만 필드는 비공개이므로 테스트 할 수 없다.
```java
public class Customer
{
    private CustomerStatus _status = CustomerStatus.Regular;
    
    public void Promote()
    {
        _status = CustomerStatus.Preferred;
    }
    
    public decimal GetDiscount()
    {
        return _status == CustomerStatus.Preferred ? 0.05m : 0m;
    }
}

public enum CustomerStatus
{   
    Regular,
    Preferred 
}
```
해결책은 필드를 공개하는 것이지만 이는 안티패턴이다.  
대신 제품코드가 이 클래스를 어떻게 사용하는지를 살펴보면 된다.    
제품코드가 관심을 갖는 정보를 승격 후 고객이 받는 할인 뿐이다.  
이것이 테스트에서 확인할 사항이다.  
- 새로 생성된 고객은 할인이 없음
- 업그레이드 시 5%할인율 적용

### 1.3 테스트로 유출된 도메인 지식
도메인 지식을 테스트로 유출하는 것은 안티패턴이다.  
보통 복잡한 알고리즘을 다루는 테스트에서 일어난다.  
예시) 계산 알고리즘  
```
public static class Calculator
{
    public static int Add(int value1, int value2)
    {
        return value1 + value2;
    }
}
```
```
public class CalculatorTests
{
    [Fact]
    public void Adding_two_numbers()
    {
        int value1 = 1;
        int value2 = 3;
        int expected = value1 + value2; // 알고리즘 유출

        int actual = Calculator.Add(value1, value2);
        Assert.Equal(expected, actual);
    }
}
```
테스트 하는 과정에서 알고리즘(예제에서는 단순하지만)이 유출되었다.    
테스트를 작성할 때는 특정 구현을 암시하면 안된다.  
알고리즘을 복제하는 대신 다음 예제와 같이 테스트에 예상 결과값을 하드코딩하는 것이 낫다.
```
public class CalculatorTests
{
    [Theory]
    [InlineData(1, 3)]
    [InlineData(11, 33)]
    [InlineData(100, 500)]
    public void Adding_two_numbers(int value1, int value2)
    {
        int expected = value1 + value2; 
        int actual = Calculator.Add(value1, value2);
        Assert.Equal(expected, actual);
    }
}
```

### 11.4 코드 오염
코드 오염도 안티패턴이다.  
코드 오염은 테스트만 필요한 제품코드를 추가하는 것이다.    
코드 오염은 종종 다양한 유형의 스위치 형태를 취한다.  
```
public class Logger
{
    private readonly bool _isTestEnvironment;
    public Logger(bool isTestEnvironment)
    {
        _isTestEnvironment = isTestEnvironment;
    }
    
public void Log(string text)
    {
        if (_isTestEnvironment)
            return;
        /* Log the text */
    }
}

public void Some_test()
{
    var logger = new Logger(true); // 테스트 환경을을 나타내고자 매개변수를 true로 설정
    ...
}
```
코드 오염의 문제는 테스트 코드와 제품 코드가 혼재돼 유지비가 증가하는 것이다.  
이러한 안티패턴을 방지하려면 테스트 코드를 제품 코드 베이스와 분리해야 한다.


### 11.5 구체 클래스를 목으로 처리하기
구체 클래스를 목으로 처리해서 본래 클래스의 기능 일부를 보존할 수 있다.  
때로는 유용할 수 있지만 단일 책임 원칙을 위배한다는 단점이 있다.  

예시) 특정 고객에게 배달된 모든 배송물의 무게와 비용 같은 고객 통계를 수집하고 계산한다.  
이 클래스는 외부 서비스에서 검색한 배달 목록을 기반으로 계산한다.
```

public class StatisticsCalculator
{
    public (double totalWeight, double totalCost) Calculate(
        int customerId)
    {
        List<DeliveryRecord> records = GetDeliveries(customerId);
        double totalWeight = records.Sum(x => x.Weight);
        double totalCost = records.Sum(x => x.Cost);
        return (totalWeight, totalCost);
    }
    public List<DeliveryRecord> GetDeliveries(int customerId)
    {
        /* 프로세스 외부 의존성을 호출해 배달 목록 조회 */
    }
}
```
StatisticsCalculator에는 비관리 의존성과 통신하는 책임과 통계를 계산하는 책이 있다.  
책임이 서로 관련이 없음에도 결합되어 있다.  
이는 별도로 분리하는 것이 좋다.
```
public class DeliveryGateway : IDeliveryGateway
{
    public List<DeliveryRecord> GetDeliveries(int customerId)
    {
        /* 프로세스 외부 의존성을 호출해 배달 목록 조회 */
    }
}

public class StatisticsCalculator
{
    public (double totalWeight, double totalCost) Calculate(
        List<DeliveryRecord> records)
    {
        double totalWeight = records.Sum(x => x.Weight);
        double totalCost = records.Sum(x => x.Cost);
        return (totalWeight, totalCost);
    }
}
```
비관리의존성과 통신하는 책임은 이제 DeliveryGateway로 넘어갔다.  
이 게이트 웨이 뒤에 인터페이스가 있으므로 구체 클래스 대신 인터페이스를 목에 사용할 수 도 있다.


### 11.6 시간 처리하기
많은 어플리케이션의 기능에서는 현재 날짜와 시간에 대한 접근이 필요하다.  
시간에 따라 달라지는 기능을 테스트하면 거짓 양성이 발생할 수 있다.   
실행 단계의 시간이 검증 단계의 시간과 다를 수 있다.  
이 의존성을 안정화하는 방법  
첫번째, 앰비언트 컨텍스트 
```
public static class DateTimeServer
{
    private static Func<DateTime> _func;
    public static DateTime Now => _func();
    public static void Init(Func<DateTime> func)
    {
        _func = func;
    }
}
DateTimeServer.Init(() => DateTime.Now); // 운영 환경 초기화 코드
DateTimeServer.Init(() => new DateTime(2020, 1, 1));  // 단위 테스트 환경 초기화 코드
```
안티 패턴이다, 제품 코드를 오염시키고 테스트를 더 어렵게 한다.    
정적 필드는 테스트 간에 공유하는 의존성을 도입해 해당 테스트를 통합 테스트 영역으로 전환한다.

두번째, 명시적 의존성으로서의 시간  
서비스 또는 일반 값으로 시간 의존성을 명시적으로 주입하는 것이다.  
이 두가지 옵션중에서는 시간을 서비스로 주입하는 것보다는 값으로 주입하는 것이 더 낫다.  
제품 코드에서 일반 값으로 작업하는 것이 더 쉽고, 테스트에서 해당 값을 스텀으로 처리하기도 더 쉽다.   
하지만 항상 일반 값으로 주입 할수는 없을 것이다. 의존성 주입 프레임워크가 값 객체와 잘 어울리지 않기 때문이다.
