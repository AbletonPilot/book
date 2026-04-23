
<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### 런타임에 제어 양보하기

[“첫 Async 프로그램”][async-program]<!-- ignore --> 절에서, 각 await 지점마다
러스트가 런타임에게 기다리는 future가 준비되지 않았다면 태스크를 일시 중지하고
다른 것으로 전환할 기회를 준다고 했습니다. 반대도 참입니다. 러스트는 _오직_
await 지점에서만 async 블록을 일시 중지하고 런타임에 제어를 되돌려 줍니다.
await 지점 사이의 모든 것은 동기적입니다.

즉 await 지점 없이 async 블록에서 많은 일을 하면, 그 future가 다른 future들이
진행하는 것을 막습니다. 이를 하나의 future가 다른 future들을 _굶긴다(starving)_
고 부르기도 합니다. 일부 경우에는 큰 문제가 아닐 수 있습니다. 그러나 값비싼
설정이나 오래 걸리는 작업을 하거나, 특정 작업을 무한히 계속할 future가 있다면
런타임에 언제, 어디서 제어를 되돌릴지 생각해야 합니다.

기아(starvation) 문제를 보여 주기 위해 오래 걸리는 연산을 시뮬레이션한 뒤, 그것
을 푸는 방법을 탐구해 봅시다. Listing 17-14는 `slow` 함수를 소개합니다.

<Listing number="17-14" caption="느린 연산을 시뮬레이션하기 위해 `thread::sleep` 사용하기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

이 코드는 `slow` 호출이 현재 스레드를 몇 밀리초 동안 블록하도록 `trpl::sleep`
대신 `std::thread::sleep`을 사용합니다. 오래 걸리고 블로킹되는 실제 세계 연산
을 대신하는 데 `slow`를 사용할 수 있습니다.

Listing 17-15에서는 한 쌍의 future에서 이런 종류의 CPU 바운드 작업을 하는 것을
흉내 내기 위해 `slow`를 사용합니다.

<Listing number="17-15" caption="느린 연산을 시뮬레이션하기 위해 `slow` 함수 호출하기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

각 future는 일련의 느린 연산을 수행한 _뒤에야_ 런타임에 제어를 되돌려 줍니다.
이 코드를 실행하면 다음 출력을 볼 수 있습니다.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

두 URL을 가져오는 future를 경쟁시키기 위해 `trpl::select`를 사용한 Listing
17-5처럼, `select`는 `a`가 끝나자마자 완료됩니다. 그러나 두 future의 `slow`
호출 사이에는 어떤 교차도 없습니다. `a` future는 `trpl::sleep` 호출이 기다려질
때까지 모든 작업을 하고, 그 다음 `b` future가 자체 `trpl::sleep` 호출이 기다
려질 때까지 모든 작업을 하고, 마지막으로 `a` future가 완료됩니다. 두 future
가 느린 태스크 사이에 진행할 수 있도록 하려면 런타임에 제어를 되돌릴 수 있는
await 지점이 필요합니다. 즉 기다릴 수 있는 무언가가 필요합니다!

Listing 17-15에서 이미 이런 종류의 전달이 일어나는 것을 볼 수 있습니다. `a`
future 끝의 `trpl::sleep`을 제거하면 `b` future가 _전혀_ 실행되지 않은 채 완료
될 것입니다. Listing 17-16처럼 연산이 진행을 번갈아 하게 하는 시작점으로
`trpl::sleep` 함수를 사용해 봅시다.

<Listing number="17-16" caption="연산이 진행을 번갈아 하게 하기 위해 `trpl::sleep` 사용하기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

각 `slow` 호출 사이에 await 지점이 있는 `trpl::sleep` 호출을 추가했습니다.
이제 두 future의 작업이 교차합니다.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

`a` future는 `trpl::sleep`을 호출하기 전에 `slow`를 호출하므로 `b`에 제어를
넘기기 전에 잠시 실행되지만, 그 후로는 한 쪽이 await 지점에 부딪힐 때마다
future들이 번갈아 왔다 갔다 합니다. 이 경우에는 모든 `slow` 호출 후에 그렇게
했지만, 우리에게 가장 말이 되는 어떤 방식으로든 작업을 나눌 수 있습니다.

그러나 여기서 우리는 정말로 _자고_ 싶지는 않습니다. 가능한 한 빠르게 진행하고
싶을 뿐입니다. 제어만 런타임에 되돌리면 됩니다. `trpl::yield_now` 함수를
사용해 직접 그렇게 할 수 있습니다. Listing 17-17에서는 모든 `trpl::sleep`
호출을 `trpl::yield_now`로 바꿉니다.

<Listing number="17-17" caption="연산이 진행을 번갈아 하게 하기 위해 `yield_now` 사용하기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

이 코드는 실제 의도에 대해 더 명확하고, `sleep`을 쓰는 것보다 상당히 빠를 수
있습니다. `sleep` 같은 타이머는 보통 얼마나 세밀할 수 있는지에 한계가 있기
때문입니다. 예를 들어 우리가 쓰는 `sleep` 버전은 `Duration`을 1나노초로 전달
해도 항상 최소 1밀리초 잡니다. 다시 말하지만 현대 컴퓨터는 _빠릅니다_. 1밀리
초에 많은 일을 할 수 있습니다!

이는 프로그램이 다른 어떤 일을 하고 있느냐에 따라 async가 계산 바운드 태스크
에도 유용할 수 있음을 의미합니다. 프로그램의 다른 부분 간 관계를 구조화하는
유용한 도구를 제공하기 때문입니다(대신 async 상태 기계 오버헤드라는 비용이
있지만요). 이것은 _협력적 멀티태스킹(cooperative multitasking)_ 의 한 형태로,
각 future가 await 지점을 통해 언제 제어를 넘길지 결정할 힘을 가집니다. 따라서
각 future는 너무 오래 블록하지 않을 책임도 집니다. 일부 러스트 기반 임베디드
운영체제에서는 이것이 _유일한_ 종류의 멀티태스킹입니다!

실제 세계 코드에서는 물론 매 줄마다 함수 호출과 await 지점을 교대하지는 않을
것입니다. 이런 식으로 제어를 양보하는 것은 비교적 저렴하지만, 공짜는 아닙니다.
많은 경우 계산 바운드 태스크를 쪼개려는 시도가 그것을 상당히 느리게 만들 수
있으므로, 연산이 잠시 블록하게 두는 것이 _전반적_ 성능을 위해 때때로 더
좋습니다. 코드의 실제 성능 병목이 무엇인지 항상 측정해 보세요. 그러나 동시적
으로 일어나기를 기대한 많은 작업이 직렬로 일어나는 것을 보고 있다면 그 근본
동역학은 염두에 두는 것이 중요합니다!

### 자체 Async 추상화 만들기

future들을 함께 구성해 새 패턴을 만들 수도 있습니다. 예를 들어 이미 가지고
있는 async 구성 요소로 `timeout` 함수를 만들 수 있습니다. 끝나면 결과는 또
다른 async 추상화를 만드는 데 사용할 수 있는 또 다른 구성 요소가 될 것입니다.

Listing 17-18은 느린 future와 함께 이 `timeout`이 어떻게 동작하기를 기대
하는지 보여 줍니다.

<Listing number="17-18" caption="시간 제한을 두고 느린 연산을 실행하기 위해 상상한 `timeout` 사용하기" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

구현해 봅시다! 시작하며 `timeout`의 API를 생각해 봅시다.

- 기다릴 수 있도록 자체가 async 함수여야 합니다.
- 첫 번째 매개변수는 실행할 future여야 합니다. 어떤 future와도 동작하도록
  제네릭으로 만들 수 있습니다.
- 두 번째 매개변수는 기다릴 최대 시간이 됩니다. `Duration`을 사용하면 `trpl::sleep`
  에 쉽게 전달할 수 있습니다.
- `Result`를 반환해야 합니다. future가 성공적으로 완료되면 `Result`는 future
  가 만들어 낸 값과 함께 `Ok`가 됩니다. 시간 제한이 먼저 지나가면 `Result`는
  시간 제한이 기다린 기간과 함께 `Err`이 됩니다.

Listing 17-19는 이 선언을 보여 줍니다.

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-19" caption="`timeout`의 시그니처 정의하기" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

그것으로 타입 목표는 충족됩니다. 이제 필요한 _동작_ 을 생각해 봅시다. 전달된
future를 기간에 대해 경쟁시키고 싶습니다. 기간으로 타이머 future를 만드는 데
`trpl::sleep`을 사용하고, 호출자가 전달한 future와 그 타이머를 함께 실행하는
데 `trpl::select`를 사용할 수 있습니다.

Listing 17-20에서 `trpl::select`를 기다린 결과에 매칭해 `timeout`을 구현합니다.

<Listing number="17-20" caption="`select`와 `sleep`으로 `timeout` 정의하기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

`trpl::select`의 구현은 공정하지 않습니다. 인수는 항상 전달된 순서로 폴링
합니다(다른 `select` 구현은 어느 인수를 먼저 폴링할지 무작위로 선택합니다).
따라서 `max_time`이 매우 짧은 기간이더라도 `future_to_try`가 완료될 기회를
가질 수 있도록 `select`에 먼저 `future_to_try`를 전달합니다. `future_to_try`
가 먼저 끝나면 `select`는 `future_to_try`의 출력과 함께 `Left`를 반환합니다.
`timer`가 먼저 끝나면 `select`는 타이머의 출력 `()`와 함께 `Right`를 반환합니다.

`future_to_try`가 성공해 `Left(output)`을 얻으면 `Ok(output)`을 반환합니다.
대신 sleep 타이머가 지나가서 `Right(())`를 얻으면 `_`로 `()`를 무시하고
`Err(max_time)`을 반환합니다.

그것으로 두 다른 async 보조 요소로 만든 동작하는 `timeout`이 있습니다. 이
코드를 실행하면 시간 제한 후 실패 모드를 출력합니다.

```text
Failed after 2 seconds
```

future가 다른 future와 구성되므로, 더 작은 async 구성 요소로 정말 강력한
도구를 만들 수 있습니다. 예를 들어 같은 접근으로 시간 제한과 재시도를 결합
할 수 있고, 그것을 다시 네트워크 호출 같은 연산(Listing 17-5 같은)과 함께
쓸 수 있습니다.

실제로는 보통 `async`와 `await`로 직접 작업하고, 부차적으로 `select` 같은
함수와 `join!` 같은 매크로로 가장 바깥의 future들이 실행되는 방식을 제어합니다.

이제 동시에 여러 future와 작업하는 여러 방법을 보았습니다. 다음으로 _stream_
으로 시간에 걸쳐 순차적으로 여러 future를 다루는 법을 살펴봅시다.

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
