<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

## Stream: 순차적인 Future

이 장 앞부분의 [“메시지 전달”][17-02-messages]<!-- ignore --> 절에서 async
채널의 수신자를 어떻게 썼는지 떠올려 보세요. async `recv` 메서드는 시간에
걸쳐 일련의 항목을 만들어 냅니다. 이는 _stream_ 이라고 알려진 훨씬 더 일반
적인 패턴의 한 사례입니다. 많은 개념이 자연스럽게 stream으로 표현됩니다.
큐에서 사용 가능해지는 항목, 전체 데이터 집합이 컴퓨터 메모리에 담기에 너무
클 때 파일 시스템에서 점진적으로 끌어오는 데이터 조각, 시간에 걸쳐 네트워크로
도착하는 데이터 같은 것입니다. stream은 future이므로 어떤 다른 종류의 future
와도 함께 쓸 수 있고 흥미로운 방식으로 결합할 수 있습니다. 예를 들어 너무
많은 네트워크 호출을 일으키지 않도록 이벤트를 묶거나, 오래 걸리는 연산의
순차에 시간 제한을 두거나, 불필요한 작업을 피하도록 사용자 인터페이스 이벤트
를 제한할 수 있습니다.

13장의 [“`Iterator` 트레이트와 `next` 메서드”][iterator-trait]<!-- ignore -->
절에서 Iterator 트레이트를 살필 때 일련의 항목을 본 적이 있지만, 이터레이터와
async 채널 수신자에는 두 가지 차이가 있습니다. 첫 번째 차이는 시간입니다.
이터레이터는 동기적이고, 채널 수신자는 비동기적입니다. 두 번째 차이는 API
입니다. `Iterator`로 직접 작업할 때는 동기 `next` 메서드를 호출합니다. 특히
`trpl::Receiver` stream에서는 대신 비동기 `recv` 메서드를 호출했습니다. 그
외에는 이 API들이 매우 비슷해 보이고, 그 유사성은 우연이 아닙니다. stream은
비동기 형태의 순회와 비슷합니다. 그러나 `trpl::Receiver`가 구체적으로 메시지
수신을 기다리는 반면, 범용 stream API는 훨씬 광범위합니다. `Iterator`가 하는
방식으로 다음 항목을 제공하되, 비동기적으로 제공합니다.

러스트에서 이터레이터와 stream의 유사성은 우리가 어떤 이터레이터로부터도
stream을 만들 수 있음을 의미합니다. 이터레이터처럼 `next` 메서드를 호출한 뒤
출력을 기다림으로써 stream과 작업할 수 있습니다. Listing 17-21처럼요. 아직
컴파일되지는 않습니다.

<Listing number="17-21" caption="이터레이터로부터 stream을 만들고 그 값을 출력하기" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

수 배열로 시작해 이터레이터로 변환한 다음 `map`을 호출해 모든 값을 두 배로
만듭니다. 그런 다음 `trpl::stream_from_iter` 함수를 사용해 이터레이터를
stream으로 변환합니다. 다음으로 `while let` 루프로 stream의 항목을 도착하는
대로 순회합니다.

안타깝게도 코드를 실행하려 하면 컴파일되지 않고 대신 `next` 메서드를 사용할
수 없다고 보고합니다.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-21
cargo build
copy only the error output
-->

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

이 출력이 설명하듯 컴파일러 오류의 이유는 `next` 메서드를 쓰려면 올바른
트레이트가 스코프에 있어야 하기 때문입니다. 지금까지의 논의를 고려하면 그
트레이트가 `Stream`일 것이라고 합리적으로 기대할 수 있지만, 실제로는
`StreamExt`입니다. _extension_ 의 줄임말인 `Ext`는 한 트레이트를 다른
트레이트로 확장하는, 러스트 커뮤니티의 흔한 패턴입니다.

`Stream` 트레이트는 `Iterator`와 `Future` 트레이트를 효과적으로 결합하는
낮은 수준의 인터페이스를 정의합니다. `StreamExt`는 `Stream` 위에 더 높은
수준의 API 집합을 제공합니다. `next` 메서드뿐 아니라 `Iterator` 트레이트가
제공하는 것과 유사한 다른 유틸리티 메서드도 포함합니다. `Stream`과 `StreamExt`
는 아직 러스트 표준 라이브러리의 일부가 아니지만, 대부분의 생태계 크레이트는
유사한 정의를 사용합니다.

컴파일러 오류의 수정은 Listing 17-22처럼 `trpl::StreamExt`에 대한 `use` 구문
을 추가하는 것입니다.

<Listing number="17-22" caption="이터레이터를 stream의 기반으로 성공적으로 사용하기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:all}}
```

</Listing>

이 모든 조각을 합치면 이 코드는 우리가 원하는 대로 동작합니다! 더욱이 `StreamExt`
가 스코프에 있으므로 이터레이터와 마찬가지로 그 모든 유틸리티 메서드를 쓸
수 있습니다.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
