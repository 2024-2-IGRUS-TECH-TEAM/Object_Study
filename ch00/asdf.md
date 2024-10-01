## 도입부

프로그래밍 패러다임은 개발자 공동체가 동일한 프로그래밍 스타일과 모델을 공유할 수 있게 함으로써 불필요한 부분에 대한 의견 충돌을 방지하며, 프로그래밍 패러다임을 교육시킴으로써 동일한 규칙과 방법을 공유하는 개발자로 성장할 수 있도록 준비시킨다.

이 책은 객체지향에 대한 다양한 오해를 제거함으로써 객체지향 프로그래밍을 하는 개발자들이 동일한 규칙과 표준에 따라 프로그램을 작성하도록 한다.

## 1장 _ 객체, 설계

글래스에 따르면, 이론 뒤에 실무가 따르는 것이 아니라, 성공적인 실무 뒤에 효과적인 이론이 뒤따른다. 실제로, 소프트웨어의 유지보수에 있어서, 실무에서는 다양한 규모의 소프트웨어를 성공적으로 유지보수하고 있지만 그와 관련된 효과적인 이론이 발표된 적은 거의 없다.

글래스의 의견에 따라, 이론과 개념은 잠시 미루고 간단한 프로그램으로 절차지향적 프로그래밍과 객체지향적 프로그래밍에 대해 알아보자.

> **티켓 판매 애플리케이션 구현하기**
> 

우리는 작은 소극장을 경영하고 있다.

일반적인 소극장처럼, 극장에서 관람객을 입장시키고, 매표소에서 티켓을 현금으로 구매하고, 매표소에는 티켓을 파는 직원이 있다.

또한 소극장은 작은 이벤트를 하고 있는데, 추첨을 통해 선정된 관람객에게 공연을 무료로 관람할 수 있는 초대장을 발송하는 것이다.

이러한 소극장을 먼저, 절차지향적인 패러다임의 코드로 구현해보자.

먼저 이벤트 당첨자에게 발송되는 초대장을 구현하자.

```java
public class Invitation {
	private LocalDateTime when;
}
```

공연을 관람하기를 원하는 모든 사람들은 티켓을 소지하고 있어야 한다.

```java
public class Ticket {
	private Long fee;
	
	public Long getFee() {
		return fee;
	}
}
```

이벤트 당첨자는 티켓으로 교환할 초대장을 가지고 있을 것이며, 이벤트에 당첨되지 않은 관람객은 티켓을 구매할 수 있는 현금을 보유하고 있을 것이다. 

관람객은 현금, 티켓, 초대장을 담을 가방을 들고 올 수 있다고 가정하자.

```java
public class Bag {
	private Long amount;
	private Invitation invitation;
	private Ticket ticket;
	
	public boolean hasInvitation() { // 초대장을 가지고 있는지 여부
		return invitation != null;
	}
	
	public boolean hasTicket() { // 티켓을 가지고 있는지 여부
		return ticket != null;
	}
	
	public void setTicket(Ticket ticket) { // 초대장을 티켓으로 교환
		this.ticket = ticket;
	}
	
	public void minusAmount(Long amount) { // 현금 감소
		this.amount -= amount;
	}
	
	public void plusAmount(Long amount) { // 현금 증가
		this.amount += amount;
	}
}
```

위 `Bag` 클래스는 초대장(`invitation`), 티켓(`ticket`), 현금(`amount`)를 인스턴스 변수로 포함하고,

초대장의 보유 여부를 판단하는 `hasInvitation` 메소드와 티켓의 소유 여부를 판단하는 `hasTicket` 메소드, 현금을 증가시키거나 감소시키는 `plusAmount`와 `minusAmount` 메소드, 초대장을 티켓으로 교환하는 `setTicket` 메소드를 구현하고 있다.

`Bag` 인스턴스의 상태는, 현금과 초대장을 함께 보관하거나 - (1), 초대장 없이 현금만 보관하거나 - (2)의 두 가지 상황이 있을 것이다.

`Bag` 인스턴스를 생성하는 시점에 이 제약을 강제할 수 있도록 생성자를 추가해보자.

```java
public class Bag {
	public Bag(long amount) { // 초대장 없이 현금만 보관하도록 강제
		this(null, amount);
	}
	
	public Bag(Invitation invitation, long amount) { // 초대장, 현금을 보관하도록 강제
		this.invitation = invitation;
		this.amount = amount;
	}
}
```

관람객이라는 개념을 구현하는 `Audience` 클래스를 만들자.

관람객은 소지품을 보관하기 위해 가방을 소지할 수 있다.(`Bag` 인스턴스)

```java
public class Audience {
	private Bag bag;
	
	public Audience(Bag bag) {
		this.bag = bag;
	}
	
	public Bag getBag() {
		return bag;
	}
}
```

관람객이 소극장에 입장하기 위해서는 매표소에서 초대장을 티켓으로 교환 / 구매해야 한다.

따라서 매표소에는 판매할 티켓, 티켓을 판매한 금액이 보관돼 있어야 한다.

`TicketOffice` 클래스를 추가하자.

```java
public class TicketOffice {
	private Long amount;
	private List<Ticket> tickets = new ArrayList<>();
	
	public TicketOffice(Long amount, Ticket ... tickets) {
		this.amount = amount;
		this.tickets.addAll(Arrays.asList(tickets));
	}
	
	public Ticket getTicket() {
		return tickets.remove(0);
	}
	
	public void minusAmount(Long amount) {
		this.amount -= amount;
	}
	
	public void plusAmount(Long amount) {
		this.amount += amount;
	}
}

```

또한 직원은 매표소에서

초대장 → 티켓 으로 교환해 주거나

현금 → 티켓으로 교환해 주는 역할을 수행한다.

직원을 구현한 `TicketSeller` 클래스는 자신이 일하는 `TicketOffice` 를 알고 있어야 한다.

```java
public class TicketSeller {
	private TicketOffice ticketOffice;
	
	public TicketSeller(TicketOffice ticketOffice) {
		this.ticketOffice = ticketOffice;
	}
	
	public TicketOffice getTicketOffice() {
		return ticketOffice;
	}
}
	
```

지금까지 준비한 클래스들을 조합해서 관람객을 소극장에 입장시키자.

소극장을 구현하는 클래스는 `Theater` 이다. `Theater` 클래스 내에서 `enter` 메소드를 구현하자.

```java
public class Theater {
	private TicketSeller ticketSeller;
	
	public Theater(TicketSeller ticketSeller) {
		this.ticketSeller = ticketSeller;
	}
	
	public void enter(Audience audience) {
		if (audience.getBag().hasInvitation()) {
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

다음과 같이 설계한 코드는 잘 동작하긴 하지만, 문제가 있다.

> **무슨 문제가 있는데?**
> 

로버트 마틴은, 소프트웨어 모듈이 가져야 하는 세 가지 기능에 대해서 설명한다. 

그 세 가지 기능은 다음과 같다.

- 모든 모듈은 제대로 실행돼야 한다.
- 변경이 용이해야 한다.
- 이해하기 쉬워야 한다.

위 코드가 문제가 있다는 것은, 제대로 실행돼야 한다는 첫 번째 조건은 만족하지만,

변경 용이성과, 읽는 사람과의 의사소통(이해하기 쉬워야 함)이라는 목적은 만족하지 못한다.

`Theater` 클래스의 `enter` 메소드가 수행하는 일을 설명하자면,

<aside>
💡

소극장은 관람객의 가방을 열어 그 안에 초대장이 들어 있는지 살펴본다.

가방 안에 초대장이 들어 있으면 판매원은 매표소에 보관돼 있는 티켓을 관람객의 가방 안으로 옮긴다.

가방 안에 초대장이 들어 있지 않다면 관람객의 가방에서 티켓 금액만큼의 현금을 꺼내 매표소에 적립한 후에 매표소에 보관돼 있는 티켓을 관람객의 가방 안으로 옮긴다.

</aside>

여기서 문제는, 관람객과 판매원이 모두 **소극장의 통제를 받는다는 점이다.**

또한, 이 코드를 이해하기 위해서는 **여러 세부적인 내용들을 한꺼번에 기억하고 있어야 한다.**

`Theater`의 `enter` 메소드를 이해하기 위해서는 `Audience`가 `Bag`을 가지고 있고, `Bag` 안에는 현금과 티켓이 들어 있으며, `TicketSeller`가 `TicketOffice`에서 티켓을 판매하고, `TicketOffice` 안에는 돈과 티켓이 보관돼 있는 사실을 동시에 기억해야 한다… 이해하기에 너무 복잡하다.

가장 심각한 문제는, 관람객과 판매원이 모두 소극장의 통제를 받고 있기 때문에, `Audience`와 `TicketSeller`를 변경할 경우 `Theater`도 함께 변경해야 한다.

예를 들어서, 관람객이 가방을 들고 있다는 가정이 바뀌었다면?

`Audience` 클래스에서 `Bag`을 제거해야 하고, `Audience`의 `Bag` 에 접근하는 `Theater`의 `enter` 메소드 또한 수정해야 한다.

이것은 객체 사이의 **의존성(Dependency)** 에 관련된 문제이다.

의존성은 변경에 대한 영향을 암시하며, 어떤 객체가 변경될 때 그 객체에게 의존하는 다른 객체도 함께 변경될 수 있다는 사실이 내포되어 있다.

그렇기에 우리는 객체 사이의 의존성을 최소화시키고, 의존하며 협력하는 객체들의 공동체를 구축해야 한다.

객체 사이의 의존성이 과한 경우를 가리켜 **결합도(coupling)**이 높다고 말한다.

반대로, 객체들이 합리적인 수준으로 의존할 경우에는 결합도가 낮다고 말한다.

결론은, 객체 사이의 결합도를 낮춰 용이한 설계를 해야 한다는 것이다.

> **그럼 어떻게 고쳐야 할까?**
> 

위 코드는 `Theater`가 관람객의 가방과 판매원의 매표소에 직접 접근함으로써, 관람객과 판매원이 자율적으로 자신의 일을 처리하지 못하게 만든다.

고로 해결 방법은 간단하다.

`Theater`가 `Audience`와 `TicketSeller`에 관해 너무 세세한 부분까지 알지 못하도록 정보를 차단하면 된다.

관람객이 가방을 가지고 있다는 사실과, 판매원이 매표소에서 티켓을 판매한다는 사실은 `Theater`에서 알 필요가 없다. 그냥 `Theater`가 원하는 것은 관람객이 소극장에 입장하는 것 뿐이다.

따라서 관람객이 스스로 가방 안의 현금과 초대장을 처리하고, 판매원이 스스로 매표소의 티켓과 판매 요금을 다루게 한다면, 모든 문제는 해결될 것이다.

즉, 관람객과 판매원을 소극장에 얽매이지 않는 **자율적인 존재**로 만들면 된다.

`Audience`와 `TicketSeller`의 자율성을 높이기 위해 코드의 설계를 변경해보자.

1. `Theater`의 `enter` 메소드에서 `TicketOffice` 에 접근하는 모든 코드를 `TicketSeller` 내부로 숨긴다. 

```java
public class Theater {
	private TicketSeller ticketSeller;
	
	public Theater(TicketSeller ticketSeller) {
		this.ticketSeller = ticketSeller;
	}
	
	public void enter(Audience audience) {
	  // ...
	}
	
}

public class TicketSeller {
	private TicketOffice ticketOffice;
	
	public TicketSeller(TicketOffice ticketOffice) {
		this.ticketOffice = ticketOffice;
	}
	
	public void sellTo(Audience audience) {
		if (audience.getBag().hasInvitation()) {
			Ticket ticket = ticketSeller.getTicketOffice().getTicket();
			audience.getBag().setTicket(ticket);
		} else {
			Ticket ticket = ticketSeller.getTicketOffice().getTicket();
			audience.getBag().minusAmount(ticket.getFee());
			ticketOffice.plusAmount(ticket.getFee());
			audience.getBag().setTicket(ticket);
		}
	}
}
```

`TicketSeller`에서 `getTicketOffice` 메소드는 제거되었다.

`ticketOffice`에 대한 접근은 오직 `TicketSeller` 안에만 존재하게 된다.

따라서 `TicketSeller`는 `ticketOffice`에서 행해지는 일들을 스스로 수행해야 한다.

이처럼, 개념적이나 물리적으로 객체 내부의 세부적인 사항을 감추는 것을 **캡슐화(encapsulation)**라고 부른다.

**캡슐화의 목적은, 변경하기 쉬운 객체를 만드는 것이다.**

캡슐화를 통해 객체 내부로의 접근을 제한하면, **객체와 객체 사이의 결합도를 낮출 수 있다.** 이는 더 간편한 코드 변경으로 이어진다.

이제 `Theater`의 `enter` 메서드는 `sellTo` 메서드를 호출하는 간단한 코드로 바뀐다.

```java
public class Theater {
	private TicketSeller ticketSeller;
	
	public Theater(TicketSeller ticketSeller) {
		this.ticketSeller = ticketSeller;
	}
	
	public void enter(Audience audience) {
		ticketSeller.sellTo(audience);
	}
}
```

수정된 `Theater` 클래스 어디에서도 `ticketOffice`에 접근하지 않는다.

`Theater`는 오직 `TicketSeller`의 인터페이스(어떤 객체가 수행해야 하는 핵심적인 역할만 규정한 것)에만 의존하며, `TicketSeller`가 내부에 `TicketOffice` 인스턴스를 포함하고 있다는 사실은 구현(그 역할을 세부적으로 정의하는 것)의 영역에 속한다.

객체를 인터페이스와 구현으로 나누고 인터페이스만을 공개하는 것은 결합도를 낮추고 변경하기 쉬운 코드를 작성하기 위해 따라야 하는 가장 기본적인 설계 원칙이다.

1. `TicketSeller` 다음으로 `Audience`의 캡슐화를 개선한다.

`TicketSeller`는 `Audience`의 `getBag` 메소드를 호출해서 `Audience` 내부의 `Bag` 인스턴스에 직접 접근한다. `Bag` 인스턴스에 접근하는 객체가 `Theater`에서 `TicketSeller`로 바뀌었을 뿐, `Audience`는 여전히 자율적인 존재가 아니다.

매표소 티켓 판매원이 관람객의 가방을 뒤져 티켓을 찾는다고 생각해보면, 이해가 쉽다.

`TicketSeller`와 동일한 방법으로, `Audience`의 캡슐화를 개선하자. `Bag`에 접근하는 모든 로직을 `Audience` 내부로 감추자는 것이다.

`TicketSeller` 에서, `Bag`에 접근하는 모든 로직을 빼고, `Audience` 로 그 로직을 모두 옮긴다.

```java
public class Audience {
	private Bag bag;
	
	public Audience(Bag bag) {
		this.bag = bag;
	}
	
	public Long buy(Ticket ticket) {
		if (bag.hasInvitation()) {
			bag.setTicket(ticket);
			return 0L;
		} else {
			bag.setTicket(ticket);
			bag.minusAmount(ticket.getFee());
			return ticket.getFee();
		}
	}
}
```

`Audience` 에서 자신의 가방을 스스로 확인하기 때문에, 외부에서는 더이상 `Audience`가 `Bag`을 소유하고 있다는 사실을 알 필요가 없다.

이제 `TicketSeller`가 `Audience`의 인터페이스만 의존하도록 코드를 수정하자.

```java
public class TicketSeller {
	private TicketOffice ticketOffice;
	
	public TicketSeller(TicketOffice ticketOffice) {
		this.ticketOffice = ticketOffice;
	}
	
	public void sellTo(Audience audience) {
		ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket());
	}
}
```

위와 같이 구현함으로써 `TicketSeller`와 `Audience`간의 결합도가 낮아졌다.

> **뭐가 바뀐걸까?**
> 

수정된 예제 또한 첫 번째 예제와 마찬가지로 관람객들을 입장시키는 데 필요한 기능을 오류 없이 수행한다. 고로 로버트 마틴의 첫 번째 목적을 만족시킨다.

`Audience`와 `TicketSeller`는 첫 번째 예제와 달리, 자신이 가지고 있는 소지품을 스스로 관리한다.

이것은 우리의 예상과도 일치하며, 코드를 읽는 사람과의 의사소통이라는 관점에서 개선되었다.

또한, 각각의 내부 구현을 변경하더라도, `Theater`를 함께 변경할 필요가 없어졌다.

고로 변경 용이성의 측면에서도 개선되었다.

밀접하게 연관된 작업만을 수행하고 연관성 없는 작업은 다른 객체에게 위임하는 객체를 가리켜 

**응집도(cohesion)가 높다**고 말한다(객체지향의 관점에서 합리적임).

 

> **절차지향과 객체지향**
> 

수정하기 전 코드에서 `Theater`의 `enter` 메소드 안에서 `Audience`와 `TicketSeller`로부터 `Bag`과 `TicketOffice`를 가져와 관람객을 입장시키는 절차를 구현했다.

`Theater`외 모든 객체에서는 관람객 입장에 필요한 정보를 제공하고, 모든 처리는 `Theater`의 `enter` 메소드 안에 있었다.

이 관점에서 `Theater`의 `enter` 메소드는 **프로세스(Process)**이며, 그 외 객체는 **데이터(Data)**이다.

이처럼 프로세스와 데이터를 **별도의 모듈에 위치시키는 방식**을 절차지향적 프로그래밍이라고 부른다.

절차적 프로그래밍은 관람객과 판매원이 자신의 일을 스스로 수행할 것이라는 우리의 직관에 위배된다. 하지만 절차적 프로그래밍 세계에서는 관람객과 판매원이 수동적인 존재일 뿐이다(데이터).

때문에 코드를 읽는 사람과 원활하게 의사소통 할 수 없다.

또한, 절차적 프로그래밍은 변경하기 어려운 코드를 양산한다. ( 데이터는 프로세스에 의존하기 때문 )

그리고 우리는 이와 같은 절차적 프로그래밍 구현 방식을, 각 데이터와 프로세스가 동일한 모듈 내부에 위치하도록 함으로써, 객체지향 프로그래밍 구현 방식으로 바꿨다.

변경된 코드에서 `Theater`는 오직 `TicketSeller`에만 의존하며, 이러한 의존성들은 적절히 통제된다.

또한 객체 내부의 변경이 객체 외부에 파급되지 않도록 제어할 수 있기 때문에 변경이 수월하다.

설계를 어렵게 만드는 것은 **의존성**이다. 해결 방법은 불필요한 의존성을 제거함으로써 객체 사이의 **결합도**를 낮추는 것이다.

위 코드에서 결합도를 낮추기 위해 사용한 방법은 `Theater`가 몰라도 되는 세부사항을 `Audience`와 `TicketSeller` 내부로 감춰 **캡슐화** 한 것이다. 결과적으로 불필요한 세부사항을 객체 내부로 캡슐화 하는 것은 객체의 **자율성**을 높이고 **응집도** 높은 객체들의 공동체를 창조할 수 있도록 한다.

훌륭한 객체지향 설계라는 건, 불필요한 세부사항만을 캡슐화하는 자율적인 객체들이 낮은 결합도와 높은 응집도(높은 관련성을 가진 작업만 수행)를 가지고 협력하도록 최소한의 의존성만을 남기는 것이다.

> **더 개선할 수 있다**
> 

절차지향 방식으로 설계한 코드를 객체지향으로 바꿈으로써, 코드는 확실히 개선됐지만, 아직도 좋아질 여지가 있다.

`Audience` 클래스를 보자.

```java
public class Audience {
	private Bag bag;
	
	public Audience(Bag bag) {
		this.bag = bag;
	}
	
	public Long buy(Ticket ticket) {
		if (bag.hasInvitation()) {
			bag.setTicket(ticket);
			return 0L;
		} else {
			bag.setTicket(ticket);
			bag.minusAmount(ticket.getFee());
			return ticket.getFee();
		}
	}
}
```

`Audience`는 자율적인 존재다. 티켓 구매, 가방 내용물 관리를 직접 수행한다.

하지만 `Bag`은 자기 자신을 책임지지 못한다.

`Bag`을 자율적인 존재로 바꿔보자.

앞의 방법과 동일하게, `Bag` 내부 상태에 접근하는 모든 로직을 `Bag` 안으로 캡슐화해서 결합도를 낮추면 된다.

```java
public class Bag {
	private Long amount;
	private Ticket ticket;
	private Invitation invitation;
	
	public Long hold(Ticket ticket) {
		if (hasInvitation()) {
			setTicket(ticket);
			return 0L;
		} else {
			setTicket(ticket);
			minusAmount(ticket.getFee());
			return ticket.getFee();
		}
	}
	
	private void setTicket(Ticket ticket) {
		this.ticket = ticket;
	}
	
	private boolean hasInvitation() {
		return invitation != null;
	}
	
	private void minusAmount(Long amount) {
		this.amount -= amount;
	}
}

```

`Bag`의 구현을 캡슐화했으니, `Audience`를 `Bag`의 구현이 아닌 인터페이스에만 의존하도록 수정하자.

```java
public class Audience {
	public Long buy(Ticket ticket) {
		return bag.hold(ticket);
	}
}
```

`TicketSeller` 도 `TicketOffice`의 자율권을 침해한다.

`TicketSeller`가 멋대로 티켓을 팔고 티켓을 판매한 돈을 `TicketOffice`에 넣어버린다.

```java
public class TicketSeller {
	public void sellTo(Audience audience) {
		ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
	}
}
```

`TicketOffice`에 `sellTicketTo` 메소드를 추가하고 `TicketSeller`의 `sellTo`메소드의 내부 코드를 위 메소드로 옮기자.

```java
public class TicketOffice {
	public void sellTicketTo(Audience audience) {
		plusAmount(audience.buy(getTicket()));
	}
	
	private Ticket getTicket() {
		return tickets.remove(0);
	}
	
	private void plusAmount(Long amount) {
		this.amount += amount;
	}
}
```

이렇게 코드를 구현하면,

```java
public class TicketSeller {
	public void sellTo(Audience audience) {
		ticketOffice.sellTicketTo(audience);
	}
}
```

`TicketSeller` 는 `TicketOffice`의 `sellTicketTo`  메소드를 호출함으로써, `TicketOffice`의 구현이 아닌 인터페이스만 의존할 수 있게 되었다.

하지만 위와 같이 코드를 변경함으로써, `TicketOffice`와 `Audience` 사이에 의존성이 추가되었다.

`TicketOffice`의 자율성은 높였지만 전체 설계 관점에서는 결합도가 상승한 것이다.

어떤 경우에도 모든 사람들을 만족시킬 수 있는 설계를 만들 순 없다.

그러므로 설계는, 적절한 트레이드오프의 결과물이다.

> **실세계에서 우리의 직관과 충돌하는 무생물의 자율성**
> 

직관에 따르는 코드는 이해하기 쉬운 경향이 있다.

그래서 `Audience`와 `TicketSeller` 역시 스스로 자신을 책임지므로, 직관에 맞는 코드가 됐고, 결과적으로 더 이해하기 쉬워졌다.

하지만 우리는 가방, 극장같은 수동적인 무생물 역시 자기 자신을 책임지는 자율적인 존재로 취급하였다.

비록 현실에서는 수동적인 존재라고 하더라도, 객체지향의 세계에서는, 모든 것이 능동적이고 자율적인 존재로 바뀐다. 이러한 방식으로 설계하는 원칙을 가리켜 **의인화(anthropomorphism)**이라 부른다.

## 2장 _ 객체지향 프로그래밍

> **협력, 객체, 클래스**
> 

객체지향 프로그램을 작성할 때 가장 먼저 고려하는 것은 무엇인가?

아마도, 가장 먼저 어떤 클래스가 필요한지에 대해 고민할 것이다.

대부분의 사람들은 클래스를 결정한 후에 클래스에 어떤 속성과 메소드가 필요한지 고민한다.

하지만, 진정한 객체지향 패러다임으로의 전환은 클래스가 아닌 객체에 초점을 맞추어야만 한다.

그러기 위해서는, 다음 두 가지에 집중해야 한다.

1. 어떤 클래스가 필요한지를 고민하기 전에 어떤 객체들이 필요한지 고민해라.
    
    클래스는, 공통적인 상태와 행동을 공유하는 **객체들을 추상화**한 것이다. 따라서 클래스의 윤곽을 잡기 위해서는 어떤 객체들이 어떤 상태와 행동을 가지는지를 먼저 결정해야 한다.
    
    객체를 중심에 두는 접근 방법은 설계를 단순하고 깔끔하게 만든다.
    
2. 객체를 독립적인 존재가 아니라 기능을 구현하기 위해 협력하는 공동체의 일원으로 봐야 한다.
    
    객체는 홀로 존재하는 것이 아니다. 다른 객체에게 도움을 주거나 의존하면서 살아가는 협력적인 존재이다. 객체를 협력하는 공동체의 일원으로 바라보는 것은 설계를 유연하고 확장 가능하게 만든다.
    
    객체지향적으로 생각하고싶다면 객체를 고립된 존재로 바라보지 않고 협력에 참여하는 협력자로 바라봐야 한다.
    
    객체들의 모양과 윤곽이 잡히면 공통된 특성과 상태를 가진 객체들을 타입으로 분류하고, 이 타입을 기반으로 클래스를 구현해야 한다.
    

> **도메인의 구조를 따르는 프로그램 구조**
> 

소프트웨어는 사용자가 원하는 어떤 문제를 해결하기 위해 만들어진다.

예를 들어, 영화 예매 시스템이 있다고 할 때, 영화 예매 시스템의 목적은 영화를 좀 더 쉽고 빠르게 예매하려는 사용자의 문제를 해결하는 것이다.

이처럼 문제를 해결하기 위해 사용자가 프로그램을 사용하는 분야를 **도메인(domain)**이라 부른다.

---

온라인 영화 예매 시스템을 통해 이 장의 주제에 대해 접근해보자.

> **요구 조건**
> 
- 영화: 영화에 대한 기본 정보
- 상영: 관객들이 영화를 관람하는 사건

**요금 할인 조건 -**

- 할인 조건: 가격의 할인 여부
    - 순서 조건: 상영 순번을 이용하여 할인 여부를 결정하는 규칙
    - 기간 조건: 영화 상영 시작 시간을 이용하여 할인 여부를 결정하는 규칙
- 할인 정책
    - 금액 할인 정책: 예매 요금에서 일정 금액 할인
    - 비율 할인 정책: 예매 요금에서 일정 비율의 요금 할인

위와 같은 요구 조건으로, 객체지향적으로 온라인 영화 예매 시스템을 구현할 것이다.

또한 영화에는 할인 정책을 할당하지 않거나, 할당하더라도 오직 하나만 할당할 수 있고 할인 정책이 존재하는 경우에는 하나 이상의 할인 조건이 반드시 존재한다.

클래스의 이름은 대응되는 도메인 개념의 이름과 동일하거나 유사하게 지어져야 하기 때문에,

영화는 `Movie`, 상영은 `Screening` 클래스로 구현,

할인 정책은 `DiscountPolicy`, 금액 할인 정책은 `AmountDiscountPolicy` , 비율 할인 정책은 `PercentDiscountPolicy` 클래스로 구현하고, 할인 조건은 `DiscountCondition` , 순번 조건은 `SequenceCondition`, 기간 조건은 `PeriodCondition` 클래스로 구현한다. 예매라는 개념은 `Reservation` 클래스로 구현한다.

```java
public class Screening {
	private Movie movie;
	private int sequence;
	private LocalDateTime whenScreened;
	
	public Screening(Movie movie, int sequence, LocalDateTime whenScreened) {
		this.movie = movie;
		this.sequence = sequence;
		this.whenScreened = whenScreened;
	}
	
	public LocalDateTime getStartTime() {
		return whenScreened;
	}
	
	public boolean isSequence(int sequence) {
		return this.sequence == sequence;
	}
	
	public Money getMovieFee() {
		return movie.getFee();
	}
}
```

위 코드에서 주목할 점은, 인스턴스 변수는 `private`, 메소드는 `public`으로 구현되었다는 점이다.

위처럼 클래스의 내부와 외부를 구분해야 하는 이유는, 경계의 명확성이 객체의 자율성을 보장하기 때문이다.

객체는 상태와 행동을 함께 가지는 복합적인 존재이며, 스스로 판단하고 행동하는 자율적인 존재이다.

이에 쓰이는 데이터와 기능을 객체 내부로 함께 묶는 것을 캡슐화라고 부른다.

대부분의 OOP 언어들은 상태와 행동을 캡슐화하는 것에서 더 나아가 외부에서의 접근을 통제할 수 있는 접근 제어 메커니즘도 함께 제공한다. `public, protected, private`와 같은 접근 수정자가 그렇다.

캡슐화와 접근 제어는 객체를 두 부분으로 나눈다. 그 두 부분 중 하나는 외부에서 접근 가능한 부분으로, **퍼블릭 인터페이스**라고 부른다. 다른 하나는 외부에서 접근 불가능하고 오직 내부에서만 접근 가능한 부분으로, 이를 **구현**이라고 부른다.

이 **인터페이스와 구현의 분리** 원칙은 객체지향 프로그램을 만들기 위한 핵심 원칙이다.

일반적으로 객체의 상태(인스턴스 변수)는 숨기고 행동(메소드)는 외부에 공개한다.

> **협력하는 객체들의 공동체**
> 

`Screening`의 `reserve` 메소드는 영화를 예매 한 후 예매 정보를 담고 있는 `Reservation`의 인스턴스를 생성해서 반환한다.

인자인 `customer`는 예매자에 대한 정보를 담고 있고, `audienceCount`는 인원 수이다.

```java
public class Screening {
	public Reservation reserve(Customer customer, int audienceCount) {
		return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);
	}
}
```

`Screening`의 `reserve` 메소드를 보면 `calculateFee`라는 `private` 메소드를 호출해서 요금을 계산한 후, 그 결과를 `Reservation`의 생성자에 전달한다.

`calculateFee` 메소드는 요금을 계산하기 위해 다시 `Movie`의 `calculateMovieFee` 메소드를 호출한다.

`Movie`의 `calculateMovieFee` 메소드의 반환 값은 인당 예매 요금이다.

따라서 `Screening`은 전체 예매 요금을 구하기 위해 인당 예매 요금에서 인원 수를 곱한다.

```java
public class Screening {
	private Money calculateFee(int audienceCount) {
		return movie.calculateMovieFee(this).times(audienceCount);
	}
}
```

`Money`는 금액과 관련된 다양한 계산을 구현하는 클래스이다.

```java
public class Money {
	public static final Money ZERO = Money.wons(0);
	
	private final BigDecimal amount;
	
	public static Money wons(long amount) {
		return new Money(BigDecimal.valueOf(amount));
	}
	
	Money(BigDecimal amount) {
		this.amount = amount;
	}
	
	public Money plus(Money amount) {
		return new Money(this.amount.add(amount.amount));
	}
	
	public Money minus(Money amount) {
		return new Money(this.amount.subtract(amount.amount));
	}
	
	public Money times(double percent) {
		return new Money(this.amount.multiply(
			BigDecimal.valueOf(percent))
		);
	}
	
	public boolean isLessThan(Money other) {
		return amount.compareTo(other.amount) < 0;
	}
	
	public boolean isGreaterThanOrEqual(Money other) {
		return amount.compareTo(other.amount) >= 0;
```

1장 소극장 예제에서 금액을 구현하기 위해 `Long` 타입을 사용했는데, `Money` 타입처럼 저장하는 값이 금액과 관련되어 있다는 의미를 전달할 수는 없다.

또한, 금액과 관련된 로직이 서로 다른 곳에 중복되어 구현되는 것을 막을 수 없다.

객체지향의 장점은 객체를 이용해 도메인의 의미를 풍부하게 표현할 수 있다는 것이다.

**고로, 만약 한 개념이 하나의 인스턴스 변수만 포함하더라도 객체로 개념을 명시적으로 표현하는 것은 전체적인 설계의 명확성과 유연성을 높인다.**

`Reservation` 클래스는 고객, 상영 정보, 예매 요금, 인원 수를 속성으로 포함한다. 

```java
public class Reservation {
	private Customer customer;
	private Screening screening;
	private Money fee;
	private int audienceCount;
	
	public Reservation(Customer customer, Screening screening, Money fee, int audienceCount) {
		this.customer = customer;
		this.screening = screening;
		this.fee = fee;
	  this.audienceCount = audienceCount;
  }
}
```

영화를 예매하기 위해 `Screening, Movie, Reservation` 인스턴스들은 서로의 메서드를 호출하며 상호작용한다.

이처럼 시스템의 어떤 기능을 구현하기 위해 객체들 사이에 이뤄지는 상호작용을 **협력(Collaboration)**이라고 부른다.

객체지향 프로그램을 작성할 때는 먼저 협력의 관점에서 어떤 객체가 필요한지를 정하고, 객체들의 공통 상태와 행위를 구현하기 위해 클래스를 작성한다.

객체의 내부 상태는 외부에서 접근하지 못하도록 감추되, 외부에 공개하는 퍼블릭 인터페이스를 통하여 내부 상태에 접근할 수 있도록 한다. 객체가 다른 객체와 상호작용할 수 있는 유일한 방법은 메시지를 전송하는 것 뿐이다. 다른 객체에게 요청이 도착할 때 해당 객체가 메시지를 수신했다고 이야기한다.

메시지를 수신한 객체는 스스로의 결정에 따라 자율적으로 메시지를 처리할 방법을 결정한다.

이때 수신한 메시지를 처리하기 위한 자신만의 방법을 **메소드**라고 부른다.

**쉽게 말하면, 송신하는 객체가 수신하는 객체에게 메시지를 보내면, 송신하는 객체는 수신 객체가 그 메시지에 응답할 수 있다고 믿고 보내고, 수신하는 객체는 자율적인 방법(메소드)에 따라서 그 메시지에 응답한다.**

> **할인 요금 계산을 위한 협력 시작하기**
> 

```java
public class Movie {
	// ...
	// ...
	public money calculateMoneyFee(Screening screening) {
		return fee.minus(discountPolicy.calculateDiscountAmount(screening));
	}
}
```

다음 `calculatemovieFee` 메소드는 `discountPolicy` 에 `calculateDiscountAmount` 메시지를 전송해 할인 요금을 반환받는다. ( 이 과정에서 `Movie` 클래스는 `discountPolicy` 안에 `calculateDiscountAmount` 메소드가 있는지 없는지 모른다. 그냥 메시지를 보내고 응답을 기다릴 뿐이다 )

```java
public abstract class DiscountPolicy {
	// ...
	// ...
	public Money calculateDiscountAmount(Screening screening) {
			for(DiscountCondition each : conditions) {
				if (each.isSatisfiedBy(screening)) {
					return getDiscountAmount(screening);
				}
			}			
			return Money.ZERO;
	}
}
		
```

`DiscountPolicy`는 할인 여부와 요금 계산에 필요한 전체적인 흐름은 정의하지만, 실제로 요금을 계산하는 부분은 추상 메서드인 `getDiscountAmount`에게 위임한다.

실제로는 `DiscountPolicy`를 상속받은 자식 클래스에서 오버라이딩한 메소드가 실행될 것이다.

이처럼 부모 클래스에 기본적인 알고리즘의 흐름을 구현하고 중간에 필요한 처리를 자식 클래스에게 위임하는 디자인 패턴을 TEMPLATE METHOD 패턴이라고 부른다.

다음 두 가지 할인 조건은 각각 `SequenceCondition`과 `PeriodCondition` 이라는 클래스로 구현하고,

`DiscountPolicy`의 `getDiscountAmount` 메소드를 오버라이딩하는 `AmountDiscountPolicy` 클래스로 할인 정책을 구현한다.

그러면, `Movie`는 `DiscountPolicy`에 의존하고, `DiscountPolicy`는 `DiscountCondition`에 의존하고, 그 세부 내용인 `AmountDiscountPolicy`와 `PercentDiscountpolicy`는 `DiscountPolicy`에 의존하고, `SequenceCondition`과 `PeriodCondition`은 `DiscountCondition`에 의존한다.

`Movie`가 몰라도 되는 할인 금액, 할인 충족 조건을 각각 다른 객체에 숨겨 캡슐화하였다.

<aside>
💡

오버라이딩과 오버로딩

오버라이딩은 부모 클래스에 정의된 같은 이름, 같은 파라미터 목록을 가진 메소드를 자식 클래스에서 재정의 하는 경우를 가리킨다.

오버로딩은 메소드의 이름은 같지만 제공되는 파라미터의 목록이 다르다. 오버로딩한 메소드는 원래의 메소드를 가리지 않기 때문에 두 메소드는 공존한다. 

</aside>

> **코드 상의 의존성과 실행 가능 의존성**
> 

실제 위 코드에서 `Movie`는 `DiscountPolicy` 에만 의존하고 있으나, 실행 시점에는 `Movie`의 인스턴스가 `AmountDiscountPolicy`나 `PercentDiscountPolicy`에 의존하게 된다.

여기서 알 수 있는 점은, **코드 상의 의존성과 실행 시점의 의존성이 서로 다를 수 있다는 점이다.** 

중요한 사실은 코드의 의존성과 실행 시점의 의존성이 다르면 다를수록, 코드를 이해하기 어려워진다.  코드를 이해하려면 객체를 생성하고 연결하는 부분을 찾아야 하기 때문이다. 반면에, 코드의 의존성과 실행 시점의 의존성이 다르면 다를수록 코드는 더 유연해지고 확장 가능해진다. 자율성이 보장되기 때문이다.

이 부분에서도 또한 설계는 트레이드오프의 산물이라는 것을 잘 보여준다.

설계가 유연해질수록 코드를 이해하고 디버깅하기는 점점 더 어려워진다. 반면에 유연성을 억제하면 코드를 이해하고 디버깅하기는 쉬워지지만 재사용성과 확장 가능성은 낮아진다. 무조건 유연한 설계도, 무조건 이해하기 쉬운 코드도 정답이 아니다. 우리는 그 안에서 융통성을 찾아야 한다.

> **상속**
> 

상속은 두 클래스 사이의 관계를 정의하는 방법이다.

코드를 제공하는 클래스를 부모 클래스, 코드를 제공 받는 클래스를 자식 클래스라 부른다. 

상속이 가치 있는 이뉴는 부모 클래스가 제공하는 모든 인터페이스를 자식 클래스가 물려받을 수 있기 때문이다.

> **다형성**
> 

메시지와 메소드는 다른 개념이다.

`Movie` 는 `DiscountPolicy` 의 인스턴스 변수에게 `calculateDiscountAmount` 메시지를 전송한다. 이때 실행되는 메소드는 `Movie`와 상호작용하기 위해 연결된 객체의 클래스가 무엇인지에 따라 달라진다. 코드 상에서 `Movie` 클래스는 `DiscountPolicy` 클래스에게 메시지를 전송하지만 실행 시점에 실제로 실행되는 메소드는 `Movie`와 협력하는 객체의 실제 클래스가 무엇인지에 따라 달라진다. 이를 **다형성**이라 부른다.

(컴파일 시간 의존성 == 코드 상의 의존성인가?)

**다형성이란, 동일한 메시지를 수신했을 때 객체의 타입에 따라 다르게 응답할 수 있는 능력을 의미한다.** 

다형성은 실행될 메소드를 컴파일 시점이 아닌 **실행 시점**에 결정한다. 다시 말해 메시지와 메소드를 실행 시점에 바인딩한다는 것이다. 이를 지연 바인딩 혹은 동적 바인딩이라고 부른다.

그에 반해, 컴파일 시점에 실행될 함수를 결정하는 것을 초기 바인딩 혹은 정적 바인딩이라고 부른다.

객체지향은 지연 바인딩의 메커니즘을 사용하기 때문에, 하나의 메시지를 선택적으로 서로 다른 메소드에 연결할 수 있다. ( 다형성 )

<aside>
💡

정적 바인딩에 관한 간단한 설명

정적 바인딩은 컴파일 시점에 메소드나 변수와의 연결이 결정되는 경우이다.

컴파일러가 함수나 메소드의 정확한 호출 대상을 알 수 있기에, 빠르고 효율적이다.

대표적인 예로는 함수 오버로딩이 있다.

```java
class Animal {
public:
	void sound() {
		cout << "Animal sound" << endl;
	}
};

int main() {
	Animal animal;
	animal.sound(); // 컴파일 시점에 Animal의 sound와 연결됨
	return 0;
}
```

</aside>

<aside>
💡

동적 바인딩에 관한 간단할 설명

동적 바인딩은 실행 시점에 메소드나 변수와의 연결이 결정된다.

다형성을 가능하게 하는 핵심 메커니즘 중 하나이다.

```java
int main() {
    Animal* animal;

    Dog dog;
    Cat cat;

    animal = &dog; // 런타임 시점에 어떤 함수가 실행될지 결정
    animal->sound();  // "Dog barks" 출력: 동적 바인딩

    animal = &cat;
    animal->sound();  // "Cat meows" 출력: 동적 바인딩

    return 0;
}

```

</aside>

> **추상화와 유연성**
> 

객체지향에서 추상적이라는 것은, 어떤 일을 어떻게 하는지 구체적인 내용을 감추고, 그 일을 해야 한다는 사실만을 명확하게 정의하는 것이다. ( 고로 인터페이스는 모든 메서드가 추상적이다 )

`DiscountPolicy` 와 `DiscountCondition`의 자식 클래스를 생략하여 코드 구조를 나타내보자.

`Movie` → `DiscountPolicy` → `DiscountCondition`의 꼴이 된다.

여기서 알 수 있는 추상화의 장점은, 추상화의 계층만 따로 놓고 보면 요구사항의 정책을 높은 수준에서 서술할 수 있다는 것이다. 또한 추상화를 이용하면 설계가 좀 더 유연해진다.

요구사항의 정책을 높은 수준에서 표현할 수 있다는 말이 무슨 말일까?

위의 계층을 하나의 문장으로 정리하면, 

“영화 예매 요금은 최대 하나의 할인 정책과 다수의 할인 조건을 이용해 계산할 수 있다.”

로 표현할 수 있다.

이 문장은,

“영화의 예매 요금은 ‘금액 할인 정책’과 ‘두 개의 순서 조건, 한 개의 기간 조건’을 이용해서 계산할 수 있다” 라는 문장을 포괄한다.

이처럼 추상화를 사용하면 세부적인 내용을 무시한 채 상위 정책을 쉽고 간단하게 표현할 수 있다.

추상화를 이용해 상위 정책을 표현하면 기존의 구조를 수정하지 않고도 새로운 기능을 쉽게 추가하고 확장할 수 있기에, 설계를 유연하게 만든다.

> **상속의 문제점**
> 

상속은 객체지향에서 코드를 재사용하기 위해 널리 사용되는 기법이지만, 두 가지 문제점이 존재한다. 그 중 하나는 캡슐화를 위반한다는 것이고, 나머지 하나는 설계를 유연하지 못하게 만든다는 것이다.

상속을 이용하기 위해서는 부모 클래스의 내부 구조를 잘 알고 있어야 한다. `AmountDiscountMovie`와 `PercentDiscountMovie`를 구현하는 개발자는 부모 클래스인 `Movie`의 `calculateMovieFee` 메소드 안에서 추상 메소드인 `getDiscountAmount` 메소드를 호출한다는 사실을 알고 있어야 한다. 

결과적으로, **상속을 이용하면 부모 클래스의 구현이 자식 클래스에게 노출되기 때문에 캡슐화가 약화된다.**

자식 클래스와 부모 클래스의 결합성이 높아지기 때문에, 부모 클래스를 변경할 때 자식 클래스도 함께 변경될 확률을 높인다.

> **합성**
> 

실행 시점에 금액 할인 정책인 영화를 비율 할인 정책으로 변경한다고 가정하자.

`Movie`에 `DiscountPolicy`를 변경할 수 있는 `changeDiscountPolicy` 메소드를 추가하자.

```java
public class Movie {
	private DiscountPolicy discountPolicy;
	
	public void changeDiscountPolicy(DiscountPolicy discountPolicy) {
		this.discountPolicy = discountPolicy;
	}
}
```

그럼 영화의 금액 할인 정책을 금액 할인에서 비율 할인으로 바꾸고자 할 때,

```java
Movie avartar = new Movie("아바타",
													Duration.ofMinutes(120),
													Money.wons(10000),
													new AmountDiscountPolicy(Money.wons(800), ...));
avatar.changeDiscountPolicy(new PercentDiscountPolicy(0.1, ...));							
```

위 예제에서, `Movie`는 요금을 계산하기 위해 `DiscountPolicy`의 코드를 재사용한다.

이 방법이 상속과 다른 점은 상속이 부모 클래스의 코드와 자식 클래스의 코드를 컴파일 시점에 하나의 단위로 강하게 결합하는 데 비해, `Movie`는 `DiscountPolicy`가 외부에 `calculateDiscountAmount` 메소드를 제공한다는 사실만 알고 내부 구현에 대해서는 전혀 알지 못한다.

이처럼 인터페이스에 정의된 메시지를 통해서만 코드를 재사용하는 방법을 **합성**이라 한다.

합성은 인터페이스에 정의된 메시지를 통해서만 재사용이 가능하기 때문에 구현을 효과적으로 캡슐화 할 수 있다. 상속은 클래스를 통해 강하게 결합되는 데 비해 합성은 메시지를 통해 느슨하게 결합된다. 따라서, 코드 재사용을 위해서는 상속보다는 합성을 선호하는 것이 더 좋은 방법이다.

> **그럼 설계 과정에서 합성만 사용하면 되는거야?**
> 

합성이 상속보다 좋은 설계 방법이라고 해서 상속을 절대 사용하지 말라는 건 아니다.

대부분의 설계에서는 상속과 합성을 같이 사용한다.

코드를 재사용하는 경우에는 상속보다 합성을 선호하는 것이 옳지만, 다형성을 위해 인터페이스를 재사용하는 경우에는 상속과 합성을 함께 조합해서 사용할 수밖에 없다.

<aside>
💡

상속의 예

상속이란, 어떤 클래스가 다른 클래스로부터 물려받아 기능을 확장하는 방식이다.

자식 클래스는 부모 클래스의 속성을 물려받아 그대로 사용하거나, 필요에 따라 재정의(오버라이딩) 할 수 있다.

```java
#include <iostream>
using namespace std;

// 부모 클래스
class Animal {
public:
    void eat() {
        cout << "Animal is eating" << endl;
    }
};

// 자식 클래스: Animal로부터 상속
class Dog : public Animal {
public:
    void bark() {
        cout << "Dog is barking" << endl;
    }
};

int main() {
    Dog myDog;
    myDog.eat();   // 상속받은 Animal의 기능 사용
    myDog.bark();  // Dog의 고유 기능
    return 0;
}

```

</aside>

<aside>
💡

합성의 예

합성이란, 객체가 다른 객체를 포함하는 방식이다.

한 클래스가 다른 클래스의 객체를 멤버로 가지고, 그 객체를 통해 동작이나 기능을 수행하는 방식이다.

```java
#include <iostream>
using namespace std;

// 다른 클래스
class Engine {
public:
    void start() {
        cout << "Engine is starting" << endl;
    }
};

// 합성을 이용한 Car 클래스
class Car {
private:
    Engine engine;  // Car는 Engine을 가지고 있음 (합성)
public:
    void drive() {
        engine.start();  // 엔진의 기능을 이용함
        cout << "Car is driving" << endl;
    }
};

int main() {
    Car myCar;
    myCar.drive();  // 엔진을 통해 차가 운전함
    return 0;
}

```

</aside>

위 코드처럼, 상속은 부모 클래스의 내부 구현이 자식 클래스에 그대로 노출되는 반면,

합성은 포함된 객체의 내부 구현이 외부에 감춰진다. (캡슐화)

## 3장 _ 역할, 책임, 협력

2장의 영화 예매 시스템에서, 다양한 객체들이 영화 예매라는 기능을 구현하기 위해 메시지를 주고받으면서 상호작용했다. 이처럼 객체들이 애플리케이션의 기능을 구현하기 위해 수행하는 상호작용을 협력이라고 한다.

객체가 협력에 참여하기 위해 수행하는 로직은 책임이라 한다.

객체들이 협력 안에서 수행하는 책임들이 모여 객체가 수행하는 역할을 구성한다.

> **협력**
> 

협력이란, 어떤 객체가 다른 객체에게 무엇인가를 요청하는 것이다. 한 객체는 어떤 것이 필요할 때 다른 객체에게 전적으로 위임하거나 서로 협력한다. 즉, 두 객체가 상호작용을 통해 더 큰 책임을 수행하는 것이다. 객체 사이의 협력을 설계할 때에는 객체를 서로 분리된 인스턴스가 아닌, 협력하는 파트너로 인식해야 한다.

2장의 영화 예매 시스템에서, `Screening`이 `Movie`에게 처리를 위임하는 이유는 요금을 계산하는 데 필요한 기본 요금과 할인 정책을 가장 잘 알고 있는 객체가 `Movie`였기 때문이다.

요금을 계산하는 작업을 `Screening`이 수행했다면 `Movie`의 인스턴스 변수인 `fee`와 `discountPolicy`에 직접 접근해야 했을 것이므로, 캡슐화를 위반했을 것이다.

자신이 할 수 없는 일을 다른 객체에게 위임하면 협력에 참여하는 객체들의 전체적인 자율성을 향상시킬 수 있다.

정리하면, 자율적인 객체는 자신에게 할당된 책임을 수행하던 중에 필요한 정보를 알지 못하거나 외부의 도움이 필요한 경우 적절한 객체에게 메시지를 전송해서 협력을 요청한다. 메시지를 수신한 객체 역시 메시지를 처리하던 중에 직접 처리할 수 없는 정보나 행동이 필요한 경우 또 다른 객체에게 협력을 요청한다.

> **협력이 설계를 위한 문맥을 결정한다**
> 

애플리케이션 안에 어떤 객체가 필요하다면, 그 이유는 그 객체가 어떤 협력에 참여하고 있기 때문이다. 그리고 그 객체가 어떤 협력에 참여할 수 있는 이유는, 협력에 필요한 적절한 행동을 보유하고 있기 때문이다.

고로, 객체의 행동을 결정하는 것은 객체가 참여하고 있는 협력이다. 협력은 객체가 필요한 이유와 객체가 수행하는 행동의 동기를 제공한다.

객체의 행동을 결정하는 것이 협력이라면, 객체의 상태를 결정하는 것은 행동이다. 객체의 상태는 그 객체가 행동을 수행하는 데 필요한 정보가 무엇인지로 결정된다.

객체는 자신의 상태를 스스로 결정하고 관리하는 자율적인 존재이기 때문에, 객체가 수행하는 행동에 필요한 상태도 함께 가지고 있어야 한다.

상태는 객체가 행동하는 데 필요한 정보에 의해 결정되고, 행동은 협력 안에서 객체가 처리할 메시지로 결정된다. ( 협력 → 행동 → 상태 )

> **책임**
> 

객체를 설계하기 위해 필요한 문맥인 협력이 갖춰졌다면? 협력에 필요한 행동을 수행할 수 있는 적절한 객체를 찾아야 한다.

책임이란 객체가 **유지해야 하는 정보(아는 것)와 수행할 수 있는 행동(하는 것)**에 대해 개략적으로 서술한 문장이다.

영화 예매 시스템에서 `Screening`의 책임은? 영화를 예매하는 것이다. 그러기 위해선 상영할 영화를 알고 있어야 한다. 이는 유지해야 하는 정보(아는 것)과 관련된 책임이다.

`Movie`의 책임은? 요금을 계산하는 것이다. 이를 수행하기 위해선 예매 가격을 계산해야 한다. 이것은 수행할 수 있는 행동(하는 것)과 관련된 책임이다.

중요한 사실은, 아는 것과 하는 것이 밀접하게 연관돼 있다는 점이다. 객체는 자신이 맡은 책임을 수행하는 데 필요한 정보를 알 책임이 있다. 또한, 그 객체는 자신이 할 수 없는 작업을 도와줄 객체를 알 책임이 있다. **어떤 책임을 수행하기 위해서는 그 책임을 수행하는데 필요한 정보도 함께 알아야 할 책임이 있는 것이다.**

객체지향 설계에서 가장 중요한 것은 책임이다. 객체의 구현 방법은 상대적으로 책임보다는 덜 중요하며 책임을 결정한 다음에 고민해도 늦지 않다.

> **책임 할당**
> 

자율적인 객체를 만드는 가장 기본적인 방법은 책임을 수행하는 데 필요한 정보를 가장 잘 알고 있는 전문가에게 그 책임을 할당하는 것이다.

이를 책임 할당을 위한 INFORMATION EXPERT(정보 전문가) 패턴이라 부른다.

객체에게 책임을 할당하기 위해서는 협력이라는 문맥을 정의해야 한다.

영화 예매 시스템을 예로 들어 정보 전문가에게 책임을 할당하는 법을 알아보자.

객체가 책임을 수행하게 하는 유일한 방법은 메시지를 전송하는 것이므로, 예매하라 라는 이름의 메시지로 협력을 시작하자.

`1: 예매하라 ->` 

이 메시지를 수행하기 위한 최적의 정보 전문가는 누구인가? 상영 시간과 기본 요금을 알고 있는 `Screening`이 될 것이다.

`1: 예매하라 -> screening` 

또한, `Screening`은 예매 가격을 계산하는 건 정보가 없기 때문에, 외부의 객체에게 가격 계산을 요청해야 한다.

`1: 예매하라 -> screening 2: 가격을 계산하라 ->`

또 가격을 계산하는 데 있어 최적의 정보 전문가는? 가격과 할인 정책을 알고 있는 `Movie`이다.

`1: 예매하라 -> screening 2: 가격을 계산하라 -> movie` 

또 가격을 계산하기 위해서는 할인 요금이 필요하지만, `Movie`는 그에 맞는 정보 전문가가 아니기에, 또 다른 외부 객체와 협력해야 한다.

이처럼 객체지향 설계는 협력에 필요한 메시지를 찾고 메시지에 적절한 객체를 선택하는 반복적인 과정을 통하여 이뤄진다.

> **책임 주도 설계**
> 

지금까지 살펴본 내용은, 협력을 설계하기 위해서 책임에 초점을 맞춰야 한다는 내용이었다.

이처럼 책임을 찾고, 그를 수행하기에 적격인 객체를 찾아 책임을 할당하는 방식으로 협력을 설계하는 방법을 **책임 주도 설계(Responsibility-Driven Design, RDD)** 라 부른다.

책임 주도 설계의 과정은 다음과 같다.

- 시스템이 사용자에게 제공해야 하는 기능인 시스템 책임을 파악한다.
- 시스템 책임을 더 작은 책임으로 분할한다.
- 분할된 책임을 수행할 수 있는 적절한 객체 또는 역할을 찾아 책임을 할당한다.
- 객체가 책임을 수행하는 도중 다른 객체의 도움이 필요한 경우 이를 책임질 적절한 객체 또는 역할을 찾는다.
- 해당 객체 또는 역할에게 책임을 할당함으로써 두 객체가 협력하게 한다.

> **객체가 메시지를 선택하는 것이 아니라, 메시지가 객체를 선택해야 한다**
> 

객체가 메시지를 선택하는 것이 아니라, 메시지가 객체를 선택해야 한다.

그에는 두 가지 이유가 있다.

1. 객체가 최소한의 인터페이스를 가질 수 있게 된다.
    
    이렇게 하면, 필요한 메시지가 식별될 때까지 객체의 인터페이스에 어떤 것도 추가하지 않기 때문에, 필요한 크기의 인터페이스를 가질 수 있다.
    
2. 객체가 추상적인 인터페이스를 가질 수 있게 된다.
    
    객체의 인터페이스는 무엇을 하는지는 표현해야 하지만, 어떻게 수행하는지를 노출해서는 안 된다. 메시지는 외부의 객체가 요청하는 무언가를 의미하기 때문에 메시지를 먼저 식별하면 무엇을 수행할지에 초점을 맞추는 인터페이스를 얻을 수 있다.
    

> **행동이 상태를 결정한다**
> 

객체의 행동은 객체가 협력에 참여할 수 있는 유일한 방법이다. 객체가 협력에 적합한지를 결정하는 것은 객체의 상태가 아니라 행동이다.

초보자들은 먼저 객체에 필요한 상태가 무엇인지를 결정하고, 그 후에 상태에 필요한 행동을 결정한다(이게 나다…).

이런 방식은 객체의 내부 구현이 객체의 퍼블릭 인터페이스에 노출되도록 만들기 때문에 캡슐화를 저해한다.

중요한 것은 객체의 상태가 아니라 행동이다.

객체가 가질 수 있는 상태는 행동을 결정하고 나서야 결정할 수 있다.

협력이 객체의 행동을 결정하고, 객체의 행동이 상태를 결정한다. 

그러고나서 그 행동이 바로 그 객체의 책임이 된다.

> **역할**
> 

객체가 어떤 특정한 협력 안에서 수행하는 책임의 집합을 역할이라고 부른다.

위에서 언급한 영화 예매 협력에서 `예매하라`라는 메시지를 처리하기에 적합한 객체로 `Screening`을 선택했는데, 하나의 단계처럼 보이는 이 설계 단계는 사실 두 단계가 합쳐진 것이다.

첫 번째 단계는 `예매하라` 메시지를 수행하기에 가장 적격인 역할을 찾는 것이고, 두 번째 단계는 역할을 수행한 객체로 `Screening` 인스턴스를 선택하는 것이다.

> **그래서 역할이 왜 필요한 건데? - 역할은 다른 것으로 교체할 수 있는 책임의 집합이다**
> 

아까 영화 예매 시스템을 RDD 방법으로 설계할 때,

`1: 예매하라 -> screening 2: 가격을 계산하라 -> movie -> 3: 할인 요금을 계산하라`

의 형식으로 설계했다.

할인 정책에는 금액 할인 정책과, 비율 할인 정책이 있었다.

만약 역할이라는 개념을 고려하지 않고, 객체에게 책임을 할당한다고 가정해보면?

`... -> 3: 할인 요금을 계산하라 -> AmountDiscountPolicy`

`... -> 3: 할인 요금을 계산하라 -> PercentDiscountPolicy`

이러한 방법으로 두 협력을 구현하면 대부분의 코드가 중복될 것이다.

따라서, 문제를 해결하기 위해서는 객체가 아닌 책임에 초점을 맞추어야 한다.

책임의 관점에서 두 협력을 바라보면 `AmountDiscountPolicy`와 `PercentDiscountPolicy` 모두 할인 요금 계산이라는 동일한 책임을 수행한다.

따라서, ‘객체라는 존재를 지우고 할인 요금을 계산하라’ 라는 메시지에 응답할 수 있는 대표자를 생각한다면 두 협력을 하나로 통합할 수 있을 것이다.

이 대표자를 협력 안에서 두 종류의 객체를 교대로 바꿔 끼울 수 있는 일종의 슬롯으로 생각할 수 있으며, 이 슬롯이 바로 역할이다.

요점은, 동일한 책임을 수행하는 역할을 기반으로 두 개의 협력을 하나로 통합할 수 있다는 것이다. 그럼으로써 중복되는 코드를 제거할 수 있다.

> **한 종류의 객체만 협력에 참여할 때: 객체 vs 역할?**
> 

역할은 객체가 참여할 수 있는 교체될 수 있는 슬롯이다.

그런데, 오직 한 종류의 객체만 협력에 참여하는 상황이라면?

레베카 워프스브록에 따르면, 협력에 참여하는 후보가 여러 객체에 의해 수행될 필요가 있다면 그 후보는 역할이 되지만 단지 한 종류의 객체만이 협력에 참여해야 한다면, 그 후보는 객체가 된다.

협력에 적합한 책임을 수행하는 대상이 하나임 → 객체

협력에 적합한 책임을 수행하는 대상이 여럿일 수 있음 → 역할

> **역할과 추상화의 관련성**
> 

역할은 공통의 책임을 바탕으로 객체의 종류를 숨기기 때문에 이런 관점에서 역할을 객체의 추상화로 볼 수 있다. 고로 추상화가 가지는 장점은 역할에도 동일하게 적용된다.

> **객체는 배우와 배역이다**
> 

객체는 다양한 역할을 가질 수 있다. 객체는 협력에 참여할 때 협력 안에서 하나의 역할로 보여진다. 동일한 객체라 하더라도, 객체가 참여하는 협력에 따라 객체의 역할은 계속해서 바뀐다. 객체는 다수의 역할을 보유할 수 있지만 객체가 참여하는 특정 협력은 객체의 한 가지 역할만 바라볼 수 있다.

이는, 배우가 하나의 연극에서 오직 하나의 배역을 연기하는 것과 동일하다. 한 배우는 어떤 드라마에서는 악역을 연기하고, 어떤 드라마에서는 선한 아버지를 연기할 수 있다.

객체에서도 마찬가지로, 객체가 다른 협력에 참여할 때는 이전의 역할은 잊혀지고 해당 협력에서 바라보는 역할의 측면에서 보여질 것이다.