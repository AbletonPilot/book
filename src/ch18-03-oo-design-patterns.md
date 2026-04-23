## 객체 지향 디자인 패턴 구현하기

_상태 패턴(state pattern)_ 은 객체 지향 디자인 패턴의 하나입니다. 이 패턴의 핵심은 어떤 값이 내부적으로 가질 수 있는 상태들의 집합을 정의한다는 것입니다. 상태들은 _상태 객체(state objects)_ 들의 집합으로 표현되며, 값의 동작은 그 상태에 따라 변합니다. 우리는 “초안(draft)”, “검토(review)”, “게시(published)” 중 하나의 상태 객체가 될 상태를 담는 필드를 가진 블로그 게시물 구조체 예제를 만들어 볼 것입니다.

상태 객체들은 기능을 공유합니다. 물론 러스트에서는 객체와 상속이 아니라 구조체와 트레이트를 사용합니다. 각 상태 객체는 자신의 동작과, 언제 다른 상태로 바뀌어야 하는지 결정하는 책임을 가집니다. 상태 객체를 담고 있는 값은 상태들의 서로 다른 동작이나 언제 상태 사이를 전이해야 하는지에 대해 아무것도 알지 못합니다.

상태 패턴을 사용하는 장점은, 프로그램의 비즈니스 요구사항이 바뀔 때 상태를 담고 있는 값의 코드나 그 값을 사용하는 코드를 변경할 필요가 없다는 점입니다. 상태 객체 중 하나의 코드만 갱신해서 그 규칙을 바꾸거나, 아니면 상태 객체를 더 추가하기만 하면 됩니다.

먼저 좀 더 전통적인 객체 지향 방식으로 상태 패턴을 구현할 것입니다. 그런 다음 러스트에서 좀 더 자연스러운 방식의 접근을 사용할 것입니다. 상태 패턴을 사용해 블로그 게시물 워크플로를 점진적으로 구현해 봅시다.

최종적으로 동작은 다음과 같을 것입니다.

1. 블로그 게시물은 빈 초안으로 시작합니다.
1. 초안이 완성되면 게시물에 대한 검토가 요청됩니다.
1. 게시물이 승인되면 게시됩니다.
1. 오직 게시된 블로그 게시물만 출력할 콘텐츠를 반환하므로, 승인되지 않은 게시물이 실수로 게시되는 일은 없습니다.

게시물에 시도되는 그 외의 변경은 어떤 효과도 없어야 합니다. 예를 들어 검토를 요청하기 전에 초안 블로그 게시물을 승인하려고 하면, 게시물은 게시되지 않은 초안 상태로 남아 있어야 합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-traditional-object-oriented-attempt"></a>

### 전통적인 객체 지향 스타일로 시도하기

같은 문제를 해결하기 위해 코드를 구성하는 방법은 무수히 많고, 각각 다른 트레이드오프를 가집니다. 이 절의 구현은 좀 더 전통적인 객체 지향 스타일로, 러스트로도 작성할 수 있지만 러스트의 강점 중 일부를 활용하지는 못합니다. 나중에 같은 객체 지향 디자인 패턴을 사용하지만 객체 지향 경험이 있는 프로그래머에게는 덜 익숙해 보일 수 있는 방식으로 구성된 다른 해법을 보일 것입니다. 두 해법을 비교하여, 다른 언어의 코드와는 다르게 러스트 코드를 설계하는 트레이드오프를 경험해 볼 것입니다.

Listing 18-11은 이 워크플로를 코드 형태로 보여 줍니다. 이는 우리가 `blog`라는 라이브러리 크레이트로 구현할 API의 사용 예시입니다. `blog` 크레이트를 아직 구현하지 않았으므로 컴파일되지 않습니다.

<Listing number="18-11" file-name="src/main.rs" caption="우리의 `blog` 크레이트가 가졌으면 하는 동작을 보여 주는 코드">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

사용자가 `Post::new`로 새로운 초안 블로그 게시물을 만들 수 있게 하고 싶습니다. 블로그 게시물에 텍스트를 추가할 수 있게 하고 싶습니다. 승인 전에 게시물의 콘텐츠를 즉시 가져오려 하면, 게시물은 아직 초안 상태이므로 어떤 텍스트도 받지 못해야 합니다. 시연 목적으로 코드에 `assert_eq!`를 추가했습니다. 이를 위한 훌륭한 단위 테스트는 초안 블로그 게시물이 `content` 메서드에서 빈 문자열을 반환하는지 단언하는 것이 될 텐데, 이 예제에서는 테스트를 작성하지는 않을 것입니다.

다음으로, 게시물에 대한 검토 요청을 가능하게 하고, 검토를 기다리는 동안에는 `content`가 빈 문자열을 반환하게 하고 싶습니다. 게시물이 승인되면 게시되어야 하며, 이는 `content`가 호출될 때 게시물의 텍스트가 반환된다는 뜻입니다.

크레이트에서 우리가 상호작용하는 유일한 타입은 `Post` 타입이라는 점에 주목하세요. 이 타입은 상태 패턴을 사용하며, 게시물이 가질 수 있는 다양한 상태(초안, 검토, 게시)를 표현하는 세 상태 객체 중 하나가 될 값을 담습니다. 한 상태에서 다른 상태로의 변경은 `Post` 타입 내부에서 관리됩니다. 상태는 우리 라이브러리 사용자가 `Post` 인스턴스에서 호출하는 메서드에 응답해 변경되지만, 사용자는 상태 변경을 직접 관리할 필요가 없습니다. 또한 사용자는 게시물을 검토받기 전에 게시하는 것과 같은 상태 관련 실수를 할 수 없습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-post-and-creating-a-new-instance-in-the-draft-state"></a>

#### `Post` 정의와 새 인스턴스 만들기

라이브러리 구현을 시작합시다! 어떤 콘텐츠를 담는 공개 `Post` 구조체가 필요하다는 것을 알고 있으니, Listing 18-12와 같이 구조체 정의와 `Post` 인스턴스를 만들기 위한 연관된 공개 `new` 함수로 시작하겠습니다. 또한 `Post`의 모든 상태 객체가 가져야 할 동작을 정의할 비공개 `State` 트레이트를 만들 것입니다.

그리고 `Post`는 상태 객체를 담기 위해 `state`라는 비공개 필드 안에 `Option<T>`로 감싼 `Box<dyn State>` 트레이트 객체를 가질 것입니다. 왜 `Option<T>`가 필요한지는 잠시 뒤에 보게 됩니다.

<Listing number="18-12" file-name="src/lib.rs" caption="`Post` 구조체와 새 `Post` 인스턴스를 만드는 `new` 함수, `State` 트레이트, `Draft` 구조체의 정의">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

`State` 트레이트는 서로 다른 게시물 상태들이 공유할 동작을 정의합니다. 상태 객체는 `Draft`, `PendingReview`, `Published`이며, 모두 `State` 트레이트를 구현할 것입니다. 지금은 트레이트에 메서드가 없고, 게시물이 시작해야 하는 상태인 `Draft` 상태부터 정의하는 것으로 시작합니다.

새 `Post`를 만들 때 `state` 필드를 `Box`를 담은 `Some` 값으로 설정합니다. 이 `Box`는 새 `Draft` 구조체 인스턴스를 가리킵니다. 이는 새 `Post` 인스턴스를 만들 때마다 항상 초안으로 시작함을 보장합니다. `Post`의 `state` 필드가 비공개이므로, 다른 어떤 상태로도 `Post`를 만들 방법이 없습니다! `Post::new` 함수에서는 `content` 필드를 새로운 빈 `String`으로 설정합니다.

#### 게시물 콘텐츠의 텍스트 저장하기

Listing 18-11에서 `add_text`라는 메서드를 호출해 `&str`을 전달하면 그것이 블로그 게시물의 텍스트 콘텐츠로 추가될 수 있기를 원한다는 것을 보았습니다. 이를 메서드로 구현하는 이유는, `content` 필드를 `pub`으로 노출하지 않음으로써 나중에 `content` 필드의 데이터가 어떻게 읽히는지를 제어하는 메서드를 구현할 수 있게 하기 위해서입니다. `add_text` 메서드는 꽤 단순하므로, Listing 18-13의 구현을 `impl Post` 블록에 추가합시다.

<Listing number="18-13" file-name="src/lib.rs" caption="게시물의 `content`에 텍스트를 추가하는 `add_text` 메서드 구현">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

`add_text` 메서드는 `add_text`가 호출되는 `Post` 인스턴스를 변경하므로 `self`에 대한 가변 참조를 받습니다. 그런 다음 `content`에 있는 `String`에 대해 `push_str`을 호출하고 `text` 인수를 전달해 저장된 `content`에 추가합니다. 이 동작은 게시물이 어떤 상태인지에 의존하지 않으므로 상태 패턴의 일부가 아닙니다. `add_text` 메서드는 `state` 필드와 전혀 상호작용하지 않지만, 우리가 지원하고 싶은 동작의 일부입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-the-content-of-a-draft-post-is-empty"></a>

#### 초안 게시물의 콘텐츠가 비어 있도록 보장하기

`add_text`를 호출해 게시물에 어떤 콘텐츠를 추가한 뒤에도, 게시물은 여전히 초안 상태이므로 `content` 메서드는 Listing 18-11의 첫 번째 `assert_eq!`에서 보이듯 빈 문자열 슬라이스를 반환하기를 원합니다. 지금은 이 요구사항을 충족하는 가장 단순한 것, 즉 항상 빈 문자열 슬라이스를 반환하는 것으로 `content` 메서드를 구현합시다. 게시물의 상태를 변경할 수 있게 만들어 게시될 수 있도록 한 뒤에 이 부분을 변경하겠습니다. 지금까지 게시물은 오직 초안 상태에만 있을 수 있으므로 게시물의 콘텐츠는 항상 비어 있어야 합니다. Listing 18-14는 이 자리표시자 구현을 보여 줍니다.

<Listing number="18-14" file-name="src/lib.rs" caption="`Post`의 `content` 메서드가 항상 빈 문자열 슬라이스를 반환하도록 자리표시자 구현 추가">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

이렇게 추가된 `content` 메서드 덕분에, Listing 18-11에서 첫 번째 `assert_eq!`까지의 모든 부분은 의도대로 동작합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>
<a id="requesting-a-review-changes-the-posts-state"></a>

#### 게시물 검토 요청, 즉 게시물의 상태 변경

다음으로, 게시물의 검토를 요청하는 기능을 추가해야 합니다. 이는 게시물 상태를 `Draft`에서 `PendingReview`로 바꿔야 합니다. Listing 18-15는 이 코드를 보여 줍니다.

<Listing number="18-15" file-name="src/lib.rs" caption="`Post`와 `State` 트레이트에 `request_review` 메서드 구현하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

`Post`에 `self`에 대한 가변 참조를 받는 공개 메서드 `request_review`를 부여합니다. 그런 다음 현재 `Post`의 상태에 대해 내부 `request_review` 메서드를 호출하고, 이 두 번째 `request_review` 메서드는 현재 상태를 소비하고 새로운 상태를 반환합니다.

`State` 트레이트에 `request_review` 메서드를 추가합니다. 이제 트레이트를 구현하는 모든 타입은 `request_review` 메서드를 구현해야 합니다. 메서드의 첫 매개변수로 `self`, `&self`, `&mut self`가 아니라 `self: Box<Self>`를 사용한다는 점에 주목하세요. 이 문법은 메서드가 그 타입을 담은 `Box`에 대해 호출될 때만 유효하다는 뜻입니다. 이 문법은 `Box<Self>`의 소유권을 가져가서 이전 상태를 무효화하므로, `Post`의 상태 값이 새로운 상태로 변환될 수 있습니다.

이전 상태를 소비하기 위해 `request_review` 메서드는 상태 값의 소유권을 가져가야 합니다. 이때 `Post`의 `state` 필드에 있는 `Option`이 등장합니다. `take` 메서드를 호출해 `state` 필드에서 `Some` 값을 꺼내고 그 자리에 `None`을 남겨 둡니다. 러스트는 구조체에 채워지지 않은 필드를 둘 수 없게 하기 때문입니다. 이렇게 함으로써 `state` 값을 빌리지 않고 `Post` 밖으로 옮길 수 있습니다. 그런 다음 게시물의 `state` 값을 이 연산의 결과로 설정합니다.

`self.state = self.state.request_review();` 같은 코드로 직접 설정하지 않고 `state`를 일시적으로 `None`으로 설정해야 하는 이유는 `state` 값의 소유권을 얻기 위해서입니다. 이는 새로운 상태로 변환한 뒤에 `Post`가 이전 `state` 값을 사용할 수 없도록 보장합니다.

`Draft`의 `request_review` 메서드는 게시물이 검토를 기다리는 상태를 나타내는 새로운 `PendingReview` 구조체의 박싱된 인스턴스를 새로 반환합니다. `PendingReview` 구조체도 `request_review` 메서드를 구현하지만, 어떤 변환도 하지 않습니다. 대신 자기 자신을 반환합니다. 왜냐하면 이미 `PendingReview` 상태인 게시물에 검토를 요청하면 그대로 `PendingReview` 상태에 머물러야 하기 때문입니다.

이제 상태 패턴의 장점이 보이기 시작합니다. `Post`의 `request_review` 메서드는 `state` 값이 무엇이든 동일합니다. 각 상태가 자신의 규칙에 책임을 집니다.

`Post`의 `content` 메서드는 빈 문자열 슬라이스를 반환하는 그대로 둡니다. 이제 `Post`는 `Draft` 상태뿐만 아니라 `PendingReview` 상태에도 있을 수 있지만, `PendingReview` 상태에서도 같은 동작을 원합니다. Listing 18-11은 이제 두 번째 `assert_eq!` 호출까지 동작합니다!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>
<a id="adding-approve-to-change-the-behavior-of-content"></a>

#### `content`의 동작을 바꾸는 `approve` 추가

`approve` 메서드는 `request_review` 메서드와 비슷할 것입니다. Listing 18-16에서 보이듯, 현재 상태가 승인되었을 때 가져야 한다고 하는 값으로 `state`를 설정합니다.

<Listing number="18-16" file-name="src/lib.rs" caption="`Post`와 `State` 트레이트에 `approve` 메서드 구현하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

`State` 트레이트에 `approve` 메서드를 추가하고, `State`를 구현하는 새로운 구조체인 `Published` 상태를 추가합니다.

`PendingReview`에서의 `request_review`와 비슷한 방식으로, `Draft`에서 `approve` 메서드를 호출하면 `approve`가 `self`를 반환하므로 아무런 효과가 없습니다. `PendingReview`에서 `approve`를 호출하면 `Published` 구조체의 박싱된 새 인스턴스를 반환합니다. `Published` 구조체는 `State` 트레이트를 구현하며, `request_review` 메서드와 `approve` 메서드 둘 다에 대해 자기 자신을 반환합니다. 그런 경우들에는 게시물이 `Published` 상태에 머물러야 하기 때문입니다.

이제 `Post`의 `content` 메서드를 갱신해야 합니다. `content`가 반환하는 값이 `Post`의 현재 상태에 의존하기를 원하므로, Listing 18-17에서 보이듯 `Post`가 자신의 `state`에 정의된 `content` 메서드에 위임하도록 만들 것입니다.

<Listing number="18-17" file-name="src/lib.rs" caption="`State`의 `content` 메서드에 위임하도록 `Post`의 `content` 메서드 갱신">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

목표는 이러한 모든 규칙을 `State`를 구현하는 구조체들 안에 두는 것이므로, `state`의 값에 대해 `content` 메서드를 호출하면서 게시물 인스턴스(즉 `self`)를 인수로 전달합니다. 그런 다음 `state` 값에 대해 `content` 메서드를 사용한 결과로 반환된 값을 반환합니다.

`Option`에 대해 `as_ref` 메서드를 호출하는 이유는 값의 소유권이 아니라 `Option` 안 값에 대한 참조를 원하기 때문입니다. `state`가 `Option<Box<dyn State>>`이므로, `as_ref`를 호출하면 `Option<&Box<dyn State>>`가 반환됩니다. `as_ref`를 호출하지 않으면, 함수 매개변수의 빌린 `&self`에서 `state`를 옮길 수 없으므로 오류가 발생할 것입니다.

그런 다음 `unwrap` 메서드를 호출하는데, `Post`의 메서드들이 그 메서드들이 끝났을 때 `state`가 항상 `Some` 값을 담도록 보장한다는 것을 알기 때문에 결코 패닉을 일으키지 않으리라는 것을 압니다. 이는 컴파일러가 이해할 수는 없지만 우리는 `None` 값이 결코 나올 수 없다는 것을 아는 9장의 [“컴파일러보다 더 많은 정보를 가지고 있을 때”][more-info-than-rustc]<!-- ignore --> 절에서 이야기한 사례 중 하나입니다.

이 시점에서 `&Box<dyn State>`에 대해 `content`를 호출하면, `&`와 `Box`에 deref 강제 변환이 일어나 `content` 메서드가 결국 `State` 트레이트를 구현한 타입에서 호출됩니다. 즉, `State` 트레이트 정의에 `content`를 추가해야 하며, Listing 18-18에서 보이듯 어떤 상태에 따라 어떤 콘텐츠를 반환할지에 대한 로직을 거기에 두게 됩니다.

<Listing number="18-18" file-name="src/lib.rs" caption="`State` 트레이트에 `content` 메서드 추가">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

`content` 메서드에 빈 문자열 슬라이스를 반환하는 기본 구현을 추가합니다. 즉, `Draft`와 `PendingReview` 구조체에는 `content`를 구현할 필요가 없습니다. `Published` 구조체는 `content` 메서드를 재정의하고 `post.content`의 값을 반환할 것입니다. 편리하긴 하지만, `State`의 `content` 메서드가 `Post`의 콘텐츠를 결정하게 만드는 것은 `State`의 책임과 `Post`의 책임 사이 경계를 흐리게 만듭니다.

10장에서 논의했듯이 이 메서드에는 라이프타임 표기가 필요하다는 점에 유의하세요. 인수로 `post`에 대한 참조를 받고 그 `post`의 일부에 대한 참조를 반환하므로, 반환되는 참조의 라이프타임은 `post` 인수의 라이프타임과 관련됩니다.

이제 끝났습니다—Listing 18-11의 모든 코드가 동작합니다! 블로그 게시물 워크플로의 규칙들로 상태 패턴을 구현했습니다. 규칙과 관련된 로직은 `Post` 전체에 흩어져 있지 않고 상태 객체들 안에 살고 있습니다.

> ### 왜 열거형이 아닌가?
>
> 가능한 게시물 상태들을 변형으로 가지는 열거형을 왜 사용하지 않았는지 궁금했을 수도 있습니다. 그것도 분명 가능한 해법입니다. 시도해 보고 최종 결과를 비교해 어느 쪽이 마음에 드는지 확인해 보세요! 열거형을 사용할 때의 한 가지 단점은 열거형 값을 검사하는 모든 곳에서 가능한 모든 변형을 처리하기 위해 `match` 표현식 등이 필요해진다는 점입니다. 이는 이 트레이트 객체 해법보다 더 반복적일 수 있습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="trade-offs-of-the-state-pattern"></a>

#### 상태 패턴 평가하기

러스트가 객체 지향 상태 패턴을 구현해 게시물이 각 상태에서 가져야 할 다양한 종류의 동작을 캡슐화할 수 있음을 보였습니다. `Post`의 메서드들은 다양한 동작에 대해 아무것도 알지 못합니다. 코드를 그렇게 구성한 덕분에, 게시된 게시물이 어떻게 다르게 동작할 수 있는지 알기 위해서는 한 곳—`Published` 구조체에 대한 `State` 트레이트 구현—만 보면 됩니다.

상태 패턴을 사용하지 않는 대안적 구현을 만들었다면, 대신 `Post`의 메서드 안에 또는 게시물의 상태를 검사하고 그곳에서 동작을 바꾸는 `main` 코드 안에 `match` 표현식을 사용했을지도 모릅니다. 그렇다면 게시된 상태인 게시물의 모든 함의를 이해하기 위해 여러 곳을 봐야 했을 것입니다.

상태 패턴을 사용하면 `Post`의 메서드와 우리가 `Post`를 사용하는 곳에는 `match` 표현식이 필요하지 않으며, 새로운 상태를 추가하려면 새 구조체를 추가하고 그 한 구조체에 대한 트레이트 메서드들을 한 곳에서 구현하기만 하면 됩니다.

상태 패턴을 사용하는 구현은 더 많은 기능을 추가하기 위해 확장하기 쉽습니다. 상태 패턴을 사용하는 코드를 유지보수하는 단순함을 보기 위해, 다음 제안 중 몇 가지를 시도해 보세요.

- 게시물의 상태를 `PendingReview`에서 다시 `Draft`로 바꾸는 `reject` 메서드를 추가합니다.
- 상태가 `Published`로 바뀌기 전에 `approve`를 두 번 호출하도록 요구합니다.
- 게시물이 `Draft` 상태일 때만 사용자가 텍스트 콘텐츠를 추가할 수 있게 합니다. 힌트: 콘텐츠에 대해 무엇이 변경될 수 있는지에 대한 책임은 상태 객체가 가지되, `Post`를 수정하는 책임은 갖지 않게 하세요.

상태 패턴의 한 가지 단점은 상태들이 상태 사이의 전이를 구현하기 때문에, 일부 상태가 서로 결합되어 있다는 점입니다. 만약 `PendingReview`와 `Published` 사이에 `Scheduled` 같은 또 다른 상태를 추가한다면, `PendingReview`의 코드를 `Scheduled`로 전이하도록 변경해야 합니다. 새로운 상태를 추가했을 때 `PendingReview`가 변경될 필요가 없다면 일이 줄어들겠지만, 그건 다른 디자인 패턴으로 옮겨 가야 한다는 뜻입니다.

또 다른 단점은 일부 로직이 중복되었다는 것입니다. 중복을 일부 제거하기 위해, `State` 트레이트에서 `request_review`와 `approve` 메서드의 기본 구현을 만들어 `self`를 반환하게 시도해 볼 수도 있습니다. 그러나 이는 동작하지 않을 것입니다. `State`를 트레이트 객체로 사용할 때, 트레이트는 구체적인 `self`가 정확히 무엇인지 알지 못하므로 반환 타입을 컴파일 시점에 알 수 없습니다. (이는 앞에서 언급한 dyn 호환성 규칙 중 하나입니다.)

또 다른 중복은 `Post`의 `request_review`와 `approve` 메서드가 비슷하게 구현되어 있다는 점입니다. 두 메서드 모두 `Post`의 `state` 필드에 대해 `Option::take`를 사용하고, `state`가 `Some`이면 감싸진 값의 같은 메서드 구현에 위임한 뒤 그 결과를 새 `state` 필드 값으로 설정합니다. 이 패턴을 따르는 메서드가 `Post`에 많다면, 반복을 제거하기 위해 매크로를 정의하는 것을 고려해 볼 수도 있습니다(20장의 [“매크로”][macros]<!-- ignore --> 절 참고).

객체 지향 언어에서 정의된 그대로 상태 패턴을 구현함으로써 우리는 러스트의 강점을 충분히 활용하지 못하고 있습니다. 무효한 상태와 전이를 컴파일 시점 오류로 만들 수 있는, `blog` 크레이트에 대해 가할 수 있는 변경을 살펴봅시다.

### 상태와 동작을 타입으로 인코딩하기

상태 패턴을 다른 트레이드오프 집합으로 다시 생각하는 방법을 보이겠습니다. 외부 코드가 상태와 전이를 전혀 알지 못하도록 그것들을 완전히 캡슐화하는 대신, 상태들을 서로 다른 타입으로 인코딩할 것입니다. 그 결과 러스트의 타입 검사 시스템은 게시된 게시물만 허용되는 곳에서 초안 게시물을 사용하려는 시도를 컴파일 오류로 막을 것입니다.

Listing 18-11의 `main`의 첫 부분을 살펴봅시다.

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

여전히 `Post::new`를 사용해 초안 상태로 새 게시물을 만들고, 게시물의 콘텐츠에 텍스트를 추가하는 것은 가능하게 합니다. 그러나 초안 게시물에 빈 문자열을 반환하는 `content` 메서드를 두는 대신, 초안 게시물에는 `content` 메서드를 아예 두지 않게 만들 것입니다. 그렇게 하면 초안 게시물의 콘텐츠를 가져오려고 시도할 때, 그 메서드가 존재하지 않는다는 컴파일 오류가 발생합니다. 그 결과 초안 게시물의 콘텐츠를 운영 환경에서 실수로 표시하는 일은 그 코드가 컴파일조차 되지 않으므로 불가능해집니다. Listing 18-19는 `Post` 구조체와 `DraftPost` 구조체의 정의, 그리고 각 구조체의 메서드를 보여 줍니다.

<Listing number="18-19" file-name="src/lib.rs" caption="`content` 메서드를 가진 `Post`와 `content` 메서드가 없는 `DraftPost`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

`Post`와 `DraftPost` 구조체 모두 블로그 게시물 텍스트를 저장하는 비공개 `content` 필드를 가집니다. 상태의 인코딩을 구조체의 타입으로 옮기고 있으므로 구조체에는 더 이상 `state` 필드가 없습니다. `Post` 구조체는 게시된 게시물을 나타내며, `content`를 반환하는 `content` 메서드를 가집니다.

여전히 `Post::new` 함수가 있지만, `Post`의 인스턴스를 반환하는 대신 `DraftPost`의 인스턴스를 반환합니다. `content`가 비공개이고 `Post`를 반환하는 함수가 없으므로, 지금은 `Post`의 인스턴스를 만드는 것이 불가능합니다.

`DraftPost` 구조체에는 `add_text` 메서드가 있어, 이전처럼 `content`에 텍스트를 추가할 수 있습니다. 다만 `DraftPost`에는 `content` 메서드가 정의되어 있지 않다는 점에 주목하세요! 이제 프로그램은 모든 게시물이 초안 게시물로 시작하고, 초안 게시물은 콘텐츠를 표시할 수 없도록 보장합니다. 이런 제약을 우회하려는 어떤 시도든 컴파일 오류로 이어질 것입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="implementing-transitions-as-transformations-into-different-types"></a>

그렇다면 게시된 게시물은 어떻게 얻을까요? 초안 게시물은 게시되기 전에 검토되고 승인되어야 한다는 규칙을 강제하고 싶습니다. 검토 대기 상태의 게시물도 여전히 어떤 콘텐츠도 표시해서는 안 됩니다. Listing 18-20에서 보이듯, 또 다른 구조체 `PendingReviewPost`를 추가하고, `DraftPost`에 `PendingReviewPost`를 반환하는 `request_review` 메서드를 정의하며, `PendingReviewPost`에 `Post`를 반환하는 `approve` 메서드를 정의해 이 제약들을 구현해 봅시다.

<Listing number="18-20" file-name="src/lib.rs" caption="`DraftPost`에 대해 `request_review`를 호출해 만들어지는 `PendingReviewPost`와, `PendingReviewPost`를 게시된 `Post`로 바꾸는 `approve` 메서드">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

`request_review`와 `approve` 메서드는 `self`의 소유권을 가져가서 `DraftPost`와 `PendingReviewPost` 인스턴스를 소비하고, 각각 `PendingReviewPost`와 게시된 `Post`로 변환합니다. 이 방식이라면 `request_review`를 호출한 뒤 남아 있는 `DraftPost` 인스턴스가 없을 테고, 그 다음도 마찬가지입니다. `PendingReviewPost` 구조체에는 `content` 메서드가 정의되어 있지 않으므로, `DraftPost`와 마찬가지로 그 콘텐츠를 읽으려는 시도는 컴파일 오류로 이어집니다. `content` 메서드가 정의되어 있는 게시된 `Post` 인스턴스를 얻는 유일한 방법은 `PendingReviewPost`에 대해 `approve` 메서드를 호출하는 것이고, `PendingReviewPost`를 얻는 유일한 방법은 `DraftPost`에 대해 `request_review` 메서드를 호출하는 것이므로, 이제 블로그 게시물 워크플로를 타입 시스템으로 인코딩한 셈입니다.

그러나 `main`에도 약간의 변경이 필요합니다. `request_review`와 `approve` 메서드는 자신이 호출된 구조체를 수정하는 대신 새 인스턴스를 반환하므로, 반환된 인스턴스를 저장하기 위해 `let post =` 섀도잉 할당을 더 추가해야 합니다. 또한 초안과 검토 대기 게시물의 콘텐츠가 빈 문자열이라는 단언을 가질 수 없으며, 그 단언이 필요하지도 않습니다. 그 상태의 게시물의 콘텐츠를 사용하려는 코드는 더 이상 컴파일할 수 없기 때문입니다. 갱신된 `main`의 코드는 Listing 18-21에 나와 있습니다.

<Listing number="18-21" file-name="src/main.rs" caption="블로그 게시물 워크플로의 새 구현을 사용하도록 `main`을 수정">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

`main`에서 `post`를 재할당해야 하도록 변경한 결과, 이 구현은 더 이상 객체 지향 상태 패턴을 그대로 따르지는 않습니다. 상태 사이의 변환이 더 이상 `Post` 구현 안에 완전히 캡슐화되지 않기 때문입니다. 그러나 그 대신 우리가 얻은 것은, 컴파일 시점에 일어나는 타입 시스템과 타입 검사 덕분에 무효한 상태가 이제 불가능해졌다는 점입니다! 이는 게시되지 않은 게시물의 콘텐츠 표시 같은 특정 버그가 운영 환경에 도달하기 전에 발견되도록 보장합니다.

Listing 18-21 이후의 `blog` 크레이트에 대해 이 절의 시작 부분에서 제안한 작업들을 시도해, 이 버전의 코드 설계에 대해 어떻게 생각하는지 확인해 보세요. 일부 작업은 이 설계에서는 이미 완료되었을 수 있다는 점에 유의하세요.

러스트가 객체 지향 디자인 패턴을 구현할 수 있음을 살펴보았지만, 상태를 타입 시스템으로 인코딩하는 것과 같은 다른 패턴들도 러스트에서 사용할 수 있습니다. 이 패턴들은 서로 다른 트레이드오프를 가집니다. 객체 지향 패턴에 매우 익숙할지도 모르지만, 러스트의 기능을 활용하기 위해 문제를 다시 생각해 보면 컴파일 시점에 일부 버그를 막는 것과 같은 이점을 얻을 수 있습니다. 객체 지향 언어에는 없는 소유권 같은 특정 기능 때문에, 러스트에서 객체 지향 패턴이 항상 최선의 해결책은 아닐 것입니다.

## 요약

이 장을 읽고 나서 러스트가 객체 지향 언어라고 생각하든 아니든, 이제 트레이트 객체를 사용해 러스트에서 일부 객체 지향 기능을 얻을 수 있다는 사실을 알게 되었습니다. 동적 디스패치는 약간의 런타임 성능을 대가로 코드에 어느 정도의 유연성을 줄 수 있습니다. 이 유연성을 사용해 코드의 유지보수성에 도움이 되는 객체 지향 패턴을 구현할 수 있습니다. 러스트에는 또한 객체 지향 언어에는 없는 소유권 같은 다른 기능들이 있습니다. 객체 지향 패턴이 러스트의 강점을 활용하는 최선의 방법이 항상은 아니지만, 사용 가능한 선택지입니다.

다음으로, 많은 유연성을 가능하게 하는 또 다른 러스트 기능인 패턴을 살펴보겠습니다. 책 전반에 걸쳐 패턴을 잠시 살펴보았지만, 그 모든 능력을 본 적은 아직 없습니다. 시작합시다!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
