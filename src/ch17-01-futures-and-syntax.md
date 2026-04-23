## Future와 Async 문법

러스트의 비동기 프로그래밍의 핵심 요소는 _future_ 와 러스트의 `async`, `await`
키워드입니다.

_future_ 는 지금은 준비되지 않았을 수 있지만 언젠가 미래에 준비될 값입니다.
(이 같은 개념은 많은 언어에 _task_ 나 _promise_ 같은 다른 이름으로 나타납니다.)
러스트는 `Future` 트레이트를 기본 구성 요소로 제공하므로, 서로 다른 비동기
연산을 서로 다른 자료 구조로 구현하되 공통 인터페이스를 갖도록 할 수 있습니다.
러스트에서 future는 `Future` 트레이트를 구현한 타입입니다. 각 future는 자신의
진행 상황과 “준비됨”이 무엇인지에 관한 자체 정보를 담고 있습니다.

블록과 함수에 `async` 키워드를 적용해 그것들이 중단되고 재개될 수 있음을
명시할 수 있습니다. async 블록이나 async 함수 내에서는 `await` 키워드로
_future를 기다릴_(즉 준비되기를 기다릴) 수 있습니다. async 블록이나 함수
내에서 future를 기다리는 지점은 모두 그 블록이나 함수가 일시 중지되고 재개될
수 있는 잠재적 지점입니다. future에 값이 이미 사용 가능한지 확인하는 과정을
_폴링(polling)_ 이라고 합니다.

C#과 자바스크립트 같은 다른 일부 언어도 async 프로그래밍에 `async`와 `await`
키워드를 사용합니다. 그 언어들에 익숙하다면 러스트의 문법 처리 방식에 몇 가지
중요한 차이를 알아차릴 수 있습니다. 거기에는 좋은 이유가 있으며, 곧 보게
될 것입니다!

러스트로 async 코드를 작성할 때 대부분의 시간을 `async`와 `await` 키워드와
함께 보냅니다. 러스트는 `for` 루프를 `Iterator` 트레이트를 사용한 동등한 코드
로 컴파일하듯, 이들을 `Future` 트레이트를 사용한 동등한 코드로 컴파일합니다.
그러나 러스트가 `Future` 트레이트를 제공하기 때문에, 필요할 때 자신만의
자료 타입에 대해서도 구현할 수 있습니다. 이 장 전체에서 볼 많은 함수들은
자체 `Future` 구현을 가진 타입을 반환합니다. 이 트레이트의 정의는 이 장
마지막에서 다시 살피고 동작 방식을 더 깊이 파헤치겠지만, 지금은 앞으로 나아
가기에 이 정도 세부 사항이면 충분합니다.

이 모든 것이 다소 추상적으로 느껴질 수 있으니, 첫 async 프로그램인 작은 웹
스크래퍼를 작성해 봅시다. 명령행에서 두 URL을 전달받아 둘을 동시에 가져오고,
먼저 완료되는 쪽의 결과를 반환하겠습니다. 이 예제는 새 문법이 꽤 많이 있겠
지만, 걱정 마세요. 필요한 모든 것을 진행하면서 설명하겠습니다.

## 첫 Async 프로그램

생태계의 여러 부분을 저글링하기보다 async 학습에 집중하기 위해, 우리는 `trpl`
크레이트를 만들었습니다(`trpl`은 “The Rust Programming Language”의 줄임말).
이 크레이트는 여러분에게 필요한 모든 타입, 트레이트, 함수를 주로
[`futures`][futures-crate]<!-- ignore -->와 [`tokio`][tokio]<!-- ignore -->
크레이트로부터 재내보내기합니다. `futures` 크레이트는 async 코드에 대한
러스트 실험의 공식적 보금자리이며, 실제로 `Future` 트레이트가 원래 설계된
곳입니다. 토키오(Tokio)는 오늘날 러스트에서 가장 널리 쓰이는 async 런타임
이며, 특히 웹 애플리케이션에 많이 쓰입니다. 훌륭한 다른 런타임도 있고, 그
런타임들이 여러분 목적에 더 적합할 수도 있습니다. 우리는 `trpl`이 잘
테스트되고 널리 쓰이기 때문에 내부적으로 `tokio` 크레이트를 사용합니다.

일부 경우 `trpl`은 이 장과 관련된 세부 사항에 집중하도록 원래 API의 이름을
바꾸거나 감쌉니다. 크레이트가 무엇을 하는지 이해하고 싶다면, [그 소스
코드][crate-source]를 확인해 보기를 권합니다. 각 재내보내기가 어느 크레이트에서
왔는지 볼 수 있고, 크레이트가 무엇을 하는지 설명하는 광범위한 주석을 남겨
두었습니다.

`hello-async`라는 이름의 새 바이너리 프로젝트를 만들고 `trpl` 크레이트를
의존성으로 추가하세요.

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

이제 `trpl`이 제공하는 여러 조각을 사용해 첫 async 프로그램을 작성할 수
있습니다. 두 웹 페이지를 가져와 각각에서 `<title>` 요소를 뽑고, 전체 과정을
먼저 마친 페이지의 제목을 출력하는 작은 명령행 도구를 만들 것입니다.

### page_title 함수 정의하기

한 페이지 URL을 매개변수로 받아 그것에 요청을 보내고 `<title>` 요소의 텍스트를
반환하는 함수를 작성하는 것부터 시작합시다(Listing 17-1 참고).

<Listing number="17-1" file-name="src/main.rs" caption="HTML 페이지의 title 요소를 가져오는 async 함수 정의하기">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

먼저 `page_title`이라는 함수를 정의하고 `async` 키워드로 표시합니다. 그런 다음
`trpl::get` 함수로 전달된 URL을 가져오고, `await` 키워드를 추가해 응답을
기다립니다. `response`의 텍스트를 얻기 위해 그 `text` 메서드를 호출하고,
다시 `await` 키워드로 그것을 기다립니다. 이 두 단계 모두 비동기적입니다.
`get` 함수의 경우 서버가 응답의 첫 부분을 보낼 때까지 기다려야 하며, 이
부분은 HTTP 헤더, 쿠키 등을 포함하고 응답 본문과 별도로 전달될 수 있습니다.
특히 본문이 매우 크면 전부 도착하는 데 시간이 걸릴 수 있습니다. 응답 _전부_
가 도착하기를 기다려야 하기 때문에 `text` 메서드도 비동기적입니다.

이 두 future를 모두 명시적으로 기다려야 합니다. 러스트의 future는 _게으르기_
때문입니다. `await` 키워드로 요청하기 전까지는 아무 일도 하지 않습니다.
(사실 future를 사용하지 않으면 러스트는 컴파일러 경고를 보여 줍니다.) 이는
13장의 [“이터레이터로 연속된 항목 처리하기”][iterators-lazy]<!-- ignore -->
절에서 이터레이터에 대한 논의를 떠올리게 할 수 있습니다. 이터레이터는
`next` 메서드를 호출하기 전까지는 — 직접 호출하든 `for` 루프나 내부적으로
`next`를 쓰는 `map` 같은 메서드를 사용하든 — 아무 일도 하지 않습니다. 마찬가지
로 future도 명시적으로 요청하기 전까지 아무 일도 하지 않습니다. 이 게으름 덕분
에 러스트는 실제로 필요하기 전까지는 async 코드를 실행하지 않을 수 있습니다.

> 참고: 이것은 16장의 [“`spawn`으로 새 스레드 만들기”][thread-spawn]<!-- ignore -->
> 절에서 `thread::spawn`을 사용할 때 본 동작과 다릅니다. 거기서는 다른 스레드
> 에 전달한 클로저가 즉시 실행되기 시작했습니다. 많은 다른 언어가 async에
> 접근하는 방식과도 다릅니다. 그러나 이터레이터처럼 러스트가 성능 보장을
> 제공할 수 있도록 하는 데 중요합니다.

`response_text`를 얻으면 `Html::parse`를 사용해 `Html` 타입의 인스턴스로
파싱할 수 있습니다. 이제 원시 문자열 대신 HTML을 더 풍부한 자료 구조로 다룰
수 있는 자료 타입이 있습니다. 특히 `select_first` 메서드를 사용해 주어진 CSS
선택자의 첫 인스턴스를 찾을 수 있습니다. 문자열 `"title"`을 전달하면, 문서에
있다면 첫 번째 `<title>` 요소를 얻습니다. 일치하는 요소가 없을 수 있으므로
`select_first`는 `Option<ElementRef>`를 반환합니다. 마지막으로 `Option::map`
메서드를 사용하는데, 이는 `Option`에 있을 때 항목과 작업하게 해 주고 없으면
아무 일도 하지 않게 해 줍니다. (여기에 `match` 식을 사용할 수도 있지만
`map`이 더 관용적입니다.) `map`에 제공한 함수의 본문에서 `title`에 `inner_html`
을 호출해 그 내용을 얻는데, 이는 `String`입니다. 모든 것을 마치면 `Option<String>`
을 갖게 됩니다.

러스트의 `await` 키워드는 기다리려는 식 _앞_ 이 아니라 _뒤_ 에 온다는 점에
유의하세요. 즉 _후위(postfix)_ 키워드입니다. 다른 언어에서 `async`를 써 봤다면
익숙한 것과 다를 수 있지만, 러스트에서는 메서드 체인 작업이 훨씬 좋아집니다.
그 결과 Listing 17-2처럼 `page_title`의 본문을 `trpl::get`과 `text` 함수 호출을
사이사이 `await`로 함께 체인하도록 바꿀 수 있습니다.

<Listing number="17-2" file-name="src/main.rs" caption="`await` 키워드로 체이닝">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

이로써 첫 async 함수를 성공적으로 작성했습니다! `main`에 호출하는 코드를
추가하기 전에 우리가 작성한 것과 그 의미에 대해 조금 더 이야기해 봅시다.

러스트가 `async` 키워드로 표시된 _블록_ 을 보면, `Future` 트레이트를 구현하는
고유한 익명 자료 타입으로 컴파일합니다. 러스트가 `async`로 표시된 _함수_ 를
보면, 본문이 async 블록인 비동기가 아닌 함수로 컴파일합니다. async 함수의
반환 타입은 컴파일러가 그 async 블록을 위해 만든 익명 자료 타입의 타입입니다.

따라서 `async fn`을 작성하는 것은 반환 타입의 _future_ 를 반환하는 함수를
작성하는 것과 동등합니다. 컴파일러에게 Listing 17-1의 `async fn page_title`
같은 함수 정의는 대략 다음과 같이 정의된 비동기가 아닌 함수와 동등합니다.

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

변환된 버전의 각 부분을 살펴봅시다.

- 10장의 [“매개변수로서의 트레이트”][impl-trait]<!-- ignore --> 절에서 논의한
  `impl Trait` 문법을 사용합니다.
- 반환된 값은 연관 타입 `Output`을 가진 `Future` 트레이트를 구현합니다.
  `Output` 타입이 `async fn` 버전의 `page_title`의 원래 반환 타입과 같은
  `Option<String>`임에 유의하세요.
- 원래 함수의 본문에서 호출된 모든 코드는 `async move` 블록으로 감싸져
  있습니다. 블록은 식임을 기억하세요. 이 전체 블록이 함수에서 반환되는
  식입니다.
- 이 async 블록은 방금 설명한 것처럼 `Option<String>` 타입의 값을 생성합니다.
  그 값은 반환 타입의 `Output` 타입과 일치합니다. 이는 여러분이 본 다른
  블록과 같습니다.
- 새 함수 본문은 `url` 매개변수를 사용하는 방식 때문에 `async move` 블록
  입니다. (`async`와 `async move`에 대해서는 이 장 나중에 훨씬 더 이야기하겠
  습니다.)

이제 `main`에서 `page_title`을 호출할 수 있습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id ="determining-a-single-pages-title"></a>

### 런타임으로 Async 함수 실행하기

우선 Listing 17-3처럼 하나의 페이지에 대한 제목을 얻겠습니다. 안타깝게도 이
코드는 아직 컴파일되지 않습니다.

<Listing number="17-3" file-name="src/main.rs" caption="사용자가 제공한 인수로 `main`에서 `page_title` 함수 호출하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

12장의 [“명령행 인수 받기”][cli-args]<!-- ignore --> 절에서 명령행 인수를
얻었던 같은 패턴을 따릅니다. 그런 다음 URL 인수를 `page_title`에 전달하고
결과를 기다립니다. future가 만들어 내는 값이 `Option<String>`이므로, 페이지에
`<title>`이 있었는지에 따라 다른 메시지를 출력하기 위해 `match` 식을 사용합
니다.

`await` 키워드를 사용할 수 있는 유일한 장소는 async 함수나 블록 안이며, 러스트
는 특별한 `main` 함수를 `async`로 표시하도록 허용하지 않습니다.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

`main`이 `async`로 표시될 수 없는 이유는 async 코드에는 _런타임(runtime)_, 즉
비동기 코드 실행의 세부 사항을 관리하는 러스트 크레이트가 필요하기 때문입니다.
프로그램의 `main` 함수는 런타임을 _초기화_ 할 수는 있지만, 그 _자체_ 가 런타임
은 아닙니다. (잠시 후 그 이유를 더 보겠습니다.) async 코드를 실행하는 모든
러스트 프로그램은 future를 실행하는 런타임을 설정하는 장소가 최소 하나는
있습니다.

async를 지원하는 대부분의 언어는 런타임을 번들하지만, 러스트는 그렇지
않습니다. 대신 여러 다른 async 런타임이 있고, 각각 자기 사용 사례에 적합한
다른 트레이드오프를 만듭니다. 예를 들어 많은 CPU 코어와 큰 RAM을 가진 고처리량
웹 서버는 단일 코어, 작은 RAM, 힙 할당 능력이 없는 마이크로컨트롤러와는 매우
다른 요구를 가집니다. 그 런타임을 제공하는 크레이트들은 파일이나 네트워크
I/O 같은 흔한 기능의 async 버전도 함께 제공하는 경우가 많습니다.

여기와 이 장 나머지에서는 `trpl` 크레이트의 `block_on` 함수를 사용하겠습니다.
이 함수는 future를 인수로 받고, 이 future가 완료될 때까지 현재 스레드를
블록합니다. 내부적으로 `block_on`을 호출하면 전달된 future를 실행하는 데
사용되는 `tokio` 크레이트를 사용해 런타임을 설정합니다(`trpl` 크레이트의
`block_on` 동작은 다른 런타임 크레이트의 `block_on` 함수와 비슷합니다).
future가 완료되면 `block_on`은 future가 만들어 낸 값을 반환합니다.

`page_title`이 반환한 future를 `block_on`에 직접 전달하고, 완료되면 Listing
17-3에서 하려 한 것처럼 결과 `Option<String>`에 `match`를 할 수 있습니다.
그러나 이 장의 대부분의 예제(그리고 실제 세계의 대부분 async 코드)에서는 단
한 번의 async 함수 호출 이상을 할 것이므로, 대신 `async` 블록을 전달하고
`page_title` 호출의 결과를 명시적으로 기다리겠습니다. Listing 17-4처럼요.

<Listing number="17-4" caption="`trpl::block_on`으로 async 블록 기다리기" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

이 코드를 실행하면 애초에 기대한 동작을 얻습니다.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run -- "https://www.rust-lang.org"
# copy the output here
-->

```console
$ cargo run -- "https://www.rust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

휴 — 드디어 동작하는 async 코드가 생겼습니다! 그러나 두 사이트를 서로 경쟁
시키는 코드를 추가하기 전에, future가 어떻게 동작하는지에 잠깐 주의를
돌려 봅시다.

각 _await 지점(await point)_, 즉 코드가 `await` 키워드를 사용하는 모든 장소는
제어가 런타임에 넘겨지는 지점을 나타냅니다. 그것이 동작하도록 하려면, 러스트는
async 블록에 관련된 상태를 추적해야 합니다. 그래서 런타임이 다른 작업을
시작한 다음, 첫 작업을 다시 진행할 준비가 되면 돌아올 수 있도록 말이죠. 이는
보이지 않는 상태 기계(state machine)로, 각 await 지점에서 현재 상태를 저장
하도록 다음과 같은 열거형을 작성한 것과 마찬가지입니다.

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

그러나 각 상태 간 전환 코드를 손으로 작성하는 것은 지루하고 오류가 나기
쉽습니다. 특히 나중에 기능과 상태를 더 추가해야 할 때 말이죠. 다행히 러스트
컴파일러가 async 코드의 상태 기계 자료 구조를 자동으로 만들고 관리해 줍니다.
자료 구조 주변의 일반 빌림과 소유권 규칙은 모두 여전히 적용되며, 다행히
컴파일러가 그것을 검사하는 것까지 처리해 주고 유용한 오류 메시지를 제공합니다.
이 장 나중에 그중 몇 가지를 살펴보겠습니다.

궁극적으로 무언가가 이 상태 기계를 실행해야 하며, 그 무언가는 런타임입니다.
(그래서 런타임을 알아볼 때 _executor(실행기)_ 라는 언급을 마주칠 수 있습니다.
실행기는 async 코드의 실행을 담당하는 런타임의 부분입니다.)

이제 컴파일러가 Listing 17-3에서 왜 `main` 자체를 async 함수로 만들지 못하게
했는지 알 수 있습니다. `main`이 async 함수였다면, `main`이 반환하는 어떤
future든 그 상태 기계를 다른 무언가가 관리해야 할 텐데, `main`은 프로그램의
시작점입니다! 대신 `main`에서 `trpl::block_on` 함수를 호출해 런타임을 설정하고
`async` 블록이 반환한 future를 완료될 때까지 실행했습니다.

> 참고: 일부 런타임은 매크로를 제공해 async `main` 함수를 _쓸 수_ 있게 해
> 줍니다. 그 매크로들은 `async fn main() { ... }`을 일반 `fn main`으로 다시
> 쓰는데, 이는 Listing 17-4에서 손으로 한 것과 같은 일을 합니다. `trpl::block_on`
> 이 하는 방식대로 future를 완료될 때까지 실행하는 함수를 호출합니다.

이제 이 조각들을 합쳐서 어떻게 동시적 코드를 작성할 수 있는지 봅시다.

<!-- Old headings. Do not remove or links may break. -->

<a id="racing-our-two-urls-against-each-other"></a>

### 두 URL을 동시에 경쟁시키기

Listing 17-5에서는 명령행에서 전달된 두 다른 URL로 `page_title`을 호출하고,
먼저 끝나는 future를 선택해 경쟁시킵니다.

<Listing number="17-5" caption="어느 URL이 먼저 반환되는지 보기 위해 두 URL에 대해 `page_title` 호출하기" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

사용자가 제공한 각 URL로 `page_title`을 호출하는 것부터 시작합니다. 결과
future를 `title_fut_1`과 `title_fut_2`로 저장합니다. future는 게으르고 아직
기다리지 않았기 때문에 아직 아무 일도 하지 않음을 기억하세요. 그런 다음
future를 `trpl::select`에 전달하면, 전달된 future 중 어느 것이 먼저 완료
되는지를 나타내는 값을 반환합니다.

> 참고: 내부적으로 `trpl::select`는 `futures` 크레이트에 정의된 더 일반적인
> `select` 함수 위에 구축되어 있습니다. `futures` 크레이트의 `select` 함수는
> `trpl::select`가 할 수 없는 많은 일을 할 수 있지만, 지금 건너뛸 수 있는
> 추가 복잡성도 있습니다.

어느 future든 정당하게 “이길” 수 있으므로 `Result`를 반환하는 것은 말이
안 됩니다. 대신 `trpl::select`는 우리가 본 적 없는 타입 `trpl::Either`를
반환합니다. `Either` 타입은 두 경우를 가진다는 점에서 `Result`와 다소
비슷합니다. 그러나 `Result`와 달리 `Either`에는 성공이나 실패의 개념이
내장되어 있지 않습니다. 대신 `Left`와 `Right`를 사용해 “둘 중 하나”를
나타냅니다.

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

`select` 함수는 첫 번째 인수가 이기면 그 future의 출력과 함께 `Left`를
반환하고, 두 번째 future 인수가 이기면 _그_ 출력과 함께 `Right`를 반환합니다.
이는 함수를 호출할 때 인수가 나타나는 순서와 일치합니다. 첫 번째 인수가 두
번째 인수의 왼쪽에 있습니다.

또한 전달된 같은 URL을 반환하도록 `page_title`을 업데이트합니다. 그러면 먼저
반환된 페이지에 해결할 수 있는 `<title>`이 없더라도 여전히 의미 있는 메시지를
출력할 수 있습니다. 그 정보를 사용 가능하게 두고, 어느 URL이 먼저 끝났는지와,
있다면 그 URL의 웹 페이지에 대한 `<title>`이 무엇인지를 나타내도록 `println!`
출력을 업데이트해 마무리합니다.

이제 작은 동작하는 웹 스크래퍼를 만들었습니다! 두어 개의 URL을 골라 명령행
도구를 실행해 보세요. 어떤 사이트는 일관되게 다른 것보다 빠른 반면, 다른
경우에는 빠른 사이트가 실행마다 다를 수 있음을 발견할지도 모릅니다. 더
중요하게도 future 작업의 기본을 배웠으니, 이제 async로 할 수 있는 것을 더
깊이 파헤칠 수 있습니다.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
