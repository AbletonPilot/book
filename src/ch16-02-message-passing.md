<!-- Old headings. Do not remove or links may break. -->

<a id="using-message-passing-to-transfer-data-between-threads"></a>

## 메시지 전달로 스레드 간 데이터 이전하기

안전한 동시성을 보장하기 위한 점점 인기 있는 접근 방식 중 하나는 메시지 전달
(message passing)입니다. 스레드나 액터가 데이터를 담은 메시지를 서로 보내
소통합니다. [Go 언어 문서](https://golang.org/doc/effective_go.html#concurrency)
의 슬로건으로 이 아이디어를 표현하면 다음과 같습니다. “메모리를 공유하여
소통하지 말고, 대신 소통하여 메모리를 공유하라.”

메시지 보내기 동시성을 달성하기 위해 러스트 표준 라이브러리는 채널(channel)
의 구현을 제공합니다. _채널_ 은 한 스레드에서 다른 스레드로 데이터가 보내지는
일반 프로그래밍 개념입니다.

프로그래밍의 채널을 시내나 강과 같은 방향성 있는 물의 통로처럼 상상할 수
있습니다. 강에 고무 오리 같은 것을 넣으면 수로의 끝까지 하류로 흘러갑니다.

채널은 송신자(transmitter)와 수신자(receiver) 두 반쪽으로 이루어집니다. 송신자
쪽은 강에 고무 오리를 넣는 상류 위치이고, 수신자 쪽은 고무 오리가 결국 도달
하는 하류입니다. 여러분 코드의 한 부분은 보내고 싶은 데이터로 송신자에 메서드
를 호출하고, 다른 부분은 도착한 메시지를 수신 쪽에서 확인합니다. 송신자나
수신자 반쪽 중 하나가 드롭되면 채널이 _닫혔다(closed)_ 고 합니다.

여기서는 값을 생성해 채널로 내려보내는 한 스레드와, 값을 받아 출력하는 다른
스레드를 가진 프로그램을 만들어 보겠습니다. 이 기능을 보여 주기 위해 채널로
스레드 간에 간단한 값을 보내겠습니다. 기법에 익숙해지면 채팅 시스템이나 많은
스레드가 계산의 일부를 수행해 결과를 집계하는 한 스레드로 보내는 시스템처럼,
서로 소통해야 하는 어떤 스레드에든 채널을 사용할 수 있습니다.

먼저 Listing 16-6에서 채널을 만들지만 그것으로 아무 일도 하지 않습니다. 아직
컴파일되지 않는데, 러스트가 채널로 보내려는 값의 타입을 알 수 없기 때문입니다.

<Listing number="16-6" file-name="src/main.rs" caption="채널을 만들고 두 반쪽을 `tx`와 `rx`에 할당하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

`mpsc::channel` 함수로 새 채널을 만듭니다. `mpsc`는 _multiple producer, single
consumer_ 의 약자입니다. 간단히 말해 러스트 표준 라이브러리가 채널을 구현하는
방식은, 채널이 값을 생성하는 여러 _보내는_ 쪽을 가질 수 있지만 그 값을 소비
하는 _받는_ 쪽은 하나만 가질 수 있음을 의미합니다. 여러 시내가 하나의 큰 강
으로 함께 흘러든다고 상상해 보세요. 어떤 시내든 내려보낸 모든 것은 끝에서
하나의 강에 도달합니다. 지금은 단일 생산자로 시작하지만, 예제가 동작하게 되면
여러 생산자를 추가하겠습니다.

`mpsc::channel` 함수는 튜플을 반환하는데, 첫 번째 요소는 보내는 쪽 — 송신자
— 이고 두 번째 요소는 받는 쪽 — 수신자 — 입니다. 약어 `tx`와 `rx`는 많은
분야에서 각각 _transmitter_ 와 _receiver_ 를 전통적으로 지칭하는 데 사용되므로,
각 쪽을 나타내도록 변수 이름을 그렇게 지었습니다. 튜플을 분해하는 패턴과 함께
`let` 구문을 사용하고 있습니다. `let` 구문에서 패턴 사용과 분해에 대해서는
19장에서 논의하겠습니다. 지금은 `let` 구문을 이런 식으로 사용하는 것이
`mpsc::channel`이 반환한 튜플의 조각을 추출하는 편리한 방법임을 알아 두세요.

보내는 쪽을 생성된 스레드로 옮기고 하나의 문자열을 보내 생성된 스레드가 메인
스레드와 소통하도록 해 봅시다. Listing 16-7처럼요. 이는 상류에 고무 오리를
넣거나 한 스레드에서 다른 스레드로 채팅 메시지를 보내는 것과 같습니다.

<Listing number="16-7" file-name="src/main.rs" caption='`tx`를 생성된 스레드로 옮기고 `"hi"` 보내기'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

다시 `thread::spawn`을 사용해 새 스레드를 만들고, 생성된 스레드가 `tx`를
소유하도록 `move`를 사용해 클로저로 `tx`를 옮깁니다. 생성된 스레드는 채널을
통해 메시지를 보낼 수 있도록 송신자를 소유해야 합니다.

송신자에는 우리가 보내고 싶은 값을 받는 `send` 메서드가 있습니다. `send`
메서드는 `Result<T, E>` 타입을 반환하므로, 수신자가 이미 드롭되었고 값을
보낼 곳이 없다면 보내기 작업은 오류를 반환합니다. 이 예제에서는 오류가 나면
패닉하도록 `unwrap`을 호출합니다. 그러나 실제 애플리케이션에서는 적절히
처리할 것입니다. 적절한 오류 처리 전략은 9장을 복습하세요.

Listing 16-8에서는 메인 스레드의 수신자에서 값을 얻습니다. 이는 강의 끝에서
물에서 고무 오리를 꺼내거나 채팅 메시지를 받는 것과 같습니다.

<Listing number="16-8" file-name="src/main.rs" caption='메인 스레드에서 `"hi"` 값을 받아 출력하기'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

수신자에는 유용한 메서드 두 개가 있습니다. `recv`와 `try_recv`입니다. _receive_
의 약자인 `recv`를 사용하고 있습니다. 이 메서드는 메인 스레드의 실행을 블록하
고 채널로 값이 보내질 때까지 기다립니다. 값이 보내지면 `recv`는 그것을
`Result<T, E>`로 반환합니다. 송신자가 닫히면 `recv`는 더 이상 오는 값이 없음을
알리기 위해 오류를 반환합니다.

`try_recv` 메서드는 블록하지 않고 대신 즉시 `Result<T, E>`를 반환합니다.
메시지가 있으면 메시지를 담은 `Ok` 값을, 이번에 메시지가 없으면 `Err` 값을
반환합니다. 이 스레드가 메시지를 기다리는 동안 해야 할 다른 작업이 있다면
`try_recv`를 사용하는 것이 유용합니다. `try_recv`를 가끔 호출하고 메시지가
있으면 처리하고, 그렇지 않으면 잠시 다른 작업을 한 뒤 다시 확인하는 루프를
작성할 수 있습니다.

이 예제에서는 단순함을 위해 `recv`를 사용했습니다. 메인 스레드에는 메시지를
기다리는 것 외에 할 일이 없으므로 메인 스레드를 블록하는 것이 적절합니다.

Listing 16-8의 코드를 실행하면 메인 스레드에서 출력된 값을 볼 것입니다.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

완벽합니다!

<!-- Old headings. Do not remove or links may break. -->

<a id="channels-and-ownership-transference"></a>

### 채널을 통한 소유권 이전

메시지 전송에서 소유권 규칙은 안전한 동시성 코드를 작성하는 데 도움이 되므로
매우 중요한 역할을 합니다. 러스트 프로그램 전체에 걸쳐 소유권에 대해 생각하는
것의 이점은 동시성 프로그래밍에서 오류를 방지하는 것입니다. 채널과 소유권이
어떻게 함께 문제를 방지하는지 보여 주는 실험을 해 봅시다. 채널로 `val` 값을
내려보낸 _후에_ 생성된 스레드에서 그 값을 사용해 보겠습니다. 이 코드가 허용
되지 않는 이유를 보려면 Listing 16-9의 코드를 컴파일해 보세요.

<Listing number="16-9" file-name="src/main.rs" caption="채널로 내려보낸 후에 `val` 사용 시도하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

여기서 우리는 `tx.send`로 `val`을 채널로 내려보낸 후에 `val`을 출력하려 합니다.
이를 허용하는 것은 나쁜 생각일 것입니다. 값이 다른 스레드로 보내지면, 우리가
그 값을 다시 사용하려 하기 전에 그 스레드가 값을 수정하거나 드롭할 수 있습니다.
잠재적으로 다른 스레드의 수정은 일관되지 않거나 존재하지 않는 데이터로 인해
오류나 예기치 않은 결과를 일으킬 수 있습니다. 그러나 Listing 16-9의 코드를
컴파일하려 하면 러스트는 오류를 줍니다.

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

우리의 동시성 실수가 컴파일 타임 오류를 일으켰습니다. `send` 함수는 매개변수의
소유권을 가져가고, 값이 이동되면 수신자가 그 소유권을 가져갑니다. 이것이
값을 보낸 후에 실수로 다시 사용하는 것을 막아 줍니다. 소유권 시스템이 모든
것이 괜찮은지 검사합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="sending-multiple-values-and-seeing-the-receiver-waiting"></a>

### 여러 값 보내기

Listing 16-8의 코드는 컴파일되고 실행되었지만, 두 개의 별개 스레드가 채널을
통해 서로 이야기하고 있음을 명확하게 보여 주지는 않았습니다.

Listing 16-10에서는 Listing 16-8의 코드가 동시적으로 실행되고 있음을 증명할
몇 가지 수정을 했습니다. 생성된 스레드가 이제 여러 메시지를 보내고 각 메시지
사이에 1초씩 멈춥니다.

<Listing number="16-10" file-name="src/main.rs" caption="여러 메시지를 보내고 각각 사이에 일시 정지하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

이번에는 생성된 스레드가 메인 스레드로 보내고 싶은 문자열의 벡터를 가지고
있습니다. 그것들을 순회하며 각각을 개별적으로 보내고, 1초의 `Duration` 값으로
`thread::sleep` 함수를 호출해 각각 사이에 일시 정지합니다.

메인 스레드에서는 더 이상 `recv` 함수를 명시적으로 호출하지 않습니다. 대신
`rx`를 이터레이터로 취급합니다. 받은 각 값에 대해 그것을 출력합니다. 채널이
닫히면 순회가 끝납니다.

Listing 16-10의 코드를 실행하면 각 줄 사이에 1초 일시 정지와 함께 다음 출력을
볼 것입니다.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

메인 스레드의 `for` 루프에 일시 정지하거나 지연시키는 코드가 없으므로, 메인
스레드가 생성된 스레드로부터 값을 받기를 기다리고 있음을 알 수 있습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-multiple-producers-by-cloning-the-transmitter"></a>

### 여러 생산자 만들기

앞서 `mpsc`가 _multiple producer, single consumer_ 의 약자라고 언급했습니다.
`mpsc`를 활용해 Listing 16-10의 코드를 확장해 같은 수신자로 값을 보내는 여러
스레드를 만들어 봅시다. Listing 16-11처럼 송신자를 복제해 그렇게 할 수 있습니다.

<Listing number="16-11" file-name="src/main.rs" caption="여러 생산자에서 여러 메시지 보내기">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

이번에는 첫 번째 생성된 스레드를 만들기 전에 송신자에 `clone`을 호출합니다.
이는 첫 번째 생성된 스레드에 전달할 수 있는 새 송신자를 줍니다. 원래 송신자는
두 번째 생성된 스레드에 전달합니다. 이렇게 하면 각각 다른 메시지를 한 수신자로
보내는 두 스레드가 생깁니다.

코드를 실행하면 출력은 다음과 비슷해 보일 것입니다.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

시스템에 따라 값들이 다른 순서로 보일 수 있습니다. 이것이 동시성을 흥미롭게도,
어렵게도 만드는 점입니다. 여러 스레드에 다른 값으로 `thread::sleep`을 실험하면
각 실행이 더 비결정적이 되고 매번 다른 출력을 만들어 냅니다.

이제 채널이 어떻게 동작하는지 보았으니 다른 동시성 방법을 살펴봅시다.
