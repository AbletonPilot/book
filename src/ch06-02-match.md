<!-- Old headings. Do not remove or links may break. -->

<a id="the-match-control-flow-operator"></a>

## `match` 흐름 제어 구조

러스트에는 값을 일련의 패턴과 비교한 뒤 어느 패턴에 매칭되는지에 따라 코드를
실행하게 해 주는 `match`라는 아주 강력한 흐름 제어 구조가 있습니다. 패턴은 리터럴
값, 변수 이름, 와일드카드, 그 외 많은 것들로 구성될 수 있습니다. [19장][ch19-00-patterns]
<!-- ignore -->에서는 다양한 종류의 패턴과 그 기능을 모두 다룹니다. `match`의 힘은
패턴의 표현력과, 컴파일러가 모든 가능한 경우가 처리되었음을 확인해 준다는 사실
에서 나옵니다.

`match` 표현식을 동전 분류 기계처럼 생각해 보세요. 동전이 다양한 크기의 구멍이
나 있는 트랙을 따라 미끄러져 내려오고, 각 동전은 자신이 들어맞는 첫 번째 구멍에
빠집니다. 같은 방식으로 값이 `match`의 각 패턴을 거치면서, 값이 “들어맞는” 첫
패턴에서 그 값이 연관된 코드 블록으로 떨어져 실행 중 사용됩니다.

동전 이야기가 나온 김에, 동전을 예로 들어 `match`를 사용해 봅시다! Listing 6-3에
보이듯, 알 수 없는 미국 동전을 받아서 계수 기계와 비슷한 방식으로 어느 동전인지
판별하고 그 값을 센트 단위로 반환하는 함수를 작성할 수 있습니다.

<Listing number="6-3" caption="열거형과, 그 열거형의 배리언트를 패턴으로 가지는 `match` 표현식">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

`value_in_cents` 함수의 `match`를 뜯어봅시다. 먼저 `match` 키워드와 그 뒤에 표현식
이 옵니다. 여기서는 그 표현식이 `coin` 값입니다. 이것은 `if`와 함께 쓰는 조건
표현식과 아주 비슷해 보이지만 큰 차이가 있습니다. `if`에서는 조건이 불리언 값으로
평가되어야 하지만, 여기서는 어떤 타입이든 될 수 있습니다. 이 예에서 `coin`의
타입은 첫 줄에서 정의한 `Coin` 열거형입니다.

그다음은 `match`의 갈래(arm)들입니다. 갈래는 두 부분으로 이뤄집니다. 패턴과 일부
코드입니다. 여기서 첫 번째 갈래의 패턴은 값 `Coin::Penny`이고, 그 뒤의 `=>`
연산자는 패턴과 실행할 코드를 구분합니다. 이 경우 코드는 단순히 값 `1`입니다. 각
갈래는 쉼표로 다음 갈래와 구분됩니다.

`match` 표현식이 실행되면, 결과 값을 순서대로 각 갈래의 패턴과 비교합니다. 어떤
패턴이 값과 매칭되면, 그 패턴과 연관된 코드가 실행됩니다. 그 패턴이 값과 매칭되지
않으면, 실행은 다음 갈래로 계속됩니다. 동전 분류 기계에서와 비슷합니다. 필요한
만큼 많은 갈래를 둘 수 있습니다. Listing 6-3의 `match`는 네 개의 갈래를 가집니다.

각 갈래와 연관된 코드는 표현식이고, 매칭되는 갈래에 있는 표현식의 결과 값이
`match` 표현식 전체가 반환하는 값이 됩니다.

Listing 6-3처럼 각 갈래가 단지 값 하나를 반환하는 것처럼 갈래 코드가 짧다면, 보통
중괄호를 쓰지 않습니다. `match` 갈래에서 여러 줄의 코드를 실행하고 싶다면 반드시
중괄호를 사용해야 하고, 그 경우 갈래 뒤의 쉼표는 선택 사항이 됩니다. 예를 들어,
다음 코드는 메서드가 `Coin::Penny`로 호출될 때마다 “Lucky penny!”를 출력하지만,
여전히 블록의 마지막 값인 `1`을 반환합니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### 값에 바인딩되는 패턴

`match` 갈래의 또 다른 유용한 기능은, 패턴에 매칭되는 값들의 일부에 바인딩할 수
있다는 점입니다. 이것이 열거형 배리언트에서 값을 뽑아낼 수 있는 방법입니다.

예시로, 열거형 배리언트 중 하나가 안에 데이터를 담도록 바꿔 봅시다. 1999년부터
2008년까지 미국은 50개 주 각각에 대한 서로 다른 디자인의 쿼터(quarter)를 한 면에
주조했습니다. 다른 동전은 주 디자인이 들어가지 않았으니, 쿼터만 이런 추가 값을
가집니다. 이 정보를 `enum`에 추가하려면 `Quarter` 배리언트가 안에 `UsState` 값을
저장하도록 바꾸면 됩니다. Listing 6-4에서 이를 수행했습니다.

<Listing number="6-4" caption="`Quarter` 배리언트가 `UsState` 값도 담는 `Coin` 열거형">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

친구가 50개 주의 쿼터를 모두 모으려 하고 있다고 상상해 봅시다. 동전 종류별로 잔돈을
정리하면서, 각 쿼터에 연관된 주 이름도 외쳐서 친구가 아직 갖고 있지 않은 주라면
자신의 컬렉션에 추가할 수 있도록 해 주려고 합니다.

이 코드의 match 표현식에서는 `Coin::Quarter` 배리언트의 값에 매칭되는 패턴에 `state`
라는 변수를 추가합니다. `Coin::Quarter`가 매칭되면, `state` 변수는 그 쿼터의 주
값에 바인딩됩니다. 그런 다음 다음과 같이 그 갈래의 코드에서 `state`를 사용할 수
있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

`value_in_cents(Coin::Quarter(UsState::Alaska))`를 호출한다면, `coin`은
`Coin::Quarter(UsState::Alaska)`가 될 것입니다. 이 값을 각 match 갈래와 비교해
나가면, `Coin::Quarter(state)`에 도달하기 전까지는 어느 것에도 매칭되지 않습니다.
그 시점에 `state`의 바인딩은 값 `UsState::Alaska`가 됩니다. 그런 다음 `println!`
표현식에서 그 바인딩을 사용할 수 있고, 이로써 `Quarter`에 해당하는 `Coin` 열거형
배리언트 안에서 내부 주 값을 꺼내게 됩니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="matching-with-optiont"></a>

### `Option<T>`와 `match` 패턴

앞 절에서는 `Option<T>`를 사용할 때 `Some` 경우에서 내부의 `T` 값을 꺼내고 싶었
습니다. `Coin` 열거형에서 했던 것처럼, `Option<T>` 또한 `match`를 사용해 처리할
수 있습니다! 동전을 비교하는 대신 `Option<T>`의 배리언트를 비교하지만, `match`
표현식이 동작하는 방식은 동일합니다.

`Option<i32>`를 받아, 안에 값이 있으면 그 값에 1을 더하고, 안에 값이 없으면 `None`
값을 반환하고 어떤 연산도 시도하지 않는 함수를 작성한다고 해 봅시다.

이 함수는 `match` 덕분에 아주 쉽게 작성할 수 있고, Listing 6-5처럼 생겼습니다.

<Listing number="6-5" caption="`Option<i32>`에 `match` 표현식을 사용하는 함수">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

`plus_one`의 첫 실행을 좀 더 자세히 살펴봅시다. `plus_one(five)`를 호출하면,
`plus_one` 본문의 변수 `x`는 `Some(5)` 값을 가집니다. 그런 다음 그 값을 각 match
갈래와 비교합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

`Some(5)` 값은 패턴 `None`과 매칭되지 않으므로 다음 갈래로 넘어갑니다.

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)`가 `Some(i)`와 매칭되나요? 됩니다! 같은 배리언트입니다. `i`는 `Some`이
담고 있는 값에 바인딩되므로, `i`는 `5`라는 값을 가집니다. 그런 다음 match 갈래의
코드가 실행되어, `i` 값에 1을 더하고 결과인 `6`을 담은 새 `Some` 값을 만듭니다.

이제 Listing 6-5에서 `x`가 `None`인 `plus_one`의 두 번째 호출을 살펴봅시다.
`match`에 들어가 첫 갈래와 비교합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

매칭됩니다! 더할 값이 없으므로 프로그램은 멈추고 `=>` 오른쪽의 `None` 값을 반환
합니다. 첫 갈래가 매칭되었기 때문에 다른 갈래들은 비교되지 않습니다.

`match`와 열거형을 함께 쓰는 것은 많은 상황에서 유용합니다. 러스트 코드에서 이
패턴을 아주 자주 보게 될 것입니다. 열거형에 대해 `match`하고, 변수를 내부 데이터에
바인딩한 뒤, 그에 기반해 코드를 실행하는 패턴이죠. 처음에는 조금 까다롭지만, 익숙
해지면 모든 언어에 이 기능이 있었으면 좋겠다는 생각이 들 것입니다. 꾸준히 사용자가
가장 좋아하는 기능 중 하나입니다.

### match는 철저해야 합니다

`match`에 대해 논의해야 할 또 다른 측면이 있습니다. 갈래의 패턴들이 모든 가능성을
반드시 다루어야 한다는 점입니다. 컴파일되지 않는 다음 버전의 `plus_one` 함수를
봅시다. 버그가 있습니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

`None` 경우를 처리하지 않았으므로, 이 코드는 버그를 일으킬 것입니다. 다행히 러스트가
잡을 줄 아는 버그입니다. 이 코드를 컴파일하려고 하면 다음 에러가 납니다.

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

러스트는 우리가 모든 가능한 경우를 다루지 않았음을 알고 있고, 심지어 우리가 잊은
패턴이 무엇인지도 알고 있습니다! 러스트에서 match는 *철저(exhaustive)*합니다. 코드가
유효하려면 마지막 가능성까지 모두 소진해야 합니다. 특히 `Option<T>`의 경우, 러스트
가 `None` 경우를 명시적으로 처리하는 것을 잊지 못하도록 막아 주기 때문에, 값이
null일 수 있는데 값이 있다고 가정하는 일을 막아 주고, 앞서 이야기한 10억 달러짜리
실수를 불가능하게 만듭니다.

### 캐치올 패턴과 `_` 자리표시자

열거형을 사용해, 몇 가지 특정 값에는 특별한 동작을 취하고 나머지 모든 값에는
기본 동작 하나를 취하도록 할 수도 있습니다. 주사위를 굴려 3이 나오면 플레이어는
움직이지 않고 대신 멋진 새 모자를 받고, 7이 나오면 플레이어가 멋진 모자를 잃고,
그 외의 값에 대해서는 게임 보드에서 그만큼의 칸을 움직이는 게임을 구현한다고
상상해 보세요. 이 로직을 구현하는 `match`는 다음과 같으며, 주사위 결과는 임의의
값 대신 하드코딩했고, 나머지 로직은 실제 구현이 이 예제의 범위를 벗어나므로 본문이
없는 함수들로 표현했습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

첫 두 갈래의 패턴은 리터럴 값 `3`과 `7`입니다. 나머지 가능한 값 모두를 다루는
마지막 갈래의 패턴은 우리가 `other`라는 이름으로 선택한 변수입니다. `other`
갈래에서 실행되는 코드는 그 변수를 `move_player` 함수에 전달하면서 사용합니다.

`u8`이 가질 수 있는 모든 값들을 나열하지 않았음에도 이 코드가 컴파일되는 이유는,
마지막 패턴이 구체적으로 나열되지 않은 모든 값에 매칭되기 때문입니다. 이 캐치올
(catch-all) 패턴이 `match`가 반드시 철저해야 한다는 요구를 충족시켜 줍니다. 패턴은
순서대로 평가되므로, 캐치올 갈래는 반드시 마지막에 두어야 한다는 점에 유의하세요.
캐치올 갈래를 더 앞에 두면 다른 갈래들은 절대 실행되지 않을 것입니다. 그래서
러스트는 캐치올 뒤에 갈래를 추가하면 경고를 해 줄 것입니다!

러스트는 캐치올이 필요하지만 그 캐치올 패턴의 값을 *사용*하고 싶지는 않을 때 사용할
수 있는 패턴도 가지고 있습니다. `_`는 어떤 값에도 매칭되고 그 값에 바인딩하지
않는 특별한 패턴입니다. 이것은 그 값을 사용하지 않겠다고 러스트에게 알려 주며,
러스트는 사용하지 않은 변수에 대해 경고하지 않을 것입니다.

게임의 규칙을 바꿔 봅시다. 이제 3이나 7이 아닌 다른 값이 나오면 다시 굴려야
합니다. 캐치올 값을 더 이상 사용할 필요가 없으므로, 코드를 변경해 `other`라는
이름의 변수 대신 `_`를 사용할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

이 예시 또한 마지막 갈래에서 다른 모든 값을 명시적으로 무시하고 있으므로, 철저성
요구를 충족합니다. 아무것도 잊지 않았습니다.

마지막으로, 3이나 7이 아닌 값이 나오면 자신의 차례에서 아무 일도 일어나지 않도록
게임의 규칙을 한 번 더 바꿔 봅시다. 이를 표현하기 위해 [“튜플 타입”][tuples]
<!-- ignore --> 절에서 언급한 빈 튜플 타입인 유닛 값을 `_` 갈래에 딸린 코드로
사용할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

여기서는 앞선 갈래의 패턴 어느 것에도 매칭되지 않은 다른 값을 사용할 일이 없고,
이 경우에는 어떤 코드도 실행하지 않겠다고 러스트에게 명시적으로 말하고 있습니다.

패턴과 매칭에 대해서는 [19장][ch19-00-patterns]<!-- ignore -->에서 더 다룰 내용
이 있습니다. 지금은 `match` 표현식이 다소 장황한 상황에서 유용할 수 있는 `if let`
문법으로 넘어가겠습니다.

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch19-00-patterns]: ch19-00-patterns.html
