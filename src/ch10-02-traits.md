<!-- Old headings. Do not remove or links may break. -->

<a id="traits-defining-shared-behavior"></a>

## 트레이트로 공유 동작 정의하기

_트레이트(trait)_ 는 특정 타입이 가지며 다른 타입과 공유할 수 있는 기능을
정의합니다. 트레이트를 사용하면 공유 동작을 추상적으로 정의할 수 있습니다.
_트레이트 경계(trait bound)_ 를 사용하면 제네릭 타입이 특정 동작을 가진 임의의
타입이 될 수 있음을 명시할 수 있습니다.

> 참고: 트레이트는 다른 언어에서 흔히 _인터페이스(interface)_ 라고 부르는 기능과
> 비슷하지만, 몇 가지 차이점이 있습니다.

### 트레이트 정의하기

어떤 타입의 동작은 그 타입에서 호출할 수 있는 메서드로 구성됩니다. 모든 타입에
대해 같은 메서드를 호출할 수 있다면, 그 타입들은 같은 동작을 공유하는 셈입니다.
트레이트 정의는 어떤 목적을 달성하는 데 필요한 동작 집합을 정의하기 위해 메서드
시그니처를 모아 두는 방법입니다.

예를 들어 다양한 종류와 분량의 텍스트를 담는 여러 구조체가 있다고 해 봅시다.
특정 장소에 접수된 뉴스 스토리를 담는 `NewsArticle` 구조체와, 최대 280자까지
가질 수 있고 그것이 새 글인지, 리포스트인지, 다른 글에 대한 답글인지 나타내는
메타데이터가 있는 `SocialPost`가 있다고 해 봅시다.

우리는 `NewsArticle`이나 `SocialPost` 인스턴스에 저장된 데이터를 요약해서 보여
줄 수 있는 `aggregator`라는 미디어 애그리게이터 라이브러리 크레이트를 만들고
싶습니다. 이를 위해 각 타입에서 요약을 얻어야 하고, 인스턴스에서 `summarize`
메서드를 호출해 그 요약을 요청할 것입니다. Listing 10-12는 이 동작을 표현하는
공개 `Summary` 트레이트의 정의를 보여 줍니다.

<Listing number="10-12" file-name="src/lib.rs" caption="`summarize` 메서드가 제공하는 동작으로 구성된 `Summary` 트레이트">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

여기서는 `trait` 키워드 다음에 트레이트 이름을 적어 트레이트를 선언하고 있으며,
이 경우 이름은 `Summary`입니다. 또한 몇 가지 예제에서 볼 예정처럼 이 크레이트에
의존하는 다른 크레이트도 이 트레이트를 사용할 수 있도록 트레이트를 `pub`으로
선언합니다. 중괄호 안에는 이 트레이트를 구현하는 타입의 동작을 기술하는 메서드
시그니처를 선언합니다. 이 경우에는 `fn summarize(&self) -> String`입니다.

메서드 시그니처 뒤에 중괄호로 구현을 제공하는 대신 세미콜론을 사용합니다. 이
트레이트를 구현하는 각 타입은 메서드 본문에 대한 자체 동작을 제공해야 합니다.
컴파일러는 `Summary` 트레이트를 가진 모든 타입이 정확히 이 시그니처로 정의된
`summarize` 메서드를 가지도록 강제합니다.

트레이트 본문에는 여러 메서드가 있을 수 있습니다. 메서드 시그니처는 한 줄에
하나씩 나열되며, 각 줄은 세미콜론으로 끝납니다.

### 타입에 트레이트 구현하기

이제 `Summary` 트레이트 메서드의 원하는 시그니처를 정의했으므로, 미디어
애그리게이터의 타입들에 이를 구현할 수 있습니다. Listing 10-13은 `NewsArticle`
구조체에 `Summary` 트레이트를 구현한 예를 보여 줍니다. 이 구현은 표제, 저자,
장소를 사용해 `summarize`의 반환 값을 만듭니다. `SocialPost` 구조체의 경우에는
게시물 내용이 이미 280자로 제한되어 있다고 가정하고, 사용자 이름에 이어 게시물
전체 텍스트를 붙여 `summarize`를 정의합니다.

<Listing number="10-13" file-name="src/lib.rs" caption="`NewsArticle`과 `SocialPost` 타입에 `Summary` 트레이트 구현하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

타입에 트레이트를 구현하는 것은 일반 메서드를 구현하는 것과 비슷합니다. 차이점은
`impl` 뒤에 구현하려는 트레이트 이름을 쓰고, 그다음 `for` 키워드를 사용하며,
그다음 트레이트를 구현할 타입의 이름을 명시한다는 것입니다. `impl` 블록 안에는
트레이트 정의에서 정의한 메서드 시그니처를 둡니다. 각 시그니처 뒤에 세미콜론을
붙이는 대신 중괄호를 사용하고, 특정 타입에 대해 트레이트의 메서드가 가져야 할
구체적 동작으로 메서드 본문을 채웁니다.

이제 라이브러리가 `NewsArticle`과 `SocialPost`에 `Summary` 트레이트를 구현했으므로,
크레이트 사용자는 일반 메서드를 호출하듯이 `NewsArticle`과 `SocialPost`
인스턴스에서 트레이트 메서드를 호출할 수 있습니다. 다른 점은 사용자가 타입뿐
아니라 트레이트도 스코프로 가져와야 한다는 것입니다. 다음은 바이너리 크레이트가
우리의 `aggregator` 라이브러리 크레이트를 사용하는 예입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

이 코드는 `1 new post: horse_ebooks: of course, as you probably already know, people`을
출력합니다.

`aggregator` 크레이트에 의존하는 다른 크레이트도 `Summary` 트레이트를 스코프로
가져와 자신의 타입에 `Summary`를 구현할 수 있습니다. 한 가지 유의할 제약은,
트레이트를 타입에 구현하려면 트레이트 또는 타입 중 적어도 하나(혹은 둘 다)가 우리
크레이트 내에 정의되어 있어야 한다는 점입니다. 예를 들어 `SocialPost`는 우리
`aggregator` 크레이트의 로컬 타입이므로, `Display` 같은 표준 라이브러리
트레이트를 `aggregator` 크레이트 기능의 일부로 `SocialPost` 같은 사용자 정의
타입에 구현할 수 있습니다. 또한 `Summary` 트레이트가 `aggregator` 크레이트의
로컬이므로 `aggregator` 크레이트 내에서 `Vec<T>`에 `Summary`를 구현할 수도
있습니다.

그러나 외부 트레이트를 외부 타입에 구현할 수는 없습니다. 예를 들어 `Display`와
`Vec<T>`가 모두 표준 라이브러리에 정의되어 있고 `aggregator` 크레이트의 로컬이
아니므로, `aggregator` 크레이트 내에서 `Vec<T>`에 `Display` 트레이트를 구현할 수
없습니다. 이 제약은 _일관성(coherence)_ 이라는 속성의 일부이며, 더 구체적으로는
부모 타입이 존재하지 않아 이름 지어진 _고아 규칙(orphan rule)_ 에 해당합니다.
이 규칙은 다른 사람의 코드가 내 코드를 망가뜨리지 못하게 하고, 그 반대도
마찬가지가 되도록 보장합니다. 이 규칙이 없다면 두 크레이트가 동일 타입에 동일
트레이트를 구현할 수 있고, 러스트는 어느 구현을 써야 할지 알 수 없을 것입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-implementations"></a>

### 기본 구현 사용하기

때때로 모든 타입에 모든 메서드 구현을 요구하지 않고, 트레이트의 일부 또는 모든
메서드에 기본 동작을 두는 것이 유용할 때가 있습니다. 그런 다음 특정 타입에
트레이트를 구현할 때, 각 메서드의 기본 동작을 유지하거나 재정의할 수 있습니다.

Listing 10-14에서는 Listing 10-12처럼 메서드 시그니처만 정의하는 대신,
`Summary` 트레이트의 `summarize` 메서드에 기본 문자열을 지정합니다.

<Listing number="10-14" file-name="src/lib.rs" caption="`summarize` 메서드의 기본 구현을 가진 `Summary` 트레이트 정의하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

기본 구현을 사용해 `NewsArticle` 인스턴스를 요약하려면 `impl Summary for NewsArticle {}`
처럼 빈 `impl` 블록을 지정합니다.

더 이상 `NewsArticle`에 `summarize` 메서드를 직접 정의하지 않더라도, 우리는
기본 구현을 제공했고 `NewsArticle`이 `Summary` 트레이트를 구현함을 명시했습니다.
그 결과 다음과 같이 여전히 `NewsArticle` 인스턴스에서 `summarize` 메서드를 호출할
수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

이 코드는 `New article available! (Read more...)`을 출력합니다.

기본 구현을 만든다고 해서 Listing 10-13의 `SocialPost`에 대한 `Summary` 구현을
바꿀 필요는 없습니다. 그 이유는 기본 구현을 재정의하는 문법이 기본 구현이 없는
트레이트 메서드를 구현하는 문법과 같기 때문입니다.

기본 구현은, 설령 같은 트레이트의 다른 메서드에 기본 구현이 없더라도 그 메서드를
호출할 수 있습니다. 이런 방식으로 트레이트는 많은 유용한 기능을 제공하고
구현자에게는 그중 일부만 지정하도록 요구할 수 있습니다. 예를 들어 `Summary`
트레이트에 구현이 필수인 `summarize_author` 메서드를 두고, `summarize_author`를
호출하는 기본 구현을 가진 `summarize` 메서드를 정의할 수 있습니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

이 버전의 `Summary`를 사용하려면, 타입에 트레이트를 구현할 때 `summarize_author`만
정의해 주면 됩니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

`summarize_author`를 정의한 뒤에는 `SocialPost` 구조체 인스턴스에서 `summarize`를
호출할 수 있고, `summarize`의 기본 구현이 우리가 제공한 `summarize_author`의
정의를 호출합니다. 우리는 `summarize_author`를 구현했기 때문에, 추가 코드를
작성하지 않고도 `Summary` 트레이트가 `summarize` 메서드의 동작을 제공해 준
셈입니다. 다음은 그 모습입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

이 코드는 `1 new post: (Read more from @horse_ebooks...)`를 출력합니다.

같은 메서드의 재정의 구현에서 기본 구현을 호출하는 것은 불가능함에 유의하세요.

<!-- Old headings. Do not remove or links may break. -->

<a id="traits-as-parameters"></a>

### 매개변수로서의 트레이트

이제 트레이트를 정의하고 구현하는 방법을 알게 되었으니, 트레이트를 사용해 여러
타입을 받는 함수를 정의하는 방법을 살펴봅시다. Listing 10-13에서 `NewsArticle`과
`SocialPost` 타입에 구현한 `Summary` 트레이트를 사용하여, `item` 매개변수에서
`summarize` 메서드를 호출하는 `notify` 함수를 정의할 것입니다. 이때 `item`은
`Summary` 트레이트를 구현하는 어떤 타입입니다. 이를 위해 다음과 같이 `impl Trait`
문법을 사용합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

`item` 매개변수의 구체적 타입 대신 `impl` 키워드와 트레이트 이름을 명시합니다.
이 매개변수는 지정된 트레이트를 구현하는 모든 타입을 받아들입니다. `notify`의
본문에서는 `Summary` 트레이트에서 온 `summarize` 같은 메서드라면 `item`에 호출할
수 있습니다. `notify`에 `NewsArticle`이나 `SocialPost` 인스턴스를 전달할 수
있습니다. `String`이나 `i32` 같은 다른 타입으로 이 함수를 호출하는 코드는
`Summary`를 구현하지 않으므로 컴파일되지 않습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### 트레이트 경계 문법

`impl Trait` 문법은 간단한 경우에 잘 동작하지만, 사실 _트레이트 경계(trait
bound)_ 라는 더 긴 형식의 문법 설탕(syntax sugar)입니다. 이는 다음과 같은 모습을
하고 있습니다.

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

이 더 긴 형식은 이전 절의 예제와 동일하지만 좀 더 장황합니다. 트레이트 경계는
제네릭 타입 매개변수 선언과 함께 꺾쇠괄호 안, 콜론 뒤에 배치합니다.

`impl Trait` 문법은 간단한 경우 편리하고 코드를 간결하게 해 주지만, 더 완전한
트레이트 경계 문법은 다른 경우에 더 복잡한 표현을 가능하게 합니다. 예를 들어
`Summary`를 구현하는 두 매개변수를 가질 수 있습니다. `impl Trait` 문법으로
표현하면 다음과 같습니다.

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

`item1`과 `item2`가 서로 다른 타입(단, 둘 다 `Summary`를 구현)이 되는 것을 허용
하고 싶다면 `impl Trait`을 사용하는 것이 적절합니다. 그러나 두 매개변수가 반드시
같은 타입이어야 하도록 강제하려면, 다음과 같이 트레이트 경계를 사용해야 합니다.

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

`item1`과 `item2` 매개변수의 타입으로 지정된 제네릭 타입 `T`는, `item1`과
`item2`의 인수로 전달된 값의 구체적 타입이 반드시 같아야 하도록 함수를 제약합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-multiple-trait-bounds-with-the--syntax"></a>

#### `+` 문법으로 여러 트레이트 경계 지정하기

여러 개의 트레이트 경계를 지정할 수도 있습니다. `notify`가 `item`에 대해
`summarize`뿐 아니라 디스플레이 포매팅까지 사용하도록 하고 싶다고 해 봅시다.
`notify` 정의에서 `item`은 `Display`와 `Summary` 둘 다를 구현해야 함을
명시합니다. `+` 문법을 사용해 다음과 같이 할 수 있습니다.

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

`+` 문법은 제네릭 타입의 트레이트 경계에도 유효합니다.

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

두 트레이트 경계가 지정되면, `notify`의 본문에서는 `summarize`를 호출하고 `{}`로
`item`을 포매팅할 수 있습니다.

#### `where` 절로 트레이트 경계 명확하게 하기

너무 많은 트레이트 경계를 사용하는 데에는 단점이 있습니다. 각 제네릭에는 자신만의
트레이트 경계가 있어서, 제네릭 타입 매개변수가 여러 개인 함수에서는 함수 이름과
매개변수 목록 사이에 많은 트레이트 경계 정보가 들어가 함수 시그니처를 읽기
어렵게 만들 수 있습니다. 이 때문에 러스트는 함수 시그니처 뒤 `where` 절 안에
트레이트 경계를 지정하는 대체 문법을 제공합니다. 그래서 다음 대신,

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

다음과 같이 `where` 절을 사용할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

이 함수의 시그니처는 덜 어지럽습니다. 많은 트레이트 경계가 없는 함수처럼, 함수
이름, 매개변수 목록, 반환 타입이 서로 가깝게 붙어 있습니다.

### 트레이트를 구현하는 타입 반환하기

다음과 같이 반환 위치에서도 `impl Trait` 문법을 사용해 어떤 트레이트를 구현하는
타입의 값을 반환할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

반환 타입으로 `impl Summary`를 사용함으로써, `returns_summarizable` 함수가
구체적 타입을 명시하지 않고 `Summary` 트레이트를 구현하는 어떤 타입을 반환한다는
것을 명시합니다. 여기서 `returns_summarizable`은 `SocialPost`를 반환하지만, 이
함수를 호출하는 코드는 그것을 알 필요가 없습니다.

반환 타입을 그것이 구현하는 트레이트만으로 지정할 수 있는 능력은, 13장에서
다루는 클로저와 이터레이터의 맥락에서 특히 유용합니다. 클로저와 이터레이터는
컴파일러만 아는 타입이나 매우 긴 이름을 갖는 타입을 만들어 냅니다. `impl Trait`
문법을 사용하면 긴 타입 이름을 적지 않고도, 함수가 `Iterator` 트레이트를 구현하는
어떤 타입을 반환함을 간결하게 명시할 수 있습니다.

그러나 `impl Trait`은 단일 타입을 반환할 때만 사용할 수 있습니다. 예를 들어
반환 타입을 `impl Summary`로 지정하고 `NewsArticle`이나 `SocialPost` 중 하나를
반환하는 다음 코드는 동작하지 않습니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

`impl Trait` 문법이 컴파일러에서 구현되는 방식의 제약 때문에, `NewsArticle` 또는
`SocialPost` 중 하나를 반환하는 것은 허용되지 않습니다. 이러한 동작을 가진 함수를
작성하는 방법은 18장의 [“트레이트 객체로 공유 동작 추상화하기”][trait-objects]<!-- ignore -->
절에서 다룹니다.

### 트레이트 경계로 메서드를 조건부로 구현하기

제네릭 타입 매개변수를 사용하는 `impl` 블록에 트레이트 경계를 두면, 지정된
트레이트를 구현하는 타입에 대해 메서드를 조건부로 구현할 수 있습니다. 예를 들어
Listing 10-15의 `Pair<T>` 타입은 항상 `Pair<T>`의 새 인스턴스를 반환하는 `new`
함수를 구현합니다(5장의 [“메서드 문법”][methods]<!-- ignore --> 절에서 `Self`가
`impl` 블록의 타입에 대한 타입 별칭임을 떠올려 보세요. 이 경우에는 `Pair<T>`
입니다). 그러나 다음 `impl` 블록에서는, 내부 타입 `T`가 비교를 가능하게 하는
`PartialOrd` 트레이트 _와_ 출력을 가능하게 하는 `Display` 트레이트를 둘 다 구현할
때만 `Pair<T>`에 `cmp_display` 메서드를 구현합니다.

<Listing number="10-15" file-name="src/lib.rs" caption="트레이트 경계에 따라 제네릭 타입에 메서드를 조건부로 구현하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

다른 트레이트를 구현하는 어떤 타입에 대해서도 트레이트를 조건부로 구현할 수
있습니다. 트레이트 경계를 만족하는 어떤 타입에든 트레이트를 구현하는 것을
_블랭킷 구현(blanket implementation)_ 이라고 부르며, 이는 러스트 표준
라이브러리에서 광범위하게 사용됩니다. 예를 들어 표준 라이브러리는 `Display`
트레이트를 구현하는 어떤 타입에든 `ToString` 트레이트를 구현합니다. 표준
라이브러리의 `impl` 블록은 다음 코드와 비슷한 모습입니다.

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

표준 라이브러리에 이 블랭킷 구현이 있기 때문에, `Display`를 구현하는 어떤
타입에서든 `ToString` 트레이트가 정의한 `to_string` 메서드를 호출할 수 있습니다.
예를 들어 정수는 `Display`를 구현하므로 다음처럼 정수를 대응되는 `String` 값으로
바꿀 수 있습니다.

```rust
let s = 3.to_string();
```

블랭킷 구현은 트레이트 문서의 “Implementors” 섹션에 표시됩니다.

트레이트와 트레이트 경계 덕분에 우리는 제네릭 타입 매개변수를 사용해 중복을
줄이면서도, 그 제네릭 타입이 특정 동작을 갖도록 컴파일러에 명시하는 코드를
작성할 수 있습니다. 그러면 컴파일러는 트레이트 경계 정보를 사용해 코드와 함께
사용되는 모든 구체적 타입이 올바른 동작을 제공하는지 검사할 수 있습니다. 동적
타입 언어에서는 메서드를 정의하지 않은 타입에서 메서드를 호출하면 런타임 오류가
날 것입니다. 그러나 러스트는 이러한 오류를 컴파일 타임으로 옮겨, 코드가 실행되기
전에 문제를 고치도록 강제합니다. 또한 런타임에 동작을 검사하는 코드를 작성할
필요도 없습니다. 컴파일 타임에 이미 검사했기 때문입니다. 그렇게 해도 제네릭의
유연성은 포기하지 않은 채 성능이 개선됩니다.

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[methods]: ch05-03-method-syntax.html#method-syntax
