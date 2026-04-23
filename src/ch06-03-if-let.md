## `if let`과 `let...else`로 간결한 흐름 제어

`if let` 문법은 `if`와 `let`을 결합해서, 하나의 패턴에 매칭되는 값을 처리하고
나머지는 무시하는 덜 장황한 방법을 제공합니다. Listing 6-6의 프로그램은 `config_max`
변수의 `Option<u8>` 값에 대해 `match`를 수행하지만, 값이 `Some` 배리언트일 때에만
코드를 실행하고 싶어 합니다.

<Listing number="6-6" caption="값이 `Some`일 때에만 코드를 실행하고 싶은 `match`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

값이 `Some`이라면, 패턴에서 값을 변수 `max`에 바인딩해 `Some` 배리언트 안의 값을
출력합니다. `None` 값에 대해서는 아무것도 하고 싶지 않습니다. 그런데 `match`
표현식을 만족시키기 위해 하나의 배리언트만 처리한 뒤에도 `_ => ()`를 추가해야
하는데, 이는 성가신 보일러플레이트 코드입니다.

대신 `if let`을 사용하면 더 짧게 쓸 수 있습니다. 다음 코드는 Listing 6-6의 `match`
와 같은 동작을 합니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

`if let` 문법은 등호를 사이에 두고 패턴과 표현식을 받습니다. 동작 방식은 `match`
와 같습니다. 표현식은 `match`에 주어지고, 패턴은 그 첫 번째 갈래입니다. 이 경우
패턴은 `Some(max)`이고, `max`는 `Some` 안의 값에 바인딩됩니다. 그런 다음 대응하는
`match` 갈래에서 `max`를 사용했던 것과 똑같은 방식으로 `if let` 블록 본문에서
`max`를 사용할 수 있습니다. `if let` 블록의 코드는 값이 패턴과 매칭될 때에만 실행
됩니다.

`if let`을 사용하면 타이핑과 들여쓰기가 줄고 보일러플레이트 코드가 줄어듭니다.
그러나 `match`가 강제해 주는 철저성 검사는 잃습니다. 철저성 검사는 우리가 어떤
경우를 처리하는 것을 잊지 않도록 해 주는 기능이죠. `match`와 `if let` 중 무엇을
선택할지는 여러분의 특정 상황에서 무엇을 하려는지, 그리고 간결함을 얻는 대신
철저성 검사를 잃는 것이 적절한 절충인지에 달려 있습니다.

다시 말해, `if let`을 값이 하나의 패턴에 매칭될 때 코드를 실행하고 다른 모든
값은 무시하는 `match`의 구문 설탕(syntax sugar)으로 생각할 수 있습니다.

`if let`에는 `else`를 포함할 수도 있습니다. `else`와 함께 가는 코드 블록은,
`if let`과 `else`에 상응하는 `match` 표현식에서 `_` 경우에 갔을 코드 블록과 같은
역할을 합니다. Listing 6-4의 `Coin` 열거형 정의를 떠올려 보세요. `Quarter` 배리
언트가 `UsState` 값도 담고 있었습니다. 쿼터의 주를 외치는 동시에 쿼터가 아닌 동전의
개수를 모두 세고 싶다면, 다음과 같이 `match` 표현식으로 할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

또는 다음처럼 `if let`과 `else` 표현식을 쓸 수도 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## `let...else`로 “행복 경로(Happy Path)”에 머무르기

흔한 패턴은, 값이 존재할 때 어떤 계산을 수행하고 그렇지 않을 때에는 기본값을
반환하는 것입니다. `UsState` 값을 가진 동전 예제를 이어서, 쿼터에 새겨진 주가
얼마나 오래된 주인지에 따라 재미있는 말을 하고 싶다면, `UsState`에 주의 연식을
확인하는 메서드를 다음과 같이 도입할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

그런 다음, Listing 6-7처럼 `if let`을 사용해 동전의 종류에 매칭하고, 조건 본문
안에 `state` 변수를 도입할 수 있습니다.

<Listing number="6-7" caption="`if let` 안에 중첩된 조건문으로 주가 1900년에 존재했는지 확인하기">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

이것도 일은 해내지만, 일을 `if let` 문의 본문 안으로 밀어 넣었고, 수행해야 할
일이 더 복잡하다면 최상위 분기들이 어떻게 관련되는지 정확히 따라가기 어려울 수
있습니다. Listing 6-8처럼, 표현식이 값을 만들어 낸다는 사실을 이용해 `if let`에서
`state`를 만들거나, 일찍 반환하도록 할 수도 있습니다. (`match`로도 비슷하게 할
수 있습니다.)

<Listing number="6-8" caption="`if let`으로 값을 만들거나 일찍 반환하기">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

이것 또한 나름대로 따라가기가 조금 성가십니다! `if let`의 한 분기는 값을 만들어
내고, 다른 분기는 함수에서 아예 반환해 버립니다.

이 흔한 패턴을 더 보기 좋게 표현하기 위해, 러스트에는 `let...else`가 있습니다.
`let...else` 문법은 `if let`과 아주 비슷하게 왼쪽에 패턴을, 오른쪽에 표현식을
받지만 `if` 분기가 없고 오직 `else` 분기만 있습니다. 패턴이 매칭되면 패턴의 값이
바깥 스코프의 변수에 바인딩됩니다. 패턴이 매칭되지 *않으면* 프로그램은 `else`
갈래로 흘러가며, 이 갈래는 반드시 함수에서 반환해야 합니다.

Listing 6-9에서 Listing 6-8을 `if let` 대신 `let...else`를 사용해 표현하면 어떻게
보이는지 확인할 수 있습니다.

<Listing number="6-9" caption="`let...else`로 함수를 흐르는 경로를 명확히 하기">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

이 방식으로 함수의 메인 본문이 “행복 경로(happy path)”에 머무른다는 점에 유의
하세요. `if let`처럼 두 분기 사이에 흐름 제어가 크게 달라지지 않습니다.

프로그램의 로직이 `match`로 표현하기에 너무 장황한 상황이라면, `if let`과
`let...else`도 러스트 도구 상자에 있다는 것을 기억하세요.

## 요약

지금까지 열거된 값들의 집합 중 하나가 될 수 있는 커스텀 타입을 만들기 위해
열거형을 사용하는 방법을 다뤘습니다. 표준 라이브러리의 `Option<T>` 타입이 에러를
막기 위해 타입 시스템을 활용하는 데 어떻게 도움이 되는지 보여 드렸습니다. 열거형
값이 안에 데이터를 담을 때, 다뤄야 하는 경우가 몇 개인지에 따라 `match` 또는
`if let`을 사용해 그 값을 꺼내고 사용할 수 있습니다.

이제 여러분의 러스트 프로그램은 구조체와 열거형으로 도메인의 개념을 표현할 수
있습니다. API에서 사용할 커스텀 타입을 만드는 일은 타입 안전성을 보장해 줍니다.
컴파일러는 여러분의 함수가 기대하는 타입의 값만 받도록 확실히 해 줄 것입니다.

잘 조직화되어 있고 사용하기 직관적이며 사용자가 필요한 것만 정확히 노출하는
API를 제공하기 위해, 이제 러스트의 모듈로 넘어가 봅시다.
