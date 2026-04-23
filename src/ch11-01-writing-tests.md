## 테스트 작성하는 방법

_테스트(test)_ 는 비테스트 코드가 기대한 방식대로 동작하는지 검증하는 러스트
함수입니다. 테스트 함수의 본문은 보통 다음 세 가지 동작을 수행합니다.

- 필요한 데이터나 상태를 준비한다.
- 테스트하려는 코드를 실행한다.
- 결과가 기대한 것과 같은지 단언(assert)한다.

이러한 동작을 수행하는 테스트를 작성하기 위해 러스트가 제공하는 기능들을
살펴봅시다. 여기에는 `test` 속성, 몇 가지 매크로, `should_panic` 속성이
포함됩니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-anatomy-of-a-test-function"></a>

### 테스트 함수 구성하기

가장 단순한 형태의 러스트 테스트는 `test` 속성이 붙은 함수입니다. 속성은 러스트
코드 조각에 대한 메타데이터이며, 한 가지 예로 5장에서 구조체와 함께 사용한
`derive` 속성이 있습니다. 함수를 테스트 함수로 바꾸려면 `fn` 앞 줄에 `#[test]`를
추가하면 됩니다. `cargo test` 명령으로 테스트를 실행하면, 러스트는 애너테이션이
붙은 함수들을 실행하는 테스트 러너 바이너리를 빌드해 각 테스트 함수가 통과했는지
실패했는지 보고합니다.

카고로 새 라이브러리 프로젝트를 만들면, 테스트 함수를 포함한 테스트 모듈이
자동으로 생성됩니다. 이 모듈은 새 프로젝트를 시작할 때마다 정확한 구조와 문법을
매번 찾아보지 않아도 되도록 테스트 작성 템플릿을 제공합니다. 원하는 만큼 추가
테스트 함수와 테스트 모듈을 더할 수 있습니다!

실제 코드를 테스트하기 전에, 먼저 템플릿 테스트로 실험하면서 테스트가 어떻게
동작하는지 몇 가지 측면을 살펴보겠습니다. 그런 다음 우리가 작성한 코드를
호출하고 그 동작이 올바른지 단언하는 실전 테스트를 작성합니다.

두 숫자를 더하는 `adder`라는 새 라이브러리 프로젝트를 만들어 봅시다.

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

`adder` 라이브러리의 _src/lib.rs_ 파일 내용은 Listing 11-1과 같을 것입니다.

<Listing number="11-1" file-name="src/lib.rs" caption="`cargo new`가 자동으로 생성한 코드">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

파일은 테스트할 대상이 있도록 예시용 `add` 함수로 시작합니다.

지금은 `it_works` 함수에만 집중합시다. `#[test]` 애너테이션에 유의하세요. 이
속성은 이 함수가 테스트 함수임을 가리키므로, 테스트 러너가 이 함수를 테스트로
취급해야 함을 알 수 있습니다. 공통 시나리오를 준비하거나 공통 작업을 수행하기
위해 `tests` 모듈에 테스트가 아닌 함수도 둘 수 있으므로, 어느 함수가 테스트인지
항상 표시해야 합니다.

예시 함수 본문은 `assert_eq!` 매크로를 사용해 `add`를 2와 2로 호출한 결과를 담고
있는 `result`가 4와 같은지 단언합니다. 이 단언은 일반적인 테스트의 형식을 보여
주는 예시 역할을 합니다. 이 테스트가 통과하는지 실행해 봅시다.

`cargo test` 명령은 프로젝트의 모든 테스트를 실행합니다. Listing 11-2처럼
말이죠.

<Listing number="11-2" caption="자동 생성된 테스트를 실행한 결과">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

카고가 테스트를 컴파일하고 실행했습니다. `running 1 test`라는 줄이 보입니다.
다음 줄은 생성된 테스트 함수의 이름인 `tests::it_works`와, 그 테스트를 실행한
결과가 `ok`임을 보여 줍니다. 전체 요약인 `test result: ok.`는 모든 테스트가
통과했음을 의미하며, `1 passed; 0 failed`라고 쓰인 부분은 통과 또는 실패한
테스트 수의 합계를 나타냅니다.

특정 상황에서 테스트가 실행되지 않도록 무시됨 상태로 표시할 수도 있습니다.
이에 대해서는 이 장의 [“명시적으로 요청했을 때만 테스트 실행하기”][ignoring]<!-- ignore -->
절에서 다룹니다. 여기서는 그렇게 하지 않았으므로 요약에 `0 ignored`가 보입니다.
`cargo test` 명령에 인수를 전달해 이름이 특정 문자열과 일치하는 테스트만 실행할
수도 있는데, 이를 _필터링(filtering)_ 이라고 하며 [“이름으로 테스트 일부만
실행하기”][subset]<!-- ignore --> 절에서 다룹니다. 여기서는 실행할 테스트를
필터링하지 않았으므로 요약의 끝에 `0 filtered out`이 표시됩니다.

`0 measured` 통계는 성능을 측정하는 벤치마크 테스트를 위한 것입니다. 이 글을
쓰는 시점에서 벤치마크 테스트는 나이틀리(nightly) 러스트에서만 사용할 수
있습니다. 자세한 내용은 [벤치마크 테스트 문서][bench]를 참고하세요.

`Doc-tests adder`로 시작하는 다음 테스트 출력 부분은 문서 테스트의 결과입니다.
아직 문서 테스트는 없지만, 러스트는 API 문서에 있는 코드 예제를 모두 컴파일할
수 있습니다. 이 기능은 문서와 코드가 동기화되도록 도와줍니다! 문서 테스트
작성법은 14장의 [“테스트로서의 문서 주석”][doc-comments]<!-- ignore --> 절에서
다룹니다. 지금은 `Doc-tests` 출력은 무시하겠습니다.

이제 테스트를 우리 필요에 맞게 바꿔 봅시다. 먼저 `it_works` 함수 이름을 다음과
같이 `exploration` 같은 다른 이름으로 바꿔 봅시다.

<span class="filename">파일명: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

그런 다음 `cargo test`를 다시 실행합니다. 이제 출력에 `it_works` 대신
`exploration`이 나타납니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

이제 또 다른 테스트를 추가할 텐데, 이번에는 실패하는 테스트를 만들어 보겠습니다!
테스트는 테스트 함수 안에서 무언가가 패닉할 때 실패합니다. 각 테스트는 새 스레드
에서 실행되며, 메인 스레드가 테스트 스레드가 죽은 것을 보면 그 테스트는 실패한
것으로 표시됩니다. 9장에서 패닉을 일으키는 가장 단순한 방법이 `panic!` 매크로를
호출하는 것임을 이야기했습니다. `another`라는 함수로 새 테스트를 입력해,
_src/lib.rs_ 파일이 Listing 11-3과 같이 보이게 하세요.

<Listing number="11-3" file-name="src/lib.rs" caption="`panic!` 매크로를 호출하기 때문에 실패할 두 번째 테스트 추가하기">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

`cargo test`로 테스트를 다시 실행합니다. 출력은 Listing 11-4처럼 `exploration`
테스트는 통과했고 `another`는 실패했음을 보여 주어야 합니다.

<Listing number="11-4" caption="한 테스트는 통과하고 한 테스트는 실패한 경우의 테스트 결과">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check the line number of the panic matches the line number in the following paragraph
 -->

`ok` 대신 `test tests::another` 줄은 `FAILED`를 보여 줍니다. 개별 결과와 요약
사이에 새로운 섹션 두 개가 나타납니다. 첫 번째 섹션은 각 테스트 실패의 상세
원인을 표시합니다. 이 경우 `tests::another`가 _src/lib.rs_ 파일의 17번째
줄에서 `Make this test fail` 메시지와 함께 패닉해 실패했다는 상세 정보를
얻습니다. 다음 섹션은 실패한 모든 테스트의 이름만 나열하며, 테스트가 많고
실패 출력이 길 때 유용합니다. 실패한 테스트의 이름을 사용해 그 테스트만 실행
하여 더 쉽게 디버깅할 수 있습니다. 테스트 실행 방법에 대해서는 [“테스트 실행
방법 제어하기”][controlling-how-tests-are-run]<!-- ignore --> 절에서 더
이야기합니다.

요약 줄이 마지막에 표시됩니다. 전체 결과는 `FAILED`입니다. 하나는 통과하고
하나는 실패했습니다.

여러 상황에서 테스트 결과가 어떻게 보이는지 확인했으니, 이제 `panic!` 외에
테스트에 유용한 매크로들을 살펴보겠습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="checking-results-with-the-assert-macro"></a>

### `assert!`로 결과 검사하기

표준 라이브러리가 제공하는 `assert!` 매크로는 테스트에서 어떤 조건이 `true`로
평가됨을 보장하고 싶을 때 유용합니다. `assert!` 매크로에는 불(Boolean) 값으로
평가되는 인수를 하나 전달합니다. 값이 `true`면 아무 일도 일어나지 않고 테스트는
통과합니다. 값이 `false`면 `assert!` 매크로가 `panic!`을 호출해 테스트를
실패시킵니다. `assert!` 매크로를 사용하면 코드가 우리가 의도한 대로 동작하는지
검사하는 데 도움이 됩니다.

5장의 Listing 5-15에서 `Rectangle` 구조체와 `can_hold` 메서드를 사용했고, 이를
여기 Listing 11-5에서 다시 보여 줍니다. 이 코드를 _src/lib.rs_ 파일에 넣고
`assert!` 매크로로 테스트를 작성해 봅시다.

<Listing number="11-5" file-name="src/lib.rs" caption="5장의 `Rectangle` 구조체와 `can_hold` 메서드">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

`can_hold` 메서드는 불 값을 반환하므로 `assert!` 매크로에 완벽히 어울리는 쓰임새
입니다. Listing 11-6에서는 너비 8, 높이 7인 `Rectangle` 인스턴스를 만들고,
너비 5, 높이 1인 다른 `Rectangle` 인스턴스를 그 안에 담을 수 있는지 단언하는
방식으로 `can_hold` 메서드를 연습하는 테스트를 작성합니다.

<Listing number="11-6" file-name="src/lib.rs" caption="더 큰 사각형이 더 작은 사각형을 실제로 담을 수 있는지 검사하는 `can_hold` 테스트">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

`tests` 모듈 안의 `use super::*;` 줄에 유의하세요. `tests` 모듈은 7장의
[“모듈 트리에서 항목을 참조하는 경로”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->
절에서 다룬 일반적인 가시성 규칙을 따르는 일반 모듈입니다. `tests` 모듈은 내부
모듈이므로, 외부 모듈의 테스트 대상 코드를 내부 모듈의 스코프로 가져와야 합니다.
여기서는 글롭(glob)을 사용하므로, 외부 모듈에서 정의한 모든 것이 이 `tests`
모듈에서 사용 가능해집니다.

테스트 이름을 `larger_can_hold_smaller`로 지었고, 필요한 두 `Rectangle`
인스턴스를 만들었습니다. 그런 다음 `assert!` 매크로를 호출하며
`larger.can_hold(&smaller)`의 결과를 전달했습니다. 이 식은 `true`를 반환할
것으로 예상되므로 테스트는 통과해야 합니다. 확인해 봅시다!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

통과합니다! 이제 다른 테스트를 추가해, 이번에는 작은 사각형이 큰 사각형을 담을
수 _없다_ 는 것을 단언해 봅시다.

<span class="filename">파일명: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

이 경우 `can_hold` 함수의 올바른 결과는 `false`이므로, `assert!` 매크로에 넘기기
전에 결과를 부정해야 합니다. 결과적으로 `can_hold`가 `false`를 반환하면 테스트가
통과합니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

두 테스트가 통과했습니다! 이제 코드에 버그를 심으면 테스트 결과가 어떻게
달라지는지 살펴봅시다. `can_hold` 메서드의 구현에서 너비를 비교할 때 부등호
`>`를 `<`로 바꿔 보겠습니다.

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

테스트를 실행하면 다음과 같은 결과가 나옵니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

테스트가 버그를 잡았습니다! `larger.width`가 `8`이고 `smaller.width`가 `5`이므로
`can_hold`의 너비 비교는 이제 `false`를 반환합니다. 8은 5보다 작지 않기 때문입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-equality-with-the-assert_eq-and-assert_ne-macros"></a>

### `assert_eq!`와 `assert_ne!`로 동등성 테스트하기

기능을 검증하는 흔한 방법 중 하나는 테스트 대상 코드의 결과와 기대값이 같은지를
테스트하는 것입니다. `assert!` 매크로에 `==` 연산자를 사용한 식을 전달해서 할
수도 있지만, 이는 매우 흔한 테스트이므로 표준 라이브러리는 이 테스트를 더
편리하게 수행하는 매크로 한 쌍, 즉 `assert_eq!`와 `assert_ne!`을 제공합니다. 이
매크로들은 각각 두 인수의 동등(equal) 여부와 부동등(not equal) 여부를 비교합니다.
또한 단언이 실패하면 두 값을 출력해 주므로 테스트가 _왜_ 실패했는지 더 쉽게
알 수 있습니다. 반면 `assert!` 매크로는 `==` 식이 `false` 값을 얻었다는 것만
나타낼 뿐, `false` 값으로 이어진 실제 값들은 출력하지 않습니다.

Listing 11-7에서는 매개변수에 `2`를 더하는 `add_two`라는 함수를 작성하고,
`assert_eq!` 매크로로 이 함수를 테스트합니다.

<Listing number="11-7" file-name="src/lib.rs" caption="`assert_eq!` 매크로로 `add_two` 함수 테스트하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

통과하는지 확인해 봅시다!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

`add_two(2)` 호출 결과를 담는 `result`라는 변수를 만든 다음, `result`와 `4`를
`assert_eq!` 매크로의 인수로 전달했습니다. 이 테스트의 출력 줄은
`test tests::it_adds_two ... ok`이며, `ok` 텍스트는 테스트가 통과했음을 나타냅니다!

코드에 버그를 심어 `assert_eq!`가 실패할 때 어떻게 보이는지 확인해 봅시다.
`add_two` 함수의 구현을 `3`을 더하도록 바꿔 보세요.

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

다시 테스트를 실행합니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

테스트가 버그를 잡았습니다! `tests::it_adds_two` 테스트가 실패했고, 실패한 단언이
`left == right`이며 `left`와 `right`가 어떤 값인지 메시지가 알려 줍니다. 이
메시지는 디버깅을 시작하는 데 도움이 됩니다. `add_two(2)` 호출 결과를 넣은
`left` 인수는 `5`였고, `right` 인수는 `4`였습니다. 테스트가 많을 때 이런 메시지가
특히 유용할 수 있음을 짐작할 수 있습니다.

일부 언어와 테스트 프레임워크에서는 동등성 단언 함수의 매개변수를 `expected`와
`actual`이라고 부르며, 인수를 지정하는 순서가 중요합니다. 그러나 러스트에서는
`left`와 `right`라고 부르며, 기대하는 값과 코드가 생성하는 값을 지정하는 순서는
중요하지 않음에 유의하세요. 이 테스트의 단언을 `assert_eq!(4, result)`로
작성해도 `` assertion `left == right` failed ``라고 표시되는 동일한 실패
메시지가 나옵니다.

`assert_ne!` 매크로는 우리가 전달한 두 값이 다르면 통과하고 같으면 실패합니다.
이 매크로는 값이 _무엇이 될지_ 는 확신할 수 없지만 _무엇이 되어서는 안 되는지_
는 확실히 아는 경우에 가장 유용합니다. 예를 들어 입력을 어떤 식으로든 바꾼다고
보장된 함수를 테스트하는데, 입력이 변경되는 방식이 테스트를 실행하는 요일에
따라 달라진다면, 함수의 출력이 입력과 다르다는 것을 단언하는 것이 가장 좋을
수도 있습니다.

내부적으로 `assert_eq!`와 `assert_ne!` 매크로는 각각 `==`와 `!=` 연산자를
사용합니다. 단언이 실패하면 이 매크로들은 인수를 디버그 포매팅으로 출력하므로,
비교되는 값들은 `PartialEq`와 `Debug` 트레이트를 구현해야 합니다. 모든 원시
타입과 대부분의 표준 라이브러리 타입은 이 트레이트들을 구현합니다. 여러분이
직접 정의한 구조체와 열거형에 대해서는, 해당 타입의 동등성을 단언하기 위해
`PartialEq`를 구현해야 합니다. 또한 단언이 실패할 때 값을 출력하기 위해 `Debug`
도 구현해야 합니다. 5장 Listing 5-12에서 언급했듯이 두 트레이트 모두 파생
가능한 트레이트이므로, 구조체나 열거형 정의에 `#[derive(PartialEq, Debug)]`
애너테이션을 추가하는 것만으로 보통 충분합니다. 이와 그 외 파생 가능한
트레이트에 대해서는 부록 C [“파생 가능한 트레이트”][derivable-traits]<!-- ignore -->
를 참고하세요.

### 사용자 정의 실패 메시지 추가하기

`assert!`, `assert_eq!`, `assert_ne!` 매크로에 실패 메시지와 함께 출력할 사용자
정의 메시지를 선택적 인수로 추가할 수 있습니다. 필수 인수 뒤에 지정된 인수들은
모두 `format!` 매크로(8장의 [“`+` 연산자 또는 `format!` 매크로로 이어
붙이기”][concatenating]<!-- ignore -->에서 다룸)에 그대로 전달되므로, `{}`
자리 표시자가 있는 포맷 문자열과 그 자리에 들어갈 값들을 전달할 수 있습니다.
사용자 정의 메시지는 단언이 무엇을 의미하는지 문서화하는 데 유용합니다. 테스트가
실패했을 때 코드의 어떤 부분이 문제인지 더 잘 파악할 수 있습니다.

예를 들어 이름으로 사람에게 인사하는 함수가 있고, 함수에 전달한 이름이 출력에
나타나는지 테스트하고 싶다고 해 봅시다.

<span class="filename">파일명: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

이 프로그램의 요구 사항은 아직 확정되지 않았고, 인사말 시작 부분의 `Hello`
텍스트가 바뀔 가능성이 높다고 판단됩니다. 요구 사항이 바뀔 때마다 테스트를
업데이트하고 싶지 않으므로, `greeting` 함수가 반환한 값과의 정확한 동등성을
검사하는 대신 출력에 입력 매개변수의 텍스트가 포함되는지만 단언하겠습니다.

기본 테스트 실패 메시지가 어떻게 보이는지 확인하기 위해, `greeting`이 `name`을
제외하도록 바꿔 버그를 심어 봅시다.

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

이 테스트를 실행하면 다음이 출력됩니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

이 결과는 단지 단언이 실패했으며 어느 줄에 있는지만 알려 줍니다. 더 유용한
실패 메시지는 `greeting` 함수의 값을 출력해 줄 것입니다. `greeting` 함수에서
얻은 실제 값을 자리 표시자에 채운 포맷 문자열로 구성된 사용자 정의 실패
메시지를 추가해 봅시다.

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

이제 테스트를 실행하면 더 유익한 오류 메시지를 얻습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

테스트 출력에서 우리가 실제로 얻은 값을 확인할 수 있어서, 기대했던 일 대신 무슨
일이 일어났는지 디버깅하는 데 도움이 됩니다.

### `should_panic`으로 패닉 검사하기

반환 값을 검사하는 것뿐 아니라, 코드가 오류 조건을 기대대로 처리하는지 확인하는
것도 중요합니다. 예를 들어 9장 Listing 9-13에서 만든 `Guess` 타입을 생각해
봅시다. `Guess`를 사용하는 다른 코드는 `Guess` 인스턴스가 1에서 100 사이의
값만 담고 있다는 보장에 의존합니다. 그 범위 밖의 값으로 `Guess` 인스턴스를
만들려는 시도가 패닉을 일으키도록 보장하는 테스트를 작성할 수 있습니다.

이를 위해 테스트 함수에 `should_panic` 속성을 추가합니다. 함수 내부 코드가
패닉하면 테스트는 통과하고, 패닉하지 않으면 실패합니다.

Listing 11-8은 `Guess::new`의 오류 조건이 우리가 기대하는 상황에서 발생하는지
확인하는 테스트를 보여 줍니다.

<Listing number="11-8" file-name="src/lib.rs" caption="어떤 조건이 `panic!`을 일으키는지 테스트하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

`#[should_panic]` 속성은 `#[test]` 속성 뒤, 적용할 테스트 함수 앞에 둡니다. 이
테스트가 통과할 때의 결과를 살펴봅시다.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

잘 나옵니다! 이제 `new` 함수에서 값이 100보다 클 때 패닉을 일으키는 조건을
제거해 코드에 버그를 심어 봅시다.

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Listing 11-8의 테스트를 실행하면 실패합니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

이 경우 아주 도움이 되는 메시지가 나오지는 않지만, 테스트 함수에
`#[should_panic]`이 달린 것을 보면 우리가 얻은 실패가 테스트 함수 안의 코드가
패닉하지 않았다는 의미임을 알 수 있습니다.

`should_panic`을 사용하는 테스트는 정확도가 떨어질 수 있습니다. 우리가 기대한
이유와 다른 이유로 테스트가 패닉하더라도 `should_panic` 테스트는 통과할 수
있기 때문입니다. `should_panic` 테스트를 더 정확하게 만들려면 `should_panic`
속성에 선택적 `expected` 매개변수를 추가할 수 있습니다. 그러면 테스트 하니스는
실패 메시지가 제공된 텍스트를 포함하는지 확인합니다. 예를 들어 Listing 11-9는
값이 너무 작은지 너무 큰지에 따라 `new` 함수가 서로 다른 메시지로 패닉하는
`Guess` 코드를 변경한 것입니다.

<Listing number="11-9" file-name="src/lib.rs" caption="특정 부분 문자열을 포함한 패닉 메시지의 `panic!`을 테스트하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

`should_panic` 속성의 `expected` 매개변수에 넣은 값이 `Guess::new` 함수가
패닉하면서 출력하는 메시지의 부분 문자열이기 때문에 이 테스트는 통과합니다.
기대하는 전체 패닉 메시지를 지정할 수도 있었는데, 이 경우라면
`Guess value must be less than or equal to 100, got 200`이 됩니다. 무엇을
지정할지는 패닉 메시지가 얼마나 유일하거나 동적인지, 그리고 테스트를 얼마나
정확하게 만들고 싶은지에 달려 있습니다. 이 경우 패닉 메시지의 부분 문자열만
있어도 테스트 함수 안의 코드가 `else if value > 100` 경우를 실행한다는 것을
보장하기에 충분합니다.

`expected` 메시지가 있는 `should_panic` 테스트가 실패할 때 어떻게 되는지 보기
위해, 이번에도 `if value < 1` 블록과 `else if value > 100` 블록의 본문을
서로 바꿔 코드에 버그를 심어 봅시다.

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

이번에는 `should_panic` 테스트를 실행하면 실패합니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

실패 메시지는 이 테스트가 우리 기대대로 실제로 패닉했지만, 패닉 메시지에 기대
문자열 `less than or equal to 100`이 포함되지 않았음을 나타냅니다. 이 경우
우리가 얻은 패닉 메시지는 `Guess value must be greater than or equal to 1, got 200`
이었습니다. 이제 버그가 어디 있는지 파악해 나갈 수 있습니다!

### 테스트에서 `Result<T, E>` 사용하기

지금까지 우리의 테스트들은 실패할 때 모두 패닉했습니다. `Result<T, E>`를
사용하는 테스트도 작성할 수 있습니다! 다음은 Listing 11-1의 테스트를 패닉하는
대신 `Result<T, E>`를 사용하고 `Err`를 반환하도록 재작성한 것입니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

`it_works` 함수는 이제 `Result<(), String>` 반환 타입을 가집니다. 함수 본문에서는
`assert_eq!` 매크로를 호출하는 대신, 테스트가 통과하면 `Ok(())`를, 실패하면
안에 `String`을 담은 `Err`를 반환합니다.

테스트가 `Result<T, E>`를 반환하도록 작성하면 테스트 본문에서 물음표 연산자를
사용할 수 있게 됩니다. 테스트 안의 어떤 작업이 `Err` 배리언트를 반환하면 실패
해야 하는 테스트를 편리하게 작성할 수 있는 방법입니다.

`Result<T, E>`를 사용하는 테스트에는 `#[should_panic]` 애너테이션을 사용할 수
없습니다. 어떤 작업이 `Err` 배리언트를 반환함을 단언하려면 `Result<T, E>` 값에
물음표 연산자를 사용하지 _말고_, 대신 `assert!(value.is_err())`를 사용하세요.

이제 테스트를 작성하는 여러 방법을 알게 되었으니, 테스트를 실행할 때 내부적으로
어떤 일이 일어나는지와 `cargo test`에서 사용할 수 있는 다양한 옵션을 살펴봅시다.

[concatenating]: ch08-02-strings.html#concatenating-with--or-format
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
