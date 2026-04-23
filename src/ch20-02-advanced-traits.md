## 고급 트레이트

10장의 [“트레이트로 공통 동작 정의하기”][traits]<!-- ignore --> 절에서 트레이트를 처음 다루었지만, 더 고급 세부 사항은 논의하지 않았습니다. 이제 러스트에 대해 더 많이 알게 되었으니 세세한 내용으로 들어갈 수 있습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>
<a id="associated-types"></a>

### 연관 타입으로 트레이트 정의하기

_연관 타입(associated types)_ 은 타입 자리표시자를 트레이트와 연결하여, 트레이트 메서드 정의가 시그니처에 이 자리표시자 타입을 사용할 수 있게 해 줍니다. 트레이트 구현자는 특정 구현에서 자리표시자 타입 대신 사용할 구체 타입을 명시할 것입니다. 이 방식으로, 트레이트가 구현될 때까지 그 타입이 정확히 무엇인지 알 필요 없이 어떤 타입들을 사용하는 트레이트를 정의할 수 있습니다.

이 장에서 다루는 대부분의 고급 기능들이 거의 필요 없다고 설명해 왔습니다. 연관 타입은 그 중간쯤에 있습니다. 책의 나머지 부분에서 설명한 기능들보다는 더 드물게 사용되지만, 이 장에서 논의되는 다른 많은 기능들보다는 더 흔히 사용됩니다.

연관 타입을 가진 트레이트의 한 예는 표준 라이브러리가 제공하는 `Iterator` 트레이트입니다. 연관 타입의 이름은 `Item`이며, `Iterator` 트레이트를 구현하는 타입이 순회하는 값들의 타입을 대신합니다. `Iterator` 트레이트의 정의는 Listing 20-13과 같습니다.

<Listing number="20-13" caption="연관 타입 `Item`을 가진 `Iterator` 트레이트의 정의">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

타입 `Item`은 자리표시자이고, `next` 메서드의 정의는 그것이 `Option<Self::Item>` 타입의 값을 반환하리라는 것을 보여 줍니다. `Iterator` 트레이트의 구현자들은 `Item`에 대한 구체 타입을 명시하고, `next` 메서드는 그 구체 타입의 값을 담은 `Option`을 반환할 것입니다.

연관 타입은 제네릭이 어떤 타입을 다룰 수 있는지 명시하지 않고 함수를 정의할 수 있게 해 준다는 점에서 제네릭과 비슷한 개념처럼 보일 수 있습니다. 두 개념의 차이를 살펴보기 위해, `Item` 타입이 `u32`임을 명시한 `Counter`라는 타입에 대한 `Iterator` 트레이트의 구현을 살펴봅시다.

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

이 문법은 제네릭의 문법과 비교할 만해 보입니다. 그렇다면 왜 Listing 20-14에서처럼 `Iterator` 트레이트를 그냥 제네릭으로 정의하지 않는 걸까요?

<Listing number="20-14" caption="제네릭을 사용한 가상의 `Iterator` 트레이트 정의">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

차이는, Listing 20-14처럼 제네릭을 사용할 때는 각 구현에서 타입을 표기해야 한다는 점입니다. 우리는 또한 `Iterator<String> for Counter`나 어떤 다른 타입을 구현할 수도 있으므로, `Counter`에 대한 `Iterator`의 여러 구현을 가질 수 있습니다. 다시 말해, 트레이트가 제네릭 매개변수를 가지면, 그 트레이트는 매번 제네릭 타입 매개변수의 구체 타입을 바꾸어 가며 한 타입에 대해 여러 번 구현될 수 있습니다. `Counter`에 `next` 메서드를 사용할 때, 어떤 `Iterator` 구현을 사용하고 싶은지 나타내기 위해 타입 표기를 제공해야 할 것입니다.

연관 타입을 사용하면 타입을 표기할 필요가 없습니다. 한 타입에 대해 트레이트를 여러 번 구현할 수 없기 때문입니다. 연관 타입을 사용하는 정의가 있는 Listing 20-13에서, `impl Iterator for Counter`는 단 하나만 있을 수 있으므로 `Item`의 타입을 단 한 번만 선택할 수 있습니다. `Counter`에서 `next`를 호출하는 모든 곳에서 `u32` 값의 이터레이터를 원한다고 명시할 필요가 없습니다.

연관 타입은 또한 트레이트의 계약의 일부가 됩니다. 트레이트의 구현자는 연관 타입 자리표시자를 대신할 타입을 제공해야 합니다. 연관 타입은 종종 그 타입이 어떻게 사용될지를 설명하는 이름을 가지며, API 문서에 연관 타입을 문서화하는 것은 좋은 관행입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-generic-type-parameters-and-operator-overloading"></a>

### 기본 제네릭 매개변수와 연산자 오버로딩 사용

제네릭 타입 매개변수를 사용할 때, 제네릭 타입에 대한 기본 구체 타입을 명시할 수 있습니다. 이는 기본 타입이 동작한다면 트레이트의 구현자가 구체 타입을 명시할 필요가 없게 해 줍니다. 제네릭 타입을 선언할 때 `<PlaceholderType=ConcreteType>` 문법으로 기본 타입을 명시합니다.

이 기법이 유용한 상황의 좋은 예는 _연산자 오버로딩(operator overloading)_ 입니다. 연산자 오버로딩에서는 특정 상황에서 (`+` 같은) 연산자의 동작을 사용자 정의합니다.

러스트는 직접 연산자를 만들거나 임의의 연산자를 오버로딩하는 것을 허용하지 않습니다. 그러나 `std::ops`에 나열된 연산과 그에 대응하는 트레이트를, 연산자에 연관된 트레이트를 구현함으로써 오버로딩할 수 있습니다. 예를 들어, Listing 20-15에서 두 `Point` 인스턴스를 더하기 위해 `+` 연산자를 오버로딩합니다. `Point` 구조체에 `Add` 트레이트를 구현해 이를 합니다.

<Listing number="20-15" file-name="src/main.rs" caption="`Point` 인스턴스에 대해 `+` 연산자를 오버로딩하기 위해 `Add` 트레이트 구현하기">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

`add` 메서드는 두 `Point` 인스턴스의 `x` 값과 두 `Point` 인스턴스의 `y` 값을 더해 새 `Point`를 만듭니다. `Add` 트레이트는 `add` 메서드에서 반환되는 타입을 결정하는 `Output`이라는 연관 타입을 가집니다.

이 코드에서 기본 제네릭 타입은 `Add` 트레이트 안에 있습니다. 다음은 그 정의입니다.

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

이 코드는 대체로 친숙해 보일 것입니다. 메서드 하나와 연관 타입 하나를 가진 트레이트입니다. 새로운 부분은 `Rhs=Self`입니다. 이 문법은 _기본 타입 매개변수(default type parameters)_ 라고 부릅니다. (“right-hand side”의 줄임인) `Rhs` 제네릭 타입 매개변수는 `add` 메서드에서 `rhs` 매개변수의 타입을 정의합니다. `Add` 트레이트를 구현할 때 `Rhs`에 대한 구체 타입을 지정하지 않으면, `Rhs`의 타입은 `Self`로 기본 설정되며, 이는 우리가 `Add`를 구현하는 타입이 됩니다.

`Point`에 대해 `Add`를 구현했을 때 `Rhs`의 기본값을 사용했는데, 두 `Point` 인스턴스를 더하고 싶었기 때문입니다. 기본값을 사용하지 않고 `Rhs` 타입을 사용자 정의하고 싶은 `Add` 트레이트 구현 예를 살펴봅시다.

서로 다른 단위로 값을 담는 두 구조체 `Millimeters`와 `Meters`가 있습니다. 기존 타입을 다른 구조체로 얇게 감싸는 이런 방식을 _newtype 패턴_ 이라고 알려져 있는데, 이는 [“newtype 패턴으로 외부 트레이트 구현하기”][newtype]<!-- ignore --> 절에서 더 자세히 설명합니다. 밀리미터 값을 미터 값에 더하고 `Add`의 구현이 변환을 올바르게 수행하기를 원합니다. Listing 20-16에서 보이듯, `Meters`를 `Rhs`로 사용해 `Millimeters`에 대한 `Add`를 구현할 수 있습니다.

<Listing number="20-16" file-name="src/lib.rs" caption="`Millimeters`와 `Meters`를 더하기 위해 `Millimeters`에 `Add` 트레이트 구현하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

`Millimeters`와 `Meters`를 더하기 위해, `Self`의 기본값을 사용하는 대신 `impl Add<Meters>`를 명시해 `Rhs` 타입 매개변수의 값을 설정합니다.

기본 타입 매개변수는 주로 두 가지 방식으로 사용됩니다.

1. 기존 코드를 깨지 않고 타입을 확장하기 위해
2. 대부분의 사용자가 필요로 하지 않을 특정 경우에 사용자 정의를 허용하기 위해

표준 라이브러리의 `Add` 트레이트는 두 번째 목적의 예입니다. 보통은 같은 종류의 두 타입을 더하지만, `Add` 트레이트는 그 이상으로 사용자 정의할 수 있는 능력을 제공합니다. `Add` 트레이트 정의에 기본 타입 매개변수를 사용한다는 것은 대부분의 경우 추가 매개변수를 지정할 필요가 없다는 뜻입니다. 다시 말해, 약간의 구현 보일러플레이트가 필요하지 않아 트레이트를 사용하기 더 쉬워집니다.

첫 번째 목적은 두 번째와 비슷하지만 반대 방향입니다. 기존 트레이트에 타입 매개변수를 추가하고 싶다면, 그것에 기본값을 줘 기존 구현 코드를 깨지 않고 트레이트의 기능을 확장할 수 있습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>
<a id="disambiguating-between-methods-with-the-same-name"></a>

### 같은 이름의 메서드 모호성 해소

러스트에서는 한 트레이트가 다른 트레이트의 메서드와 같은 이름의 메서드를 갖는 것을 막지 않으며, 한 타입에 두 트레이트 모두를 구현하는 것도 막지 않습니다. 트레이트의 메서드와 같은 이름의 메서드를 타입에 직접 구현하는 것도 가능합니다.

같은 이름의 메서드를 호출할 때, 어느 것을 사용하고 싶은지 러스트에 알려야 합니다. Listing 20-17의 코드를 봅시다. `Pilot`과 `Wizard`라는 두 트레이트를 정의했고, 둘 다 `fly`라는 메서드를 가지고 있습니다. 그런 다음 이미 `fly`라는 메서드가 직접 구현되어 있는 `Human` 타입에 두 트레이트 모두를 구현합니다. 각 `fly` 메서드는 다른 일을 합니다.

<Listing number="20-17" file-name="src/main.rs" caption="`fly` 메서드를 가지도록 두 트레이트가 정의되어 `Human` 타입에 구현되고, `Human`에 직접 `fly` 메서드도 구현됩니다.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

`Human` 인스턴스에서 `fly`를 호출하면, Listing 20-18에서 보이듯 컴파일러는 기본적으로 그 타입에 직접 구현된 메서드를 호출합니다.

<Listing number="20-18" file-name="src/main.rs" caption="`Human` 인스턴스에서 `fly` 호출하기">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

이 코드를 실행하면 `*waving arms furiously*`가 출력되며, 러스트가 `Human`에 직접 구현된 `fly` 메서드를 호출했음을 보여 줍니다.

`Pilot` 트레이트나 `Wizard` 트레이트의 `fly` 메서드를 호출하려면, 어느 `fly` 메서드를 의미하는지를 명시하기 위해 더 명시적인 문법을 사용해야 합니다. Listing 20-19는 이 문법을 보여 줍니다.

<Listing number="20-19" file-name="src/main.rs" caption="어느 트레이트의 `fly` 메서드를 호출할지 명시하기">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

메서드 이름 앞에 트레이트 이름을 명시하면 어느 `fly` 구현을 호출할지 러스트에 분명히 합니다. `Human::fly(&person)`이라고도 쓸 수 있는데, 이는 Listing 20-19에서 사용한 `person.fly()`와 같습니다. 다만 모호성을 해소할 필요가 없다면 이쪽이 좀 더 길어 작성하기 번거롭습니다.

이 코드를 실행하면 다음을 출력합니다.

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

`fly` 메서드는 `self` 매개변수를 받으므로, 두 _타입_ 이 같은 _트레이트_ 를 구현한다면, 러스트는 `self`의 타입에 따라 어느 트레이트 구현을 사용할지 알아낼 수 있습니다.

그러나 메서드가 아닌 연관 함수에는 `self` 매개변수가 없습니다. 같은 함수 이름을 가진 메서드가 아닌 함수를 정의하는 여러 타입이나 트레이트가 있을 때, 완전 한정 문법을 사용하지 않으면 러스트는 어느 타입을 의미하는지 항상 알 수는 없습니다. 예를 들어, Listing 20-20에서는 모든 강아지 이름을 Spot이라고 짓고 싶어 하는 동물 보호소를 위한 트레이트를 만듭니다. 메서드가 아닌 연관 함수 `baby_name`을 가진 `Animal` 트레이트를 만듭니다. `Animal` 트레이트는 `Dog` 구조체에 구현되어 있고, `Dog`에는 메서드가 아닌 연관 함수 `baby_name`도 직접 제공합니다.

<Listing number="20-20" file-name="src/main.rs" caption="연관 함수가 있는 트레이트와 같은 이름의 연관 함수를 가지면서 그 트레이트를 구현하기도 한 타입">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

모든 강아지의 이름을 Spot으로 짓는 코드를 `Dog`에 정의된 `baby_name` 연관 함수에 구현합니다. `Dog` 타입은 또한 모든 동물이 가지는 특성을 설명하는 트레이트 `Animal`을 구현합니다. 새끼 개는 강아지(puppy)라고 부르며, 이는 `Animal` 트레이트와 연관된 `baby_name` 함수의 `Dog`에 대한 `Animal` 트레이트 구현에 표현되어 있습니다.

`main`에서는 `Dog::baby_name` 함수를 호출하는데, 이는 `Dog`에 직접 정의된 연관 함수를 호출합니다. 이 코드는 다음을 출력합니다.

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

이 출력은 우리가 원한 것이 아닙니다. 우리는 `Dog`에 구현한 `Animal` 트레이트의 일부인 `baby_name` 함수를 호출해 코드가 `A baby dog is called a puppy`를 출력하기를 원합니다. Listing 20-19에서 사용한 트레이트 이름을 명시하는 기법은 여기서는 도움이 되지 않습니다. `main`을 Listing 20-21의 코드로 바꾸면 컴파일 오류를 받게 됩니다.

<Listing number="20-21" file-name="src/main.rs" caption="`Animal` 트레이트의 `baby_name` 함수를 호출하려고 시도하지만, 러스트가 어느 구현을 사용해야 할지 모릅니다">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

`Animal::baby_name`은 `self` 매개변수가 없고, `Animal` 트레이트를 구현하는 다른 타입이 있을 수도 있으므로, 러스트는 어느 `Animal::baby_name` 구현을 원하는지 알아낼 수 없습니다. 다음과 같은 컴파일러 오류를 받게 됩니다.

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

모호성을 해소하고 다른 타입이 아닌 `Dog`에 대한 `Animal`의 구현을 사용하고 싶다고 러스트에 알리려면, 완전 한정 문법(fully qualified syntax)을 사용해야 합니다. Listing 20-22는 완전 한정 문법을 어떻게 사용하는지 보여 줍니다.

<Listing number="20-22" file-name="src/main.rs" caption="`Dog`에 구현된 `Animal` 트레이트의 `baby_name` 함수를 호출하고자 함을 명시하기 위해 완전 한정 문법 사용하기">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

각괄호 안에 타입 표기를 제공하여, 이 함수 호출에 대해 `Dog` 타입을 `Animal`로 취급하고 싶다는 것을 말함으로써 `Dog`에 구현된 `Animal` 트레이트의 `baby_name` 메서드를 호출하고 싶다는 것을 러스트에 알립니다. 이제 이 코드는 우리가 원하는 것을 출력합니다.

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

일반적으로 완전 한정 문법은 다음과 같이 정의됩니다.

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

메서드가 아닌 연관 함수의 경우 `receiver`가 없습니다. 다른 인수들의 목록만 있을 것입니다. 함수나 메서드를 호출하는 모든 곳에서 완전 한정 문법을 사용할 수 있습니다. 그러나 러스트가 프로그램의 다른 정보로부터 알아낼 수 있는 이 문법의 어떤 부분이라도 생략할 수 있습니다. 같은 이름을 사용하는 여러 구현이 있고 어느 구현을 호출하고 싶은지 식별하는 데 러스트의 도움이 필요할 때만 이 더 장황한 문법을 사용하면 됩니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### 슈퍼트레이트 사용

때로는 다른 트레이트에 의존하는 트레이트 정의를 작성할 수 있습니다. 어떤 타입이 첫 번째 트레이트를 구현하기 위해 그 타입이 두 번째 트레이트도 구현하도록 요구하고 싶을 때입니다. 그렇게 하면 트레이트 정의가 두 번째 트레이트의 연관 항목을 활용할 수 있습니다. 트레이트 정의가 의존하는 트레이트는 그 트레이트의 _슈퍼트레이트(supertrait)_ 라고 부릅니다.

예를 들어, 주어진 값을 별표로 둘러싸 출력하도록 포맷한 `outline_print` 메서드를 가진 `OutlinePrint` 트레이트를 만들고 싶다고 합시다. 즉, `(x, y)`로 결과가 나오도록 표준 라이브러리 트레이트 `Display`를 구현하는 `Point` 구조체가 있을 때, `x`가 `1`이고 `y`가 `3`인 `Point` 인스턴스에서 `outline_print`를 호출하면 다음을 출력해야 합니다.

```text
**********
*        *
* (1, 3) *
*        *
**********
```

`outline_print` 메서드의 구현에서 `Display` 트레이트의 기능을 사용하고 싶습니다. 따라서 `OutlinePrint` 트레이트가 `Display`도 구현하면서 `OutlinePrint`가 필요로 하는 기능을 제공하는 타입에 대해서만 동작할 것임을 명시해야 합니다. 트레이트 정의에서 `OutlinePrint: Display`를 명시함으로써 그렇게 할 수 있습니다. 이 기법은 트레이트에 트레이트 바운드를 추가하는 것과 비슷합니다. Listing 20-23은 `OutlinePrint` 트레이트의 구현을 보여 줍니다.

<Listing number="20-23" file-name="src/main.rs" caption="`Display`의 기능을 요구하는 `OutlinePrint` 트레이트 구현하기">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

`OutlinePrint`가 `Display` 트레이트를 요구한다고 명시했으므로, `Display`를 구현하는 어떤 타입에 대해서든 자동으로 구현되는 `to_string` 함수를 사용할 수 있습니다. 트레이트 이름 뒤에 콜론과 `Display` 트레이트를 명시하지 않고 `to_string`을 사용하려고 하면, 현재 스코프에서 `&Self` 타입에 대해 `to_string`이라는 이름의 메서드를 찾을 수 없다는 오류를 받게 됩니다.

`Display`를 구현하지 않는 타입, 예컨대 `Point` 구조체에 `OutlinePrint`를 구현하려고 시도하면 어떤 일이 일어나는지 봅시다.

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

`Display`가 필요하지만 구현되지 않았다는 오류를 받습니다.

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

이를 고치려면, `Point`에 `Display`를 구현하고 `OutlinePrint`가 요구하는 제약을 만족시킵니다.

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

그러면 `Point`에 `OutlinePrint` 트레이트를 구현하는 것이 성공적으로 컴파일될 것이고, `Point` 인스턴스에 `outline_print`를 호출해 별표 윤곽 안에 표시할 수 있습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a>
<a id="using-the-newtype-pattern-to-implement-external-traits"></a>

### newtype 패턴으로 외부 트레이트 구현

10장의 [“타입에 트레이트 구현하기”][implementing-a-trait-on-a-type]<!-- ignore --> 절에서, 트레이트 또는 타입(또는 둘 다)이 우리 크레이트 로컬일 때만 그 타입에 트레이트를 구현할 수 있다는 고아 규칙(orphan rule)을 언급했습니다. 튜플 구조체에 새로운 타입을 만드는 newtype 패턴을 사용하면 이 제한을 우회할 수 있습니다. (튜플 구조체는 5장의 [“튜플 구조체로 다른 타입 만들기”][tuple-structs]<!-- ignore --> 절에서 다루었습니다.) 튜플 구조체는 필드 하나를 가지며, 트레이트를 구현하고자 하는 타입을 얇게 감싸는 래퍼가 될 것입니다. 그러면 래퍼 타입은 우리 크레이트 로컬이 되어, 래퍼에 트레이트를 구현할 수 있습니다. _Newtype_ 은 Haskell 프로그래밍 언어에서 유래한 용어입니다. 이 패턴을 사용해도 런타임 성능 페널티는 없으며, 래퍼 타입은 컴파일 시점에 사라집니다.

예시로, `Vec<T>`에 `Display`를 구현하고 싶다고 해 봅시다. `Display` 트레이트와 `Vec<T>` 타입이 우리 크레이트 바깥에서 정의되었으므로 고아 규칙은 우리가 직접 그렇게 하는 것을 막습니다. `Vec<T>` 인스턴스를 담는 `Wrapper` 구조체를 만들고, Listing 20-24에서 보이듯 `Wrapper`에 `Display`를 구현하고 `Vec<T>` 값을 사용할 수 있습니다.

<Listing number="20-24" file-name="src/main.rs" caption="`Display`를 구현하기 위해 `Vec<String>` 주위에 `Wrapper` 타입 만들기">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

`Wrapper`가 튜플 구조체이고 `Vec<T>`가 튜플의 인덱스 0번 항목이므로, `Display`의 구현은 내부 `Vec<T>`에 접근하기 위해 `self.0`을 사용합니다. 그러면 `Wrapper`에서 `Display` 트레이트의 기능을 사용할 수 있습니다.

이 기법을 사용하는 단점은 `Wrapper`가 새 타입이므로, 그것이 담고 있는 값의 메서드들을 가지지 못한다는 것입니다. `Wrapper`를 정확히 `Vec<T>`처럼 다룰 수 있도록 메서드들이 `self.0`에 위임하도록 `Vec<T>`의 모든 메서드를 `Wrapper`에 직접 구현해야 할 것입니다. 새 타입이 내부 타입이 가진 모든 메서드를 갖기를 원한다면, 내부 타입을 반환하도록 `Wrapper`에 `Deref` 트레이트를 구현하는 것이 한 해결책이 될 것입니다(15장의 [“스마트 포인터를 일반 참조처럼 다루기”][smart-pointer-deref]<!-- ignore --> 절에서 `Deref` 트레이트 구현을 논의했습니다). `Wrapper` 타입이 내부 타입의 모든 메서드를 갖기를 원하지 않는다면—예를 들어 `Wrapper` 타입의 동작을 제한하기 위해—원하는 메서드만 수동으로 구현해야 할 것입니다.

이 newtype 패턴은 트레이트가 관여하지 않을 때도 유용합니다. 초점을 바꾸어 러스트의 타입 시스템과 상호작용하는 몇 가지 고급 방법들을 살펴봅시다.

[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits]: ch10-02-traits.html
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
