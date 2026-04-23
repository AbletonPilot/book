<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

## Async로 동시성 적용하기

이 절에서는 16장에서 스레드로 다룬 것과 같은 동시성 도전의 일부를 async에
적용해 보겠습니다. 핵심 아이디어의 많은 부분은 거기서 이미 이야기했으므로,
이 절에서는 스레드와 future의 차이점에 집중하겠습니다.

많은 경우 async를 사용해 동시성을 다루는 API는 스레드를 사용하는 것과 매우
비슷합니다. 다른 경우에는 꽤 다르게 끝납니다. 스레드와 async 간 API가 _비슷해
보이는_ 경우에도 종종 다른 동작을 가지며, 거의 항상 다른 성능 특성을 가집니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### `spawn_task`로 새 태스크 만들기

16장의 [“`spawn`으로 새 스레드 만들기”][thread-spawn]<!-- ignore --> 절에서
처음 다룬 연산은 두 개의 다른 스레드에서 수를 세는 것이었습니다. async로 같은
것을 해 봅시다. `trpl` 크레이트는 `thread::spawn` API와 매우 비슷해 보이는
`spawn_task` 함수와, `thread::sleep` API의 async 버전인 `sleep` 함수를 제공
합니다. 이 둘을 함께 사용해 Listing 17-6처럼 수 세기 예제를 구현할 수 있습니다.

<Listing number="17-6" caption="메인 태스크가 다른 것을 출력하는 동안 한 가지를 출력할 새 태스크 만들기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

시작점으로 최상위 함수가 async일 수 있도록 `main` 함수를 `trpl::block_on`으로
설정합니다.

> 참고: 이 장 이 시점부터 모든 예제는 `main`에 `trpl::block_on`을 사용한 정확히
> 같은 래핑 코드를 포함할 것이므로, `main`처럼 종종 생략하겠습니다. 여러분
> 코드에는 잊지 말고 포함하세요!

그런 다음 그 블록 안에 두 개의 루프를 작성하며, 각 루프에는 다음 메시지를
보내기 전에 0.5초(500밀리초) 기다리는 `trpl::sleep` 호출이 있습니다. 한
루프는 `trpl::spawn_task`의 본문에 두고, 다른 하나는 최상위 `for` 루프에
둡니다. `sleep` 호출 뒤에도 `await`를 추가합니다.

이 코드는 스레드 기반 구현과 유사하게 동작합니다. 여러분 터미널에서 실행
할 때 메시지가 다른 순서로 나타날 수 있다는 사실도 포함해서 말이죠.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

이 버전은 메인 async 블록 본문의 `for` 루프가 끝나자마자 중단됩니다. `main`
함수가 끝날 때 `spawn_task`로 생성된 태스크가 종료되기 때문입니다. 태스크의
완료까지 계속 실행하려면, 첫 태스크가 완료되기를 기다리도록 join 핸들을 사용
해야 합니다. 스레드에서는 스레드가 실행을 마칠 때까지 “블록”하기 위해 `join`
메서드를 사용했습니다. Listing 17-7에서는 태스크 핸들 자체가 future이기 때문에
같은 일을 하도록 `await`를 사용할 수 있습니다. 그 `Output` 타입은 `Result`
이므로 기다린 후 `unwrap`도 합니다.

<Listing number="17-7" caption="태스크를 완료까지 실행하기 위해 join 핸들과 `await` 사용하기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

이 업데이트된 버전은 _두_ 루프가 모두 끝날 때까지 실행됩니다.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

지금까지 async와 스레드는 문법만 다를 뿐 비슷한 결과를 주는 것 같습니다. join
핸들에 `join`을 호출하는 대신 `await`를 사용하고, `sleep` 호출을 기다린 것
정도입니다.

더 큰 차이는 이것을 하기 위해 또 다른 운영체제 스레드를 생성할 필요가 없었
다는 점입니다. 사실 여기서는 태스크를 생성할 필요조차 없습니다. async 블록은
익명 future로 컴파일되므로, 각 루프를 async 블록에 넣고 `trpl::join` 함수를
사용해 런타임이 둘 다 완료까지 실행하도록 할 수 있습니다.

16장의 [“모든 스레드가 끝날 때까지 기다리기”][join-handles]<!-- ignore -->
절에서 `std::thread::spawn`을 호출할 때 반환되는 `JoinHandle` 타입의 `join`
메서드 사용법을 보여 줬습니다. `trpl::join` 함수는 유사하지만 future용입니다.
두 future를 주면, 둘 다 완료되었을 때 각 future의 출력을 담은 튜플을 출력으로
하는 새 single future를 만들어 냅니다. 따라서 Listing 17-8에서는 `fut1`과
`fut2` 둘 다 끝나기를 기다리기 위해 `trpl::join`을 사용합니다. `fut1`과
`fut2`를 기다리는 것이 아니라 `trpl::join`이 만들어 낸 새 future를 기다립니다.
출력이 두 유닛 값을 담은 튜플일 뿐이므로 무시합니다.

<Listing number="17-8" caption="`trpl::join`으로 두 익명 future 기다리기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

이것을 실행하면 두 future가 모두 완료까지 실행되는 것을 볼 수 있습니다.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

이제 매번 정확히 같은 순서가 보일 것이며, 이는 Listing 17-7에서 스레드와
`trpl::spawn_task`로 본 것과 매우 다릅니다. 이는 `trpl::join` 함수가 _공정
(fair)_ 하기 때문입니다. 각 future를 동등하게 자주 확인하며 번갈아 가고, 다른
한 쪽이 준비되었더라도 한 쪽이 앞서 달리게 두지 않습니다. 스레드에서는 운영
체제가 어떤 스레드를 확인하고 얼마나 오래 실행하게 할지 결정합니다. async
러스트에서는 런타임이 어떤 태스크를 확인할지 결정합니다. (실제로는 async 런
타임이 동시성 관리의 일부로 내부적으로 운영체제 스레드를 사용할 수 있기
때문에 세부 사항은 복잡해지며, 공정성 보장이 런타임에 더 많은 작업이 될 수
있습니다. 그래도 가능합니다!) 런타임이 주어진 연산에 공정성을 보장할 필요는
없으며, 공정성을 원할지 선택할 수 있는 다른 API를 제공하는 경우가 많습니다.

이 future 기다리기에 대한 변형들을 시도해 보고 어떤 결과가 나오는지 보세요.

- 루프 중 하나 또는 둘 다의 async 블록을 제거합니다.
- 각 async 블록을 정의한 직후에 기다립니다.
- 첫 번째 루프만 async 블록으로 감싸고, 두 번째 루프 본문 이후에 결과 future
  를 기다립니다.

추가 도전으로, 코드를 실행하기 _전에_ 각 경우에 출력이 무엇일지 알아낼 수
있는지 봐 보세요!

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>
<a id="counting-up-on-two-tasks-using-message-passing"></a>

### 메시지 전달로 두 태스크 간 데이터 보내기

future 간 데이터 공유도 익숙할 것입니다. 다시 메시지 전달을 사용하지만, 이번
에는 타입과 함수의 async 버전으로 합니다. 스레드 기반과 future 기반 동시성의
주요 차이점을 보여 주기 위해, 16장의 [“메시지 전달로 스레드 간 데이터
이전하기”][message-passing-threads]<!-- ignore --> 절과는 약간 다른 경로를
가겠습니다. Listing 17-9에서는 별개의 스레드를 생성한 것과 달리 별개의 태스크
를 생성하지 _않고_, 단일 async 블록으로 시작하겠습니다.

<Listing number="17-9" caption="async 채널을 만들고 두 반쪽을 `tx`와 `rx`에 할당하기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

여기서는 16장에서 스레드와 함께 썼던 multiple-producer, single-consumer 채널
API의 async 버전인 `trpl::channel`을 사용합니다. API의 async 버전은 스레드
기반 버전과 약간만 다릅니다. 불변 수신자 `rx` 대신 가변 수신자를 사용하고,
`recv` 메서드는 값 자체를 바로 만드는 대신 우리가 기다려야 할 future를 만듭니다.
이제 송신자에서 수신자로 메시지를 보낼 수 있습니다. 별개의 스레드나 심지어
태스크를 생성할 필요가 없다는 점에 유의하세요. 단지 `rx.recv` 호출을 기다리기만
하면 됩니다.

`std::mpsc::channel`의 동기 `Receiver::recv` 메서드는 메시지를 받을 때까지
블록합니다. `trpl::Receiver::recv` 메서드는 async이므로 그렇지 않습니다. 블록
하는 대신 메시지를 받거나 채널의 보내는 쪽이 닫힐 때까지 제어를 런타임에
되돌려 줍니다. 대조적으로 `send` 호출은 블록하지 않으므로 기다리지 않습니다.
보내는 채널이 제한이 없기 때문에 그럴 필요가 없습니다.

> 참고: 이 모든 async 코드가 `trpl::block_on` 호출 안의 async 블록에서 실행
> 되므로, 그 안의 모든 것은 블록을 피할 수 있습니다. 그러나 그 _밖_ 의 코드는
> `block_on` 함수가 반환할 때까지 블록됩니다. 이것이 `trpl::block_on` 함수의
> 전체 요점입니다. 어떤 async 코드 집합에서 어디서 블록할지 — 따라서 동기와
> async 코드 사이를 어디서 전환할지 — _선택_ 할 수 있게 해 줍니다.

이 예제에서 두 가지에 유의하세요. 첫째, 메시지는 바로 도착합니다. 둘째,
여기서 future를 사용하지만 아직 동시성은 없습니다. 이 리스트의 모든 것은
future가 관여하지 않은 것처럼 순서대로 일어납니다.

일련의 메시지를 보내고 사이에 sleep을 넣어 첫 번째 부분을 해결해 봅시다.
Listing 17-10처럼요.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="async 채널로 여러 메시지를 보내고 받으며 각 메시지 사이에 `await`로 sleep 하기" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

메시지를 보내는 것 외에도 받아야 합니다. 이 경우에는 얼마나 많은 메시지가
들어오는지 알고 있으므로 `rx.recv().await`를 네 번 호출해 수동으로 할 수
있습니다. 그러나 실제 세계에서는 보통 _알 수 없는_ 수의 메시지를 기다릴
것이므로, 더 이상 메시지가 없음을 판단할 때까지 계속 기다려야 합니다.

Listing 16-10에서는 동기 채널에서 받은 모든 항목을 처리하기 위해 `for` 루프를
사용했습니다. 그러나 러스트에는 _비동기로 생성된_ 일련의 항목과 `for` 루프를
함께 쓸 방법이 아직 없으므로, 본 적 없는 루프 — `while let` 조건 루프 — 를
사용해야 합니다. 이는 6장의 [“`if let`과 `let...else`로 간결한 제어 흐름”][if-let]<!-- ignore -->
절에서 본 `if let` 구성의 루프 버전입니다. 루프는 명시한 패턴이 값과 계속
일치하는 한 실행을 계속합니다.

`rx.recv` 호출은 future를 만들어 내고, 우리는 그것을 기다립니다. 런타임은
준비될 때까지 future를 일시 중지합니다. 메시지가 도착하면 future는 메시지가
도착할 때마다 `Some(message)`로 해결됩니다. 채널이 닫히면 _어떤_ 메시지가
도착했는지 여부와 관계없이 future는 대신 `None`으로 해결되어 더 이상 값이
없음을, 따라서 폴링을 — 즉 기다리기를 — 멈춰야 함을 나타냅니다.

`while let` 루프가 이 모든 것을 모읍니다. `rx.recv().await` 호출 결과가
`Some(message)`이면 `if let`과 마찬가지로 메시지에 접근해 루프 본문에서 사용
할 수 있습니다. 결과가 `None`이면 루프가 끝납니다. 루프가 한 번 완료될 때마다
다시 await 지점에 부딪히므로, 런타임이 다시 그것을 일시 중지하고 또 다른
메시지가 도착할 때까지 기다립니다.

이제 코드가 모든 메시지를 성공적으로 보내고 받습니다. 안타깝게도 여전히 몇 가지
문제가 있습니다. 우선 메시지가 0.5초 간격으로 도착하지 않습니다. 프로그램을
시작한 후 2초(2,000밀리초) 뒤에 한꺼번에 도착합니다. 또 이 프로그램은 절대
종료되지 않습니다! 대신 새 메시지를 영원히 기다립니다. <kbd>ctrl</kbd>-<kbd>C</kbd>
로 종료해야 합니다.

#### Async 블록 내의 코드는 선형으로 실행됩니다

메시지가 사이의 지연 없이 전체 지연 후 한꺼번에 들어오는 이유부터 살펴봅시다.
주어진 async 블록 내에서 `await` 키워드가 코드에 나타나는 순서가 프로그램이
실행될 때 그들이 실행되는 순서이기도 합니다.

Listing 17-10에는 async 블록이 하나뿐이므로 그 안의 모든 것이 선형으로 실행
됩니다. 여전히 동시성은 없습니다. 모든 `tx.send` 호출이 모든 `trpl::sleep`
호출과 그에 연관된 await 지점 사이사이 섞여 발생합니다. 그 후에야 `while let`
루프가 `recv` 호출의 `await` 지점 중 하나라도 통과할 수 있게 됩니다.

각 메시지 사이에 sleep 지연이 일어나는 우리가 원하는 동작을 얻으려면, Listing
17-11처럼 `tx`와 `rx` 연산을 자체 async 블록에 넣어야 합니다. 그러면 런타임은
Listing 17-8처럼 `trpl::join`을 사용해 각각을 따로 실행할 수 있습니다. 다시
한 번, 개별 future가 아니라 `trpl::join` 호출 결과를 기다립니다. 개별 future를
순차로 기다리면 결국 우리가 _하지 않으려는_ 바로 그 순차 흐름으로 돌아갈
것입니다.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="`send`와 `recv`를 각자의 `async` 블록으로 분리하고 그 블록의 future를 기다리기" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Listing 17-11의 업데이트된 코드로 메시지는 2초 후 한꺼번에가 아니라 500밀리초
간격으로 출력됩니다.

#### async 블록으로 소유권 이동하기

그러나 `while let` 루프가 `trpl::join`과 상호작용하는 방식 때문에 프로그램은
여전히 종료되지 않습니다.

- `trpl::join`이 반환한 future는 그것에 전달된 _두_ future 모두 완료되어야만
  완료됩니다.
- `tx_fut` future는 `vals`의 마지막 메시지를 보낸 후 sleep을 마치면 완료됩니다.
- `rx_fut` future는 `while let` 루프가 끝날 때까지 완료되지 않습니다.
- `while let` 루프는 `rx.recv`를 기다린 결과가 `None`을 만들 때까지 끝나지
  않습니다.
- `rx.recv`를 기다리면 채널의 다른 쪽이 닫힌 후에만 `None`을 반환합니다.
- 채널은 우리가 `rx.close`를 호출하거나 송신자 쪽 `tx`가 드롭될 때만 닫힙니다.
- 우리는 `rx.close`를 어디에서도 호출하지 않고, `tx`는 `trpl::block_on`에 전달된
  가장 바깥 async 블록이 끝날 때까지 드롭되지 않습니다.
- 그 블록은 `trpl::join`이 완료되는 데 블록되어 있어서 끝날 수 없고, 이는
  이 목록의 맨 위로 돌아갑니다.

지금은 메시지를 보내는 async 블록이 메시지 전송에 소유권이 필요하지 않기
때문에 `tx`를 _빌리기만_ 하지만, 우리가 `tx`를 그 async 블록으로 _이동_ 할 수
있다면 그 블록이 끝날 때 드롭될 것입니다. 13장의 [“참조를 캡처하거나 소유권
이동하기”][capture-or-move]<!-- ignore --> 절에서 클로저와 `move` 키워드
사용법을 배웠고, 16장의 [“스레드에 `move` 클로저 사용하기”][move-threads]<!-- ignore -->
절에서 논의한 것처럼 스레드 작업 시 데이터를 클로저로 이동해야 할 때가 많습
니다. 같은 기본 동역학이 async 블록에 적용되므로 `move` 키워드는 클로저와
마찬가지로 async 블록과도 동작합니다.

Listing 17-12에서는 메시지를 보내는 데 사용되는 블록을 `async`에서 `async move`
로 바꿉니다.

<Listing number="17-12" caption="완료 시 올바르게 종료되는 Listing 17-11 코드의 수정판" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

이 버전의 코드를 실행하면 마지막 메시지가 보내지고 받아진 후 우아하게 종료
됩니다. 다음으로 하나 이상의 future에서 데이터를 보내려면 무엇을 바꿔야 할지
봅시다.

#### `join!` 매크로로 여러 future 조인하기

이 async 채널도 다중 생산자 채널이므로, Listing 17-13처럼 여러 future에서
메시지를 보내고 싶다면 `tx`에 `clone`을 호출할 수 있습니다.

<Listing number="17-13" caption="async 블록과 함께 여러 생산자 사용하기" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

먼저 `tx`를 복제해 첫 번째 async 블록 밖에 `tx1`을 만듭니다. 이전에 `tx`로 한
것처럼 `tx1`을 그 블록으로 이동합니다. 그런 다음 나중에 원래 `tx`를 _새_ async
블록으로 이동해 거기서 약간 더 느린 지연으로 더 많은 메시지를 보냅니다. 이
새 async 블록을 메시지를 받는 async 블록 뒤에 두었지만, 앞에 두어도 괜찮았을
것입니다. 중요한 것은 future가 생성되는 순서가 아니라 기다려지는 순서입니다.

메시지 보내기 async 블록 둘 다 `async move` 블록이어야 `tx`와 `tx1`이 그
블록들이 끝날 때 드롭됩니다. 그렇지 않으면 처음에 시작한 것과 같은 무한 루프
로 다시 돌아갈 것입니다.

마지막으로 `trpl::join`에서 `trpl::join!`으로 바꿔 추가 future를 처리합니다.
`join!` 매크로는 컴파일 타임에 future의 수를 아는 임의의 수의 future를
기다립니다. 이 장 나중에 알 수 없는 수의 future 컬렉션을 기다리는 법을 논의
하겠습니다.

이제 두 보내는 future 모두의 메시지가 보이고, 보내는 future들이 보낸 후
약간 다른 지연을 사용하므로 메시지도 그 다른 간격으로 받아집니다.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

future 간 데이터 보내기에 메시지 전달 사용법, async 블록 내 코드가 어떻게
순차로 실행되는지, async 블록으로 소유권 이동법, 여러 future 조인법을 살펴
봤습니다. 다음으로 런타임에 다른 태스크로 전환할 수 있다고 알리는 법과 그
이유를 논의하겠습니다.

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
