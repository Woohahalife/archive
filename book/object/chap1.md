![object01.png](image%2Fobject01.png)

#### 구현 내용의 문제점

##### 소프트웨어 모듈이 가져야 하는 세가지 기능
- 실행 중에 제대로 동작해야 한다.
- 변경을 위해 존재 -> 간단한 작업 만으로도 변경이 가능해야함
- 코드를 읽는 사람과 의사소통 -> 개발자가 쉽게 읽고 이해할 수 있어야 함

```java
/**  
 * 극장을 구현한 클래스  
 */  
public class Theater {  
  
    private TicketSeller ticketSeller;  
  
    public Theater(TicketSeller ticketSeller) {  
        this.ticketSeller = ticketSeller;  
    }  
  
    // 관람객의 입장 로직 구현  
    public void enter(Audience audience) {  
  
        if(audience.getBag().hasInvitation()) { // 가방 안에 초대장이 들어 있는지 확인  
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();  
  
            audience.getBag().setTicket(ticket);  
        } else {  
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();  
  
            audience.getBag().minusAmount(ticket.getFee());  
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());  
  
            audience.getBag().setTicket(ticket);  
        }  
    }  
}
```

구현 코드는 관람객을 입장 시키는데 필요한 기능을 오류없이 정확하게 수행하고 있음
하지만 변경 용이성과 읽는 사람과의 의사소통 목적은 만족하지 못함

##### 예상을 빗나가는 코드

- 소극장은 관람객의 가방을 열어 그 안에 초대장이 들어있는지 살펴봄
- 가방 안에 초대장이 있다면 판매원은 매표소에 보관된 티켓을 가방으로 옮김
- 초대장이 없다면 관람객의 가방에서 티켓 금액만큼의 현금을 꺼내 매표소에 적립
- 매표소에 보관되어 있는 티켓을 관람객의 가방 안으로 옮김

##### 문제 1
관람객과 판매원이 소극장의 통제를 받는 수동적 존재이다.
- 소극장 입장에서는 관람객 허락없이 가방을 뒤지고 있음(검증문)
- 소극장에서 판매원의 허락없이 매표소의 티켓과 현금을 제어하고 있음

현실 상황에서는 관람객이 직접 자신의 가방에서 초대장을 꺼내 판매원에게 건냄
티켓 구매 상황 발생시에도 관람객이 직접 돈을 꺼내 판매원에게 지불
판매원도 직접 매표소의 티켓을 꺼내 관람객에게 건내고 관람객에게서 돈을 받아 매표소에 보관함

> 제대로 동작 하지만 코드의 동작 방식이 현실 상황에서의 상식과 완전히 다르게 동작함

##### 문제 2

`Theater`의 `enter()` 메서드에서 너무 많은 세부 사항을 다루고 있음
- `enter()`메서드에서 `Audience`, `Bag`, `TickerSeller` 의 세부적인 내용을 전부 알고 있어야함
- 코드 작성자 뿐만 아니라 읽는 사람에게도 부담이 된다.
- `Audience`와 `TickerSeller`를 변경할 경우 `Theater`도 함께 변경해야함

#### 변경에 취약한 코드

위의 코드는 **변경에 취약하다는 점**이 가장 큰 문제가 됨
- 관람객이 가방을 들고있다는 가정이 바뀔 경우 `Theater`의 `enter()`메서드 역시 수정됨
- `Theater`는 관람객이 가방을 들고 있고, 판매원이 매표소에서만 티켓을 판매한다는 세부 내용에 의존해 동작함
- 세부 내용 중 하나라도 바뀌면 해당 클래스 뿐만 아니라 의존하는 `Theater`도 함께 변경해야 함

**의존성은 변경에 대한 영향을 암시한다.** 어떤 객체가 변경될 때 그 객체에 의존하는 다른 객체도 함께 변경될 수 있다는 사실이 내포되어 있다.
-> **구현에 필요한 최소한의 의존성만 유지하고 불필요한 의존성을 제거하는 것이 중요**

설계 목표 : 객체 사이의 결합도를 낮춰 변경이 용이한 설계를 만드는 것

### 설계 개선

코드를 이해하기 어려운 이유는 `Theater`가 관람객의 가방과 매표소에 직접 접근하기 때문
- `Theater`가 `Audience`와 `TickerSeller`에 결합된다는 것을 의미
- `Audience`와 `TickerSeller`를 변경할 때 `Theater`도 함께 변경해야함

`Theater`가 `Audience`와 `TickerSeller`에 관해 세세한 부분까지 알지 못하도록 정보를 차단하자
- 관람객이 가방을 가지고 있다는 사실과 판매원이 매표소에서 티켓을 판매한다는 사실을 `Theater`가 알아야 할 필요가 없다.
- `Theater`는 관람객이 소극장에 입장하는 것만 알면 된다.
- **관람객이 스스로 가방안의 현금과 초대장을 처리하고, 판매원이 스스로 매표소의 티켓과 요금을 다루면 됨**

> **관람객과 판매원을 자율적인 존재로 만들면 되는 것**

```java
public class Theater {  
  
    private TicketSeller ticketSeller;  
  
    public Theater(TicketSeller ticketSeller) {  
        this.ticketSeller = ticketSeller;  
    }  
  
    // 관람객의 입장 로직 구현  
    public void enter(Audience audience) {  
  
        ticketSeller.sellTo(audience); // 세부 로직을 캡슐화  
    }  
}


public class TicketSeller {  
  
    private TicketOffice ticketOffice;  
  
    public TicketSeller(TicketOffice ticketOffice) {  
        this.ticketOffice = ticketOffice;  
    }

//	public TicketOffice getTicketOffice() {  
//	    return ticketOffice;  
//	}
  
    // 티켓을 TicketSeller가 직접 판매  
    public void sellTo(Audience audience) {  
  
        if(audience.getBag().hasInvitation()) { // 가방 안에 초대장이 들어 있는지 확인  
            Ticket ticket = ticketOffice.getTicket();  
  
            audience.getBag().setTicket(ticket);  
        } else {  
            Ticket ticket = ticketOffice.getTicket();  
  
            audience.getBag().minusAmount(ticket.getFee());  
            ticketOffice.plusAmount(ticket.getFee());  
  
            audience.getBag().setTicket(ticket);  
        }  
    }  
}
```

- `enter()` 메서드에서 `TicketOffice`에 접근해 티켓을 판매하는 코드를 `TicketSeller` 내부로 숨겼다.
- `TicketOffice`에 접근해 티켓을 판매 & 교환하는 행위는 `TicketSeller`가 직접 수행하게 되었다.
- `Theater`에서는 이제 `TicketSeller`의 `TicketOffice`에 직접 접근할 수 없다.
- `Theater`는 `TicketSeller`가 `sellTo()`메세지를 이해하고 응답할 수 있다는 사실만 알고 있다.

**캡슐화를 통해 객체 내부의 직접 접근을 제한하고 객체 사이의 결합도를 낮출 수 있다.**
- `Theater`는 `TicketSeller`의 **인터페이스(interface)**에만 의존한다.
- `TicketSeller`가 내부에 `TicketOffice` 인스턴스를 포함하고 있다는 사실은 **구현(implementation)**영역에 속한다.
> 객체를 인터페이스와 구현으로 나누고 인터페이스만을 공개하는 것은 객체 사이의 결합도를 낮추고 변경하기 쉬운 코드 작성을 위해 필요한 가장 기본적 설계원칙이다.

![object02.png](image%2Fobject02.png)

`Theater`의 로직을 `TicketSeller`로 이동시킨 결과, `Theater`에서 `TicketOffice`로의 의존성이 제거되었다.

```java
public class TicketSeller {  
  
    private TicketOffice ticketOffice;  
  
    public TicketSeller(TicketOffice ticketOffice) {  
        this.ticketOffice = ticketOffice;  
    }  
  
    // 티켓을 TicketSeller가 직접 판매  
    public void sellTo(Audience audience) {  
  
        ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket())); // 티켓 구매 금액에 따라 office의 보유 금액 증액  
    }  
}

public class Audience {  
  
    private Bag bag;  
  
    public Audience(Bag bag) {  
        this.bag = bag;  
    }  
  
//    public Bag getBag() {  
//        return bag;  
//    }  
  
    // 관람객이 직접 가방 안의 초대장을 확인 & 직접 금액 지불  
    public Long buy(Ticket ticket) {  
  
        if(bag.hasInvitation()) { // 가방 안에 초대장이 들어 있는지 확인  
            bag.setTicket(ticket);  
  
            return 0L; // 반환금액 없음  
        } else {  
  
            bag.setTicket(ticket);  
            bag.minusAmount(ticket.getFee()); // 보유 금액 차감  
  
            return ticket.getFee(); // 티켓 구매 금액 반환  
        }  
    }  
}
```

`TicketSeller`와 `Audience`간의 의존성도 동일한 방법으로 캡슐화를 통해 개선할 수 있다.
`TicketSeller`에서 `Bag`에 접근하는 모든 로직을 `Audience` 내부에 `buy()`메서드를 추가해 로직을 감추고, `Audience`가 직접 `Bag`안에 초대장 유무 검증 및 금액 지출 행위 등을 수행하도록 변경했다.

![object03.png](image%2Fobject03.png)

- `TickerSeller`는 `Audience`의 인터페이스에만 의존하도록 수정했다.
- `Audience`와 `TicketSeller`는 내부 구현을 외부에 노출하지 않고 자신의 문제를 스스로 해결한다.

### 개선내용

- `Theater`는 `TicketSeller` 내부에 대해서는 전혀 알지 못한다.
- `TicketSeller`도 `Audience`의 내부에 대해서 전혀 알지 못한다.
- `Audience`와 `TicketSeller`는 자신이 가지고 있는 소지품을 스스로 관리한다.
- `Audience`와 `TicketSeller`의 내부 구현을 변경하더라도 `Theater`를 함께 변경할 필요가 없어졌다.

> **객체 내부의 상태를 캡슐화하고 객체 간의 메시지를 통해서만 상호작용 하도록 개선했다.**

### 캡슐화와 응집도

밀접하게 연관된 작업만을 수행하고 연관성 없는 작업은 다른 객체에게 위임하는 객체를 **응집도(cohesion)**가 높다고 말한다.
- 자신의 데이터를 스스로 처리하는 자율적인 객체를 만들면 결합도를 낮추고 응집도를 높일 수 있다.
- 응집도를 높이기 위해선 객체 스스로 자신의 데이터를 책임져야함

>  **외부의 간섭을 최대한 배제하고 메시지를 통해서만 협력하는 자율적인 객체들의 공동체를 만들어야한다.**

### 설계가 왜 필요한가

설계는 코드를 배치하는 것이다.
- 설계는 코드를 작성하는 매 순칸 코드를 어떻게 배치할 것인지를 결정하는 과정에서 나온다.
- 설계는 코드 작성의 일부이며 코드를 작성하지 않고는 검증할 수 없다.

좋은 설계란 **오늘 요구하는 기능을 온전히 수행하면서 내일의 변경을 매끄럽게 수용할 수 있는 설계**다.
- 요구사항은 항상 변경된다.
- 개발 시작 시점에 모든 요구사항을 수집하는 것은 불가능하다.
- 개발이 진행되는 동안 요구사항은 결국 바뀐다.

#### 객체지향 설계

객체지향 프로그래밍은 의존성을 효율적으로 통제할 수 있는 다양한 방법을 제공함으로써 요구사항 변경에 좀 더 수월하게 대응할 수 있는 가능성을 높여준다.
- 객체지향 패러다임은 실제 세상을 바라보는 방식대로 코드를 작성할 수 있게 돕는다.
- 객체는 자신의 데이터를 스스로 책임지는 자율적인 존재이다.
- 실제 세상에 대해 예상하는 방식대로 객체의 행동을 보장함으로써 코드를 좀 더 쉽게 이해할 수 있다.

> 훌륭한 객체지향 설계란 협력하는 객체 사이의 의존성을 적절하게 관리하는 설계이다.