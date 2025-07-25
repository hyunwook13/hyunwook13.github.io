---
layout: post
title: 26장 - 메인 컴포넌트
tags: [clean-architecture]
date: 2025-07-29 10:00:00 +0900
---

- 모든 시스템에는 최소 하나의 컴포넌트가 존재
    - 이 컴포넌트가 다른 컴포넌트들을 생성, 조정, 관리
    - 저자는 이것을 Main 컴포넌트라고 함

### 궁극적인 세부사항

- 메인 컴포넌트는 궁극적인 세부사항, 가장 낮은 수준의 정책, 메인 시스템의 초기진입점
- 운영체제를 제외한 어떤 것도 의존하지 않음
- 메인은 모든 팩토리와 전략, 그리고 시스템 전반을 담당하는 나머지 기반 설비를 생성한 후, 시스템에서 더 높은 수준을 담당하는 부분으로 제어권을 넘기는 역할을 맡음

→ 의존성 주입 프레임워크를 이용해 의존성을 주입하는 일은 바로 이 메인 컴포넌트에서 이뤄져야 한다. 

- 메인에 의존성이 일단 주입되고 나면, 메인은 의존성 주입 프레임워크를 사용하지 않고도 일반적인 방식으로 의존성을 분배할 수 있어야 한다.
    - 메인을 지저분한 컴포넌트 중에서도 가장 지저분한 컴포넌트라고 생각해보자

```java
public class Main implements HtwMessageReceiver {
	private static HuntTheWumpus game;
	private static int hitPoints = 10;
	private static final List<String> caverns = new ArrayList<>(;
	private static final String[] environments = new String[] { 
		"bright"
		"humid",
		"dry",
		"creepy",
		"ugly",
		"foggy",
		"hot",
		"cold",
		"drafty",
		"dreadful"
	};
	
	private static final String[] shapes = new String[] {
		"round",
		"square",
		"oval",
		"irregular",
		"long"
		"craggy",
		"rough",
		"tall",
		"narrow"
	};
	
	private static final String[] cavernTypes = new String[] {
		"cavern",
		"room"
		"chamber",
		"catacomb",
		"crevasse",
		"cell",
		"tunnel",
		"passageway",
		"hall",
		"expanse"
	};
	
	private static final Stringll adornments = new String[] {
		"smelling of sulfur",
		"with engravings on the walls",
		"with a bumpy floor"
		"",
		"littered with garbage",
		"spattered with guano",
		"with piles of Wumpus droppings",
		"with bones scattered around",
		"with a corpse on the floor",
		"that seems to vibrate",
		"that feels stuffy",
		"that fills you with dread"
	}
};
```

- 아래의 코드를 확인해보면 모든 경우에 대해 팩토리 메소드를 만들어두었음

```java
public static void main(String[] args) throws IOException {
	game = HtwFactory-makeGame ("htw. game. Hunt TheWumpusFacade", new Main());
	createMap();
	BufferedReader br = new BufferedReader (new InputStreamReader (System. in));
	game.makeRestCommand().execute();
	while (true) {
		System.out.println(game.getPlayerCavern();
		System.out.println("Health: " + hitPoints + " arrows: " + game.getQuiver());
		HuntTheWumpus.Command c = game.makeRestCommand () ;
		System.out.println("›>");
		String command = br. readLine();
		if (command.equalsIgnoreCase("e"))
			c = game.makeMoveCommand (EAST) ;
		else if (command. equals IgnoreCase ("w") )
			c = game.makeMoveCommand (WEST) ;
		else if (command. equalsIgnoreCase ("n" ))
			c = game.makeMoveCommand (NORTH) ;
		else if (command. equalsIgnoreCase("s") )
			c = game. makeMoveCommand (SOUTH) ;
		else if (command. equals IgnoreCase("r"))
			c = game.makeRestCommand () ;
		else if (command. equalsIgnoreCase("sw"))
			c = game.makeShootCommand (WEST) ;
		else if (command. equals IgnoreCase("se"))
			c = game.makeShootCommand (EAST);
		else if (command. equalsIgnoreCase("sn"))
			c = game. makeShootCommand (NORTH) ;
		else if (command. equalsIgnoreCase("ss" ))
			c = game. makeShootCommand (SOUTH) ;
		else if (command. equalsIgnoreCase("q"))
			return;
		c. execute();
	}
}
```

- main 함수에서 주목할 점이 더 있다. 바로 입력 스트림 생성 부분, 게임의 메 인 루프 처리, 간단한 입력 명령어 해석 등은 main 함수에서 모두 처리하지만, 명령어를 실제로 처리하는 일은 다른 고수준 컴포넌트로 위임한다는 사실이다.
- 마지막으로 지도 생성 역시 main에서 처리한다는 점도 주목하자.

```java
private static void createMap() {
	int Caverns = (int) (Math. random () * 30.0 + 10.0);
	while (nCaverns— > 0)
		caverns. add (makeName()) ;
		for (String cavern : caverns) ‹
			maybeConnectCavern(cavern, NORTH);
			maybeConnectCavern (cavern, SOUTH);
			maybeConnectCavern(cavern, EAST);
			maybeConnectCavern(cavern, WEST);
		}
	String playerCavern = anyCavern();
	game. setPlayerCavern (playerCavern);
	game. setWumpusCavern(anyOther (playerCavern));
	game.addBatCavern(anyOther (playerCavern));
	game. addBatCavern(anyOther (playerCavern) );
	game.addBatCavern (anyOther(playerCavern)) ;
	game.addPitCavern (anyOther (playerCavern));
	game.addPitCavern(any0ther(playerCavern) );
	game.addPitCavern(any0ther(playerCavern) );
	game.setQuiver (5) ;
	}
// 이하 코드 생략
}
```

- 요지는 메인은 클린 아키텍처에서 가장 바깥 원에 위치하는, 지저분한 저수준 모듈이라는 점이다.
- 메인은 고수준의 시스템을 위한 모든 것을 로드한 후, 제어권을 고수준의 시스템에게 넘긴다.

---

### 정리

> “Main은 시스템의 모든 걸 조립하고, 고수준 로직에게 제어권을 넘기는 지저분한 조립자다.”
> 
> 그래서 외부 기술에 대한 의존성, 초기화, 입출력은 여기서만 하고, 나머지는 깔끔한 도메인 객체들이 책임져야 한다는 거야.
>