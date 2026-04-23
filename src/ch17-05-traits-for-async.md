<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

## Async 관련 트레이트 자세히 살펴보기

이 장 전체에서 `Future`, `Stream`, `StreamExt` 트레이트를 다양한 방식으로
사용했습니다. 그러나 지금까지는 그것들이 어떻게 동작하는지, 어떻게 서로
맞물리는지의 세부 사항에 너무 깊이 들어가는 것을 피했습니다. 일상적인 러스트
작업에는 대부분 그것으로 충분합니다. 그러나 때때로 이 트레이트들의 더 많은
세부 사항과 `Pin` 타입 및 `Unpin` 트레이트를 이해해야 하는 상황을 마주할
것입니다. 이 절에서는 그런 시나리오에 도움이 될 만큼만 파고들되, _정말로_
깊은 탐구는 다른 문서에 맡기겠습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### `Future` 트레이트

`Future` 트레이트가 어떻게 동작하는지 더 자세히 살펴보는 것부터 시작하겠습니다.
러스트가 정의하는 방식은 다음과 같습니다.

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

이 트레이트 정의는 새 타입 몇 개와 본 적 없는 문법도 포함하므로 정의를 조각
별로 살펴봅시다.

먼저 `Future`의 연관 타입 `Output`은 future가 무엇으로 해결되는지 말합니다.
이는 `Iterator` 트레이트의 `Item` 연관 타입과 유사합니다. 둘째, `Future`에는
`self` 매개변수로 특별한 `Pin` 참조를 받고 `Context` 타입에 대한 가변 참조를
받아 `Poll<Self::Output>`을 반환하는 `poll` 메서드가 있습니다. 잠시 후에 `Pin`
과 `Context`에 대해 더 이야기하겠습니다. 지금은 메서드가 반환하는 `Poll`
타입에 집중합시다.

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

이 `Poll` 타입은 `Option`과 유사합니다. 값을 가진 배리언트 `Ready(T)`와, 그렇지
않은 배리언트 `Pending`이 있습니다. 그러나 `Poll`은 `Option`과는 꽤 다른 의미
입니다! `Pending` 배리언트는 future가 여전히 할 일이 있으므로 호출자가 나중에
다시 확인해야 함을 나타냅니다. `Ready` 배리언트는 `Future`가 작업을 마쳤고
`T` 값이 사용 가능함을 나타냅니다.

> 참고: `poll`을 직접 호출해야 하는 경우는 드물지만, 그래야 한다면 대부분의
> future에서 future가 `Ready`를 반환한 후에 호출자가 `poll`을 다시 호출해서는
> 안 된다는 점을 염두에 두세요. 많은 future는 준비된 후 다시 폴링되면 패닉
> 합니다. 다시 폴링해도 안전한 future는 그 문서에서 명시적으로 그렇게
> 말합니다. 이는 `Iterator::next`의 동작과 유사합니다.

`await`를 사용하는 코드를 보면 러스트는 내부적으로 `poll`을 호출하는 코드로
컴파일합니다. 하나의 URL에 대한 페이지 제목이 해결되면 그것을 출력한 Listing
17-4를 되돌아보면, 러스트는 그것을 (정확히는 아니지만) 대략 다음과 같이
컴파일합니다.

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // But what goes here?
    }
}
```

future가 여전히 `Pending`일 때는 무엇을 해야 할까요? future가 마침내 준비될
때까지 몇 번이고 다시 시도할 어떤 방법이 필요합니다. 다시 말해 루프가 필요
합니다.

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // continue
        }
    }
}
```

그러나 러스트가 그대로 그 코드로 컴파일한다면 모든 `await`가 블로킹될 것
입니다. 우리가 원하는 것과 정반대입니다! 대신 러스트는 루프가 이 future의
작업을 일시 중지하고 다른 future에서 작업한 다음 나중에 이것을 다시 확인할
수 있는 무언가에 제어를 넘길 수 있도록 보장합니다. 우리가 보았듯 그 무언가가
async 런타임이며, 이 스케줄링과 조율 작업이 그 주요 일 중 하나입니다.

[“메시지 전달로 두 태스크 간 데이터 보내기”][message-passing]<!-- ignore -->
절에서 `rx.recv`를 기다리는 것을 설명했습니다. `recv` 호출은 future를 반환하고,
그 future를 기다리면 폴링합니다. 런타임은 채널이 닫힐 때 `Some(message)`든
`None`이든 준비될 때까지 future를 일시 중지한다고 언급했습니다. `Future`
트레이트, 구체적으로 `Future::poll`에 대한 더 깊은 이해로 이것이 어떻게 동작
하는지 볼 수 있습니다. `poll`이 `Poll::Pending`을 반환할 때 런타임은 future가
준비되지 않았음을 압니다. 반대로 `poll`이 `Poll::Ready(Some(message))`나
`Poll::Ready(None)`을 반환하면 런타임은 future가 준비되었음을 알고 진행합니다.

런타임이 이것을 어떻게 하는지에 대한 정확한 세부 사항은 이 책의 범위를 넘어
가지만, 핵심은 future의 기본 메커니즘을 보는 것입니다. 런타임은 자신이 책임
지는 각 future를 _폴링_ 하고, 준비되지 않은 경우 future를 다시 자도록 둡니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>
<a id="the-pin-and-unpin-traits"></a>

### `Pin` 타입과 `Unpin` 트레이트

Listing 17-13에서 `trpl::join!` 매크로로 세 future를 기다렸습니다. 그러나 런
타임까지는 알려지지 않을 어떤 수의 future를 담는 벡터 같은 컬렉션을 가지는
일이 흔합니다. Listing 17-13을 Listing 17-23의 코드로 바꿔서 세 future를
벡터에 넣고 대신 `trpl::join_all` 함수를 호출해 봅시다. 아직 컴파일되지
않습니다.

<Listing number="17-23" caption="컬렉션의 future 기다리기"  file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:here}}
```

</Listing>

12장의 “`run`에서 오류 반환하기” 절에서 한 것처럼 각 future를 _트레이트 객체_
로 만들기 위해 `Box`에 넣습니다. (트레이트 객체는 18장에서 자세히 다룹니다.)
트레이트 객체를 사용하면 이 타입들이 만들어 낸 각 익명 future를 같은 타입으로
다룰 수 있습니다. 모두 `Future` 트레이트를 구현하기 때문입니다.

이는 놀라울 수 있습니다. 어쨌든 어떤 async 블록도 아무것도 반환하지 않으므로
각각 `Future<Output = ()>`을 만들어 냅니다. 그러나 `Future`는 트레이트이고,
컴파일러는 출력 타입이 동일하더라도 각 async 블록에 대해 고유한 열거형을
만든다는 점을 기억하세요. 두 개의 서로 다른 손으로 쓴 구조체를 `Vec`에 넣을
수 없는 것처럼, 컴파일러가 생성한 열거형도 섞을 수 없습니다.

그런 다음 future 컬렉션을 `trpl::join_all` 함수에 전달하고 결과를 기다립니다.
그러나 이는 컴파일되지 않습니다. 오류 메시지의 관련 부분은 다음과 같습니다.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23
cargo build
copy *only* the final `error` block from the errors
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

이 오류 메시지의 note는 값을 _핀(pin)_ 하기 위해 `pin!` 매크로를 써야 한다고
알려 줍니다. 이는 값이 메모리에서 이동하지 않도록 보장하는 `Pin` 타입 안에
값을 넣는 것을 의미합니다. 오류 메시지는 `dyn Future<Output = ()>`가 `Unpin`
트레이트를 구현해야 하는데 현재는 그렇지 않기 때문에 pin이 필요하다고
말합니다.

`trpl::join_all` 함수는 `JoinAll`이라는 구조체를 반환합니다. 그 구조체는 타입
`F`에 대해 제네릭이며, `F`는 `Future` 트레이트를 구현하도록 제한됩니다.
`await`로 future를 직접 기다리면 future를 암묵적으로 핀합니다. 그래서 future
를 기다리고 싶은 모든 곳에서 `pin!`을 쓸 필요가 없습니다.

그러나 여기서는 future를 직접 기다리지 않습니다. 대신 `join_all` 함수에 future
컬렉션을 전달해 새 future인 JoinAll을 구성합니다. `join_all`의 시그니처는
컬렉션의 항목 타입이 모두 `Future` 트레이트를 구현하도록 요구하며, `Box<T>`는
자신이 감싸는 `T`가 `Unpin` 트레이트를 구현하는 future일 때만 `Future`를 구현
합니다.

흡수할 게 많습니다! 이를 정말로 이해하려면 `Future` 트레이트가 실제로 어떻게
동작하는지, 특히 pin 관련해서 조금 더 깊이 들어가 봅시다. `Future` 트레이트의
정의를 다시 봅시다.

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Required method
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

`cx` 매개변수와 그 `Context` 타입은 런타임이 여전히 게으르면서도 주어진 future
를 언제 확인할지 실제로 아는 방법의 핵심입니다. 다시 말하지만 그것이 어떻게
동작하는지 세부 사항은 이 장의 범위를 넘어가며, 보통 사용자 정의 `Future`
구현을 작성할 때만 이것에 대해 생각할 필요가 있습니다. 대신 `self`의 타입에
집중하겠습니다. 우리가 `self`에 타입 애너테이션이 있는 메서드를 본 것은 이번이
처음입니다. `self`에 대한 타입 애너테이션은 다른 함수 매개변수의 타입 애너
테이션처럼 동작하지만 두 가지 핵심 차이가 있습니다.

- 메서드가 호출되기 위해 `self`가 어떤 타입이어야 하는지 러스트에게 알려
  줍니다.
- 아무 타입이나 될 수는 없습니다. 메서드가 구현된 타입이나, 그 타입에 대한
  참조나 스마트 포인터, 또는 그 타입에 대한 참조를 감싸는 `Pin`으로 제한됩니다.

이 문법에 대해서는 [18장][ch-18]<!-- ignore -->에서 더 봅니다. 지금은 future를
폴링해 `Pending`인지 `Ready(Output)`인지 확인하려면 타입에 대한 `Pin`으로 감싼
가변 참조가 필요하다는 것만 알면 됩니다.

`Pin`은 `&`, `&mut`, `Box`, `Rc` 같은 포인터 같은 타입을 위한 래퍼입니다.
(기술적으로 `Pin`은 `Deref`나 `DerefMut` 트레이트를 구현하는 타입과 함께
동작하지만, 이는 참조와 스마트 포인터로만 작업하는 것과 사실상 동등합니다.)
`Pin`은 자체가 포인터가 아니며, `Rc`와 `Arc`가 참조 카운팅으로 하는 것처럼
자체 동작을 가지지도 않습니다. 포인터 사용에 제약을 강제하기 위해 컴파일러가
쓸 수 있는 순수한 도구입니다.

`await`가 `poll` 호출로 구현된다는 점을 떠올리면 앞서 본 오류 메시지 설명을
시작할 수 있지만, 그것은 `Pin`이 아니라 `Unpin`에 관한 것이었습니다. 그렇다면
`Pin`과 `Unpin`이 정확히 어떻게 관련되어 있고, `Future`가 `poll`을 호출하기
위해 `self`가 `Pin` 타입에 있어야 하는 이유는 무엇일까요?

이 장 앞부분에서 future의 일련의 await 지점들이 상태 기계로 컴파일되며, 컴파일
러는 그 상태 기계가 빌림과 소유권을 포함한 러스트의 모든 일반 안전 규칙을
따르도록 한다는 것을 떠올려 보세요. 그것을 가능하게 하기 위해 러스트는 한
await 지점과 다음 await 지점 또는 async 블록 끝 사이에 어떤 데이터가 필요한지
살핍니다. 그런 다음 컴파일된 상태 기계에 해당 배리언트를 만듭니다. 각 배리언트
는 그 소스 코드 섹션에서 사용될 데이터에 대한 필요한 접근 — 그 데이터의
소유권을 가져가든 가변 또는 불변 참조를 얻든 — 을 가집니다.

여기까지는 좋습니다. 주어진 async 블록에서 소유권이나 참조에 대해 잘못된 것이
있으면 빌림 검사기가 알려 줄 것입니다. 그 블록에 해당하는 future를 이리저리
옮기고 싶을 때 — `join_all`에 전달하기 위해 `Vec`에 넣는 것처럼 — 상황이 더
까다로워집니다.

future를 이동할 때 — `join_all`과 이터레이터로 쓰기 위해 자료 구조에 푸시
하든 함수에서 반환하든 — 그것은 실제로 러스트가 우리를 위해 만든 상태 기계를
이동하는 것을 의미합니다. 그리고 러스트의 대부분 다른 타입과 달리, 러스트가
async 블록용으로 만드는 future는 그림 17-4의 단순화된 그림에서 보이듯이 주어진
배리언트의 필드에 자기 자신에 대한 참조로 끝날 수 있습니다.

<figure>

<img alt="future fut1을 나타내는 단일 열, 3행 테이블. 처음 두 행에 데이터 값 0과 1이 있고, 세 번째 행에서 두 번째 행으로 화살표가 가리켜 future 내부의 내부 참조를 나타냅니다." src="img/trpl17-04.svg" class="center" />

<figcaption>그림 17-4: 자기 참조 자료 타입</figcaption>

</figure>

그러나 기본적으로 자기 자신에 대한 참조를 가진 객체는 이동하기에 안전하지
않습니다. 참조는 항상 그것이 참조하는 무언가의 실제 메모리 주소를 가리키기
때문입니다(그림 17-5 참고). 자료 구조 자체를 이동하면, 그 내부 참조는 예전
위치를 가리키게 남습니다. 그러나 그 메모리 위치는 이제 유효하지 않습니다. 한
가지로, 자료 구조를 변경할 때 그 값은 업데이트되지 않을 것입니다. 다른 — 더
중요한 — 점은 이제 컴퓨터가 그 메모리를 다른 목적으로 자유롭게 재사용할 수
있다는 것입니다! 나중에 완전히 관련 없는 데이터를 읽게 될 수 있습니다.

<figure>

<img alt="각각 하나의 열과 세 개의 행을 가진 fut1과 fut2라는 두 future를 묘사하는 두 테이블. future를 fut1에서 fut2로 이동한 결과를 나타냅니다. 첫 번째 fut1은 회색으로 표시되어 알 수 없는 메모리를 나타내는 물음표가 각 인덱스에 있습니다. 두 번째 fut2는 첫 번째와 두 번째 행에 0과 1이 있고, 세 번째 행에서 fut1의 두 번째 행으로 화살표가 가리켜 이동 전 future의 예전 메모리 위치를 참조하는 포인터를 나타냅니다." src="img/trpl17-05.svg" class="center" />

<figcaption>그림 17-5: 자기 참조 자료 타입을 이동한 안전하지 않은 결과</figcaption>

</figure>

이론적으로 러스트 컴파일러는 객체가 이동될 때마다 그 객체에 대한 모든 참조를
업데이트하려고 할 수 있지만, 이는 많은 성능 오버헤드를 더할 수 있습니다. 특히
참조의 전체 웹이 업데이트되어야 한다면 말이죠. 대신 해당 자료 구조가 _메모리
에서 이동하지 않도록_ 확인할 수 있다면 어떤 참조도 업데이트할 필요가 없습니다.
이것이 바로 러스트의 빌림 검사기가 하는 일입니다. 안전한 코드에서는 활성
참조가 있는 어떤 항목도 이동하지 못하게 막습니다.

`Pin`은 그 위에 쌓아 우리에게 필요한 정확한 보장을 제공합니다. 값에 대한
포인터를 `Pin`으로 감싸 _핀_ 하면 더 이상 이동할 수 없습니다. 따라서
`Pin<Box<SomeType>>`이 있으면 `Box` 포인터가 _아니라_ 실제로 `SomeType` 값을
핀합니다. 그림 17-6은 이 과정을 보여 줍니다.

<figure>

<img alt="나란히 배치된 세 개의 상자. 첫 번째는 “Pin”, 두 번째는 “b1”, 세 번째는 “pinned”로 표시됩니다. “pinned” 안에는 “fut”라는 테이블이 있으며, 단일 열을 가지며 자료 구조의 각 부분에 해당하는 셀이 있는 future를 나타냅니다. 첫 셀에 값 “0”, 두 번째 셀에서 네 번째이자 마지막 셀로 화살표가 나가고, 그 셀에 값 “1”이 있으며, 세 번째 셀에는 점선과 생략 부호가 있어 자료 구조의 다른 부분이 있을 수 있음을 나타냅니다. 전체적으로 “fut” 테이블은 자기 참조적인 future를 나타냅니다. “Pin” 상자에서 화살표가 나와 “b1”을 거쳐 “pinned” 안의 “fut” 테이블에서 끝납니다." src="img/trpl17-06.svg" class="center" />

<figcaption>그림 17-6: 자기 참조 future 타입을 가리키는 `Box` 핀하기</figcaption>

</figure>

사실 `Box` 포인터는 여전히 자유롭게 이동할 수 있습니다. 기억하세요. 우리가
중요하게 여기는 것은 궁극적으로 참조되는 데이터가 제자리에 있도록 확인하는
것입니다. 그림 17-7처럼 포인터가 이동하지만 _그것이 가리키는 데이터_ 는 같은
장소에 있다면 잠재적 문제가 없습니다. (독립적인 연습으로, `Pin`이 `Box`를
감싼 경우에 이를 어떻게 할지 알아내기 위해 이 타입들과 `std::pin` 모듈의
문서를 살펴보세요.) 핵심은 자기 참조 타입 자체가 여전히 핀되어 있기 때문에
이동할 수 없다는 것입니다.

<figure>

<img alt="앞 다이어그램과 동일하게 세 대강의 열로 배치된 네 개의 상자. 두 번째 열이 바뀌었습니다. 이제 두 번째 열에는 “b1”과 “b2”라는 두 상자가 있고, “b1”은 회색으로 표시되며 “Pin”에서의 화살표는 “b1” 대신 “b2”를 통과해 포인터가 “b1”에서 “b2”로 이동했지만 “pinned”의 데이터는 이동하지 않았음을 나타냅니다." src="img/trpl17-07.svg" class="center" />

<figcaption>그림 17-7: 자기 참조 future 타입을 가리키는 `Box`를 이동하기</figcaption>

</figure>

그러나 대부분의 타입은 `Pin` 포인터 뒤에 있더라도 완전히 안전하게 이동할 수
있습니다. 항목에 내부 참조가 있을 때만 pin에 대해 생각할 필요가 있습니다. 수와
불(Boolean) 같은 원시 값은 명백히 내부 참조가 없으므로 안전합니다. 러스트에서
보통 다루는 대부분의 타입도 마찬가지입니다. 예를 들어 걱정 없이 `Vec`를 이동
할 수 있습니다. 지금까지 본 것을 고려하면 `Pin<Vec<String>>`이 있다면 `Vec<String>`
에 다른 참조가 없을 때는 항상 이동이 안전함에도 `Pin`이 제공하는 안전하지만
제한적인 API를 통해 모든 일을 해야 합니다. 이런 경우에 항목을 이동해도 괜찮다고
컴파일러에게 알릴 방법이 필요합니다. 그리고 그것이 `Unpin`이 등장하는 지점입니다.

`Unpin`은 16장에서 본 `Send`와 `Sync` 트레이트와 유사한 마커 트레이트이므로,
자체 기능이 없습니다. 마커 트레이트는 주어진 트레이트를 구현한 타입을 특정
맥락에서 쓰는 것이 안전하다고 컴파일러에게 알리기 위해서만 존재합니다. `Unpin`
은 주어진 타입이 해당 값이 안전하게 이동될 수 있는지에 대한 어떤 보장도 유지
할 필요가 _없다_ 고 컴파일러에게 알립니다.

<!--
  The inline `<code>` in the next block is to allow the inline `<em>` inside it,
  matching what NoStarch does style-wise, and emphasizing within the text here
  that it is something distinct from a normal type.
-->

`Send`와 `Sync`처럼, 컴파일러는 안전하다고 증명할 수 있는 모든 타입에 대해
자동으로 `Unpin`을 구현합니다. `Send`와 `Sync`와 유사한 특수한 경우는
`Unpin`이 타입에 대해 _구현되지 않는_ 경우입니다. 이에 대한 표기는
<code>impl !Unpin for <em>SomeType</em></code>이며, <code><em>SomeType</em></code>
은 `Pin` 안의 그 타입에 대한 포인터가 사용될 때마다 그 보장을 유지해야 안전한
타입의 이름입니다.

다시 말해 `Pin`과 `Unpin`의 관계에 대해 염두에 둘 두 가지가 있습니다. 첫째,
`Unpin`이 “정상” 경우이고 `!Unpin`은 특수 경우입니다. 둘째, 타입이 `Unpin`을
구현하는지 `!Unpin`을 구현하는지는 <code>Pin<&mut <em>SomeType</em>></code>
처럼 그 타입에 대한 핀된 포인터를 사용할 때 _에만_ 중요합니다.

이를 구체화하기 위해 `String`을 생각해 보세요. 길이와 이를 구성하는 유니코드
문자가 있습니다. 그림 17-8처럼 `String`을 `Pin`으로 감쌀 수 있습니다. 그러나
`String`은 러스트의 대부분 다른 타입처럼 자동으로 `Unpin`을 구현합니다.

<figure>

<img alt="왼쪽에 “Pin”이라는 레이블이 붙은 상자와 오른쪽에 “String”이라는 레이블이 붙은 상자로 향하는 화살표. “String” 상자는 문자열의 길이를 나타내는 데이터 5usize와, 이 String 인스턴스에 저장된 문자열 “hello”의 문자를 나타내는 문자 “h”, “e”, “l”, “l”, “o”를 담습니다. 점선 직사각형이 “String” 상자와 그 레이블을 둘러싸지만, “Pin” 상자는 둘러싸지 않습니다." src="img/trpl17-08.svg" class="center" />

<figcaption>그림 17-8: `String` 핀하기. 점선은 `String`이 `Unpin` 트레이트를 구현하므로 핀되지 않았음을 나타냅니다</figcaption>

</figure>

그 결과 `String`이 대신 `!Unpin`을 구현했다면 허용되지 않을 일을 할 수 있습
니다. 그림 17-9처럼 메모리의 정확히 같은 위치에서 한 문자열을 다른 문자열로
교체하는 것 같은 일입니다. 이는 `Pin` 계약을 위반하지 않습니다. `String`에는
이동을 안전하지 않게 만드는 내부 참조가 없기 때문입니다. 그것이 바로 `!Unpin`
이 아니라 `Unpin`을 구현하는 이유입니다.

<figure>

<img alt="앞 예제의 같은 “hello” 문자열 데이터가 이제 “s1”이라고 레이블되고 회색으로 표시됩니다. 앞 예제의 “Pin” 상자는 이제 “s2”라고 레이블된 다른 String 인스턴스를 가리킵니다. s2는 유효하고 길이가 7usize이며 문자열 “goodbye”의 문자를 담습니다. s2도 Unpin 트레이트를 구현하므로 점선 직사각형으로 둘러싸입니다." src="img/trpl17-09.svg" class="center" />

<figcaption>그림 17-9: 메모리에서 `String`을 완전히 다른 `String`으로 교체하기</figcaption>

</figure>

이제 앞 Listing 17-23의 `join_all` 호출에 대해 보고된 오류를 이해할 만큼 알게
되었습니다. 우리는 원래 async 블록이 만들어 낸 future를 `Vec<Box<dyn Future<Output = ()>>>`
로 이동하려 했지만, 본 것처럼 그 future들은 내부 참조를 가질 수 있으므로
자동으로 `Unpin`을 구현하지 않습니다. 일단 핀하면 결과 `Pin` 타입을 `Vec`에
전달할 수 있으며, future의 기저 데이터가 이동되지 _않을_ 것임을 확신할 수
있습니다. Listing 17-24는 세 future가 정의된 각 곳에서 `pin!` 매크로를 호출
하고 트레이트 객체 타입을 조정해 코드를 고치는 방법을 보여 줍니다.

<Listing number="17-24" caption="future들이 벡터로 이동될 수 있도록 핀하기">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

이 예제는 이제 컴파일되고 실행됩니다. 런타임에 벡터에서 future를 추가하거나
제거하고 그들을 모두 조인할 수 있습니다.

`Pin`과 `Unpin`은 대부분 저수준 라이브러리나 런타임 자체를 만들 때 중요하며,
일상적인 러스트 코드에는 덜 중요합니다. 그러나 오류 메시지에서 이 트레이트
들을 볼 때, 이제 코드를 어떻게 고칠지에 대한 더 나은 아이디어를 갖게 될
것입니다!

> 참고: `Pin`과 `Unpin`의 이 조합은 자기 참조적이어서 도전적이었을 복잡한
> 타입들의 전체 부류를 러스트에서 안전하게 구현하는 것을 가능하게 합니다.
> `Pin`이 필요한 타입은 오늘날 async 러스트에서 가장 흔하게 나타나지만,
> 가끔은 다른 맥락에서도 볼 수 있습니다.
>
> `Pin`과 `Unpin`이 어떻게 동작하고 유지해야 할 규칙에 대한 구체적인 내용은
> `std::pin`의 API 문서에 광범위하게 다뤄지므로, 더 알고 싶다면 거기서
> 시작하는 것이 좋습니다.
>
> 내부에서 일이 어떻게 동작하는지 더 세부적으로 이해하고 싶다면, [_Asynchronous
> Programming in Rust_][async-book]의 [2장][under-the-hood]<!-- ignore -->과
> [4장][pinning]<!-- ignore -->을 보세요.

### `Stream` 트레이트

이제 `Future`, `Pin`, `Unpin` 트레이트에 대한 더 깊은 이해를 가졌으니 `Stream`
트레이트에 주의를 돌릴 수 있습니다. 이 장 앞부분에서 배운 대로, stream은
비동기 이터레이터와 유사합니다. 그러나 `Iterator`와 `Future`와 달리 이 글을
쓰는 시점에 `Stream`은 표준 라이브러리에 정의가 없습니다. 하지만 생태계
전반에서 쓰이는 `futures` 크레이트의 매우 흔한 정의는 _있습니다_.

`Stream` 트레이트가 어떻게 그것들을 합칠지 보기 전에 `Iterator`와 `Future`
트레이트 정의를 복습해 봅시다. `Iterator`에서는 순차 아이디어가 있습니다. 그
`next` 메서드는 `Option<Self::Item>`을 제공합니다. `Future`에서는 시간에
걸친 준비성의 아이디어가 있습니다. 그 `poll` 메서드는 `Poll<Self::Output>`
을 제공합니다. 시간에 걸쳐 준비되는 일련의 항목을 나타내기 위해 이 기능들을
합친 `Stream` 트레이트를 정의합니다.

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

`Stream` 트레이트는 stream이 만들어 내는 항목의 타입에 대한 `Item`이라는 연관
타입을 정의합니다. 이는 `Iterator`와 유사하게 0에서 여러 개일 수 있으며,
`Future`와 달리 항상 단 하나의 `Output`이 있는 것과는 다릅니다. 비록 유닛
타입 `()`이라도 말이죠.

`Stream`은 또한 그 항목들을 얻는 메서드를 정의합니다. 우리는 그것을 `poll_next`
라고 부르는데, 이는 `Future::poll`과 같은 방식으로 폴링하고 `Iterator::next`
와 같은 방식으로 일련의 항목을 만들어 낸다는 점을 명확히 하기 위함입니다.
그 반환 타입은 `Poll`과 `Option`을 결합합니다. 바깥 타입은 `Poll`입니다. future
처럼 준비성을 확인해야 하기 때문입니다. 안쪽 타입은 `Option`입니다. 이터레이터
처럼 더 많은 메시지가 있는지를 알려야 하기 때문입니다.

이 정의와 매우 유사한 것이 러스트 표준 라이브러리의 일부로 끝날 가능성이
높습니다. 그동안 대부분의 런타임 도구 모음의 일부이므로, 의존해도 되고, 다음에
다루는 모든 것은 일반적으로 적용되어야 합니다!

[“Stream: 순차적인 Future”][streams]<!-- ignore --> 절에서 본 예제에서는
그러나 `poll_next` _나_ `Stream`을 사용하지 않고, 대신 `next`와 `StreamExt`를
사용했습니다. 물론 우리는 future의 `poll` 메서드로 직접 작업할 _수 있는_ 것
처럼 자체 `Stream` 상태 기계를 손으로 작성해 `poll_next` API로 직접 작업할
_수 있습니다_. 그러나 `await`를 사용하는 것이 훨씬 좋고, `StreamExt` 트레이트
는 바로 그 일을 할 수 있도록 `next` 메서드를 제공합니다.

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

<!--
TODO: update this if/when tokio/etc. update their MSRV and switch to using async functions
in traits, since the lack thereof is the reason they do not yet have this.
-->

> 참고: 이 장 앞부분에서 썼던 실제 정의는 이와 약간 다르게 생겼습니다.
> 트레이트에서 async 함수 사용이 아직 지원되지 않던 러스트 버전을 지원하기
> 때문입니다. 결과적으로 다음과 같이 생겼습니다.
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> 그 `Next` 타입은 `Future`를 구현하는 `struct`로, `Next<'_, Self>`로 `self`
> 에 대한 참조의 라이프타임을 명명해 `await`가 이 메서드와 동작할 수 있게
> 합니다.

`StreamExt` 트레이트는 또한 stream과 함께 사용할 수 있는 모든 흥미로운 메서드
의 보금자리입니다. `StreamExt`는 `Stream`을 구현하는 모든 타입에 대해 자동
으로 구현되지만, 기반 트레이트에 영향을 주지 않고 편의 API를 반복할 수 있도록
커뮤니티를 위해 이 트레이트들은 별개로 정의됩니다.

`trpl` 크레이트에서 쓴 `StreamExt` 버전에서 이 트레이트는 `next` 메서드를
정의할 뿐만 아니라, `Stream::poll_next` 호출의 세부 사항을 올바르게 처리하는
`next`의 기본 구현도 제공합니다. 즉 자체 stream 자료 타입을 작성해야 할 때에도
`Stream`만 구현하면 되고, 그러면 여러분의 자료 타입을 사용하는 누구든 자동으로
`StreamExt`와 그 메서드를 사용할 수 있습니다.

저수준 세부 사항에 대해 다룰 것은 이것이 전부입니다. 마무리하며 future(stream
포함), 태스크, 스레드가 모두 어떻게 맞물리는지 생각해 봅시다!

[message-passing]: ch17-02-concurrency-with-async.md#sending-data-between-two-tasks-using-message-passing
[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#our-first-async-program
[any-number-futures]: ch17-03-more-futures.html#working-with-any-number-of-futures
[streams]: ch17-04-streams.html
