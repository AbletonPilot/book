<!-- Old headings. Do not remove or links may break. -->

<a id="turning-our-single-threaded-server-into-a-multithreaded-server"></a>
<a id="from-single-threaded-to-multithreaded-server"></a>

## 단일 스레드 서버에서 멀티스레드 서버로

지금 서버는 각 요청을 차례로 처리합니다. 즉, 첫 연결의 처리가 끝날 때까지 두 번째 연결을 처리하지 않을 것입니다. 서버가 점점 더 많은 요청을 받게 되면, 이 직렬 실행은 점점 더 비최적이 될 것입니다. 서버가 처리에 오랜 시간이 걸리는 요청을 받으면, 새 요청들이 빨리 처리될 수 있더라도 후속 요청들은 그 긴 요청이 끝날 때까지 기다려야 합니다. 이를 고쳐야 하는데, 먼저 그 문제를 실제로 살펴봅시다.

<!-- Old headings. Do not remove or links may break. -->

<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### 느린 요청 시뮬레이션

천천히 처리되는 요청이 현재 서버 구현에서 다른 요청들에 어떤 영향을 미칠 수 있는지 살펴봅니다. Listing 21-10은 응답 전에 서버를 5초간 잠재우는 시뮬레이션된 느린 응답으로 _/sleep_ 요청을 처리하는 것을 구현합니다.

<Listing number="21-10" file-name="src/main.rs" caption="5초간 잠들어 느린 요청을 시뮬레이션하기">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

이제 케이스가 세 개이므로 `if`에서 `match`로 전환했습니다. 문자열 리터럴 값들에 대해 패턴 매칭하려면 `request_line`의 슬라이스에 대해 명시적으로 매치해야 합니다. `match`는 동등성 메서드처럼 자동으로 참조와 역참조를 해 주지 않습니다.

첫 번째 갈래는 Listing 21-9의 `if` 블록과 같습니다. 두 번째 갈래는 _/sleep_ 에 대한 요청과 매치합니다. 그 요청을 받으면, 서버는 성공 HTML 페이지를 렌더링하기 전에 5초간 잠듭니다. 세 번째 갈래는 Listing 21-9의 `else` 블록과 같습니다.

우리 서버가 얼마나 원시적인지 볼 수 있습니다. 실제 라이브러리는 여러 요청의 인식을 훨씬 덜 장황하게 처리할 것입니다!

`cargo run`을 사용해 서버를 시작하세요. 그런 다음 두 브라우저 창을 엽니다. 하나는 _http://127.0.0.1:7878_ 용, 다른 하나는 _http://127.0.0.1:7878/sleep_ 용입니다. 이전처럼 _/_ URI를 몇 번 입력하면 빠르게 응답하는 것을 볼 수 있습니다. 그러나 _/sleep_ 을 입력한 다음 _/_ 를 로드하면, _/_ 가 로드되기 전에 `sleep`이 5초를 다 잘 때까지 기다리는 것을 보게 될 것입니다.

느린 요청 뒤에 요청들이 쌓이는 것을 피하기 위해 사용할 수 있는 여러 기법이 있습니다. 17장에서 했던 것처럼 async를 사용하는 것을 포함해서 말입니다. 우리가 구현할 것은 스레드 풀입니다.

### 스레드 풀로 처리량 개선하기

_스레드 풀(thread pool)_ 은 작업을 처리할 준비가 되어 기다리고 있는, 생성된 스레드들의 그룹입니다. 프로그램이 새 작업을 받으면, 풀 안의 스레드 하나에 그 작업을 할당하고 그 스레드가 작업을 처리합니다. 풀의 나머지 스레드들은 첫 번째 스레드가 처리하는 동안 들어오는 다른 작업을 처리할 수 있습니다. 첫 번째 스레드가 자기 작업의 처리를 끝내면, 새 작업을 처리할 준비가 된 유휴 스레드 풀로 돌아옵니다. 스레드 풀은 동시에 연결들을 처리할 수 있게 해서 서버의 처리량을 늘립니다.

DoS 공격으로부터 우리를 보호하기 위해 풀의 스레드 수를 작은 수로 제한할 것입니다. 만약 프로그램이 들어오는 각 요청에 대해 새 스레드를 만들게 했다면, 누군가 우리 서버에 1천만 개의 요청을 만들면 서버의 모든 자원을 다 써 버려 요청 처리를 멈추게 만드는 큰 혼란을 일으킬 수 있을 것입니다.

따라서 무제한의 스레드를 생성하는 대신, 풀에서 기다리는 고정된 수의 스레드를 가질 것입니다. 들어오는 요청들은 처리를 위해 풀로 보내집니다. 풀은 들어오는 요청들의 큐를 유지합니다. 풀의 각 스레드는 이 큐에서 요청 하나를 꺼내 처리하고 큐에 다음 요청을 요구합니다. 이 설계로 _`N`_ 개의 요청을 동시에 처리할 수 있는데, 여기서 _`N`_ 은 스레드 수입니다. 각 스레드가 오래 걸리는 요청에 응답하고 있다면 후속 요청들은 여전히 큐에 쌓일 수 있지만, 그 지점에 도달하기 전까지 처리할 수 있는 오래 걸리는 요청 수를 늘렸습니다.

이 기법은 웹 서버의 처리량을 개선하는 여러 방법 중 하나일 뿐입니다. 탐구해 볼 수 있는 다른 옵션으로는 fork/join 모델, 단일 스레드 비동기 I/O 모델, 멀티스레드 비동기 I/O 모델 등이 있습니다. 이 주제에 관심이 있다면 다른 해법에 대해 더 읽고 그것들을 구현해 볼 수 있습니다. 러스트 같은 저수준 언어로는 이 모든 옵션이 가능합니다.

스레드 풀 구현을 시작하기 전에, 풀을 사용하는 모습이 어때야 하는지 이야기해 봅시다. 코드를 설계하려고 할 때, 클라이언트 인터페이스를 먼저 작성하면 설계를 안내하는 데 도움이 될 수 있습니다. 호출하고 싶은 방식으로 구조화되도록 코드의 API를 작성한 다음, 기능을 먼저 구현하고 그 다음 공개 API를 설계하는 대신 그 구조 안에서 기능을 구현합니다.

12장의 프로젝트에서 테스트 주도 개발을 사용한 것과 비슷하게, 여기서는 컴파일러 주도 개발을 사용할 것입니다. 우리가 원하는 함수들을 호출하는 코드를 작성한 다음, 컴파일러로부터의 오류를 보고 코드가 동작하게 하기 위해 다음에 무엇을 변경해야 할지 결정합니다. 그러나 그렇게 하기 전에, 시작점으로 사용하지 _않을_ 기법을 살펴봅니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### 각 요청마다 스레드 생성하기

먼저 모든 연결마다 새 스레드를 만들었다면 우리 코드가 어땠을지 살펴봅시다. 앞에서 말했듯이, 무제한의 스레드를 잠재적으로 생성하는 문제 때문에 이것이 우리의 최종 계획은 아닙니다. 그러나 먼저 동작하는 멀티스레드 서버를 얻기 위한 시작점입니다. 그런 다음 개선으로 스레드 풀을 추가하고, 두 해법을 대조하면 더 쉬울 것입니다.

Listing 21-11은 `for` 반복 안에서 각 스트림을 다루기 위해 새 스레드를 생성하도록 `main`에 가할 변경 사항을 보여 줍니다.

<Listing number="21-11" file-name="src/main.rs" caption="각 스트림에 대해 새 스레드 생성">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

16장에서 배웠듯이 `thread::spawn`은 새 스레드를 만든 다음 새 스레드에서 클로저의 코드를 실행합니다. 이 코드를 실행하고 브라우저에서 _/sleep_ 을 로드한 다음, 두 개의 다른 브라우저 탭에서 _/_ 를 로드하면 _/_ 에 대한 요청들이 _/sleep_ 이 끝나기를 기다리지 않아도 된다는 것을 실제로 보게 될 것입니다. 그러나 앞에서 언급했듯이 이는 결국 시스템을 압도할 것입니다. 어떤 제한도 없이 새 스레드를 만들고 있을 것이기 때문입니다.

17장에서, 이것이 정확히 async와 await가 정말로 빛나는 종류의 상황임을 떠올렸을 수도 있습니다. 스레드 풀을 만들면서, async와 함께라면 어떻게 다르거나 같았을지 생각하면서 그것을 염두에 두세요.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### 한정된 수의 스레드 만들기

우리 스레드 풀이 비슷하고 친숙한 방식으로 동작하길 원해, 스레드에서 스레드 풀로 전환하는 것이 우리 API를 사용하는 코드에 큰 변경을 요구하지 않도록 합니다. Listing 21-12는 `thread::spawn` 대신 사용하고 싶은 `ThreadPool` 구조체의 가상 인터페이스를 보여 줍니다.

<Listing number="21-12" file-name="src/main.rs" caption="우리의 이상적인 `ThreadPool` 인터페이스">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

설정 가능한 수의 스레드, 이 경우 4개로 새 스레드 풀을 만들기 위해 `ThreadPool::new`를 사용합니다. 그런 다음 `for` 반복에서 `pool.execute`는 `thread::spawn`과 비슷한 인터페이스를 가져, 풀이 각 스트림에 대해 실행해야 할 클로저를 받습니다. `pool.execute`가 클로저를 받아 풀에 있는 스레드에 주어 실행하도록 구현해야 합니다. 이 코드는 아직 컴파일되지 않을 텐데, 컴파일러가 어떻게 고칠지 안내할 수 있도록 시도해 보겠습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### 컴파일러 주도 개발로 `ThreadPool` 만들기

Listing 21-12의 변경 사항을 _src/main.rs_ 에 적용한 다음, 개발을 안내하기 위해 `cargo check`의 컴파일러 오류를 사용합시다. 다음은 우리가 받는 첫 오류입니다.

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

좋습니다! 이 오류는 우리가 `ThreadPool` 타입이나 모듈이 필요하다고 알려 주므로, 이제 하나를 만들겠습니다. 우리 `ThreadPool` 구현은 우리 웹 서버가 하는 작업의 종류와 독립적일 것입니다. 그래서 `ThreadPool` 구현을 담을 `hello` 크레이트를 바이너리 크레이트에서 라이브러리 크레이트로 전환합시다. 라이브러리 크레이트로 변경한 후, 별도의 스레드 풀 라이브러리를 웹 요청 서비스뿐만 아니라 스레드 풀을 사용해 하고 싶은 어떤 작업에든 사용할 수도 있을 것입니다.

다음 내용을 담은 _src/lib.rs_ 파일을 만드세요. 이는 지금 우리가 가질 수 있는 `ThreadPool` 구조체의 가장 단순한 정의입니다.

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>


그런 다음, _src/main.rs_ 의 맨 위에 다음 코드를 추가하여 라이브러리 크레이트로부터 `ThreadPool`을 스코프로 가져오도록 _main.rs_ 파일을 편집합니다.

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

이 코드는 여전히 동작하지 않을 텐데, 우리가 다뤄야 할 다음 오류를 얻기 위해 다시 확인해 봅시다.

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

이 오류는 다음으로 `ThreadPool`에 대한 `new`라는 연관 함수를 만들어야 함을 나타냅니다. `new`는 인수로 `4`를 받을 수 있는 매개변수 하나를 가져야 하고 `ThreadPool` 인스턴스를 반환해야 함도 압니다. 이러한 특성을 가진 가장 단순한 `new` 함수를 구현해 봅시다.

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

음수 개의 스레드는 말이 안 되므로 `size` 매개변수의 타입으로 `usize`를 선택했습니다. 이 `4`를 스레드 컬렉션의 요소 수로 사용할 것임을 알고 있는데, 이는 3장의 [“정수 타입”][integer-types]<!-- ignore --> 절에서 논의한 대로 `usize` 타입의 용도입니다.

다시 코드를 확인해 봅시다.

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

이제 `ThreadPool`에 `execute` 메서드가 없어서 오류가 발생합니다. [“한정된 수의 스레드 만들기”](#creating-a-finite-number-of-threads)<!-- ignore --> 절에서, 우리 스레드 풀이 `thread::spawn`과 비슷한 인터페이스를 가져야 한다고 결정했음을 떠올려 보세요. 또한 `execute` 함수가 받은 클로저를 받아 풀의 유휴 스레드에 주어 실행하도록 구현하겠습니다.

매개변수로 클로저를 받도록 `ThreadPool`에 `execute` 메서드를 정의할 것입니다. 13장의 [“캡처된 값을 클로저 밖으로 옮기기”][moving-out-of-closures]<!-- ignore -->에서, 클로저를 매개변수로 받기 위해 세 가지 다른 트레이트(`Fn`, `FnMut`, `FnOnce`)를 사용할 수 있다는 것을 떠올려 보세요. 여기서 어느 종류의 클로저를 사용할지 결정해야 합니다. 결국 표준 라이브러리 `thread::spawn` 구현과 비슷한 일을 하게 될 것임을 알므로, `thread::spawn`의 시그니처가 매개변수에 어떤 바운드를 가지는지 볼 수 있습니다. 문서는 우리에게 다음을 보여 줍니다.

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

여기서 우리가 신경 쓰는 것은 `F` 타입 매개변수입니다. `T` 타입 매개변수는 반환 값과 관련이 있고, 우리는 그것에 신경 쓰지 않습니다. `spawn`이 `F`에 대한 트레이트 바운드로 `FnOnce`를 사용하는 것을 볼 수 있습니다. 이는 아마 우리에게도 원하는 것일 텐데, 결국 `execute`에서 받은 인수를 `spawn`에 전달하게 될 것이기 때문입니다. 요청을 처리하는 스레드는 그 요청의 클로저를 한 번만 실행할 것이므로, `FnOnce`의 `Once`와 일치한다는 점에서 우리가 사용하고 싶은 트레이트가 `FnOnce`라는 것을 더 확신할 수 있습니다.

`F` 타입 매개변수에는 또한 `Send` 트레이트 바운드와 `'static` 라이프타임 바운드가 있는데, 이는 우리 상황에서 유용합니다. 클로저를 한 스레드에서 다른 스레드로 옮기기 위해 `Send`가 필요하고, 스레드가 실행하는 데 얼마나 걸릴지 알 수 없으므로 `'static`이 필요합니다. 이 바운드들을 가진 `F` 타입의 제네릭 매개변수를 받는 `execute` 메서드를 `ThreadPool`에 만들어 봅시다.

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

여전히 `FnOnce` 다음에 `()`를 사용하는데, 이 `FnOnce`가 매개변수를 받지 않고 단위 타입 `()`를 반환하는 클로저를 나타내기 때문입니다. 함수 정의처럼 반환 타입은 시그니처에서 생략할 수 있지만, 매개변수가 없더라도 여전히 괄호가 필요합니다.

다시, 이는 `execute` 메서드의 가장 단순한 구현입니다. 아무것도 하지 않지만, 단지 코드를 컴파일하려는 것뿐입니다. 다시 확인해 봅시다.

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

컴파일됩니다! 그러나 `cargo run`을 시도해 브라우저에서 요청을 만들면 장의 시작 부분에서 보았던 오류를 브라우저에서 보게 될 것이라는 점에 유의하세요. 우리 라이브러리는 아직 `execute`에 전달된 클로저를 실제로 호출하지 않습니다!

> 참고: Haskell이나 러스트처럼 엄격한 컴파일러를 가진 언어들에 대해 들어 봤을 수 있는 말이 있습니다. “코드가 컴파일되면 동작한다”입니다. 그러나 이 말이 보편적으로 참은 아닙니다. 우리 프로젝트는 컴파일되지만 절대 아무것도 하지 않습니다! 우리가 실제 완전한 프로젝트를 만들고 있다면, 코드가 컴파일되고 _그리고_ 우리가 원하는 동작을 가지는지 확인하기 위해 단위 테스트를 작성하기 시작하기 좋은 시점일 것입니다.

생각해 봅시다. 클로저 대신 future를 실행할 것이었다면 여기에서 무엇이 달라질까요?

#### `new`에서 스레드 수 검증하기

`new`와 `execute`의 매개변수로 아무것도 하지 않고 있습니다. 이 함수들의 본문을 우리가 원하는 동작으로 구현해 봅시다. 먼저 `new`를 생각해 봅시다. 앞서 음수 개의 스레드를 가진 풀은 말이 안 되므로 `size` 매개변수에 부호 없는 타입을 선택했습니다. 그러나 0개의 스레드를 가진 풀도 말이 안 되지만, 0은 완벽하게 유효한 `usize`입니다. `ThreadPool` 인스턴스를 반환하기 전에 `size`가 0보다 큰지 확인하는 코드를 추가하고, Listing 21-13에서 보이듯 `assert!` 매크로를 사용해 0을 받으면 프로그램이 패닉하도록 합니다.

<Listing number="21-13" file-name="src/lib.rs" caption="`size`가 0이면 패닉하도록 `ThreadPool::new` 구현하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

또한 doc 주석으로 `ThreadPool`에 약간의 문서를 추가했습니다. 14장에서 논의한 대로 함수가 패닉할 수 있는 상황을 짚는 절을 추가함으로써 좋은 문서화 관행을 따랐다는 점에 주목하세요. `cargo doc --open`을 실행하고 `ThreadPool` 구조체를 클릭해 `new`에 대해 생성된 문서가 어떻게 보이는지 확인해 보세요!

여기서처럼 `assert!` 매크로를 추가하는 대신, 12장의 I/O 프로젝트의 Listing 12-9에서 `Config::build`로 했던 것처럼 `new`를 `build`로 변경하고 `Result`를 반환하게 할 수도 있습니다. 그러나 우리는 이 경우 어떤 스레드도 없이 스레드 풀을 만들려는 시도가 회복할 수 없는 오류여야 한다고 결정했습니다. 의욕이 있다면, `new` 함수와 비교하기 위해 다음 시그니처를 가진 `build`라는 이름의 함수를 작성해 보세요.

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### 스레드를 저장할 공간 만들기

이제 풀에 저장할 유효한 수의 스레드를 가지고 있음을 알 방법을 갖추었으니, 그 스레드들을 만들고 구조체를 반환하기 전에 `ThreadPool` 구조체에 저장할 수 있습니다. 그러나 어떻게 스레드를 “저장”할까요? `thread::spawn` 시그니처를 다시 살펴봅시다.

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`spawn` 함수는 `JoinHandle<T>`를 반환하는데, 여기서 `T`는 클로저가 반환하는 타입입니다. `JoinHandle`도 사용해 보고 어떻게 되는지 봅시다. 우리 경우 스레드 풀에 전달하는 클로저들은 연결을 처리하고 아무것도 반환하지 않을 것이므로, `T`는 단위 타입 `()`이 될 것입니다.

Listing 21-14의 코드는 컴파일되지만 아직 어떤 스레드도 만들지 않습니다. `ThreadPool`의 정의를 `thread::JoinHandle<()>` 인스턴스의 벡터를 담도록 변경했고, 벡터를 `size` 용량으로 초기화했고, 스레드를 만들기 위해 어떤 코드를 실행할 `for` 반복을 설정했고, 그것들을 담은 `ThreadPool` 인스턴스를 반환했습니다.

<Listing number="21-14" file-name="src/lib.rs" caption="스레드들을 담을 `ThreadPool`을 위한 벡터 만들기">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

`ThreadPool`의 벡터의 항목 타입으로 `thread::JoinHandle`을 사용하므로, 라이브러리 크레이트에 `std::thread`를 스코프로 가져왔습니다.

유효한 크기를 받으면 우리 `ThreadPool`은 `size`개의 항목을 담을 수 있는 새 벡터를 만듭니다. `with_capacity` 함수는 `Vec::new`와 같은 작업을 수행하지만 한 가지 중요한 차이가 있습니다. 벡터에 공간을 미리 할당합니다. 벡터에 `size`개의 요소를 저장해야 한다는 것을 알기 때문에, 이 할당을 미리 하는 것은 요소가 삽입될 때 자기 자신의 크기를 조정하는 `Vec::new`보다 약간 더 효율적입니다.

`cargo check`를 다시 실행하면 성공해야 합니다.

<!-- Old headings. Do not remove or links may break. -->
<a id ="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### `ThreadPool`에서 스레드로 코드 보내기

Listing 21-14의 `for` 반복에 스레드 생성에 관한 주석을 남겨 두었습니다. 여기서는 실제로 스레드를 어떻게 만드는지 살펴봅니다. 표준 라이브러리는 스레드를 만드는 방법으로 `thread::spawn`을 제공하며, `thread::spawn`은 스레드가 만들어지자마자 실행해야 할 어떤 코드를 받기를 기대합니다. 그러나 우리 경우에는 스레드를 만들고 나중에 보낼 코드를 _기다리게_ 하고 싶습니다. 표준 라이브러리의 스레드 구현은 그렇게 할 방법을 포함하고 있지 않으므로, 우리가 수동으로 구현해야 합니다.

이 동작을 구현하기 위해, 이 새 동작을 관리할 새 자료 구조를 `ThreadPool`과 스레드 사이에 도입하겠습니다. 이 자료 구조를 _Worker_ 라고 부르겠는데, 이는 풀링 구현에서 흔한 용어입니다. `Worker`는 실행해야 할 코드를 가져와 자기 스레드에서 그 코드를 실행합니다.

식당 주방에서 일하는 사람들을 떠올려 보세요. 일꾼들은 손님으로부터 주문이 들어올 때까지 기다리고, 그런 다음 그 주문을 받아 처리하는 책임을 맡습니다.

스레드 풀에 `JoinHandle<()>` 인스턴스의 벡터를 저장하는 대신, `Worker` 구조체의 인스턴스들을 저장할 것입니다. 각 `Worker`는 단일 `JoinHandle<()>` 인스턴스를 저장합니다. 그런 다음, 실행할 코드의 클로저를 받아 실행을 위해 이미 실행 중인 스레드에 보내는 메서드를 `Worker`에 구현할 것입니다. 또한 로깅이나 디버깅 시 풀의 서로 다른 `Worker` 인스턴스를 구분할 수 있도록 각 `Worker`에 `id`를 부여하겠습니다.

`ThreadPool`을 만들 때 일어날 새 과정은 다음과 같습니다. `Worker`를 이렇게 설정한 다음 클로저를 스레드로 보내는 코드를 구현하겠습니다.

1. `id`와 `JoinHandle<()>`을 담는 `Worker` 구조체를 정의합니다.
2. `ThreadPool`이 `Worker` 인스턴스의 벡터를 담도록 변경합니다.
3. `id` 번호를 받아 `id`와 빈 클로저로 생성된 스레드를 담는 `Worker` 인스턴스를 반환하는 `Worker::new` 함수를 정의합니다.
4. `ThreadPool::new`에서, `for` 반복 카운터를 사용해 `id`를 생성하고, 그 `id`로 새 `Worker`를 만들고, 그 `Worker`를 벡터에 저장합니다.

도전해 보고 싶다면, Listing 21-15의 코드를 보기 전에 직접 이 변경 사항들을 구현해 보세요.

준비되었나요? 다음은 앞의 수정 사항들을 만드는 한 가지 방법인 Listing 21-15입니다.

<Listing number="21-15" file-name="src/lib.rs" caption="스레드를 직접 담는 대신 `Worker` 인스턴스를 담도록 `ThreadPool` 수정하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

이제 `JoinHandle<()>` 인스턴스 대신 `Worker` 인스턴스를 담고 있으므로, `ThreadPool`의 필드 이름을 `threads`에서 `workers`로 변경했습니다. `for` 반복의 카운터를 `Worker::new`의 인수로 사용하고, 새 `Worker` 각각을 `workers`라는 이름의 벡터에 저장합니다.

외부 코드(예컨대 _src/main.rs_ 의 우리 서버)는 `ThreadPool` 안에서 `Worker` 구조체를 사용하는 구현 세부 사항을 알 필요가 없으므로, `Worker` 구조체와 그 `new` 함수를 비공개로 만듭니다. `Worker::new` 함수는 우리가 주는 `id`를 사용하고, 빈 클로저를 사용해 새 스레드를 생성해 만든 `JoinHandle<()>` 인스턴스를 저장합니다.

> 참고: 시스템 자원이 충분하지 않아 운영체제가 스레드를 만들 수 없으면, `thread::spawn`은 패닉합니다. 일부 스레드 생성은 성공할 수 있더라도, 그것은 우리의 전체 서버를 패닉하게 만들 것입니다. 단순함을 위해 이 동작은 괜찮지만, 운영 스레드 풀 구현에서는 [`std::thread::Builder`][builder]<!-- ignore -->와 그 [`spawn`][builder-spawn]<!-- ignore --> 메서드를 사용해 `Result`를 반환하게 하는 편이 좋을 것입니다.

이 코드는 컴파일되며, `ThreadPool::new`에 인수로 명시한 수만큼의 `Worker` 인스턴스를 저장할 것입니다. 그러나 _여전히_ `execute`에서 받은 클로저를 처리하지는 않습니다. 다음으로 그것을 어떻게 할지 살펴봅시다.

#### 채널을 통해 스레드로 요청 보내기

다음으로 다룰 문제는 `thread::spawn`에 주어진 클로저들이 정말로 아무것도 하지 않는다는 것입니다. 현재 `execute` 메서드에서 실행하고 싶은 클로저를 받습니다. 그러나 `ThreadPool`을 만드는 동안 각 `Worker`를 만들 때 `thread::spawn`에 실행할 클로저를 줘야 합니다.

방금 만든 `Worker` 구조체들이 `ThreadPool`에 보관된 큐에서 실행할 코드를 가져와 그것을 실행하기 위해 자기 스레드로 보내기를 원합니다.

16장에서 배운 채널—두 스레드 사이에서 통신하는 단순한 방법—이 이 사용 사례에 완벽할 것입니다. 채널을 작업의 큐로 기능하게 하고, `execute`는 `ThreadPool`에서 `Worker` 인스턴스로 작업을 보내며, 그것은 작업을 자기 스레드로 보냅니다. 다음은 계획입니다.

1. `ThreadPool`은 채널을 만들고 송신자(sender)를 들고 있습니다.
2. 각 `Worker`는 수신자(receiver)를 들고 있습니다.
3. 채널을 통해 보내고 싶은 클로저들을 담을 새 `Job` 구조체를 만듭니다.
4. `execute` 메서드는 실행하고 싶은 작업을 송신자를 통해 보냅니다.
5. 자기 스레드에서 `Worker`는 자신의 수신자를 반복하며 받은 작업의 클로저를 실행합니다.

Listing 21-16에서 보이듯, `ThreadPool::new`에서 채널을 만들고 `ThreadPool` 인스턴스에 송신자를 보관하는 것부터 시작합시다. `Job` 구조체는 지금은 아무것도 담지 않지만 채널을 통해 보내는 항목의 타입이 될 것입니다.

<Listing number="21-16" file-name="src/lib.rs" caption="`Job` 인스턴스를 전송하는 채널의 송신자를 저장하도록 `ThreadPool` 수정">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

`ThreadPool::new`에서 새 채널을 만들고 풀이 송신자를 들고 있게 합니다. 이는 성공적으로 컴파일됩니다.

스레드 풀이 채널을 만들 때 채널의 수신자를 각 `Worker`에 전달해 보겠습니다. `Worker` 인스턴스가 생성하는 스레드에서 수신자를 사용하고 싶으므로, 클로저에서 `receiver` 매개변수를 참조할 것입니다. Listing 21-17의 코드는 아직 완전히 컴파일되지 않을 것입니다.

<Listing number="21-17" file-name="src/lib.rs" caption="각 `Worker`에 수신자 전달하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

작고 직관적인 변경을 했습니다. `Worker::new`에 수신자를 전달하고, 그것을 클로저 안에서 사용합니다.

이 코드를 확인하려고 하면, 이 오류를 받습니다.

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

코드가 `receiver`를 여러 `Worker` 인스턴스에 전달하려고 합니다. 16장에서 떠올렸겠지만, 이는 동작하지 않을 것입니다. 러스트가 제공하는 채널 구현은 다중 _producer_, 단일 _consumer_ 입니다. 즉, 이 코드를 고치기 위해 채널의 소비 끝부분을 그냥 복제할 수는 없습니다. 또한 여러 메시지를 여러 소비자에게 여러 번 보내는 것을 원하지도 않습니다. 우리는 각 메시지가 한 번 처리되도록 여러 `Worker` 인스턴스가 있는 메시지 목록 하나를 원합니다.

또한 채널 큐에서 작업을 꺼내는 것은 `receiver`를 변경하는 것을 포함하므로, 스레드들은 `receiver`를 안전하게 공유하고 수정할 방법이 필요합니다. 그렇지 않으면 (16장에서 다룬) 경합 조건이 발생할 수 있습니다.

16장에서 논의한 스레드 안전 스마트 포인터를 떠올려 보세요. 여러 스레드에 걸쳐 소유권을 공유하고 스레드들이 값을 수정할 수 있게 하려면 `Arc<Mutex<T>>`를 사용해야 합니다. `Arc` 타입은 여러 `Worker` 인스턴스가 수신자를 소유하게 해 주고, `Mutex`는 한 번에 단 하나의 `Worker`만 수신자에서 작업을 받도록 보장합니다. Listing 21-18은 우리가 가해야 할 변경 사항을 보여 줍니다.

<Listing number="21-18" file-name="src/lib.rs" caption="`Arc`와 `Mutex`를 사용해 `Worker` 인스턴스 사이에서 수신자 공유하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

`ThreadPool::new`에서 수신자를 `Arc`와 `Mutex`로 감쌉니다. 새 `Worker`마다 `Arc`를 복제하여 참조 카운트를 늘려, `Worker` 인스턴스가 수신자의 소유권을 공유할 수 있도록 합니다.

이러한 변경 사항으로 코드가 컴파일됩니다! 거의 다 왔습니다!

#### `execute` 메서드 구현하기

마침내 `ThreadPool`에 `execute` 메서드를 구현해 봅시다. 또한 `Job`을 구조체에서 `execute`가 받는 클로저 타입을 담는 트레이트 객체에 대한 타입 별칭으로 변경할 것입니다. 20장의 [“타입 동의어와 타입 별칭”][type-aliases]<!-- ignore --> 절에서 논의했듯이, 타입 별칭은 사용을 쉽게 하기 위해 긴 타입을 더 짧게 만들어 줍니다. Listing 21-19를 보세요.

<Listing number="21-19" file-name="src/lib.rs" caption="각 클로저를 담는 `Box`를 위한 `Job` 타입 별칭을 만들고 채널로 작업 보내기">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

`execute`에서 받은 클로저로 새 `Job` 인스턴스를 만든 후, 채널의 송신 끝으로 그 작업을 보냅니다. 보내기가 실패하는 경우를 위해 `send`에 `unwrap`을 호출합니다. 예를 들어, 모든 스레드가 실행을 멈춰 수신 끝이 새 메시지 받기를 멈춘 경우 이런 일이 일어날 수 있습니다. 현재로서는 스레드 실행을 멈출 수 없습니다. 풀이 존재하는 한 스레드들은 계속 실행됩니다. `unwrap`을 사용하는 이유는 실패 케이스가 발생하지 않을 것임을 알지만 컴파일러는 그것을 모르기 때문입니다.

그러나 아직 완전히 끝난 것은 아닙니다! `Worker`에서 `thread::spawn`에 전달되는 클로저는 여전히 채널의 수신 끝을 _참조_ 만 합니다. 대신, 클로저가 영원히 반복하면서 채널의 수신 끝에 작업을 요청하고 받았을 때 그 작업을 실행하기를 원합니다. Listing 21-20에 보이는 변경 사항을 `Worker::new`에 가합시다.

<Listing number="21-20" file-name="src/lib.rs" caption="`Worker` 인스턴스의 스레드에서 작업을 받고 실행하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

여기서 먼저 뮤텍스를 획득하기 위해 `receiver`에 `lock`을 호출하고, 그런 다음 어떤 오류든 패닉하기 위해 `unwrap`을 호출합니다. 락을 획득하는 것은 뮤텍스가 _poisoned_ 상태에 있으면 실패할 수 있는데, 이는 다른 어떤 스레드가 락을 해제하지 않고 락을 들고 있는 동안 패닉했을 때 일어날 수 있습니다. 이 상황에서, 이 스레드가 패닉하도록 `unwrap`을 호출하는 것이 올바른 행동입니다. 자유롭게 이 `unwrap`을 여러분에게 의미 있는 오류 메시지가 있는 `expect`로 변경해도 됩니다.

뮤텍스의 락을 얻으면, 채널에서 `Job`을 받기 위해 `recv`를 호출합니다. 마지막 `unwrap`은 여기서도 어떤 오류든 그냥 지나가는데, `send` 메서드가 수신자가 닫혔을 때 `Err`를 반환하는 것과 비슷하게 송신자를 들고 있는 스레드가 종료된 경우 발생할 수 있습니다.

`recv` 호출은 블록되므로, 아직 작업이 없으면 현재 스레드는 작업이 사용 가능해질 때까지 기다립니다. `Mutex<T>`는 한 번에 단 하나의 `Worker` 스레드만 작업을 요청하고 있도록 보장합니다.

이제 우리 스레드 풀이 동작하는 상태입니다! `cargo run`을 실행하고 몇 가지 요청을 만들어 보세요.

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^

warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

성공입니다! 이제 비동기적으로 연결을 실행하는 스레드 풀을 가지고 있습니다. 4개를 초과하는 스레드가 만들어지지 않으므로, 서버가 많은 요청을 받아도 시스템이 과부하되지 않을 것입니다. _/sleep_ 에 요청을 만들면 서버는 다른 스레드가 그것들을 실행하게 함으로써 다른 요청들을 서비스할 수 있을 것입니다.

> 참고: _/sleep_ 을 여러 브라우저 창에서 동시에 열면, 5초 간격으로 한 번에 하나씩 로드될 수 있습니다. 일부 웹 브라우저는 캐싱을 위해 같은 요청의 여러 인스턴스를 순차적으로 실행합니다. 이 제한은 우리 웹 서버 때문이 아닙니다.

작업을 위해 클로저 대신 future를 사용했다면 Listing 21-18, 21-19, 21-20의 코드가 어떻게 달라졌을지 잠시 멈추고 생각해 볼 좋은 시점입니다. 어떤 타입이 바뀔까요? 메서드 시그니처는 어떻게 다를까요? 코드의 어느 부분이 같게 유지될까요?

17장과 19장에서 `while let` 반복에 대해 배운 후, 왜 우리가 Listing 21-21에서 보이는 것처럼 `Worker` 스레드 코드를 작성하지 않았는지 궁금했을 수도 있습니다.

<Listing number="21-21" file-name="src/lib.rs" caption="`while let`을 사용한 `Worker::new`의 대안적 구현">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

이 코드는 컴파일되고 실행되지만, 원하는 스레딩 동작을 만들어 내지는 않습니다. 느린 요청은 여전히 다른 요청들이 처리되기를 기다리게 만들 것입니다. 이유는 다소 미묘합니다. `Mutex` 구조체에는 공개 `unlock` 메서드가 없습니다. 락의 소유권이 `lock` 메서드가 반환하는 `LockResult<MutexGuard<T>>` 안의 `MutexGuard<T>`의 라이프타임에 기반하기 때문입니다. 컴파일 시점에 빌림 검사기는 `Mutex`로 보호되는 자원이 우리가 락을 들고 있지 않으면 접근될 수 없다는 규칙을 강제할 수 있습니다. 그러나 이 구현은 우리가 `MutexGuard<T>`의 라이프타임을 의식하지 않으면 의도한 것보다 더 오래 락이 유지되는 결과를 낼 수도 있습니다.

`let job = receiver.lock().unwrap().recv().unwrap();`을 사용하는 Listing 21-20의 코드는 동작합니다. `let`을 사용하면, 등호 오른쪽의 표현식에서 사용된 임시 값들은 `let` 문이 끝날 때 즉시 드롭됩니다. 그러나 `while let`(그리고 `if let`과 `match`)은 연관된 블록의 끝까지 임시 값들을 드롭하지 않습니다. Listing 21-21에서는 `job()` 호출 동안 락이 유지된 채로 남아 있어, 다른 `Worker` 인스턴스들이 작업을 받을 수 없습니다.

[type-aliases]: ch20-03-advanced-types.html#type-synonyms-and-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[moving-out-of-closures]: ch13-01-closures.html#moving-captured-values-out-of-closures
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
