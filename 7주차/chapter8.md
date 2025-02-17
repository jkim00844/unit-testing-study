# 8장 통합테스트를 하는 이유
### 8.1 통합 테스트는 무엇인가?
통합 테스트는 테스트 스위트에서 중요한 역할을 하며, 단위 테스트의 개수와 통합 테스트의 개수의 균형을 맞추는 것이 필요하다.

**통합 테스트의 역할**  
단위 테스트는 다음 세 가지 요구 사항을 충족하는 테스트다.
- 단일 동작 단위를 검증하고,
- 빠르게 수행하고,
- 다른 테스트와 별도로 처리한다.

이 세가지 요구 사항 중 하나라도 충족하지 못하면 통합테스트 범주에 속한다.    
실제로 통합테스트는 대부분 시스템이 프로세스 외부 의존성과 통합해 어떻게 작동하는지를 검증한다.   
단위테스트는 도메인 모델을 다루고 통합테스트는 프로세스 외부 의존성과 도메인 모델을 연결하는 코드를 확인한다.    

<img src="https://drive.google.com/uc?export=view&id=1Oa1j7sH1RaMWV-Qq6yEyX4_mNn8OIAuG"></br>
테스트 피라미드는 대부분의 애플리케이션에 가장 적합한 절충안을 나타낸다.  
빠르고 저렴한 단위테스트는 대부분의 예외 상황을 다루는 (=빠른 피드백, 유지 보수성)  
반면, 느리고 비용이 많이 드는 통합 테스트는 개수가 적지만 시스템 전체의 정확성을 보장한다. (=회귀 방지, 리팩터링 내성)

빠른 실패 원칙  
예기치 않은 오류가 발생하자마자 현재 연산을 중단하는 것을 의미한다.    
보통 예외를 던져서 현재 연산을 중지한다. 예외는 그 의마가 빠른 실패 원칙에 완벽히 부합한다.  
버그는 빨리 발견할 수록 더 쉽게 해결할 수 있고 빨리 실패하면 손상이 확산 되는 것을 막을 수 있다.  
이는 단위 테스트로 확인하는 것이 더 낫다.

### 8.2 어떤 프로세스 외부 의존성을 직접 테스트 해야 하는가?
프로세스 외부 의존성은 실제 의존성을 사용하거나 목으로 대체해서 테스트한다.  

**프로세스 외부 의존성의 두가지 유형**  
- 관리 의존성 : 전체를 제어할 수 있는 프로세스 외부 의존성  
이 의존성은 애플리케이션을 통해서만 접근할 수 있으며 외부 환경에서 볼 수 없다.  
예) 데이터 베이스, 외부 시스템은 보통 데이터베이스에 직접 접근하지 않고 애플리케이션에서 제공하는 API를 통해 접근한다.
- 비관리 의존성 : 전체를 제어할 수 없는 프로세스 외부 의존성    
해당 의존성과의 상호작용을 외부 환경에서 볼수 있다.  
예) SMTP 서버와 메시지 버스, 둘 다 다른 애플리케이션에서 볼 수 있는 사이드 이펙트를 발생시킨다.  

<img src="https://drive.google.com/uc?export=view&id=1uMilxVkOuZmgqym8cFrFrdZ1Nj1X1GwZ"></br>
관리 의존성과의 통신은 구현 세부 사항이므로 실제 인스턴스를 사용해 테스트해야 하고  
비관리 의존성은 식별할 수 있는 동작(계약)이므로 목으로 대체해 테스트해야 한다.  

**통합 테스트시 실제 데이터베이스를 사용할 수 없는 경우**  
때로는 관리 범위를 벗아난다는 이유로(IT 보안 정책, 테스트 데이터베이스 인스턴스를 설정하고 유지하는 비용등)  
통합 테스트에서 관리 의존성을 실제 버전으로 사용할 수 없는 경우도 있다.  
테스트 자동화 환경에 배포할 수 없는 레거시 데이터베이스를 예로 들 수 있다.    
데이터베이스를 그래도 테스트할 수 없으면 통합 테스트를 아예 작성하지 말고 도메인 모델의 단위테스트에만 집중해도 괜찮다.  
오히려 관리 의존성을 목으로 대체하면 통합테스트의 리팩터링 내성이 저하된다.  
가치가 충분하지 않는 테스트는 테스트 스위트에 있어서는 안된다.  

### 8.3 통합 테스트 : 예제
<img src="https://drive.google.com/uc?export=view&id=1JF0Aakps9cG4qVR57QkB2aK6JbDJQaLk"></br>
통합테스트 시나리오
- 데이터베이스에 사용자와 회사를 삽입하고
- 해당 데이터베이스에서 이메일 변경 시나리오를 실행하며
  - 사용자는 유형을 기업에서 일반으로 변경한 뒤, 이메일도 변경하고 회사는 직원 수를 변경한다.
  - 메시지 버스로 메시지를 보낸다.
- 데이터베이스 상태를 검증하게 된다.
데이터 베이스는 관리의존성으로 실제 인스턴스를 사용하고 메시지 버스는 목으로 대체하여 테스트 한다.

**엔드 투 엔드 테스트는 어떤가?**   
<img src="https://drive.google.com/uc?export=view&id=1gtHXAi1MXPRn_c_9EdO7e_kJlwD1otwu"></br>
엔드 투 엔드 테스트는 외부 클라이언트를 모방하므로 테스트 범위에 포함된 모든 프로세스 외부 의존성을 참조하는 배포된 버전의 애플리케이션을 테스트한다.  
관리 의존성(데이터베이스 등) 을 직접 확인하는 것이 아닌 애플리케이션을 통해 간접적으로 확인해야 한다.  

### 8.4 의존성 추상화를 위한 인테페이스 사용 
인터페이스를 사용하는 일반적인 이유
- 프로세스 외부 의존성을 추상화해 느슨한 결합을 달성하고
- 기존 코드를 변경하지 않고 새로운 기능을 추가해 공개 폐쇄 원칙(OCP)를 지키기 때문이다. 

이 두 가지 이유 모두 오해다.  
단일 구현을 위한 인터페이스는 추상화가 아니며 해당 인터페이스를 구현하는 구체 클래스보다 결합도가 낮지 않다.  
또 다른 이유는 현재 필요하지 않는 기능에 시간을 들이지 말아야 한다.   
새로운 기능을 추가가 안될 수 있기 때문에 미리 인터페이스를 만들어 코드를 증가 시키는 것은 좋지 않다.

그럼에도  
프로세스 외부 의존성에 인터페이스를 사용하는 이유는?  
실용적이고 현실적이다. 다실 말해 목을 사용하기 위함이다.  
인터페이스가 없으면 테스트 대역을 만들 수 없으므로 테스트 대상 시스템과 프로세스 외부 의존성 간의 상호 작용을 확인할 수 없다.  
따라서 의존성을 목으로 처리할 필요가 없는 한 프로세스 외부 의존성에 대해 인터페이스를 두지 않는 것이 좋다.  

### 8.5 통합테스트 모범 사례
통합 테스트를 최대한 활용하는 데 도움이 되는 몇 가지 일반적인 지침이 있다.
1. 도메인 모델의 경계 명시  
항상 도메인 모델은 코드베이스 내에서 명시적이고 잘 알려진 위치에 두어야 한다.  
단위 테스트는 도메인 모델과 알고리즘을 대상으로 하고  
통합테스트는 컨트롤러를 대상으로 한다.  
도메인 클래스와 컨트롤러 사이에 명확한 경계는 단위 테스트와 통합 테스트를 쉽게 구별할 수 있다. 

2. 애플리케이션 내 계층 줄이기  
애플리케이션에 추상 계층이 너무 많으면 코드베이스를 탐색하기 어렵고 간단한 연산이라 해도 숨은 로직을 이해하기 너무 어려워진다.   
추상화가 지나치게 많으면 단위 테스트와 통합 테스트에도 도움이 되지 않는다. 
간접 계층은 코드를 추론하는 데 부정적인 영향을 미친다. 가능한 적게 사용해라.  
대부분 백엔드 시스템에서는 도메인 모델, 애플리케이션 서비스 계층(컨트롤러), 인프라 계층 이 세가지만 활용하면 된다.  
인프라 계층은 보통 도메인 모델에 속하느 않는 알고리즘과 프로세스 외부 의존성에 접근 할 수 있는 코드로 구성된다.  

3. 순환 의존성 제거  
순환 의존성은 둘 이상의 클래스가 제대로 작동하고자 직간접적으로 서로 의존하는 것을 말하며 대표적인 예는 콜백이다.  
순환 의존성은 코드를 읽고 이해하기 어렵기 때문에 테스트하기도 어렵다.
코드 베이스에서 순환 의존성을 모두 제거하는 것은 거의 불가능하므로 최대한 작게 만드는 것이 좋다.

**테스트에서 다중 실행 구절 사용**  
테스트에서 두 개 이상의 준비나 실행 또는 검증 구절을 두는 것은 코드 악취에 해당한다.  
테스트가 여러 가지 동작 단위를 확인해서 테스트의 유지 보수성을 저해한다는 신호다.  
예를 들어 사용자 등록과 삭제와 같이 두가지 유스케이스가 있으면 하나의 통합 테스트에서 두 유스케이스를 모두 확인하려 할 수 있다.
- 준비 : 사용자 등록에 필요한 데이터 준비
- 실행 : UserController.RegisterUser() 호출
- 검증 : 등록이 성공적으로 완료됐는지 확인하기 위해 데이터베이스 조회
- 실행 : UserController.DeleteUser 호출
- 검증 : 사용자가 삭제됐는지 확인하기 위해 데이터베이스 조회

이 방식은 사용자 상태가 자연스럽게 흐르기 때문에 이상이 없어보이지만  
이러한 테스트는 초점을 잃고 순식간에 테스트가 너무 커질 수 있다.  
따라서 각 실행은 별개의 고유 테스트로 나누는 것이 좋다.  

### 8.6 로깅을 테스트하는 방법
로깅은 회색 지대로 테스트에 관해서는 어떻게 해야 할지 분명하지 않다.  
로깅은 애플리케이션의 동작에 대해 중요한 정보를 생성한다.  
그러나 로깅은 너무 보편적이므로 테스트 노력을 들일 가치가 있는지 분명하지 않다.   


텍스트 파일이나 데이터베이스와 같은 프로세스 외부 의존성에서 발생하는 사이드 이펙트를  
애플리케이션 클라이언트 또는 개발자 이외의 다른 사람이 보는 경우라면  
로깅도 식별할 수 있는 동작이므로 반드시 테스트 해야 한다.  
하지만 로깅을 보는 이가 개발자 뿐이라면 구현 세부 사항이므로 테스트해서는 안된다.  

로깅은 두가지 유형으로 나눌 수 있다. 
- 지원 로깅은 지원 담당자나 시스템 관리자가 추적할 수 있는 메시지를 생성한다. -> 테스트 O
- 진단 로깅은 개발자가 애플리케이션 내부 상황을 파악할 수 있도록 돕는다. -> 테스트 X

### 정리
통합 테스트는 시스템이 프로세스 외부 의존성과 통합해 작동하는 방식을 검증한다.  
통합 테스트는 컨트롤러를 다루고, 단위 테스트는 알고리즘과 도메인 모델을 다룬다.  
통합 테스트는 회귀 방지와 리팩터링 내성에 우수하고, 단위테스트는 유지보수성과 피드백 속도가 우수하다.  
통합 테스트의 기준은 단위 테스트 보다 높으며  
시스템이 전체적으로 올바른지 확인하기 때문에 속도가 느리고 비용이 많이 발생하므로 그 수가 적어야한다.  