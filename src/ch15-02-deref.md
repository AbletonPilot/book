<!-- Old headings. Do not remove or links may break. -->

<a id="treating-smart-pointers-like-regular-references-with-the-deref-trait"></a>
<a id="treating-smart-pointers-like-regular-references-with-deref"></a>

## 스마트 포인터를 일반 참조처럼 다루기

`Deref` 트레이트를 구현하면 _역참조 연산자(dereference operator)_ `*`의
동작을 커스터마이즈할 수 있습니다(곱셈이나 글롭 연산자와 혼동하지 마세요).
스마트 포인터를 일반 참조처럼 다룰 수 있도록 `Deref`를 구현하면, 참조에 대해
동작하는 코드를 작성한 다음 그 코드를 스마트 포인터와도 함께 사용할 수 있습니다.

먼저 역참조 연산자가 일반 참조에 어떻게 동작하는지 살펴봅시다. 그 다음
`Box<T>`처럼 동작하는 커스텀 타입을 정의해 보면서, 새로 정의된 타입에서
역참조 연산자가 참조처럼 동작하지 않는 이유를 살펴봅니다. `Deref` 트레이트
구현이 스마트 포인터가 참조와 유사한 방식으로 동작할 수 있게 해 주는 방법을
탐구하겠습니다. 그런 다음 러스트의 역참조 강제(deref coercion) 기능과 참조나
스마트 포인터 모두와 함께 동작하게 해 주는 방법을 살펴봅니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>
<a id="following-the-pointer-to-the-value"></a>

### 참조를 따라 값으로 가기

일반 참조는 포인터의 한 종류이며, 포인터를 다른 어딘가에 저장된 값으로 가는
화살표로 생각할 수 있습니다. Listing 15-6에서는 `i32` 값에 대한 참조를 만들고
역참조 연산자로 참조를 따라 값으로 갑니다.

<Listing number="15-6" file-name="src/main.rs" caption="역참조 연산자로 `i32` 값에 대한 참조 따라가기">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

변수 `x`는 `i32` 값 `5`를 가집니다. `y`를 `x`에 대한 참조와 같게 설정합니다.
`x`가 `5`와 같음을 단언할 수 있습니다. 그러나 `y`의 값에 대해 단언하려면,
컴파일러가 실제 값을 비교할 수 있도록 `*y`를 사용해 참조를 따라 그것이 가리키는
값으로 가야 합니다(그래서 _역참조(dereference)_ 입니다). `y`를 역참조하면
`y`가 가리키는 정수 값에 접근할 수 있고 그것을 `5`와 비교할 수 있습니다.

대신 `assert_eq!(5, y);`를 쓰려 하면 다음 컴파일 오류가 납니다.

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

숫자와 숫자에 대한 참조를 비교하는 것은 서로 다른 타입이므로 허용되지 않습니다.
참조가 가리키는 값으로 가려면 역참조 연산자를 사용해야 합니다.

### `Box<T>`를 참조처럼 사용하기

Listing 15-6의 코드를 참조 대신 `Box<T>`를 사용하도록 다시 쓸 수 있습니다.
Listing 15-7에서 `Box<T>`에 사용된 역참조 연산자는 Listing 15-6에서 참조에
사용된 역참조 연산자와 같은 방식으로 동작합니다.

<Listing number="15-7" file-name="src/main.rs" caption="`Box<i32>`에서 역참조 연산자 사용하기">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

Listing 15-7과 Listing 15-6의 주된 차이는, 여기서는 `y`를 `x` 값에 대한 참조가
아니라 `x`의 복사된 값을 가리키는 박스의 인스턴스로 설정한다는 점입니다.
마지막 단언에서는 `y`가 참조였을 때와 같은 방식으로 박스의 포인터를 따라가기
위해 역참조 연산자를 사용할 수 있습니다. 다음으로, 우리가 직접 박스 타입을
정의해 봄으로써 역참조 연산자를 사용할 수 있게 해 주는 `Box<T>`의 특별한
점이 무엇인지 탐구하겠습니다.

### 직접 스마트 포인터 정의하기

표준 라이브러리가 제공하는 `Box<T>` 타입과 비슷한 래퍼 타입을 만들어, 스마트
포인터 타입이 기본적으로 참조와 어떻게 다르게 동작하는지 경험해 봅시다. 그런
다음 역참조 연산자를 사용할 수 있는 능력을 어떻게 추가하는지 살펴봅니다.

> 참고: 곧 만들 `MyBox<T>` 타입과 실제 `Box<T>` 사이에는 한 가지 큰 차이가
> 있습니다. 우리 버전은 데이터를 힙에 저장하지 않습니다. 이 예제는 `Deref`에
> 집중하고 있으므로, 데이터가 실제로 어디에 저장되는지는 포인터 같은 동작보다
> 덜 중요합니다.

`Box<T>` 타입은 결국 요소가 하나인 튜플 구조체로 정의되므로, Listing 15-8은
`MyBox<T>` 타입을 같은 방식으로 정의합니다. `Box<T>`에 정의된 `new` 함수와
맞도록 `new` 함수도 정의하겠습니다.

<Listing number="15-8" file-name="src/main.rs" caption="`MyBox<T>` 타입 정의하기">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

`MyBox`라는 구조체를 정의하고, 어떤 타입의 값도 담을 수 있기를 원하므로 제네릭
매개변수 `T`를 선언합니다. `MyBox` 타입은 타입 `T`의 요소 하나를 가진 튜플
구조체입니다. `MyBox::new` 함수는 타입 `T`의 매개변수 하나를 받아, 전달된 값을
담은 `MyBox` 인스턴스를 반환합니다.

Listing 15-7의 `main` 함수를 Listing 15-8에 추가하고, `Box<T>` 대신 우리가
정의한 `MyBox<T>` 타입을 사용하도록 바꿔 봅시다. Listing 15-9의 코드는 러스트
가 `MyBox`를 역참조하는 법을 모르기 때문에 컴파일되지 않습니다.

<Listing number="15-9" file-name="src/main.rs" caption="참조 및 `Box<T>`와 같은 방식으로 `MyBox<T>`를 사용하려는 시도">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

결과 컴파일 오류는 다음과 같습니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

우리 `MyBox<T>` 타입은 그 타입에 역참조 능력을 구현하지 않았기 때문에 역참조될
수 없습니다. `*` 연산자로 역참조를 가능하게 하려면 `Deref` 트레이트를 구현합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="treating-a-type-like-a-reference-by-implementing-the-deref-trait"></a>

### `Deref` 트레이트 구현하기

10장의 [“타입에 트레이트 구현하기”][impl-trait]<!-- ignore -->에서 논의했듯이
트레이트를 구현하려면 그 트레이트의 필수 메서드 구현을 제공해야 합니다.
표준 라이브러리가 제공하는 `Deref` 트레이트는 `self`를 빌리고 내부 데이터에
대한 참조를 반환하는 `deref`라는 메서드 하나를 구현하도록 요구합니다. Listing
15-10은 `MyBox<T>`의 정의에 추가할 `Deref` 구현을 담고 있습니다.

<Listing number="15-10" file-name="src/main.rs" caption="`MyBox<T>`에 `Deref` 구현하기">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

`type Target = T;` 문법은 `Deref` 트레이트가 사용할 연관 타입을 정의합니다.
연관 타입은 제네릭 매개변수를 선언하는 약간 다른 방식이지만, 지금은 그에 대해
걱정하지 않아도 됩니다. 20장에서 더 자세히 다룹니다.

`deref` 메서드의 본문을 `&self.0`으로 채워, `deref`가 `*` 연산자로 접근하고
싶은 값에 대한 참조를 반환하게 합니다. 5장의 [“튜플 구조체로 서로 다른 타입
만들기”][tuple-structs]<!-- ignore -->에서 `.0`이 튜플 구조체의 첫 번째 값에
접근함을 떠올려 보세요. Listing 15-9에서 `MyBox<T>` 값에 `*`를 호출하는
`main` 함수는 이제 컴파일되고, 단언도 통과합니다!

`Deref` 트레이트가 없다면 컴파일러는 `&` 참조만 역참조할 수 있습니다. `deref`
메서드는 컴파일러에게 `Deref`를 구현한 어떤 타입의 값이든 받아서, `deref`
메서드를 호출해 자신이 역참조할 수 있는 참조를 얻을 수 있는 능력을 줍니다.

Listing 15-9에서 `*y`를 입력했을 때, 내부적으로 러스트는 실제로 다음 코드를
실행했습니다.

```rust,ignore
*(y.deref())
```

러스트는 `*` 연산자를 `deref` 메서드 호출과 그 다음 평범한 역참조로 대체해서,
`deref` 메서드를 호출해야 하는지 여부를 우리가 생각하지 않아도 되게 합니다.
이 러스트 기능 덕분에 우리는 일반 참조를 가지든 `Deref`를 구현한 타입을
가지든 동일하게 동작하는 코드를 작성할 수 있습니다.

`deref` 메서드가 값에 대한 참조를 반환하고, `*(y.deref())`의 괄호 밖 평범한
역참조가 여전히 필요한 이유는 소유권 시스템과 관련이 있습니다. 만약 `deref`
메서드가 참조 대신 값 자체를 반환했다면, 값이 `self`에서 이동되었을 것입니다.
이 경우나 역참조 연산자를 사용하는 대부분의 경우, 우리는 `MyBox<T>` 내부의
값의 소유권을 가져가고 싶지 않습니다.

코드에서 `*`을 사용할 때마다 `*` 연산자가 `deref` 메서드 호출과 `*` 연산자
호출로 딱 한 번만 대체된다는 점에 유의하세요. `*` 연산자의 대체는 무한히
재귀하지 않으므로, 결국 `i32` 타입의 데이터가 나오고, 이는 Listing 15-9의
`assert_eq!`에 있는 `5`와 일치합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="implicit-deref-coercions-with-functions-and-methods"></a>
<a id="using-deref-coercions-in-functions-and-methods"></a>

### 함수와 메서드에서 역참조 강제 사용하기

_역참조 강제(deref coercion)_ 는 `Deref` 트레이트를 구현하는 타입에 대한 참조를
다른 타입에 대한 참조로 변환합니다. 예를 들어 역참조 강제는 `&String`을
`&str`로 변환할 수 있는데, `String`이 `&str`를 반환하는 방식으로 `Deref`
트레이트를 구현하기 때문입니다. 역참조 강제는 러스트가 함수와 메서드의 인수에
대해 수행하는 편의 기능이며, `Deref` 트레이트를 구현하는 타입에서만 동작합니다.
특정 타입의 값에 대한 참조를 함수 또는 메서드 정의의 매개변수 타입과 일치하지
않는 인수로 함수나 메서드에 전달할 때 자동으로 일어납니다. 일련의 `deref`
메서드 호출이 우리가 제공한 타입을 매개변수가 필요로 하는 타입으로 변환합니다.

역참조 강제는 함수 및 메서드 호출을 작성하는 프로그래머가 `&`와 `*`로 많은
명시적 참조와 역참조를 추가하지 않아도 되도록 러스트에 추가되었습니다. 역참조
강제 기능은 또한 참조나 스마트 포인터 모두에 대해 동작하는 더 많은 코드를
작성할 수 있게 해 줍니다.

역참조 강제가 동작하는 모습을 보기 위해, Listing 15-8에서 정의한 `MyBox<T>`
타입과 Listing 15-10에서 추가한 `Deref` 구현을 사용하겠습니다. Listing 15-11
은 문자열 슬라이스 매개변수를 가진 함수의 정의를 보여 줍니다.

<Listing number="15-11" file-name="src/main.rs" caption="`&str` 타입 매개변수 `name`을 가진 `hello` 함수">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

예를 들어 `hello("Rust");`처럼 문자열 슬라이스를 인수로 `hello` 함수를 호출할
수 있습니다. 역참조 강제 덕분에 Listing 15-12처럼 `MyBox<String>` 타입 값에
대한 참조로 `hello`를 호출할 수 있습니다.

<Listing number="15-12" file-name="src/main.rs" caption="역참조 강제 덕분에 동작하는, `MyBox<String>` 값에 대한 참조로 `hello` 호출하기">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

여기서 우리는 `MyBox<String>` 값에 대한 참조인 `&m`을 인수로 `hello` 함수를
호출합니다. Listing 15-10에서 `MyBox<T>`에 `Deref` 트레이트를 구현했으므로,
러스트는 `deref`를 호출해 `&MyBox<String>`을 `&String`으로 바꿀 수 있습니다.
표준 라이브러리는 `String`에 문자열 슬라이스를 반환하는 `Deref` 구현을
제공하는데, 이는 `Deref`의 API 문서에 나와 있습니다. 러스트는 다시 `deref`를
호출해 `&String`을 `&str`로 바꾸고, 이는 `hello` 함수의 정의와 일치합니다.

러스트가 역참조 강제를 구현하지 않았다면, `&MyBox<String>` 타입의 값으로
`hello`를 호출하려면 Listing 15-12 대신 Listing 15-13의 코드를 작성해야 했을
것입니다.

<Listing number="15-13" file-name="src/main.rs" caption="러스트에 역참조 강제가 없었다면 작성해야 했을 코드">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

`(*m)`은 `MyBox<String>`을 `String`으로 역참조합니다. 그런 다음 `&`와 `[..]`
가 `hello`의 시그니처와 맞도록 문자열 전체에 해당하는 `String`의 문자열
슬라이스를 취합니다. 역참조 강제 없는 이 코드는 이 모든 기호가 관여해서 읽고,
쓰고, 이해하기 더 어렵습니다. 역참조 강제는 러스트가 이 변환을 자동으로
처리할 수 있게 해 줍니다.

관련된 타입에 대해 `Deref` 트레이트가 정의되어 있으면, 러스트는 타입을 분석해
`Deref::deref`를 매개변수의 타입과 맞는 참조를 얻기 위해 필요한 만큼 호출
합니다. `Deref::deref`를 삽입해야 하는 횟수는 컴파일 타임에 해결되므로, 역참조
강제를 활용하는 데 런타임 벌점은 없습니다!

<!-- Old headings. Do not remove or links may break. -->

<a id="how-deref-coercion-interacts-with-mutability"></a>

### 가변 참조에서의 역참조 강제 처리

`Deref` 트레이트를 사용해 불변 참조에서 `*` 연산자를 재정의하는 것처럼,
`DerefMut` 트레이트를 사용해 가변 참조에서 `*` 연산자를 재정의할 수 있습니다.

러스트는 다음 세 가지 경우에 타입과 트레이트 구현을 찾을 때 역참조 강제를
수행합니다.

1. `T: Deref<Target=U>`일 때 `&T`에서 `&U`로
2. `T: DerefMut<Target=U>`일 때 `&mut T`에서 `&mut U`로
3. `T: Deref<Target=U>`일 때 `&mut T`에서 `&U`로

처음 두 경우는 두 번째가 가변성을 구현한다는 점만 빼면 같습니다. 첫 번째 경우
는 `&T`가 있고 `T`가 어떤 타입 `U`에 대해 `Deref`를 구현한다면, 투명하게
`&U`를 얻을 수 있음을 말합니다. 두 번째 경우는 같은 역참조 강제가 가변 참조에
대해 일어남을 말합니다.

세 번째 경우는 더 까다롭습니다. 러스트는 가변 참조도 불변 참조로 강제
변환합니다. 그러나 그 반대는 _가능하지 않습니다_. 불변 참조는 절대 가변 참조로
강제 변환되지 않습니다. 빌림 규칙 때문에 가변 참조가 있다면 그 가변 참조가
그 데이터에 대한 유일한 참조여야 하기 때문입니다(그렇지 않으면 프로그램이
컴파일되지 않습니다). 하나의 가변 참조를 하나의 불변 참조로 변환하는 것은
빌림 규칙을 결코 깨뜨리지 않습니다. 불변 참조를 가변 참조로 변환하려면 최초의
불변 참조가 그 데이터에 대한 유일한 불변 참조여야 하지만, 빌림 규칙은 그것을
보장하지 않습니다. 따라서 러스트는 불변 참조를 가변 참조로 변환하는 것이
가능하다는 가정을 할 수 없습니다.

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
