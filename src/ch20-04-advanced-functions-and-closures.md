## 고급 함수와 클로저

이 절에서는 함수 포인터와 클로저 반환을 비롯한, 함수와 클로저에 관련된 고급 기능 몇 가지를 살펴봅니다.

### 함수 포인터

클로저를 함수에 전달하는 방법에 대해 이야기했습니다. 일반 함수도 함수에 전달할 수 있습니다! 이 기법은 새로운 클로저를 정의하는 대신 이미 정의해 둔 함수를 전달하고 싶을 때 유용합니다. 함수는 (소문자 _f_ 의) `fn` 타입으로 강제 변환됩니다. 이를 `Fn` 클로저 트레이트와 혼동하면 안 됩니다. `fn` 타입은 _함수 포인터(function pointer)_ 라고 부릅니다. 함수 포인터로 함수를 전달하면 함수를 다른 함수의 인수로 사용할 수 있습니다.

매개변수가 함수 포인터임을 명시하는 문법은 클로저의 문법과 비슷합니다. Listing 20-28에서, 매개변수에 1을 더하는 `add_one` 함수를 정의했습니다. 함수 `do_twice`는 두 매개변수를 받습니다. `i32` 매개변수를 받고 `i32`를 반환하는 어떤 함수에 대한 함수 포인터와, 하나의 `i32` 값입니다. `do_twice` 함수는 함수 `f`를 두 번 호출하면서 `arg` 값을 전달한 다음, 두 함수 호출 결과를 더합니다. `main` 함수는 `add_one`과 `5`를 인수로 `do_twice`를 호출합니다.

<Listing number="20-28" file-name="src/main.rs" caption="`fn` 타입을 사용해 함수 포인터를 인수로 받기">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

이 코드는 `The answer is: 12`를 출력합니다. `do_twice`의 매개변수 `f`가 `i32` 타입의 매개변수 하나를 받고 `i32`를 반환하는 `fn`임을 명시합니다. 그러면 `do_twice`의 본문에서 `f`를 호출할 수 있습니다. `main`에서는 함수 이름 `add_one`을 `do_twice`의 첫 인수로 전달할 수 있습니다.

클로저와 달리 `fn`은 트레이트가 아니라 타입이므로, `Fn` 트레이트 중 하나를 트레이트 바운드로 사용해 제네릭 타입 매개변수를 선언하는 대신 `fn`을 매개변수 타입으로 직접 명시합니다.

함수 포인터는 세 클로저 트레이트(`Fn`, `FnMut`, `FnOnce`) 모두를 구현하므로, 클로저를 기대하는 함수에 인수로 함수 포인터를 항상 전달할 수 있습니다. 함수가 함수든 클로저든 받을 수 있도록, 제네릭 타입과 클로저 트레이트 중 하나를 사용해 함수를 작성하는 것이 가장 좋습니다.

그렇긴 해도, 클로저를 받지 않고 `fn`만 받고 싶은 경우의 한 예는 클로저가 없는 외부 코드와 인터페이싱할 때입니다. C 함수는 함수를 인수로 받을 수 있지만, C에는 클로저가 없습니다.

인라인으로 정의된 클로저나 이름 붙은 함수 중 어느 쪽이라도 사용할 수 있는 예시로, 표준 라이브러리의 `Iterator` 트레이트가 제공하는 `map` 메서드의 사용을 살펴봅시다. 숫자의 벡터를 문자열의 벡터로 바꾸기 위해 `map` 메서드를 사용하려면, Listing 20-29처럼 클로저를 사용할 수 있습니다.

<Listing number="20-29" caption="숫자를 문자열로 변환하기 위해 `map` 메서드와 함께 클로저 사용">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

또는 클로저 대신 `map`의 인수로 함수의 이름을 사용할 수도 있습니다. Listing 20-30은 그것이 어떤 모습인지 보여 줍니다.

<Listing number="20-30" caption="숫자를 문자열로 변환하기 위해 `map` 메서드와 함께 `String::to_string` 함수 사용">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

`to_string`이라는 이름의 함수가 여러 개 사용 가능하므로, [“고급 트레이트”][advanced-traits]<!-- ignore --> 절에서 이야기한 완전 한정 문법을 사용해야 한다는 점에 주목하세요.

여기서는 `ToString` 트레이트에 정의된 `to_string` 함수를 사용하고 있는데, 표준 라이브러리는 이를 `Display`를 구현하는 어떤 타입에든 구현해 두었습니다.

6장의 [“열거형 값”][enum-values]<!-- ignore --> 절에서, 우리가 정의하는 각 열거형 변형의 이름이 초기화 함수가 되기도 한다는 것을 떠올려 보세요. 이 초기화 함수들은 클로저 트레이트를 구현하는 함수 포인터로 사용할 수 있어서, 클로저를 받는 메서드의 인수로 초기화 함수를 명시할 수 있습니다. Listing 20-31에서 보입니다.

<Listing number="20-31" caption="숫자로부터 `Status` 인스턴스를 만들기 위해 `map` 메서드와 함께 열거형 초기화 함수 사용">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

여기서는 `Status::Value`의 초기화 함수를 사용해 `map`이 호출되는 범위 안의 각 `u32` 값을 가지고 `Status::Value` 인스턴스를 만듭니다. 어떤 사람들은 이 스타일을 선호하고, 어떤 사람들은 클로저를 사용하는 것을 선호합니다. 같은 코드로 컴파일되므로, 어느 스타일이든 더 명료한 쪽을 사용하세요.

### 클로저 반환

클로저는 트레이트로 표현되므로, 클로저를 직접 반환할 수는 없습니다. 트레이트를 반환하고 싶을 만한 대부분의 경우에는, 트레이트를 구현하는 구체 타입을 함수의 반환값으로 사용할 수 있습니다. 그러나 클로저는 보통 그렇게 할 수 없습니다. 반환할 수 있는 구체 타입을 가지고 있지 않기 때문입니다. 예를 들어, 클로저가 자신의 스코프에서 어떤 값을 캡처한다면 함수 포인터 `fn`을 반환 타입으로 사용할 수 없습니다.

대신 보통은 10장에서 배운 `impl Trait` 문법을 사용합니다. `Fn`, `FnOnce`, `FnMut`을 사용해 어떤 함수 타입이든 반환할 수 있습니다. 예를 들어 Listing 20-32의 코드는 잘 컴파일됩니다.

<Listing number="20-32" caption="`impl Trait` 문법을 사용해 함수에서 클로저 반환하기">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

그러나 13장의 [“클로저 타입 추론과 표기”][closure-types]<!-- ignore --> 절에서 언급했듯이, 각 클로저는 또한 자신만의 별개의 타입입니다. 같은 시그니처를 가지지만 다른 구현을 가진 여러 함수와 함께 작업해야 한다면, 그것들에 대해 트레이트 객체를 사용해야 합니다. Listing 20-33과 같은 코드를 작성하면 무슨 일이 일어나는지 봅시다.

<Listing file-name="src/main.rs" number="20-33" caption="`impl Fn` 타입을 반환하는 함수들로 정의된 클로저들의 `Vec<T>` 만들기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

여기 두 함수 `returns_closure`와 `returns_initialized_closure`가 있는데, 둘 다 `impl Fn(i32) -> i32`를 반환합니다. 두 함수가 반환하는 클로저들이 같은 타입을 구현하더라도 서로 다르다는 점에 주목하세요. 이를 컴파일하려고 하면 러스트는 그것이 동작하지 않을 것임을 우리에게 알려 줍니다.

```text
{{#include ../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

오류 메시지는 우리가 `impl Trait`을 반환할 때마다, 러스트가 고유한 _불투명 타입(opaque type)_ 을 만든다는 것을 알려 줍니다. 불투명 타입은 러스트가 우리를 위해 무엇을 만드는지 그 세부 사항을 들여다볼 수 없는 타입이며, 우리가 직접 작성하기 위해 러스트가 생성할 타입을 짐작할 수도 없습니다. 그래서 이 함수들이 같은 트레이트 `Fn(i32) -> i32`를 구현하는 클로저들을 반환하더라도, 러스트가 각각에 대해 생성하는 불투명 타입은 서로 별개입니다. (이는 같은 출력 타입을 가지더라도 서로 다른 비동기 블록에 대해 러스트가 다른 구체 타입을 만드는 것과 비슷한데, 17장의 [“`Pin` 타입과 `Unpin` 트레이트”][future-types]<!-- ignore -->에서 본 적이 있습니다.) 이 문제에 대한 해결책은 이미 몇 번 보았습니다. Listing 20-34에서처럼 트레이트 객체를 사용할 수 있습니다.

<Listing number="20-34" caption="같은 타입을 가지도록 `Box<dyn Fn>`을 반환하는 함수들로 정의된 클로저들의 `Vec<T>` 만들기">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

이 코드는 잘 컴파일됩니다. 트레이트 객체에 대한 더 많은 내용은 18장의 [“공유된 동작을 추상화하기 위한 트레이트 객체 사용”][trait-objects]<!-- ignore --> 절을 참고하세요.

다음으로 매크로를 살펴봅시다!

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[future-types]: ch17-03-more-futures.html
[trait-objects]: ch18-02-trait-objects.html
