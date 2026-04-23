## 우아한 종료와 정리

Listing 21-20의 코드는 우리가 의도한 대로 스레드 풀을 사용해 비동기적으로 요청에 응답하고 있습니다. 직접적인 방식으로 사용하지 않고 있는 `workers`, `id`, `thread` 필드에 대한 경고 몇 개를 받는데, 이는 우리가 아무것도 정리하지 않고 있음을 상기시킵니다. 메인 스레드를 멈추기 위해 덜 우아한 <kbd>ctrl</kbd>-<kbd>C</kbd> 방법을 사용하면, 다른 모든 스레드들도 즉시 멈춥니다. 요청을 처리하는 도중이더라도 말이죠.

다음으로, 풀의 각 스레드에 `join`을 호출해 닫기 전에 그들이 작업 중인 요청을 끝낼 수 있도록 `Drop` 트레이트를 구현하겠습니다. 그런 다음, 스레드들에게 새 요청을 받기를 멈추고 종료해야 한다고 알릴 방법을 구현하겠습니다. 이 코드가 동작하는 것을 보기 위해, 두 요청만 받은 후 스레드 풀을 우아하게 종료하도록 서버를 수정하겠습니다.

진행하면서 한 가지 주목할 점은: 이 중 어느 것도 클로저 실행을 다루는 코드 부분에는 영향을 주지 않으므로, 비동기 런타임을 위한 스레드 풀을 사용하고 있었더라도 여기 있는 모든 것은 같을 것입니다.

### `ThreadPool`에 `Drop` 트레이트 구현

스레드 풀에 `Drop`을 구현하는 것부터 시작합시다. 풀이 드롭되면, 스레드들은 모두 작업을 끝내도록 join되어야 합니다. Listing 21-22는 `Drop` 구현의 첫 시도를 보여 줍니다. 이 코드는 아직 완전히 동작하지는 않을 것입니다.

<Listing number="21-22" file-name="src/lib.rs" caption="스레드 풀이 스코프를 벗어날 때 각 스레드 join하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

먼저 스레드 풀의 각 `workers`를 반복합니다. `self`가 가변 참조이고, `worker`도 변경할 수 있어야 하므로 `&mut`을 사용합니다. 각 `worker`에 대해 이 특정 `Worker` 인스턴스가 종료되고 있다는 메시지를 출력한 다음, 그 `Worker` 인스턴스의 스레드에 `join`을 호출합니다. `join` 호출이 실패하면 러스트가 패닉하고 우아하지 않은 종료로 들어가도록 `unwrap`을 사용합니다.

이 코드를 컴파일할 때 받는 오류는 다음과 같습니다.

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

이 오류는 각 `worker`의 가변 빌림만 가지고 있고 `join`은 인수의 소유권을 가져가기 때문에 `join`을 호출할 수 없다고 알려 줍니다. 이 문제를 해결하려면, `thread`를 소유한 `Worker` 인스턴스에서 스레드를 옮겨야 `join`이 그 스레드를 소비할 수 있습니다. 이를 하는 한 가지 방법은 Listing 18-15에서 취한 같은 접근법을 취하는 것입니다. `Worker`가 `Option<thread::JoinHandle<()>>`를 들고 있다면, `Option`에 `take` 메서드를 호출해 `Some` 변형에서 값을 빼내고 그 자리에 `None` 변형을 남길 수 있을 것입니다. 다시 말해, 실행 중인 `Worker`는 `thread`에 `Some` 변형을 가지고 있을 테고, `Worker`를 정리하고 싶을 때 `Worker`가 실행할 스레드를 가지지 않도록 `Some`을 `None`으로 교체합니다.

그러나 이런 일이 일어나는 _유일한_ 시점은 `Worker`를 드롭할 때일 것입니다. 그 대가로, `worker.thread`에 접근하는 어디에서나 `Option<thread::JoinHandle<()>>`를 다뤄야 할 것입니다. 관용적인 러스트는 `Option`을 꽤 많이 사용하지만, 이런 우회처럼 항상 존재할 것임을 아는 무언가를 `Option`에 감싸는 자신을 발견한다면, 코드를 더 깔끔하고 오류가 덜 나도록 만들 수 있는 다른 접근법을 찾는 것이 좋습니다.

이 경우 더 좋은 대안이 있습니다. `Vec::drain` 메서드입니다. 이 메서드는 벡터에서 어느 항목을 제거할지 명시하기 위한 범위 매개변수를 받고, 그 항목들의 이터레이터를 반환합니다. `..` 범위 문법을 전달하면 벡터에서 *모든* 값을 제거합니다.

따라서 `ThreadPool`의 `drop` 구현을 다음과 같이 갱신해야 합니다.

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

이는 컴파일러 오류를 해결하며 코드의 다른 변경을 요구하지 않습니다. drop은 패닉할 때도 호출될 수 있으므로 `unwrap`도 패닉해 이중 패닉을 일으켜 즉시 프로그램을 충돌시키고 진행 중인 어떤 정리도 끝낼 수 있다는 점에 유의하세요. 예시 프로그램에는 괜찮지만 운영 코드에는 권장되지 않습니다.

### 스레드들에게 작업 수신을 중단하라고 신호하기

지금까지 가한 모든 변경 사항으로 코드는 어떤 경고도 없이 컴파일됩니다. 그러나 나쁜 소식은, 이 코드가 아직 우리가 원하는 방식으로 동작하지는 않는다는 것입니다. 핵심은 `Worker` 인스턴스의 스레드가 실행하는 클로저의 로직입니다. 현재 우리는 `join`을 호출하지만, 그것이 스레드를 종료시키지는 않을 것입니다. 그들이 작업을 찾기 위해 영원히 `loop`하기 때문입니다. 우리의 현재 `drop` 구현으로 `ThreadPool`을 드롭하려고 하면, 메인 스레드는 첫 스레드가 끝나기를 기다리며 영원히 블록될 것입니다.

이 문제를 고치려면 `ThreadPool` `drop` 구현을 변경한 다음 `Worker` 반복을 변경해야 합니다.

먼저 스레드들이 끝나기를 기다리기 전에 명시적으로 `sender`를 드롭하도록 `ThreadPool` `drop` 구현을 변경하겠습니다. Listing 21-23은 `sender`를 명시적으로 드롭하기 위한 `ThreadPool`의 변경 사항을 보여 줍니다. 스레드와 달리, 여기서는 `Option::take`로 `ThreadPool`에서 `sender`를 옮길 수 있도록 `Option`을 사용해야 _합니다_.

<Listing number="21-23" file-name="src/lib.rs" caption="`Worker` 스레드를 join하기 전에 `sender`를 명시적으로 드롭하기">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

`sender`를 드롭하면 채널이 닫혀 더 이상 메시지가 보내지지 않을 것임을 가리킵니다. 그런 일이 일어나면, `Worker` 인스턴스들이 무한 반복에서 하는 모든 `recv` 호출이 오류를 반환할 것입니다. Listing 21-24에서, 그 경우 우아하게 반복을 빠져나가도록 `Worker` 반복을 변경하는데, 이는 `ThreadPool` `drop` 구현이 그들에게 `join`을 호출할 때 스레드들이 끝날 것임을 의미합니다.

<Listing number="21-24" file-name="src/lib.rs" caption="`recv`가 오류를 반환할 때 명시적으로 반복을 빠져나오기">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

이 코드가 동작하는 것을 보기 위해, Listing 21-25에서 보이듯 두 요청만 받은 후 서버를 우아하게 종료하도록 `main`을 수정합시다.

<Listing number="21-25" file-name="src/main.rs" caption="반복을 빠져나옴으로써 두 요청을 서비스한 후 서버 종료하기">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

실제 웹 서버가 두 요청만 서비스한 후 종료되기를 원하지는 않을 것입니다. 이 코드는 우아한 종료와 정리가 작동하는 상태에 있다는 것을 보여 주기 위한 것일 뿐입니다.

`take` 메서드는 `Iterator` 트레이트에 정의되어 있고 반복을 최대 처음 두 항목으로 제한합니다. `ThreadPool`은 `main`의 끝에서 스코프를 벗어나고 `drop` 구현이 실행될 것입니다.

`cargo run`으로 서버를 시작하고 세 요청을 만드세요. 세 번째 요청은 오류를 일으킬 것이고, 터미널에서 다음과 비슷한 출력을 보아야 합니다.

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

`Worker` ID와 출력된 메시지의 순서는 다르게 보일 수 있습니다. 메시지를 보면 이 코드가 어떻게 동작하는지 알 수 있습니다. `Worker` 인스턴스 0과 3이 첫 두 요청을 받았습니다. 서버는 두 번째 연결 후 연결 받기를 멈추었고, `ThreadPool`에 대한 `Drop` 구현은 `Worker 3`이 자기 작업을 시작하기도 전에 실행되기 시작합니다. `sender`를 드롭하면 모든 `Worker` 인스턴스의 연결을 끊고 그들에게 종료하라고 알립니다. `Worker` 인스턴스들은 연결이 끊길 때 각각 메시지를 출력하고, 그런 다음 스레드 풀은 각 `Worker` 스레드가 끝나기를 기다리기 위해 `join`을 호출합니다.

이 특정 실행의 한 가지 흥미로운 측면에 주목하세요. `ThreadPool`이 `sender`를 드롭했고, 어떤 `Worker`가 오류를 받기 전에 우리는 `Worker 0`을 join하려고 시도했습니다. `Worker 0`은 아직 `recv`로부터 오류를 받지 못했으므로, 메인 스레드는 `Worker 0`이 끝나기를 기다리며 블록되었습니다. 그 사이 `Worker 3`은 작업을 받고, 그 다음 모든 스레드가 오류를 받았습니다. `Worker 0`이 끝나면, 메인 스레드는 나머지 `Worker` 인스턴스들이 끝나기를 기다렸습니다. 그 시점에 그들은 모두 자신의 반복을 빠져나와 멈췄습니다.

축하합니다! 이제 우리는 프로젝트를 완성했습니다. 비동기적으로 응답하기 위해 스레드 풀을 사용하는 기본 웹 서버를 가지고 있습니다. 풀의 모든 스레드를 정리하는 서버의 우아한 종료를 수행할 수 있습니다.

다음은 참고를 위한 전체 코드입니다.

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

여기에서 더 많이 할 수 있을 것입니다! 이 프로젝트를 계속 향상시키고 싶다면, 다음 몇 가지 아이디어가 있습니다.

- `ThreadPool`과 그 공개 메서드에 더 많은 문서 추가하기.
- 라이브러리 기능에 대한 테스트 추가하기.
- `unwrap` 호출을 더 견고한 오류 처리로 변경하기.
- 웹 요청 서비스 외의 어떤 작업을 수행하기 위해 `ThreadPool` 사용하기.
- [crates.io](https://crates.io/)에서 스레드 풀 크레이트를 찾고, 그 크레이트를 사용해 비슷한 웹 서버를 구현하기. 그런 다음 그 API와 견고함을 우리가 구현한 스레드 풀과 비교하기.

## 요약

잘 했습니다! 책의 끝에 도달했습니다! 러스트의 이 여정에 함께해 주셔서 감사하다고 말씀드리고 싶습니다. 이제 자신의 러스트 프로젝트를 구현하고 다른 사람의 프로젝트에 도움을 줄 준비가 되었습니다. 러스트 여정에서 마주칠 어떤 도전이든 도와드리고 싶어 하는 다른 Rustaceans의 환영하는 커뮤니티가 있다는 것을 기억하세요.
