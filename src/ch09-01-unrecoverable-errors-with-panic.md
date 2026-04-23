## `panic!`을 사용한 복구 불가능한 에러

때로는 코드에서 나쁜 일이 일어나고, 그것에 대해 여러분이 할 수 있는 일이 없습니다.
이런 경우를 위해 러스트에는 `panic!` 매크로가 있습니다. 실전에서 패닉을 일으키는
두 가지 방법이 있습니다. 우리 코드가 패닉하도록 만드는 동작(예: 배열의 끝을 지난
자리 접근)을 취하거나, `panic!` 매크로를 명시적으로 호출하는 것입니다. 두 경우
모두 우리는 프로그램에서 패닉을 일으킵니다. 기본적으로 이 패닉들은 실패 메시지를
출력하고, 되감기(unwind)하고, 스택을 정리하고, 종료합니다. 환경 변수를 통해 패닉이
일어났을 때 러스트가 호출 스택을 표시하도록 해서 패닉의 원인을 추적하기 쉽게
만들 수도 있습니다.

> ### 패닉에 대한 반응으로 스택을 되감거나 즉시 종료하기
>
> 기본적으로, 패닉이 일어나면 프로그램은 *되감기(unwinding)*를 시작합니다. 이는
> 러스트가 스택을 거꾸로 올라가며 마주치는 각 함수의 데이터를 정리한다는 뜻입
> 니다. 그러나 거슬러 올라가며 정리하는 일은 많은 작업입니다. 따라서 러스트는
> 정리 없이 프로그램을 끝내는 *즉시 종료(aborting)*라는 대안을 선택할 수 있게 해
> 줍니다.
>
> 이 경우 프로그램이 사용하던 메모리는 운영체제가 정리해야 합니다. 프로젝트에서
> 결과 바이너리를 가능한 한 작게 만들어야 한다면, *Cargo.toml* 파일의 적절한
> `[profile]` 섹션에 `panic = 'abort'`를 추가해 패닉 시 되감기에서 즉시 종료로
> 전환할 수 있습니다. 예를 들어, 릴리스 모드에서 패닉 시 즉시 종료를 원한다면
> 다음을 추가하세요.
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

간단한 프로그램에서 `panic!`을 호출해 봅시다.

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

프로그램을 실행하면 다음과 비슷한 결과를 볼 수 있습니다.

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

`panic!` 호출이 마지막 두 줄에 담긴 에러 메시지를 유발합니다. 첫 줄은 우리의 패닉
메시지와 패닉이 일어난 소스 코드의 위치를 보여 줍니다. *src/main.rs:2:5*는 우리
*src/main.rs* 파일의 두 번째 줄, 다섯 번째 문자임을 나타냅니다.

이 경우, 표시된 줄은 우리 코드의 일부이고, 그 줄로 가면 `panic!` 매크로 호출을
볼 수 있습니다. 다른 경우에는 `panic!` 호출이 우리 코드가 호출하는 코드 안에 있을
수 있고, 에러 메시지가 보고하는 파일명과 줄 번호는 결국 `panic!` 호출로 이어진
우리 코드의 줄이 아니라 `panic!` 매크로가 호출된 다른 사람의 코드가 됩니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

우리는 `panic!` 호출이 왔던 함수들의 백트레이스(backtrace)를 사용해 문제를 일으키는
코드의 부분을 알아낼 수 있습니다. `panic!` 백트레이스를 사용하는 방법을 이해하기
위해, 또 다른 예를 보고, 우리 코드에서 직접 매크로를 호출하는 대신 우리 코드의
버그 때문에 라이브러리에서 `panic!` 호출이 올 때 어떤지 살펴봅시다. Listing 9-1에는
유효한 인덱스의 범위를 벗어난 벡터의 인덱스에 접근하려는 코드가 있습니다.

<Listing number="9-1" file-name="src/main.rs" caption="벡터의 끝을 지난 요소에 접근하려는 시도. `panic!` 호출을 유발합니다.">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

여기서 우리는 벡터의 100번째 요소(인덱싱이 0에서 시작하므로 인덱스 99에 있는)에
접근하려 하고 있지만, 벡터에는 요소가 세 개뿐입니다. 이 상황에서 러스트는 패닉할
것입니다. `[]`를 사용하면 요소를 반환하기로 되어 있지만, 유효하지 않은 인덱스를
건네면 러스트가 반환할 수 있는 올바른 요소가 여기엔 없습니다.

C에서는 자료 구조의 끝을 지난 자리에서 읽으려는 시도가 정의되지 않은 동작입니다.
자료 구조에 속하지 않는 메모리일지라도, 자료 구조의 그 요소에 해당할 만한 메모리
위치에 무엇이 있든 그것을 얻게 될 수 있습니다. 이를 *버퍼 오버리드(buffer
overread)*라고 부르며, 공격자가 인덱스를 조작해 자료 구조 뒤에 저장된, 접근이 허용
되지 않아야 할 데이터를 읽을 수 있다면 보안 취약점으로 이어질 수 있습니다.

이런 종류의 취약점으로부터 여러분의 프로그램을 보호하기 위해, 존재하지 않는 인덱스
의 요소를 읽으려 하면 러스트는 실행을 멈추고 계속하기를 거부합니다. 시도해서 보
겠습니다.

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

이 에러는 우리 *main.rs*의 4번째 줄을 가리키는데, 여기서 우리는 `v`에 있는 벡터의
인덱스 99에 접근하려 합니다.

`note:` 줄은 에러를 유발한 일이 정확히 무엇이었는지에 대한 백트레이스를 얻으려면
`RUST_BACKTRACE` 환경 변수를 설정할 수 있다고 알려 줍니다. *백트레이스(backtrace)*
는 이 지점에 도달하기 위해 호출된 모든 함수의 목록입니다. 러스트의 백트레이스는
다른 언어에서 그렇듯이 동작합니다. 백트레이스를 읽는 비결은, 맨 위에서 시작해서
여러분이 작성한 파일이 보일 때까지 읽는 것입니다. 그 자리가 문제가 시작된 곳입
니다. 그 자리 위의 줄들은 여러분의 코드가 호출한 코드입니다. 그 아래의 줄들은
여러분의 코드를 호출한 코드입니다. 앞뒤의 이 줄들에는 러스트 코어 코드, 표준 라이
브러리 코드, 또는 여러분이 사용하는 크레이트가 포함될 수 있습니다. `RUST_BACKTRACE`
환경 변수를 `0` 이외의 어떤 값으로든 설정해서 백트레이스를 얻어 봅시다. Listing
9-2는 여러분이 볼 것과 비슷한 출력을 보여 줍니다.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="`RUST_BACKTRACE` 환경 변수가 설정되었을 때 `panic!` 호출에 의해 생성되는 백트레이스">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

출력이 많군요! 여러분이 보는 정확한 출력은 운영체제와 러스트 버전에 따라 다를 수
있습니다. 이 정보를 담은 백트레이스를 얻으려면 디버그 심볼이 활성화되어 있어야
합니다. 디버그 심볼은 `--release` 플래그 없이 `cargo build`나 `cargo run`을 사용할
때 기본으로 활성화되며, 우리는 여기서 그렇게 했습니다.

Listing 9-2의 출력에서 백트레이스의 6번째 줄은 문제를 일으키는 우리 프로젝트의
줄을 가리킵니다. *src/main.rs*의 4번째 줄이죠. 프로그램이 패닉하기를 원하지
않는다면, 우리가 작성한 파일을 언급하는 첫 줄이 가리키는 위치에서 조사를 시작해야
합니다. 의도적으로 패닉할 코드를 작성했던 Listing 9-1에서 패닉을 고치는 방법은
벡터 인덱스의 범위를 벗어난 요소를 요청하지 않는 것입니다. 앞으로 여러분의 코드가
패닉할 때, 어떤 값들로 어떤 동작이 취해져서 패닉을 일으키는지, 그리고 그 코드가
대신 무엇을 해야 하는지 알아내야 합니다.

`panic!`으로 돌아가, 에러 상황을 처리하는 데 `panic!`을 언제 써야 하고 언제 쓰지
말아야 하는지는 이 장 뒤쪽의
[“`panic!`을 쓸 것인가 말 것인가”][to-panic-or-not-to-panic]<!-- ignore --> 절에서
다룰 것입니다. 다음으로는 `Result`를 사용해 에러에서 복구하는 방법을 살펴봅니다.

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
