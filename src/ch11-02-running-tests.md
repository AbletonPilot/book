## 테스트 실행 방법 제어하기

`cargo run`이 코드를 컴파일하고 결과 바이너리를 실행하는 것처럼, `cargo test`는
코드를 테스트 모드로 컴파일하고 결과 테스트 바이너리를 실행합니다. `cargo test`
가 생성한 바이너리의 기본 동작은 모든 테스트를 병렬로 실행하고 테스트 실행 중에
생성되는 출력을 캡처해 표시되지 않도록 함으로써, 테스트 결과와 관련된 출력을
더 읽기 쉽게 만드는 것입니다. 그러나 명령행 옵션을 지정해 이 기본 동작을 바꿀
수 있습니다.

일부 명령행 옵션은 `cargo test`에, 일부는 결과 테스트 바이너리에 전달됩니다. 두
종류의 인수를 구분하려면, `cargo test`에 전달할 인수를 먼저 나열하고 구분자 `--`
뒤에 테스트 바이너리로 전달할 인수를 나열합니다. `cargo test --help`를 실행하면
`cargo test`와 함께 사용할 수 있는 옵션이, `cargo test -- --help`를 실행하면
구분자 뒤에 사용할 수 있는 옵션이 표시됩니다. 이 옵션들은 [_The `rustc` Book_
의 “Tests” 섹션][tests]에도 문서화되어 있습니다.

[tests]: https://doc.rust-lang.org/rustc/tests/index.html

### 테스트를 병렬 또는 순차 실행하기

여러 테스트를 실행할 때 기본적으로는 스레드를 사용해 병렬로 실행됩니다. 더
빨리 끝나고 피드백을 더 빨리 받을 수 있다는 뜻입니다. 테스트가 동시에 실행되기
때문에, 테스트가 서로에 의존하지 않도록 해야 하고, 현재 작업 디렉터리나 환경
변수 같은 공유 환경을 포함한 어떤 공유 상태에도 의존하지 않도록 해야 합니다.

예를 들어 각 테스트가 _test-output.txt_ 라는 파일을 디스크에 만들고 거기에
어떤 데이터를 쓰는 코드를 실행한다고 해 봅시다. 그런 다음 각 테스트가 그 파일의
데이터를 읽고, 파일이 특정 값을 포함하는지 단언하는데, 그 값이 테스트마다
다릅니다. 테스트가 동시에 실행되기 때문에, 한 테스트가 파일에 쓰고 읽는 사이에
다른 테스트가 파일을 덮어쓸 수 있습니다. 그러면 두 번째 테스트는 코드가
틀려서가 아니라 병렬로 실행되는 동안 테스트가 서로 간섭했기 때문에 실패합니다.
한 가지 해법은 각 테스트가 다른 파일에 쓰도록 하는 것이고, 또 다른 해법은
테스트를 한 번에 하나씩 실행하는 것입니다.

테스트를 병렬로 실행하고 싶지 않거나 사용하는 스레드 수를 더 세밀하게 제어하고
싶다면, `--test-threads` 플래그와 사용할 스레드 수를 테스트 바이너리에 전달할
수 있습니다. 다음 예를 보세요.

```console
$ cargo test -- --test-threads=1
```

테스트 스레드 수를 `1`로 설정하여 프로그램에 어떤 병렬성도 사용하지 말라고
지시합니다. 하나의 스레드로 테스트를 실행하는 것은 병렬로 실행하는 것보다
시간이 더 오래 걸리지만, 테스트가 상태를 공유하더라도 서로 간섭하지 않습니다.

### 함수 출력 보기

기본적으로 테스트가 통과하면, 러스트 테스트 라이브러리는 표준 출력으로 출력된
모든 것을 캡처합니다. 예를 들어 테스트에서 `println!`을 호출하고 테스트가
통과하면, 터미널에서 `println!` 출력을 볼 수 없고 테스트가 통과했다는 줄만
보입니다. 테스트가 실패하면 표준 출력에 출력된 내용이 나머지 실패 메시지와
함께 나타납니다.

예시로 Listing 11-10은 매개변수의 값을 출력하고 10을 반환하는 엉뚱한 함수와,
통과하는 테스트 하나, 실패하는 테스트 하나를 보여 줍니다.

<Listing number="11-10" file-name="src/lib.rs" caption="`println!`을 호출하는 함수를 위한 테스트들">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

이 테스트를 `cargo test`로 실행하면 다음 출력이 보입니다.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

이 출력 어디에도, 통과하는 테스트가 실행될 때 출력되는 `I got the value 4`가
보이지 않음에 유의하세요. 그 출력은 캡처된 것입니다. 실패한 테스트의 출력인
`I got the value 8`은 테스트 요약 출력 섹션에 나타나며, 테스트 실패 원인도
함께 표시됩니다.

통과하는 테스트의 출력도 보고 싶다면, `--show-output` 플래그로 러스트가 성공한
테스트의 출력도 보여 주도록 지시할 수 있습니다.

```console
$ cargo test -- --show-output
```

Listing 11-10의 테스트를 `--show-output` 플래그와 함께 다시 실행하면 다음 출력이
보입니다.

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### 이름으로 테스트 일부만 실행하기

전체 테스트 스위트를 실행하는 데 시간이 오래 걸릴 때가 있습니다. 특정 영역의
코드 작업을 하고 있다면, 그 코드와 관련된 테스트만 실행하고 싶을 수 있습니다.
실행할 테스트의 이름(들)을 `cargo test`에 인수로 전달해 어느 테스트를 실행할지
선택할 수 있습니다.

테스트 일부만 실행하는 방법을 보이기 위해, Listing 11-11처럼 `add_two` 함수에
대한 세 개의 테스트를 먼저 만들고 그중 어느 것을 실행할지 선택해 보겠습니다.

<Listing number="11-11" file-name="src/lib.rs" caption="세 개의 서로 다른 이름을 가진 세 테스트">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

앞서 본 것처럼 인수 없이 테스트를 실행하면 모든 테스트가 병렬로 실행됩니다.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### 단일 테스트 실행하기

`cargo test`에 테스트 함수 이름을 전달해 해당 테스트만 실행할 수 있습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

`one_hundred`라는 이름의 테스트만 실행되고, 나머지 두 테스트는 이름이 일치하지
않았습니다. 테스트 출력은 끝에 `2 filtered out`을 표시해 실행되지 않은 테스트가
더 있음을 알려 줍니다.

이런 방식으로 여러 테스트 이름을 지정할 수는 없습니다. `cargo test`에 전달된 첫
번째 값만 사용됩니다. 하지만 여러 테스트를 실행하는 방법은 있습니다.

#### 여러 테스트를 필터링해서 실행하기

테스트 이름의 일부를 지정하면 그 값과 일치하는 이름의 테스트가 모두 실행됩니다.
예를 들어 우리 테스트 중 두 개의 이름에 `add`가 들어 있으므로, `cargo test add`
를 실행해 둘을 실행할 수 있습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

이 명령은 이름에 `add`가 들어 있는 모든 테스트를 실행하고, `one_hundred`라는
이름의 테스트는 걸러냈습니다. 또한 테스트가 속한 모듈 이름이 테스트 이름의
일부가 되므로, 모듈 이름으로 필터링해 한 모듈의 모든 테스트를 실행할 수 있음에
유의하세요.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-some-tests-unless-specifically-requested"></a>

### 명시적으로 요청했을 때만 테스트 실행하기

몇몇 특정 테스트는 실행 시간이 매우 오래 걸릴 수 있어서, 대부분의 `cargo test`
실행에서는 제외하고 싶을 수 있습니다. 실행하고 싶은 모든 테스트를 인수로
나열하는 대신, 아래처럼 시간이 오래 걸리는 테스트에 `ignore` 속성을 붙여 제외할
수 있습니다.

<span class="filename">파일명: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

`#[test]` 뒤에 제외하고 싶은 테스트에 `#[ignore]` 줄을 추가합니다. 이제 테스트를
실행하면 `it_works`는 실행되지만 `expensive_test`는 실행되지 않습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

`expensive_test` 함수는 `ignored`로 표시됩니다. 무시된 테스트만 실행하려면
`cargo test -- --ignored`를 사용할 수 있습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

어느 테스트를 실행할지 제어함으로써, `cargo test` 결과를 빠르게 받을 수 있도록
보장할 수 있습니다. 무시된 테스트의 결과를 확인하는 것이 합당하고 결과를 기다릴
시간이 있는 시점이 되면, 대신 `cargo test -- --ignored`를 실행하세요. 무시
여부와 관계없이 모든 테스트를 실행하려면 `cargo test -- --include-ignored`를
실행하면 됩니다.
