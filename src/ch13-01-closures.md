<!-- Old headings. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>
<a id="closures-anonymous-functions-that-capture-their-environment"></a>

## 클로저

러스트의 클로저는 변수에 저장하거나 다른 함수에 인수로 전달할 수 있는 익명
함수입니다. 클로저를 한 곳에서 만들어 두고, 다른 곳에서 다른 맥락으로 평가하기
위해 호출할 수 있습니다. 함수와 달리 클로저는 자신이 정의된 스코프에서 값을
캡처할 수 있습니다. 이러한 클로저 기능이 어떻게 코드 재사용과 동작 사용자화를
가능하게 하는지 보여 드리겠습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>
<a id="capturing-the-environment-with-closures"></a>

### 환경 캡처하기

먼저 클로저가 자신이 정의된 환경의 값을 캡처해 나중에 사용할 수 있도록 하는
방법을 살펴보겠습니다. 시나리오는 다음과 같습니다. 가끔 우리 티셔츠 회사는
프로모션으로 메일링 리스트에 있는 누군가에게 한정판 셔츠를 무료로 증정합니다.
메일링 리스트의 사람들은 선택적으로 자신의 프로필에 좋아하는 색을 추가할 수
있습니다. 무료 셔츠 당첨자가 좋아하는 색을 설정해 두었다면 그 색의 셔츠를
받습니다. 좋아하는 색을 지정하지 않았다면, 회사가 현재 가장 많이 보유하고 있는
색의 셔츠를 받습니다.

이를 구현하는 방법은 여러 가지입니다. 이 예제에서는 단순화를 위해 사용 가능한
색의 수를 제한해, 배리언트로 `Red`와 `Blue`를 가진 `ShirtColor`라는 열거형을
사용합니다. 회사의 재고는 `shirts`라는 필드를 가지는 `Inventory` 구조체로
표현합니다. `shirts` 필드는 현재 재고인 셔츠 색들을 나타내는 `Vec<ShirtColor>`
를 담습니다. `Inventory`에 정의된 `giveaway` 메서드는 무료 셔츠 당첨자의
선택적 셔츠 색 선호도를 받아 그 사람이 받을 셔츠 색을 반환합니다. 이 구성은
Listing 13-1에 나와 있습니다.

<Listing number="13-1" file-name="src/main.rs" caption="셔츠 회사의 증정 상황">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

`main`에 정의된 `store`는 이 한정판 프로모션을 위해 분배할 수 있는 파란 셔츠
두 장과 빨간 셔츠 한 장을 남겨 두고 있습니다. 빨간 셔츠를 선호하는 사용자와
선호가 없는 사용자 각각에 대해 `giveaway` 메서드를 호출합니다.

다시 말하지만, 이 코드는 여러 방식으로 구현될 수 있습니다. 여기서는 클로저에
집중하기 위해, 클로저를 사용하는 `giveaway` 메서드 본문을 제외하고는 이미 배운
개념들에 충실했습니다. `giveaway` 메서드에서는 사용자 선호를 `Option<ShirtColor>`
타입의 매개변수로 받고, `user_preference`에 `unwrap_or_else` 메서드를 호출합니다.
[`Option<T>`의 `unwrap_or_else` 메서드][unwrap-or-else]<!-- ignore -->는 표준
라이브러리에 정의되어 있습니다. 이 메서드는 인수 하나를 받습니다. 인수는
아무 인수도 받지 않고 `T` 값(`Option<T>`의 `Some` 배리언트에 저장되는 것과
같은 타입, 여기서는 `ShirtColor`)을 반환하는 클로저입니다. `Option<T>`가 `Some`
배리언트이면 `unwrap_or_else`는 `Some` 안의 값을 반환합니다. `Option<T>`가
`None` 배리언트이면 `unwrap_or_else`는 클로저를 호출하고 클로저가 반환한 값을
반환합니다.

`unwrap_or_else`의 인수로 클로저 식 `|| self.most_stocked()`를 지정합니다. 이는
매개변수가 없는 클로저입니다(클로저에 매개변수가 있다면 두 수직 파이프 사이에
나타납니다). 클로저의 본문은 `self.most_stocked()`를 호출합니다. 여기서
클로저를 정의만 해 두고, 결과가 필요하면 `unwrap_or_else`의 구현이 나중에
클로저를 평가하게 됩니다.

이 코드를 실행하면 다음이 출력됩니다.

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

여기서 흥미로운 점 하나는, 현재 `Inventory` 인스턴스에서 `self.most_stocked()`
를 호출하는 클로저를 전달했다는 것입니다. 표준 라이브러리는 우리가 정의한
`Inventory`나 `ShirtColor` 타입, 또는 이 시나리오에서 사용하려는 로직에 대해
아무것도 알 필요가 없었습니다. 클로저는 `self` `Inventory` 인스턴스에 대한
불변 참조를 캡처하고, 우리가 지정한 코드와 함께 `unwrap_or_else` 메서드에
전달합니다. 반면 함수는 이런 방식으로 환경을 캡처할 수 없습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="closure-type-inference-and-annotation"></a>

### 클로저 타입 추론과 명시

함수와 클로저에는 더 많은 차이가 있습니다. 클로저는 보통 `fn` 함수처럼 매개변수
나 반환 값의 타입을 명시하도록 요구하지 않습니다. 타입 명시는 함수에 필요한데,
타입이 사용자에게 노출되는 명시적 인터페이스의 일부이기 때문입니다. 이
인터페이스를 엄격하게 정의하는 것은 함수가 사용하고 반환하는 값의 타입에 대해
모두가 동의하도록 보장하는 데 중요합니다. 반면 클로저는 그런 식으로 노출된
인터페이스에 사용되지 않습니다. 변수에 저장되며, 라이브러리 사용자에게 이름을
붙여 노출하지 않고 사용됩니다.

클로저는 전형적으로 짧고, 임의의 시나리오가 아니라 좁은 맥락 안에서만
관련됩니다. 이러한 제한된 맥락 안에서 컴파일러는 대부분의 변수 타입을 추론할
수 있는 것과 비슷하게 매개변수와 반환 값의 타입을 추론할 수 있습니다(컴파일러
가 클로저 타입 명시를 필요로 하는 드문 경우도 있습니다).

변수처럼, 엄밀히 필요하지 않지만 장황해지는 대가로 명시성과 명확성을 높이고
싶다면 타입 명시를 추가할 수 있습니다. 클로저에 타입을 명시하는 모습은 Listing
13-2의 정의처럼 됩니다. 이 예제에서는 Listing 13-1처럼 인수로 전달하는 자리에
클로저를 정의하는 대신, 클로저를 정의해 변수에 저장합니다.

<Listing number="13-2" file-name="src/main.rs" caption="클로저에서 매개변수와 반환 값 타입을 선택적으로 명시하기">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

타입 명시를 추가하면, 클로저의 문법이 함수의 문법과 더 비슷해 보입니다. 여기서는
매개변수에 1을 더하는 함수와, 같은 동작을 하는 클로저를 비교를 위해 정의합니다.
관련된 부분을 맞추기 위해 공백을 추가했습니다. 이는 파이프 사용과 문법 일부가
선택적이라는 점만 빼면 클로저 문법이 함수 문법과 얼마나 비슷한지 보여 줍니다.

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

첫 번째 줄은 함수 정의를 보여 주고, 두 번째 줄은 완전히 애너테이트된 클로저
정의를 보여 줍니다. 세 번째 줄에서는 클로저 정의에서 타입 명시를 제거합니다.
네 번째 줄에서는 클로저 본문에 식이 하나뿐이므로 선택적인 중괄호를 제거합니다.
이 모두는 유효한 정의이며, 호출될 때 동일한 동작을 만들어 냅니다. `add_one_v3`
와 `add_one_v4` 줄은 사용처에서 타입이 추론되므로 컴파일되려면 클로저가 평가
되어야 합니다. 이는 `let v = Vec::new();`가 러스트가 타입을 추론할 수 있도록
타입 명시나 `Vec`에 삽입할 어떤 타입의 값이 필요한 것과 비슷합니다.

클로저 정의에 대해, 컴파일러는 각 매개변수와 반환 값에 대해 하나의 구체적
타입을 추론합니다. 예를 들어 Listing 13-3은 매개변수로 받은 값을 그대로 반환
하는 짧은 클로저의 정의를 보여 줍니다. 이 예제 목적을 제외하면 이 클로저는
그다지 유용하지 않습니다. 정의에 어떤 타입 명시도 추가하지 않았음에 유의
하세요. 타입 명시가 없으므로 어떤 타입으로도 클로저를 호출할 수 있으며, 여기서는
처음에 `String`으로 호출했습니다. 그런 다음 `example_closure`를 정수로
호출하려 하면 오류가 납니다.

<Listing number="13-3" file-name="src/main.rs" caption="타입이 추론되는 클로저를 두 가지 다른 타입으로 호출하려는 시도">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

컴파일러는 다음 오류를 줍니다.

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

처음 `example_closure`를 `String` 값으로 호출하면, 컴파일러는 `x`의 타입과
클로저의 반환 타입을 `String`으로 추론합니다. 그 타입들은 `example_closure`의
클로저에 고정되며, 같은 클로저를 다른 타입으로 사용하려 하면 타입 오류가 납니다.

### 참조를 캡처하거나 소유권 이동하기

클로저는 환경에서 값을 세 가지 방식으로 캡처할 수 있고, 이는 함수가 매개변수를
받는 세 가지 방식에 직접 대응됩니다. 불변으로 빌리기, 가변으로 빌리기, 소유권
가져가기입니다. 클로저는 함수 본문이 캡처된 값으로 무엇을 하는지에 따라 이 중
어떤 방식을 쓸지 결정합니다.

Listing 13-4에서는 값을 출력하는 데 불변 참조만 필요하므로 `list`라는 벡터에
대한 불변 참조를 캡처하는 클로저를 정의합니다.

<Listing number="13-4" file-name="src/main.rs" caption="불변 참조를 캡처하는 클로저 정의하고 호출하기">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

이 예제는 또한 변수가 클로저 정의에 바인딩될 수 있고, 나중에 변수 이름이
함수 이름인 것처럼 변수 이름과 괄호를 써서 클로저를 호출할 수 있음을 보여 줍니다.

같은 시각에 `list`에 대한 불변 참조가 여러 개 있을 수 있으므로, `list`는
클로저 정의 이전 코드에서도, 클로저 정의 이후 호출 이전에도, 클로저 호출 이후
에도 접근 가능합니다. 이 코드는 컴파일되고 실행되며 다음을 출력합니다.

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

다음으로 Listing 13-5에서는 `list` 벡터에 요소를 추가하도록 클로저 본문을
바꿉니다. 이제 클로저는 가변 참조를 캡처합니다.

<Listing number="13-5" file-name="src/main.rs" caption="가변 참조를 캡처하는 클로저 정의하고 호출하기">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

이 코드는 컴파일되고 실행되며 다음을 출력합니다.

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

이제 `borrows_mutably` 클로저의 정의와 호출 사이에 `println!`이 없음에
유의하세요. `borrows_mutably`가 정의될 때 `list`에 대한 가변 참조를 캡처합니다.
클로저가 호출된 뒤에는 클로저를 다시 쓰지 않으므로 가변 빌림이 끝납니다.
클로저 정의와 클로저 호출 사이에는 출력을 위한 불변 빌림이 허용되지 않습니다.
가변 빌림이 있을 때는 다른 빌림이 허용되지 않기 때문입니다. 거기에 `println!`
을 추가해 어떤 오류 메시지를 받는지 시도해 보세요!

클로저 본문이 엄밀히 소유권을 필요로 하지 않더라도, 환경에서 사용하는 값의
소유권을 클로저가 가져가게 강제하려면 매개변수 목록 앞에 `move` 키워드를
사용할 수 있습니다.

이 기법은 주로 새 스레드로 클로저를 전달하면서 데이터를 새 스레드가 소유하도록
이동할 때 유용합니다. 스레드와 그것을 왜 사용하는지는 16장에서 동시성을
이야기할 때 자세히 다루겠습니다. 지금은 `move` 키워드가 필요한 클로저로 새
스레드를 시작하는 예를 간략히 살펴봅시다. Listing 13-6은 Listing 13-4를 메인
스레드가 아닌 새 스레드에서 벡터를 출력하도록 변형한 것입니다.

<Listing number="13-6" file-name="src/main.rs" caption="스레드용 클로저가 `list`의 소유권을 가져가도록 `move` 사용하기">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

새 스레드를 시작하고, 실행할 클로저를 인수로 전달합니다. 클로저 본문은 리스트를
출력합니다. Listing 13-4에서는 `list`를 출력하는 데 필요한 최소한의 접근만
필요했기 때문에 불변 참조로만 `list`를 캡처했습니다. 이 예제에서는 클로저
본문이 여전히 불변 참조만 필요하더라도, 클로저 정의 맨 앞에 `move` 키워드를
두어 `list`가 클로저로 이동되어야 함을 명시해야 합니다. 메인 스레드가 새
스레드에 `join`을 호출하기 전에 더 많은 작업을 한다면, 새 스레드가 메인 스레드
의 나머지가 끝나기 전에 끝날 수도 있고, 메인 스레드가 먼저 끝날 수도 있습니다.
메인 스레드가 `list`의 소유권을 유지하다가 새 스레드보다 먼저 끝나 `list`를
드롭하면, 스레드의 불변 참조는 유효하지 않게 됩니다. 따라서 컴파일러는 참조가
유효하도록 `list`가 새 스레드에 주어진 클로저로 이동되도록 요구합니다. `move`
키워드를 제거하거나 클로저 정의 이후 메인 스레드에서 `list`를 사용해 어떤
컴파일러 오류를 받는지 시도해 보세요!

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>
<a id="moving-captured-values-out-of-closures-and-the-fn-traits"></a>

### 캡처된 값을 클로저 밖으로 이동하기

클로저가 정의된 환경에서 참조를 캡처하거나 값의 소유권을 캡처하면(따라서
클로저 _안으로_ 무엇이 이동되는지가 결정됨), 클로저 본문의 코드는 클로저가
나중에 평가될 때 그 참조나 값에 무슨 일이 일어나는지를 정의합니다(따라서
클로저 _밖으로_ 무엇이 이동되는지가 결정됨).

클로저 본문은 다음 중 어떤 것이든 할 수 있습니다. 캡처된 값을 클로저 밖으로
이동하기, 캡처된 값을 변경하기, 값을 이동하지도 변경하지도 않기, 애초에
환경에서 아무것도 캡처하지 않기.

클로저가 환경에서 값을 캡처하고 다루는 방식은 클로저가 구현하는 트레이트에
영향을 줍니다. 그리고 함수와 구조체는 어떤 종류의 클로저를 사용할 수 있는지
트레이트로 지정할 수 있습니다. 클로저는 본문이 값을 어떻게 다루는지에 따라,
아래 세 `Fn` 트레이트 중 하나, 둘, 또는 세 개 모두를 누적적으로 자동으로
구현합니다.

* `FnOnce`는 한 번 호출될 수 있는 클로저에 적용됩니다. 모든 클로저는 최소한 이
  트레이트를 구현합니다. 모든 클로저는 호출될 수 있기 때문입니다. 캡처된 값을
  본문 밖으로 이동하는 클로저는 `FnOnce`만 구현하고 다른 `Fn` 트레이트는 구현
  하지 않습니다. 한 번만 호출될 수 있기 때문입니다.
* `FnMut`는 캡처된 값을 본문 밖으로 이동하지는 않지만 값을 변경할 수는 있는
  클로저에 적용됩니다. 이런 클로저는 두 번 이상 호출될 수 있습니다.
* `Fn`은 캡처된 값을 본문 밖으로 이동하지도 변경하지도 않는 클로저와, 환경에서
  아무것도 캡처하지 않는 클로저에 적용됩니다. 이런 클로저는 환경을 변경하지
  않고 두 번 이상 호출될 수 있는데, 이는 동시에 여러 번 클로저를 호출하는
  경우처럼 중요한 상황에서 중요합니다.

Listing 13-1에서 사용한 `Option<T>`의 `unwrap_or_else` 메서드 정의를 살펴봅시다.

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

`T`는 `Option`의 `Some` 배리언트에 있는 값의 타입을 나타내는 제네릭 타입
이었음을 기억하세요. 그 타입 `T`는 `unwrap_or_else` 함수의 반환 타입이기도
합니다. 예를 들어 `Option<String>`에 `unwrap_or_else`를 호출하는 코드는
`String`을 얻습니다.

다음으로, `unwrap_or_else` 함수에는 추가 제네릭 타입 매개변수 `F`가 있음에
유의하세요. `F` 타입은 `f`라는 매개변수의 타입이며, `unwrap_or_else`를 호출할
때 우리가 제공하는 클로저입니다.

제네릭 타입 `F`에 지정된 트레이트 경계는 `FnOnce() -> T`이며, 이는 `F`가 한
번 호출될 수 있고, 인수를 받지 않으며, `T`를 반환할 수 있어야 한다는 뜻입니다.
트레이트 경계에 `FnOnce`를 사용하는 것은 `unwrap_or_else`가 `f`를 한 번
이상 호출하지 않는다는 제약을 표현합니다. `unwrap_or_else`의 본문을 보면,
`Option`이 `Some`이면 `f`가 호출되지 않고, `Option`이 `None`이면 `f`가 한 번
호출될 것임을 알 수 있습니다. 모든 클로저가 `FnOnce`를 구현하므로,
`unwrap_or_else`는 세 종류의 클로저를 모두 받으며 가능한 한 유연합니다.

> 참고: 우리가 하려는 일이 환경에서 값을 캡처할 필요가 없다면, `Fn` 트레이트
> 중 하나를 구현하는 무언가가 필요한 자리에 클로저 대신 함수의 이름을 사용할
> 수 있습니다. 예를 들어 `Option<Vec<T>>` 값에서, 값이 `None`이면 새로운 빈
> 벡터를 얻기 위해 `unwrap_or_else(Vec::new)`를 호출할 수 있습니다. 컴파일러는
> 함수 정의에 해당하는 `Fn` 트레이트 중 적용 가능한 것을 자동으로 구현합니다.

이제 슬라이스에 정의된 표준 라이브러리 메서드 `sort_by_key`를 살펴보며, 그것이
`unwrap_or_else`와 어떻게 다르고, `sort_by_key`가 트레이트 경계로 `FnOnce`
대신 `FnMut`를 사용하는 이유를 봅시다. 이 클로저는 슬라이스에서 현재 고려
중인 항목에 대한 참조 형태의 인수 하나를 받고, 순서를 매길 수 있는 `K` 타입의
값을 반환합니다. 이 함수는 각 항목의 특정 속성으로 슬라이스를 정렬하고 싶을
때 유용합니다. Listing 13-7에서는 `Rectangle` 인스턴스 목록을 `width` 속성으로
낮은 순에서 높은 순으로 정렬하기 위해 `sort_by_key`를 사용합니다.

<Listing number="13-7" file-name="src/main.rs" caption="`sort_by_key`로 사각형을 너비순으로 정렬하기">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

이 코드는 다음을 출력합니다.

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

`sort_by_key`가 `FnMut` 클로저를 받도록 정의된 이유는 클로저를 여러 번, 즉
슬라이스의 각 항목당 한 번씩 호출하기 때문입니다. 클로저 `|r| r.width`는
환경에서 아무것도 캡처하거나 변경하거나 이동하지 않으므로 트레이트 경계
요구 사항을 충족합니다.

반대로 Listing 13-8은 환경에서 값을 이동하므로 `FnOnce` 트레이트만 구현하는
클로저의 예를 보여 줍니다. 컴파일러는 이 클로저를 `sort_by_key`에 사용하도록
허용하지 않습니다.

<Listing number="13-8" file-name="src/main.rs" caption="`FnOnce` 클로저를 `sort_by_key`에 사용하려는 시도">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

이것은 `list`를 정렬할 때 `sort_by_key`가 클로저를 몇 번 호출하는지 세려는
억지스럽고 복잡한(동작도 하지 않는) 방법입니다. 이 코드는 클로저의 환경에서
온 `String`인 `value`를 `sort_operations` 벡터에 푸시하여 이 세기를 하려고
합니다. 클로저는 `value`를 캡처한 다음, `value`의 소유권을 `sort_operations`
벡터로 이전함으로써 `value`를 클로저 밖으로 이동합니다. 이 클로저는 한 번 호출
될 수 있습니다. 두 번째 호출은 동작하지 않습니다. `value`가 환경에 더 이상
존재하지 않아 다시 `sort_operations`에 푸시할 수 없기 때문입니다! 따라서 이
클로저는 `FnOnce`만 구현합니다. 이 코드를 컴파일하려 하면, 클로저가 `FnMut`
를 구현해야 하는데 `value`가 클로저 밖으로 이동될 수 없다는 다음 오류를 얻습니다.

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

오류는 `value`를 환경 밖으로 이동하는 클로저 본문 줄을 가리킵니다. 이를 고치
려면 클로저 본문이 값을 환경 밖으로 이동하지 않도록 바꿔야 합니다. 환경에
카운터를 두고 클로저 본문에서 그 값을 증가시키는 것은 클로저 호출 횟수를
세는 더 직관적인 방법입니다. Listing 13-9의 클로저는 `num_sort_operations`
카운터에 대한 가변 참조만 캡처하므로 두 번 이상 호출될 수 있어서 `sort_by_key`
에 사용할 수 있습니다.

<Listing number="13-9" file-name="src/main.rs" caption="`sort_by_key`와 함께 `FnMut` 클로저 사용은 허용됩니다.">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

`Fn` 트레이트는 클로저를 활용하는 함수나 타입을 정의하거나 사용할 때 중요합니다.
다음 절에서는 이터레이터를 다룹니다. 많은 이터레이터 메서드가 클로저를 인수로
받으므로, 계속 진행하면서 이 클로저 세부 사항을 염두에 두세요!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
