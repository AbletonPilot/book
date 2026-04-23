## 스레드를 사용해 코드를 동시에 실행하기

대부분의 현대 운영체제에서 실행된 프로그램의 코드는 _프로세스(process)_ 안에서
실행되며, 운영체제는 여러 프로세스를 한 번에 관리합니다. 프로그램 내에서도
동시에 실행되는 독립적인 부분을 가질 수 있습니다. 이러한 독립적인 부분을
실행하는 기능을 _스레드(thread)_ 라고 부릅니다. 예를 들어 웹 서버는 여러
요청에 동시에 응답할 수 있도록 여러 스레드를 가질 수 있습니다.

프로그램의 계산을 여러 작업을 동시에 실행하도록 여러 스레드로 나누면 성능이
개선될 수 있지만 복잡성도 더해집니다. 스레드가 동시에 실행될 수 있기 때문에,
서로 다른 스레드의 코드가 실행되는 순서에 대한 본래적인 보장은 없습니다. 이는
다음과 같은 문제로 이어질 수 있습니다.

- 스레드가 데이터나 자원에 일관되지 않은 순서로 접근하는 경합 조건(race
  conditions)
- 두 스레드가 서로를 기다려서 둘 다 계속 진행할 수 없는 교착 상태(deadlocks)
- 특정 상황에서만 발생해 안정적으로 재현하고 고치기 어려운 버그

러스트는 스레드 사용의 부정적 효과를 완화하려 시도하지만, 다중 스레드
맥락의 프로그래밍은 여전히 신중한 사고를 요구하고 단일 스레드에서 실행되는
프로그램과는 다른 코드 구조를 필요로 합니다.

프로그래밍 언어는 스레드를 몇 가지 다른 방식으로 구현하며, 많은 운영체제는
새 스레드를 만들기 위해 프로그래밍 언어가 호출할 수 있는 API를 제공합니다.
러스트 표준 라이브러리는 스레드 구현의 _1:1_ 모델을 사용하며, 이 모델에서
프로그램은 하나의 언어 스레드당 하나의 운영체제 스레드를 사용합니다. 1:1
모델과는 다른 트레이드오프를 제공하는 다른 스레딩 모델을 구현한 크레이트들도
있습니다. (다음 장에서 볼 러스트의 async 시스템도 동시성에 대한 또 다른
접근 방식을 제공합니다.)

### `spawn`으로 새 스레드 만들기

새 스레드를 만들려면 `thread::spawn` 함수를 호출하고, 새 스레드에서 실행
하려는 코드를 담은 클로저(13장에서 클로저를 다뤘습니다)를 전달합니다. Listing
16-1의 예제는 메인 스레드에서 어떤 텍스트를 출력하고, 새 스레드에서 다른
텍스트를 출력합니다.

<Listing number="16-1" file-name="src/main.rs" caption="메인 스레드가 다른 것을 출력하는 동안 한 가지를 출력할 새 스레드 만들기">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

러스트 프로그램의 메인 스레드가 완료되면, 실행이 끝났든 아니든 모든 생성된
스레드는 종료됨에 유의하세요. 이 프로그램의 출력은 매번 조금씩 다를 수 있지만
다음과 비슷해 보일 것입니다.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

`thread::sleep` 호출은 스레드의 실행을 잠시 멈추게 해 다른 스레드가 실행될
수 있게 합니다. 스레드는 아마 교대로 실행되겠지만 보장되지는 않습니다. 운영
체제가 스레드를 어떻게 스케줄링하느냐에 달려 있습니다. 이번 실행에서는
코드에서 생성된 스레드의 출력 구문이 먼저 나오지만 메인 스레드가 먼저 출력
했습니다. 그리고 생성된 스레드에 `i`가 `9`가 될 때까지 출력하라고 했지만, 메인
스레드가 종료되기 전에 `5`까지만 도달했습니다.

이 코드를 실행했는데 메인 스레드의 출력만 보이거나 어떤 겹침도 보이지 않는다면,
범위의 숫자를 늘려 운영체제가 스레드 사이를 전환할 기회를 더 만들어 보세요.

<!-- Old headings. Do not remove or links may break. -->

<a id="waiting-for-all-threads-to-finish-using-join-handles"></a>

### 모든 스레드가 끝날 때까지 기다리기

Listing 16-1의 코드는 메인 스레드가 끝나서 생성된 스레드를 대부분의 시간 너무
일찍 중단시킬 뿐만 아니라, 스레드가 실행되는 순서에 대한 보장도 없으므로
생성된 스레드가 실행될지조차 보장할 수 없습니다!

`thread::spawn`의 반환 값을 변수에 저장함으로써 생성된 스레드가 실행되지 않거나
일찍 끝나는 문제를 고칠 수 있습니다. `thread::spawn`의 반환 타입은
`JoinHandle<T>`입니다. `JoinHandle<T>`는 소유된 값으로, 그것에 `join` 메서드를
호출하면 해당 스레드가 끝날 때까지 기다립니다. Listing 16-2는 Listing 16-1에서
생성한 스레드의 `JoinHandle<T>`를 사용하는 방법과 `main`이 종료되기 전에 생성된
스레드가 끝나도록 `join`을 호출하는 방법을 보여 줍니다.

<Listing number="16-2" file-name="src/main.rs" caption="스레드가 완료될 때까지 실행됨을 보장하기 위해 `thread::spawn`에서 받은 `JoinHandle<T>` 저장하기">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

핸들에 `join`을 호출하면, 현재 실행 중인 스레드를 블록하여 그 핸들이 나타내는
스레드가 종료될 때까지 기다립니다. 스레드를 _블록(blocking)_ 한다는 것은 그
스레드가 작업을 수행하거나 종료하지 못하도록 막는다는 뜻입니다. `join` 호출을
메인 스레드의 `for` 루프 뒤에 두었으므로 Listing 16-2를 실행하면 다음과 비슷한
출력이 나올 것입니다.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

두 스레드는 계속 번갈아 가며 실행되지만, 메인 스레드는 `handle.join()` 호출
때문에 기다리고 생성된 스레드가 끝날 때까지 종료되지 않습니다.

그러나 `handle.join()`을 다음과 같이 `main`의 `for` 루프 _앞_ 으로 옮기면
어떻게 되는지 봅시다.

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

메인 스레드는 생성된 스레드가 끝날 때까지 기다린 다음 `for` 루프를 실행할
것이므로, 출력은 더 이상 교차되지 않습니다. 다음과 같이요.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

`join`을 어디서 호출하느냐 같은 작은 세부 사항도 스레드가 동시에 실행될지
여부에 영향을 줄 수 있습니다.

### 스레드에 `move` 클로저 사용하기

`thread::spawn`에 전달하는 클로저에는 `move` 키워드를 자주 사용합니다. 그러면
클로저가 환경에서 사용하는 값들의 소유권을 가져가, 그 값들의 소유권을 한
스레드에서 다른 스레드로 이전하기 때문입니다. 13장의 [“참조를 캡처하거나
소유권 이동하기”][capture]<!-- ignore -->에서 클로저 맥락의 `move`를 논의했
습니다. 이제 `move`와 `thread::spawn`의 상호작용에 더 집중하겠습니다.

Listing 16-1에서 `thread::spawn`에 전달한 클로저가 인수를 받지 않음에
유의하세요. 생성된 스레드의 코드에서 메인 스레드의 어떤 데이터도 사용하지
않습니다. 생성된 스레드에서 메인 스레드의 데이터를 사용하려면, 생성된 스레드의
클로저가 필요한 값들을 캡처해야 합니다. Listing 16-3은 메인 스레드에서 벡터를
만들고 생성된 스레드에서 사용하려는 시도를 보여 줍니다. 그러나 잠시 후 볼
것처럼 이는 아직 동작하지 않습니다.

<Listing number="16-3" file-name="src/main.rs" caption="메인 스레드가 만든 벡터를 다른 스레드에서 사용하려는 시도">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

클로저가 `v`를 사용하므로 클로저는 `v`를 캡처해 환경의 일부로 만듭니다.
`thread::spawn`은 이 클로저를 새 스레드에서 실행하므로, 그 새 스레드 내에서
`v`에 접근할 수 있어야 합니다. 그러나 이 예제를 컴파일하면 다음 오류가
납니다.

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

러스트는 `v`를 어떻게 캡처할지 _추론_ 하는데, `println!`이 `v`에 대한 참조만
필요로 하므로 클로저는 `v`를 빌리려고 합니다. 그러나 문제가 있습니다. 러스트
는 생성된 스레드가 얼마나 오래 실행될지 알 수 없으므로, `v`에 대한 참조가
항상 유효할지 알 수 없습니다.

Listing 16-4는 `v`에 대한 참조가 유효하지 않을 가능성이 더 높은 시나리오를
제공합니다.

<Listing number="16-4" file-name="src/main.rs" caption="`v`를 드롭하는 메인 스레드에서 `v`에 대한 참조를 캡처하려는 클로저를 가진 스레드">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

러스트가 이 코드를 실행하도록 허용했다면, 생성된 스레드가 전혀 실행되지 않고
즉시 백그라운드로 옮겨질 가능성이 있습니다. 생성된 스레드 내부에 `v`에 대한
참조가 있지만, 메인 스레드는 15장에서 논의한 `drop` 함수를 사용해 즉시 `v`를
드롭합니다. 그러면 생성된 스레드가 실행되기 시작할 때 `v`가 더 이상 유효하지
않으므로 그것에 대한 참조도 유효하지 않습니다. 이런!

Listing 16-3의 컴파일러 오류를 고치기 위해 오류 메시지의 조언을 사용할 수
있습니다.

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

클로저 앞에 `move` 키워드를 추가함으로써, 러스트가 값을 빌려야 한다고 추론하게
두는 대신 클로저가 사용하는 값의 소유권을 가져가도록 강제합니다. Listing
16-5에 보이는 Listing 16-3의 수정은 우리가 의도한 대로 컴파일되고 실행됩니다.

<Listing number="16-5" file-name="src/main.rs" caption="`move` 키워드를 사용해 클로저가 사용하는 값의 소유권을 가져가도록 강제하기">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

같은 방식을 사용해 메인 스레드가 `drop`을 호출하는 Listing 16-4의 코드를 `move`
클로저로 고치고 싶을 수 있습니다. 그러나 이 수정은 Listing 16-4가 하려는 일이
다른 이유로 허용되지 않으므로 동작하지 않습니다. 클로저에 `move`를 추가하면
`v`를 클로저의 환경으로 옮기게 되므로, 메인 스레드에서 더 이상 그것에 `drop`을
호출할 수 없습니다. 대신 다음 컴파일러 오류를 얻습니다.

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

러스트의 소유권 규칙이 다시 우리를 구했습니다! Listing 16-3의 코드에서 오류가
났던 것은 러스트가 보수적이어서 스레드를 위해 `v`를 빌리기만 했기 때문입니다.
이는 메인 스레드가 이론적으로 생성된 스레드의 참조를 무효화할 수 있음을
의미했습니다. `v`의 소유권을 생성된 스레드로 옮기도록 러스트에게 지시함으로써,
메인 스레드가 더 이상 `v`를 사용하지 않을 것임을 러스트에게 보장합니다. 같은
방식으로 Listing 16-4를 바꾸면, 메인 스레드에서 `v`를 사용하려 할 때 소유권
규칙을 어기게 됩니다. `move` 키워드는 러스트의 보수적인 기본 빌림 동작을
재정의하는 것이지, 우리가 소유권 규칙을 어기게 해 주는 것은 아닙니다.

이제 스레드가 무엇인지와 스레드 API가 제공하는 메서드를 다뤘으니, 스레드를
사용할 수 있는 몇 가지 상황을 살펴봅시다.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
