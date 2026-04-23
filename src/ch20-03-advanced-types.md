## 고급 타입

러스트 타입 시스템에는 지금까지 언급은 했지만 아직 논의하지 않은 몇 가지 기능들이 있습니다. 먼저 newtype을 일반적으로 살펴보면서 그것이 타입으로서 왜 유용한지 살펴봅니다. 그런 다음 newtype과 비슷하지만 약간 다른 의미론을 가진 기능인 타입 별칭(type alias)으로 넘어갑니다. `!` 타입과 동적 크기 타입에 대해서도 논의합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-for-type-safety-and-abstraction"></a>

### newtype 패턴을 사용한 타입 안전성과 추상화

이 절은 앞 절 [“newtype 패턴으로 외부 트레이트 구현하기”][newtype]<!-- ignore -->를 읽었다고 가정합니다. newtype 패턴은 지금까지 논의한 것 외에도 여러 작업에 유용한데, 여기에는 값들이 결코 혼동되지 않도록 정적으로 강제하는 것과 어떤 값의 단위를 나타내는 것이 포함됩니다. Listing 20-16에서 newtype을 사용해 단위를 나타내는 예시를 보았습니다. `Millimeters`와 `Meters` 구조체가 `u32` 값을 newtype으로 감쌌던 것을 떠올려 보세요. `Millimeters` 타입 매개변수를 가진 함수를 작성했다면, 실수로 `Meters` 타입의 값이나 보통 `u32` 값으로 그 함수를 호출하려는 프로그램은 컴파일할 수 없을 것입니다.

또한 newtype 패턴을 사용해 어떤 타입의 구현 세부 사항 일부를 추상화할 수 있습니다. 새 타입은 비공개 내부 타입의 API와 다른 공개 API를 노출할 수 있습니다.

newtype은 또한 내부 구현을 숨길 수 있습니다. 예를 들어, 사람의 ID와 이름을 연관시켜 저장하는 `HashMap<i32, String>`을 감싸는 `People` 타입을 제공할 수 있습니다. `People`을 사용하는 코드는 우리가 제공하는 공개 API, 예컨대 이름 문자열을 `People` 컬렉션에 추가하는 메서드와만 상호작용할 것입니다. 그 코드는 우리가 내부적으로 이름에 `i32` ID를 부여한다는 것을 알 필요가 없습니다. newtype 패턴은 18장의 [“구현 세부 사항을 숨기는 캡슐화”][encapsulation-that-hides-implementation-details]<!-- ignore --> 절에서 논의한 구현 세부 사항을 숨기기 위한 캡슐화를 가벼운 방식으로 달성합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-type-synonyms-with-type-aliases"></a>

### 타입 동의어와 타입 별칭

러스트는 기존 타입에 다른 이름을 주는 _타입 별칭(type alias)_ 을 선언할 수 있는 기능을 제공합니다. 이를 위해 `type` 키워드를 사용합니다. 예를 들어, 다음과 같이 `i32`에 대한 별칭 `Kilometers`를 만들 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

이제 `Kilometers`라는 별칭은 `i32`의 _동의어_ 입니다. Listing 20-16에서 만든 `Millimeters`와 `Meters` 타입과 달리, `Kilometers`는 별개의 새로운 타입이 아닙니다. `Kilometers` 타입을 가진 값들은 `i32` 타입의 값들과 동일하게 다루어집니다.

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

`Kilometers`와 `i32`는 같은 타입이므로, 두 타입의 값을 더할 수 있고 `Kilometers` 값을 `i32` 매개변수를 받는 함수에 전달할 수도 있습니다. 그러나 이 방법을 사용하면 앞에서 논의한 newtype 패턴에서 얻는 타입 검사의 이점을 얻지 못합니다. 다시 말해, 어딘가에서 `Kilometers`와 `i32` 값을 섞어 사용하면 컴파일러는 우리에게 오류를 주지 않을 것입니다.

타입 동의어의 주요 사용 사례는 반복을 줄이는 것입니다. 예를 들어, 다음과 같은 긴 타입을 가질 수 있습니다.

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

코드 곳곳에서 함수 시그니처와 타입 표기로 이 긴 타입을 작성하는 것은 지치고 오류가 나기 쉬울 수 있습니다. Listing 20-25와 같은 코드로 가득한 프로젝트를 상상해 보세요.

<Listing number="20-25" caption="긴 타입을 여러 곳에서 사용하기">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

타입 별칭은 반복을 줄여 이 코드를 더 관리하기 쉽게 만듭니다. Listing 20-26에서는 장황한 타입에 대한 `Thunk`라는 별칭을 도입했고, 그 타입의 모든 사용을 더 짧은 별칭 `Thunk`로 대체할 수 있습니다.

<Listing number="20-26" caption="반복을 줄이기 위해 타입 별칭 `Thunk` 도입">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

이 코드는 훨씬 읽고 쓰기 쉽습니다! 타입 별칭에 의미 있는 이름을 선택하면 의도를 전달하는 데도 도움이 됩니다(_thunk_ 는 나중에 평가될 코드를 가리키는 단어이므로, 저장되는 클로저에 적합한 이름입니다).

타입 별칭은 또한 반복을 줄이기 위해 `Result<T, E>` 타입과 함께 흔히 사용됩니다. 표준 라이브러리의 `std::io` 모듈을 봅시다. I/O 연산은 작업이 실패하는 상황을 처리하기 위해 종종 `Result<T, E>`를 반환합니다. 이 라이브러리에는 모든 가능한 I/O 오류를 나타내는 `std::io::Error` 구조체가 있습니다. `std::io`의 많은 함수들은 `Write` 트레이트의 다음 함수들처럼 `E`가 `std::io::Error`인 `Result<T, E>`를 반환할 것입니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>`가 많이 반복됩니다. 그래서 `std::io`에는 다음과 같은 타입 별칭 선언이 있습니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

이 선언이 `std::io` 모듈 안에 있으므로, 완전 한정 별칭 `std::io::Result<T>`를 사용할 수 있습니다. 즉, `E`가 `std::io::Error`로 채워진 `Result<T, E>`입니다. `Write` 트레이트 함수 시그니처는 결국 다음처럼 보이게 됩니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

타입 별칭은 두 가지 면에서 도움이 됩니다. 코드를 작성하기 더 쉽게 만들고 _그리고_ `std::io` 전반에 걸쳐 일관된 인터페이스를 줍니다. 그것은 별칭이므로 그저 또 하나의 `Result<T, E>`이며, 즉 `Result<T, E>`에 동작하는 어떤 메서드든 그것과 함께 사용할 수 있고, `?` 연산자 같은 특별한 문법도 사용할 수 있습니다.

### 결코 반환하지 않는 never 타입

러스트에는 타입 이론 용어로 _빈 타입(empty type)_ 으로 알려진 `!`라는 특별한 타입이 있는데, 값이 없기 때문입니다. 우리는 함수가 결코 반환하지 않을 때 반환 타입의 자리에 들어가므로 이를 _never 타입_ 이라고 부르는 것을 선호합니다. 다음은 한 예입니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

이 코드는 “함수 `bar`는 never를 반환한다”라고 읽습니다. never를 반환하는 함수는 _발산하는 함수(diverging functions)_ 라고 부릅니다. 우리는 `!` 타입의 값을 만들 수 없으므로 `bar`는 결코 반환할 수 없습니다.

그러나 결코 값을 만들 수 없는 타입이 무슨 쓸모가 있을까요? 숫자 맞추기 게임의 일부인 Listing 2-5의 코드를 떠올려 보세요. Listing 20-27에 일부를 다시 가져왔습니다.

<Listing number="20-27" caption="`continue`로 끝나는 갈래가 있는 `match`">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

당시에는 이 코드의 일부 세부 사항을 건너뛰었습니다. 6장의 [“`match` 제어 흐름 구문”][the-match-control-flow-construct]<!-- ignore --> 절에서, `match` 갈래들은 모두 같은 타입을 반환해야 한다고 논의했습니다. 그래서 예를 들어 다음 코드는 동작하지 않습니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

이 코드에서 `guess`의 타입은 정수 _이면서_ 문자열이어야 할 텐데, 러스트는 `guess`가 단 하나의 타입만 가지기를 요구합니다. 그러면 `continue`는 무엇을 반환할까요? Listing 20-27에서는 어떻게 한 갈래에서 `u32`를 반환하면서 다른 갈래는 `continue`로 끝낼 수 있었을까요?

짐작했을지도 모르지만, `continue`는 `!` 값을 가집니다. 즉, 러스트가 `guess`의 타입을 계산할 때, 두 매치 갈래를 모두 봅니다. 전자는 `u32` 값을, 후자는 `!` 값을 가집니다. `!`는 결코 값을 가질 수 없으므로, 러스트는 `guess`의 타입이 `u32`라고 결정합니다.

이 동작을 형식적으로 설명하는 방법은, `!` 타입의 표현식은 어떤 다른 타입으로도 강제 변환될 수 있다는 것입니다. `continue`는 값을 반환하지 않고 대신 제어를 반복문의 맨 위로 되돌리므로, 이 `match` 갈래를 `continue`로 끝낼 수 있습니다. 그래서 `Err` 경우에는 `guess`에 결코 값을 할당하지 않습니다.

never 타입은 `panic!` 매크로와 함께 쓰일 때도 유용합니다. 값을 만들거나 패닉하기 위해 `Option<T>` 값에 호출하는 `unwrap` 함수의 정의를 떠올려 보세요.

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

이 코드에서는 Listing 20-27의 `match`에서와 같은 일이 일어납니다. 러스트는 `val`이 타입 `T`를 가지고 `panic!`이 타입 `!`를 가짐을 봅니다. 그래서 전체 `match` 표현식의 결과는 `T`입니다. 이 코드가 동작하는 이유는 `panic!`이 값을 만들지 않기 때문입니다. 프로그램을 종료시킵니다. `None` 경우에는 `unwrap`에서 값을 반환하지 않을 것이므로, 이 코드는 유효합니다.

`!` 타입을 가지는 마지막 표현식은 `loop`입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

여기서 반복문은 결코 끝나지 않으므로 표현식의 값은 `!`입니다. 그러나 `break`를 포함했다면 그렇지 않을 것입니다. 반복문이 `break`에 도달했을 때 종료될 것이기 때문입니다.

### 동적 크기 타입과 `Sized` 트레이트

러스트는 특정 타입의 값에 얼마만큼의 공간을 할당할지와 같이, 자신의 타입에 대한 특정 세부 사항을 알아야 합니다. 이는 처음에는 타입 시스템의 한 구석을 약간 혼란스럽게 만듭니다. 바로 _동적 크기 타입(dynamically sized types)_ 이라는 개념입니다. 때로 _DST_ 또는 _크기 미정 타입(unsized types)_ 이라고도 부르는 이 타입들은, 크기를 런타임에만 알 수 있는 값을 사용해 코드를 작성하게 해 줍니다.

이 책 전반에 걸쳐 사용해 온 `str`이라는 동적 크기 타입의 세부 사항으로 들어가 봅시다. 그렇습니다, `&str`이 아니라 `str` 단독이 DST입니다. 사용자가 입력한 텍스트를 저장할 때처럼 많은 경우, 런타임이 되어야 문자열이 얼마나 긴지 알 수 있습니다. 즉, `str` 타입의 변수를 만들 수 없으며, `str` 타입의 인수를 받을 수도 없습니다. 동작하지 않는 다음 코드를 봅시다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

러스트는 특정 타입의 어떤 값에든 얼마만큼의 메모리를 할당할지 알아야 하며, 한 타입의 모든 값은 같은 양의 메모리를 사용해야 합니다. 만약 러스트가 이 코드를 작성하게 허용했다면, 이 두 `str` 값이 같은 양의 공간을 차지해야 했을 것입니다. 그러나 길이가 다릅니다. `s1`은 12바이트의 저장 공간이 필요하고 `s2`는 15바이트가 필요합니다. 이것이 동적 크기 타입을 담는 변수를 만들 수 없는 이유입니다.

그러면 어떻게 할까요? 이 경우에는 이미 답을 알고 있습니다. `s1`과 `s2`의 타입을 `str`이 아니라 문자열 슬라이스(`&str`)로 만듭니다. 4장의 [“문자열 슬라이스”][string-slices]<!-- ignore --> 절에서, 슬라이스 자료 구조가 슬라이스의 시작 위치와 길이만 저장한다는 것을 떠올려 보세요. 그래서 `&T`가 `T`가 위치한 메모리 주소를 저장하는 단일 값이라면, 문자열 슬라이스는 _두_ 값입니다. `str`의 주소와 그 길이입니다. 그런 만큼 컴파일 시점에 문자열 슬라이스 값의 크기를 알 수 있습니다. `usize` 길이의 두 배입니다. 즉, 문자열이 가리키는 문자열이 얼마나 길든, 우리는 항상 문자열 슬라이스의 크기를 압니다. 일반적으로 이것이 러스트에서 동적 크기 타입을 사용하는 방식입니다. 동적 정보의 크기를 저장하는 추가 메타데이터를 가집니다. 동적 크기 타입의 황금률은, 동적 크기 타입의 값을 항상 어떤 종류의 포인터 뒤에 두어야 한다는 것입니다.

`str`을 모든 종류의 포인터와 결합할 수 있습니다. 예를 들어 `Box<str>`이나 `Rc<str>`입니다. 사실 이 동적 크기 타입은 다른 동적 크기 타입에서도 본 적이 있습니다. 트레이트입니다. 모든 트레이트는 트레이트의 이름을 사용해 가리킬 수 있는 동적 크기 타입입니다. 18장의 [“공유된 동작을 추상화하기 위한 트레이트 객체 사용”][using-trait-objects-to-abstract-over-shared-behavior]<!-- ignore --> 절에서, 트레이트를 트레이트 객체로 사용하려면 `&dyn Trait`이나 `Box<dyn Trait>` 같은 포인터 뒤에 두어야 한다고 언급했습니다(`Rc<dyn Trait>`도 동작합니다).

DST를 다루기 위해 러스트는 어떤 타입의 크기가 컴파일 시점에 알려져 있는지 결정하는 `Sized` 트레이트를 제공합니다. 이 트레이트는 컴파일 시점에 크기가 알려진 모든 것에 자동으로 구현됩니다. 또한 러스트는 모든 제네릭 함수에 `Sized` 바운드를 암묵적으로 추가합니다. 즉, 다음과 같은 제네릭 함수 정의는

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

실제로는 다음과 같이 작성한 것처럼 다루어집니다.

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

기본적으로 제네릭 함수는 컴파일 시점에 크기가 알려진 타입에 대해서만 동작합니다. 그러나 다음 특별한 문법을 사용해 이 제한을 완화할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

`?Sized`에 대한 트레이트 바운드는 “`T`가 `Sized`일 수도 있고 아닐 수도 있다”는 의미이며, 이 표기는 제네릭 타입이 컴파일 시점에 알려진 크기를 가져야 한다는 기본 설정을 무효화합니다. 이러한 의미의 `?Trait` 문법은 `Sized`에 대해서만 사용할 수 있고, 다른 트레이트에는 사용할 수 없습니다.

또한 `t` 매개변수의 타입을 `T`에서 `&T`로 바꾸었음에 주목하세요. 타입이 `Sized`가 아닐 수 있으므로, 어떤 종류의 포인터 뒤에서 사용해야 합니다. 이 경우에는 참조를 선택했습니다.

다음으로 함수와 클로저에 대해 이야기하겠습니다!

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-construct]: ch06-02-match.html#the-match-control-flow-construct
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
