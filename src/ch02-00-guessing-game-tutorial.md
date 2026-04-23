# 숫자 맞히기 게임 만들기

실습 프로젝트를 함께 해 보며 러스트에 바로 뛰어들어 봅시다! 이 장에서는 몇 가지
자주 쓰이는 러스트 개념을 실제 프로그램에서 어떻게 사용하는지 보여 주며 소개합니다.
`let`, `match`, 메서드, 연관 함수, 외부 크레이트 등을 배우게 됩니다! 이어지는
장들에서는 이 아이디어들을 더 자세히 탐구할 것입니다. 이 장에서는 기본기만
연습해 봅니다.

고전적인 초심자용 프로그래밍 문제인 숫자 맞히기 게임을 구현할 것입니다. 동작 방식은
다음과 같습니다. 프로그램은 1부터 100 사이의 무작위 정수를 생성합니다. 그런 다음
플레이어에게 예측값을 입력하라는 프롬프트를 표시합니다. 예측값이 입력되면 프로그램은
그 값이 너무 작은지 너무 큰지 알려 줍니다. 예측값이 정답이라면 게임은 축하 메시지를
출력하고 종료합니다.

## 새 프로젝트 설정하기

새 프로젝트를 설정하려면 1장에서 만든 *projects* 디렉터리로 이동한 뒤, 다음과 같이
Cargo로 새 프로젝트를 만드세요.

```console
$ cargo new guessing_game
$ cd guessing_game
```

첫 번째 명령인 `cargo new`는 첫 번째 인자로 프로젝트 이름(`guessing_game`)을 받습니다.
두 번째 명령은 새 프로젝트 디렉터리로 이동합니다.

생성된 *Cargo.toml* 파일을 살펴봅시다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">파일명: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

1장에서 봤듯이, `cargo new`는 여러분을 위해 “Hello, world!” 프로그램을 만들어
줍니다. *src/main.rs* 파일도 한번 확인해 보세요.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

이제 이 “Hello, world!” 프로그램을 컴파일하고 곧바로 같은 명령으로 실행하기 위해
`cargo run`을 사용해 봅시다.

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

`run` 명령은 지금 이 게임에서처럼 프로젝트를 빠르게 반복하며 작업할 때 편리합니다.
각 반복을 다음 단계로 넘어가기 전에 빠르게 테스트할 수 있기 때문입니다.

*src/main.rs* 파일을 다시 엽니다. 앞으로 이 파일에 모든 코드를 작성하게 됩니다.

## 예측값 처리하기

숫자 맞히기 게임 프로그램의 첫 부분은 사용자 입력을 요청해서, 그 입력을 처리하고,
입력이 예상된 형태인지 검사합니다. 우선 플레이어가 예측값을 입력할 수 있도록 하는
것부터 시작합니다. Listing 2-1의 코드를 *src/main.rs*에 입력하세요.

<Listing number="2-1" file-name="src/main.rs" caption="사용자로부터 예측값을 받아 출력하는 코드">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

이 코드에는 정보가 많이 들어 있으니 한 줄씩 살펴봅시다. 사용자 입력을 받고 그
결과를 출력으로 내보내려면, `io` 입출력 라이브러리를 스코프로 가져와야 합니다. `io`
라이브러리는 `std`라고 부르는 표준 라이브러리에서 옵니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

러스트는 기본적으로 모든 프로그램의 스코프에 자동으로 들어오는 표준 라이브러리 항목
들의 모음을 가지고 있습니다. 이 모음을 *프렐류드(prelude)*라고 부르며, 그 안의 모든
항목은 [표준 라이브러리 문서][prelude]에서 확인할 수 있습니다.

사용하려는 타입이 프렐류드에 없다면, `use` 문으로 해당 타입을 스코프로 명시적으로
가져와야 합니다. `std::io` 라이브러리를 사용하면 사용자 입력을 받는 능력을 포함해
여러 유용한 기능을 사용할 수 있습니다.

1장에서 봤듯이 `main` 함수는 프로그램의 진입점입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

`fn` 구문은 새 함수를 선언하고, 괄호 `()`는 매개변수가 없음을 나타내며, 여는
중괄호 `{`는 함수 본문의 시작을 알립니다.

1장에서도 배웠듯이 `println!`은 문자열을 화면에 출력하는 매크로입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

이 코드는 게임이 무엇인지 알려 주고 사용자로부터 입력을 요청하는 프롬프트를
출력합니다.

### 변수에 값 저장하기

다음으로, 사용자 입력을 담을 *변수(variable)*를 만들겠습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

이제 프로그램이 흥미로워지기 시작합니다! 이 짧은 한 줄에 많은 일이 벌어지고 있습니다.
우리는 변수를 만들기 위해 `let` 문을 사용합니다. 또 다른 예시는 다음과 같습니다.

```rust,ignore
let apples = 5;
```

이 줄은 `apples`라는 이름의 새 변수를 만들고 값 `5`를 바인딩합니다. 러스트에서
변수는 기본적으로 불변(immutable)입니다. 즉, 변수에 값을 한 번 주고 나면 그 값을
바꿀 수 없다는 의미입니다. 이 개념은 3장의
[“변수와 가변성”][variables-and-mutability]<!-- ignore --> 절에서 자세히 다룹니다.
변수를 가변(mutable)으로 만들려면 변수 이름 앞에 `mut`를 붙입니다.

```rust,ignore
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

> 참고: `//` 문법은 주석을 시작하며, 줄 끝까지 이어집니다. 러스트는 주석 안의
> 내용을 모두 무시합니다. 주석에 대해서는 [3장][comments]<!-- ignore -->에서 더
> 자세히 다룹니다.

숫자 맞히기 게임 프로그램으로 돌아가 보면, 이제 `let mut guess`가 `guess`라는
이름의 가변 변수를 도입한다는 사실을 알 수 있습니다. 등호(`=`)는 러스트에게
지금 이 변수에 무언가를 바인딩하고 싶다는 것을 알려 줍니다. 등호 오른쪽에는
`guess`에 바인딩되는 값이 있는데, 이는 `String`의 새 인스턴스를 반환하는 함수인
`String::new`를 호출한 결과입니다. [`String`][string]<!-- ignore -->은 표준 라이
브러리가 제공하는 문자열 타입으로, 크기가 자라날 수 있는 UTF-8로 인코딩된 텍스트
조각입니다.

`::new` 줄의 `::` 문법은 `new`가 `String` 타입의 연관 함수(associated function)
임을 나타냅니다. *연관 함수*는 타입 위에 구현된 함수로, 여기서는 `String`에
구현되어 있습니다. 이 `new` 함수는 새로운 빈 문자열을 만듭니다. 많은 타입에서
`new` 함수를 보게 될 텐데, 이는 어떤 값을 새로 만들어 주는 함수에 흔히 붙는
이름이기 때문입니다.

정리하면, `let mut guess = String::new();` 줄은 현재 비어 있는 `String`의 새
인스턴스에 바인딩된 가변 변수를 만듭니다. 휴!

### 사용자 입력 받기

프로그램의 첫 줄에서 `use std::io;`로 표준 라이브러리의 입출력 기능을 포함시켰던
것을 떠올려 보세요. 이제 `io` 모듈의 `stdin` 함수를 호출할 텐데, 이 함수는 사용자
입력을 다룰 수 있게 해 줍니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

만약 프로그램의 시작 부분에서 `use std::io;`로 `io` 모듈을 가져오지 않았더라도,
`std::io::stdin`처럼 전체 경로를 적는 방식으로 여전히 이 함수를 사용할 수 있습니다.
`stdin` 함수는 [`std::io::Stdin`][iostdin]<!-- ignore -->의 인스턴스를 반환하는데,
이는 터미널의 표준 입력에 대한 핸들을 나타내는 타입입니다.

다음으로, `.read_line(&mut guess)` 줄은 표준 입력 핸들에
[`read_line`][read_line]<!-- ignore --> 메서드를 호출해 사용자에게서 입력을
받습니다. 또한 사용자 입력을 어느 문자열에 저장할지 `read_line`에 알려 주기 위해
인자로 `&mut guess`를 전달합니다. `read_line`이 수행하는 전체 작업은 사용자가 표준
입력에 타이핑한 내용을 가져다가 기존 문자열에 덧붙이는 것(내용을 덮어쓰지 않고)이
므로, 해당 문자열을 인자로 전달합니다. 메서드가 문자열의 내용을 바꿀 수 있어야
하기 때문에, 문자열 인자는 가변이어야 합니다.

`&`는 이 인자가 *참조(reference)*임을 나타냅니다. 참조는 같은 데이터를 메모리에
여러 번 복사하지 않고도 코드의 여러 부분에서 그 데이터에 접근할 수 있게 해 주는
방법입니다. 참조는 복잡한 기능이지만, 러스트의 큰 장점 중 하나는 이 참조를 안전
하고 쉽게 사용할 수 있다는 점입니다. 이 프로그램을 끝내기 위해서 그 세부 사항을
많이 알 필요는 없습니다. 지금은 참조도 변수와 마찬가지로 기본적으로 불변이라는
것만 알아 두면 됩니다. 따라서 참조를 가변으로 만들려면 `&guess`가 아니라 `&mut
guess`라고 써야 합니다. (4장에서 참조를 좀 더 자세히 설명합니다.)

<!-- Old headings. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### `Result`로 잠재적 실패 다루기

우리는 여전히 같은 한 줄의 코드를 이야기하고 있습니다. 이제는 세 번째 텍스트 줄에
대해 논의하는 중이지만, 그래도 이는 여전히 하나의 논리적인 코드 줄의 일부임에
유의하세요. 다음 부분은 이 메서드입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

이 코드는 다음과 같이 한 줄로 쓸 수도 있었습니다.

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

그러나 한 줄이 길면 읽기 어렵기 때문에, 나누는 편이 더 좋습니다. `.method_name()`
구문으로 메서드를 호출할 때에는, 긴 줄을 나누는 데 도움이 되도록 줄바꿈과 공백을
넣는 것이 좋을 때가 많습니다. 이제 이 줄이 하는 일을 살펴봅시다.

앞서 언급했듯이, `read_line`은 사용자가 입력한 내용을 우리가 건네준 문자열에
집어넣지만, 동시에 `Result` 값도 반환합니다. [`Result`][result]<!-- ignore -->는
흔히 *enum*이라고 부르는 [*열거형(enumeration)*][enums]<!-- ignore -->으로,
여러 가능한 상태 중 하나에 있을 수 있는 타입입니다. 각 가능한 상태를 *배리언트
(variant)*라고 부릅니다.

열거형은 [6장][enums]<!-- ignore -->에서 더 자세히 다룹니다. `Result` 타입의
용도는 에러 처리 정보를 인코딩하는 것입니다.

`Result`의 배리언트는 `Ok`와 `Err`입니다. `Ok` 배리언트는 연산이 성공했음을 의미
하며, 성공적으로 생성된 값을 담고 있습니다. `Err` 배리언트는 연산이 실패했음을
의미하며, 어떻게 또는 왜 연산이 실패했는지에 대한 정보를 담고 있습니다.

다른 모든 타입의 값들처럼, `Result` 타입의 값도 그 위에 정의된 메서드를 가집니다.
`Result`의 인스턴스에는 호출할 수 있는 [`expect` 메서드][expect]<!-- ignore -->가
있습니다. 이 `Result` 인스턴스가 `Err` 값이라면, `expect`는 프로그램을 충돌시키고
여러분이 `expect`에 인자로 전달한 메시지를 표시합니다. `read_line` 메서드가
`Err`를 반환한다면, 이는 보통 운영체제에서 비롯된 에러의 결과일 것입니다. 이
`Result` 인스턴스가 `Ok` 값이라면, `expect`는 `Ok`가 담고 있는 반환값을 꺼내서
그 값만을 여러분에게 돌려주고, 여러분은 이를 사용할 수 있습니다. 이 경우 그
값은 사용자가 입력한 바이트 수입니다.

`expect`를 호출하지 않으면 프로그램은 컴파일되지만 경고를 받게 됩니다.

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

러스트는 `read_line`이 반환한 `Result` 값을 여러분이 사용하지 않았음을 경고하며,
프로그램이 발생할 수 있는 에러를 처리하지 않았음을 알립니다.

이 경고를 없애는 올바른 방법은 실제로 에러 처리 코드를 작성하는 것이지만, 우리의
경우 문제가 생기면 이 프로그램을 그냥 충돌시키고 싶은 것이므로 `expect`를 사용해도
됩니다. 에러로부터 복구하는 방법은 [9장][recover]<!-- ignore -->에서 배웁니다.

### `println!` 자리표시자로 값 출력하기

닫는 중괄호를 빼고 나면, 지금까지의 코드에서 논의할 줄은 한 줄뿐입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

이 줄은 사용자 입력이 담긴 문자열을 출력합니다. `{}` 한 쌍의 중괄호는 자리표시자
(placeholder)입니다. `{}`를 값을 제자리에 붙잡아 두는 작은 게 집게라고 생각해
보세요. 변수의 값을 출력할 때에는 변수 이름을 중괄호 안에 넣을 수 있습니다.
표현식을 평가한 결과를 출력할 때에는 포맷 문자열 안에 빈 중괄호를 두고, 그 포맷
문자열 뒤에 쉼표로 구분된 표현식 목록을 이어서 적습니다. 표현식은 포맷 문자열의
빈 중괄호 자리표시자와 같은 순서로 출력됩니다. 변수와 표현식의 결과를 `println!`
한 번의 호출에서 출력하려면 다음과 같이 씁니다.

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

이 코드는 `x = 5 and y + 2 = 12`를 출력합니다.

### 첫 부분 테스트하기

숫자 맞히기 게임의 첫 부분을 테스트해 봅시다. `cargo run`으로 실행하세요.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

이 시점에서 게임의 첫 부분은 끝났습니다. 우리는 키보드로부터 입력을 받고 그것을
출력하고 있습니다.

## 비밀 숫자 생성하기

다음으로, 사용자가 맞히려고 할 비밀 숫자를 생성해야 합니다. 게임을 여러 번 재미있게
플레이할 수 있도록, 비밀 숫자는 매번 달라야 합니다. 게임이 너무 어렵지 않도록 1부터
100 사이의 난수를 사용하겠습니다. 러스트는 아직 표준 라이브러리에 난수 기능을
포함하지 않습니다. 그렇지만 러스트 팀은 그 기능을 제공하는
[`rand` 크레이트][randcrate]를 제공하고 있습니다.

<!-- Old headings. Do not remove or links may break. -->
<a id="using-a-crate-to-get-more-functionality"></a>

### 크레이트로 기능 확장하기

크레이트(crate)는 러스트 소스 코드 파일들의 모음이라는 점을 기억해 두세요. 우리가
지금까지 만들어 온 프로젝트는 실행 파일에 해당하는 *바이너리 크레이트(binary
crate)*입니다. `rand` 크레이트는 다른 프로그램에서 사용되도록 의도된 코드를
담고 있으며 그 자체로는 실행할 수 없는, *라이브러리 크레이트(library crate)*
입니다.

Cargo가 외부 크레이트를 조율하는 부분은 Cargo가 진짜로 빛을 발하는 지점입니다.
`rand`를 사용하는 코드를 작성하려면, 먼저 *Cargo.toml* 파일을 수정해 `rand`
크레이트를 의존성으로 포함시켜야 합니다. 지금 그 파일을 열고, Cargo가 만들어 준
`[dependencies]` 섹션 헤더 아래쪽에 다음 줄을 추가하세요. 이 자습서의 코드 예제가
동작하지 않을 수도 있으니, `rand` 버전을 우리가 여기 적어 둔 것과 정확히 동일하게
지정하세요.

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">파일명: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

*Cargo.toml* 파일에서 헤더 뒤에 오는 모든 내용은 그 섹션의 일부이며, 다른 섹션이
시작될 때까지 이어집니다. `[dependencies]`에서는 프로젝트가 어떤 외부 크레이트에
의존하는지, 그리고 그 크레이트의 어떤 버전을 필요로 하는지 Cargo에게 알려 줍니다.
여기서는 시맨틱 버저닝(semantic version) 지정자 `0.8.5`로 `rand` 크레이트를
지정합니다. Cargo는 [시맨틱 버저닝][semver]<!-- ignore -->(*SemVer*라고도 부릅
니다)을 이해하는데, 이는 버전 번호를 작성하기 위한 표준입니다. 지정자 `0.8.5`는
사실 `^0.8.5`의 줄임말로, 0.8.5 이상이지만 0.9.0 미만인 어떤 버전이든 의미합니다.

Cargo는 이 버전들이 버전 0.8.5와 호환되는 공개 API를 가진 것으로 간주합니다. 이
지정 덕분에 이 장의 코드와 여전히 컴파일되는 최신 패치 릴리스를 받게 됩니다.
0.9.0 이상의 어떤 버전도 뒤이은 예제들이 사용하는 API와 동일한 API를 가진다고
보장되지 않습니다.

이제 코드는 전혀 바꾸지 않고, Listing 2-2처럼 프로젝트를 빌드해 봅시다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="`rand` 크레이트를 의존성으로 추가한 뒤 `cargo build`를 실행한 출력">

```console
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```

</Listing>

버전 번호는 다르게 보일 수 있고(하지만 SemVer 덕분에 모두 코드와 호환됩니다!),
운영체제에 따라 다른 줄이 나타날 수 있으며, 줄의 순서도 다를 수 있습니다.

외부 의존성을 포함시키면, Cargo는 *레지스트리(registry)*에서 그 의존성이 필요로
하는 모든 것의 최신 버전을 가져옵니다. 레지스트리는 [Crates.io][cratesio]의
데이터 사본입니다. Crates.io는 러스트 생태계의 사람들이 다른 이들이 사용할 수
있도록 오픈 소스 러스트 프로젝트를 게시하는 곳입니다.

레지스트리를 갱신한 뒤, Cargo는 `[dependencies]` 섹션을 확인하고 아직 내려받지
않은 크레이트들을 내려받습니다. 이 경우 우리는 `rand`만 의존성으로 나열했지만,
Cargo는 `rand`가 동작하기 위해 의존하는 다른 크레이트들도 가져왔습니다. 크레이트를
내려받은 뒤 러스트는 그 크레이트들을 컴파일하고, 이어서 의존성들이 사용 가능한
상태로 프로젝트를 컴파일합니다.

아무것도 바꾸지 않고 바로 `cargo build`를 다시 실행하면, `Finished` 줄 이외의 다른
출력은 보이지 않을 것입니다. Cargo는 이미 의존성들을 내려받아 컴파일했다는 사실과,
여러분이 *Cargo.toml*에서 그 내용을 바꾸지 않았다는 사실을 알고 있습니다. 또한
Cargo는 여러분의 코드도 바뀌지 않았음을 알고 있으므로 코드도 다시 컴파일하지
않습니다. 할 일이 없으니 그대로 종료됩니다.

*src/main.rs* 파일을 열어 사소한 변경을 가하고 저장한 뒤 다시 빌드하면, 다음처럼
두 줄의 출력만 보일 것입니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

이 출력은 Cargo가 *src/main.rs*에 가한 작은 변경에 맞춰서만 빌드를 갱신했음을
보여 줍니다. 의존성은 바뀌지 않았으므로, Cargo는 이미 내려받아 컴파일해 둔
결과물을 재사용할 수 있음을 알고 있습니다.

<!-- Old headings. Do not remove or links may break. -->
<a id="ensuring-reproducible-builds-with-the-cargo-lock-file"></a>

#### 재현 가능한 빌드 보장하기

Cargo에는 여러분이나 다른 누구든지 코드를 빌드할 때마다 매번 동일한 산출물을
다시 만들어낼 수 있도록 보장해 주는 메커니즘이 있습니다. Cargo는 여러분이 다른
표시를 남기기 전까지는 여러분이 지정한 의존성 버전만을 사용합니다. 예를 들어,
다음 주에 `rand` 크레이트의 0.8.6 버전이 출시되고, 그 버전에 중요한 버그 수정이
포함되어 있지만 동시에 여러분의 코드를 망가뜨리는 리그레션이 포함되어 있다고
해 봅시다. 이를 다루기 위해, 러스트는 `cargo build`를 처음 실행할 때 *Cargo.lock*
파일을 만들어 둡니다. 그래서 우리는 이제 *guessing_game* 디렉터리 안에 이
파일을 가지게 됩니다.

프로젝트를 처음 빌드할 때 Cargo는 기준에 부합하는 의존성 버전들을 모두 계산해서
*Cargo.lock* 파일에 기록합니다. 이후에 프로젝트를 빌드할 때 Cargo는 *Cargo.lock*
파일이 있음을 확인하고, 버전을 다시 계산하는 대신 거기 적힌 버전을 사용합니다.
이렇게 해서 재현 가능한 빌드가 자동으로 이뤄지게 됩니다. 다시 말해, 여러분의
프로젝트는 *Cargo.lock* 파일 덕분에 여러분이 명시적으로 업그레이드하기 전까지는
0.8.5에 머물러 있게 됩니다. *Cargo.lock* 파일은 재현 가능한 빌드에 있어 중요하기
때문에, 많은 경우 프로젝트의 나머지 코드와 함께 소스 관리에 체크인됩니다.

#### 크레이트를 새 버전으로 업데이트하기

크레이트를 업데이트하고 *싶어질* 때, Cargo는 `update` 명령을 제공합니다. 이
명령은 *Cargo.lock* 파일을 무시하고, *Cargo.toml*의 지정에 맞는 모든 최신 버전을
계산합니다. 그런 다음 Cargo는 그 버전을 *Cargo.lock* 파일에 기록합니다. 그렇지
않은 경우, 기본적으로 Cargo는 0.8.5보다 크고 0.9.0보다 작은 버전만 찾습니다.
`rand` 크레이트가 새 버전 0.8.6과 0.999.0을 출시했다면, `cargo update`를 실행
했을 때 다음과 같이 보일 것입니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.999.0)
```

Cargo는 0.999.0 릴리스를 무시합니다. 이 시점에서 *Cargo.lock* 파일에도 변경이
발생한 것을 볼 수 있으며, 이제 사용하는 `rand` 크레이트의 버전이 0.8.6임이
기록됩니다. `rand` 버전 0.999.0이나 0.999.*x* 시리즈의 어떤 버전을 쓰려면,
*Cargo.toml* 파일을 다음과 같이 수정해야 합니다(뒤의 예제들은 여러분이 `rand`
0.8을 사용한다고 가정하므로 실제로 이 변경을 하지는 마세요).

```toml
[dependencies]
rand = "0.999.0"
```

다음번에 `cargo build`를 실행하면, Cargo는 사용 가능한 크레이트 레지스트리를
갱신하고, 여러분이 지정한 새 버전에 따라 `rand` 요구 사항을 다시 평가합니다.

[Cargo][doccargo]<!-- ignore -->와 [그 생태계][doccratesio]<!-- ignore -->에
대해서는 할 이야기가 훨씬 더 많고, 그 부분은 14장에서 다룹니다. 지금은 이 정도만
알아 두면 됩니다. Cargo는 라이브러리 재사용을 매우 쉽게 해 주기 때문에, 러스타
시안들은 여러 패키지를 조립해서 구성된 작은 프로젝트를 작성할 수 있습니다.

### 난수 생성하기

`rand`를 사용해 맞힐 숫자를 생성해 봅시다. 다음 단계는 Listing 2-3처럼
*src/main.rs*를 업데이트하는 것입니다.

<Listing number="2-3" file-name="src/main.rs" caption="난수 생성 코드 추가">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

먼저, `use rand::Rng;` 줄을 추가합니다. `Rng` 트레이트는 난수 생성기가 구현하는
메서드들을 정의하며, 그 메서드들을 사용하려면 이 트레이트가 스코프에 있어야
합니다. 트레이트는 10장에서 자세히 다룹니다.

그다음, 중간에 두 줄을 추가합니다. 첫 번째 줄에서는 `rand::thread_rng` 함수를
호출해 우리가 사용할 특정 난수 생성기, 즉 현재 실행 스레드에 한정된 로컬 생성기
이자 운영체제가 시드를 제공해 주는 생성기를 얻습니다. 그다음, 그 난수 생성기에
`gen_range` 메서드를 호출합니다. 이 메서드는 `use rand::Rng;` 문으로 스코프로
가져온 `Rng` 트레이트에 정의되어 있습니다. `gen_range` 메서드는 범위 표현식을
인자로 받아 그 범위 안의 난수를 생성합니다. 여기서 우리가 사용하는 범위 표현식
형태는 `start..=end` 꼴로, 시작과 끝을 모두 포함합니다. 따라서 1부터 100 사이의
숫자를 요청하려면 `1..=100`으로 지정해야 합니다.

> 참고: 어떤 크레이트에서 어떤 트레이트를 써야 하고 어떤 메서드와 함수를 호출해야
> 하는지를 그냥은 알 수 없기 때문에, 각 크레이트는 사용 방법에 대한 지시가 담긴
> 문서를 가지고 있습니다. Cargo의 또 다른 멋진 기능 하나는 `cargo doc --open`
> 명령을 실행하면 여러분의 모든 의존성이 제공하는 문서를 로컬로 빌드해서 브라우저에서
> 열어 준다는 것입니다. 예를 들어 `rand` 크레이트의 다른 기능에 관심이 있다면,
> `cargo doc --open`을 실행하고 왼쪽 사이드바에서 `rand`를 클릭해 보세요.

두 번째 새 줄은 비밀 숫자를 출력합니다. 이는 프로그램을 개발하는 동안 테스트에
유용하지만, 최종 버전에서는 삭제할 것입니다. 프로그램이 시작하자마자 정답을
출력한다면 게임이라고 할 수 없으니까요!

프로그램을 몇 번 실행해 보세요.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

서로 다른 난수가 나와야 하고, 모두 1과 100 사이의 숫자여야 합니다. 훌륭합니다!

## 예측값과 비밀 숫자 비교하기

이제 사용자 입력과 난수를 모두 가졌으니 이 둘을 비교할 수 있습니다. 그 단계는
Listing 2-4에 나와 있습니다. 뒤에서 설명하겠지만 이 코드는 아직 컴파일되지 않는
다는 점에 유의하세요.

<Listing number="2-4" file-name="src/main.rs" caption="두 숫자를 비교할 때 가능한 반환값 처리">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

먼저, `use` 문을 하나 더 추가해 표준 라이브러리의 `std::cmp::Ordering` 타입을
스코프로 가져옵니다. `Ordering` 타입은 또 다른 열거형으로, 배리언트 `Less`,
`Greater`, `Equal`을 가집니다. 이는 두 값을 비교할 때 가능한 세 가지 결과입니다.

그런 다음, `Ordering` 타입을 사용하는 다섯 줄의 새 코드를 맨 아래에 추가합니다.
`cmp` 메서드는 두 값을 비교하며, 비교할 수 있는 어떤 것에든 호출할 수 있습니다.
메서드는 비교하고자 하는 대상의 참조를 받습니다. 여기서는 `guess`를 `secret_number`
와 비교하고 있습니다. 그리고는 `use` 문으로 스코프로 가져온 `Ordering` 열거형의
배리언트를 반환합니다. 우리는 [`match`][match]<!-- ignore --> 표현식을 사용해,
`guess`와 `secret_number` 값을 사용한 `cmp` 호출에서 `Ordering`의 어느 배리언트가
반환되었는지에 따라 다음에 할 일을 결정합니다.

`match` 표현식은 *갈래(arm)*들로 구성됩니다. 갈래는 맞춰 볼 *패턴(pattern)*과,
`match`에 주어진 값이 해당 갈래의 패턴에 부합할 때 실행되어야 하는 코드로 이뤄
집니다. 러스트는 `match`에 주어진 값을 가지고 각 갈래의 패턴을 순서대로 훑어
봅니다. 패턴과 `match` 구조는 러스트의 강력한 기능입니다. 코드가 마주칠 수 있는
다양한 상황을 표현할 수 있게 해 주고, 그 모든 상황을 여러분이 반드시 처리하도록
만들어 줍니다. 이 기능들은 각각 6장과 19장에서 자세히 다룹니다.

여기서 사용하는 `match` 표현식으로 예를 하나 따라가 봅시다. 사용자가 50으로
예측했고, 이번에 무작위로 생성된 비밀 숫자가 38이라고 해 봅시다.

코드가 50과 38을 비교하면, 50이 38보다 크므로 `cmp` 메서드는 `Ordering::Greater`를
반환합니다. `match` 표현식은 `Ordering::Greater` 값을 받아 각 갈래의 패턴을
검사하기 시작합니다. 첫 번째 갈래의 패턴인 `Ordering::Less`를 보고, `Ordering::Greater`
값이 `Ordering::Less`와 일치하지 않음을 확인한 뒤, 해당 갈래의 코드를 무시하고
다음 갈래로 이동합니다. 다음 갈래의 패턴은 `Ordering::Greater`인데, 이는
`Ordering::Greater`와 *일치*합니다! 그 갈래에 연관된 코드가 실행되어 화면에
`Too big!`을 출력합니다. `match` 표현식은 첫 번째 성공적인 일치 이후 종료되므로,
이 시나리오에서는 마지막 갈래는 보지 않습니다.

하지만 Listing 2-4의 코드는 아직 컴파일되지 않습니다. 시도해 봅시다.

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

에러의 핵심은 *타입이 일치하지 않는다(mismatched types)*는 것입니다. 러스트는
강하고 정적인 타입 시스템을 가지고 있습니다. 하지만 타입 추론도 가지고 있습니다.
`let mut guess = String::new()`라고 작성했을 때, 러스트는 `guess`가 `String`
이어야 한다고 추론할 수 있었고 우리가 타입을 쓰도록 강요하지 않았습니다. 반면에
`secret_number`는 숫자 타입입니다. 러스트의 몇몇 숫자 타입은 1부터 100 사이의
값을 가질 수 있습니다. 32비트 숫자 `i32`, 부호 없는 32비트 숫자 `u32`, 64비트
숫자 `i64` 등이 그렇습니다. 다른 지정이 없으면 러스트는 기본으로 `i32`를 사용
합니다. 따라서 러스트가 다른 숫자 타입을 추론하게 만드는 타입 정보를 다른 곳에
덧붙이지 않는 한, `secret_number`의 타입은 `i32`입니다. 에러의 이유는 러스트가
문자열과 숫자 타입을 비교할 수 없기 때문입니다.

결국 우리는 프로그램이 입력으로 읽어 들이는 `String`을 숫자 타입으로 변환해,
비밀 숫자와 수치적으로 비교할 수 있도록 만들어야 합니다. `main` 함수 본문에
다음 줄을 추가해 이를 수행합니다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

추가하는 줄은 다음과 같습니다.

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

`guess`라는 변수를 만듭니다. 그런데 프로그램에는 이미 `guess`라는 변수가 있지
않나요? 맞습니다. 하지만 러스트는 고맙게도 이전의 `guess` 값을 새 값으로
*섀도잉(shadowing)*할 수 있게 해 줍니다. *섀도잉*은 예를 들어 `guess_str`과
`guess` 같은 두 개의 고유한 변수를 만들도록 강요받는 대신, `guess`라는 변수
이름을 재사용할 수 있게 해 줍니다. [3장][shadowing]<!-- ignore -->에서 더
자세히 다루겠지만, 지금은 이 기능이 값을 한 타입에서 다른 타입으로 변환할 때 자주
사용된다는 정도만 알아 두면 됩니다.

이 새 변수를 `guess.trim().parse()`라는 표현식에 바인딩합니다. 이 표현식에서의
`guess`는 원래의 `guess` 변수, 즉 문자열로 된 입력을 담고 있던 변수를 가리킵니다.
`String` 인스턴스의 `trim` 메서드는 시작과 끝의 공백을 모두 제거해 줍니다. `u32`는
수치 데이터만 담을 수 있기 때문에 문자열을 `u32`로 변환하기 전에 반드시 이 작업을
해야 합니다. 사용자가 `read_line`을 충족시키고 예측값을 입력하려면 <kbd>enter</kbd>
키를 눌러야 하고, 이 과정에서 개행 문자가 문자열에 추가됩니다. 예를 들어 사용자가
<kbd>5</kbd>를 타이핑하고 <kbd>enter</kbd>를 누르면, `guess`는 `5\n` 과 같은
형태가 됩니다. `\n`은 “개행(newline)”을 나타냅니다. (윈도우에서는 <kbd>enter</kbd>를
누르면 캐리지 리턴과 개행이 함께 들어가 `\r\n`이 됩니다.) `trim` 메서드는 `\n`
이나 `\r\n`을 제거해 결과적으로 `5`만 남깁니다.

[문자열의 `parse` 메서드][parse]<!-- ignore -->는 문자열을 다른 타입으로 변환합
니다. 여기서는 문자열을 숫자로 변환하는 데 사용합니다. 우리가 원하는 정확한 숫자
타입을 러스트에게 알려 주기 위해 `let guess: u32`를 사용합니다. `guess` 뒤의 콜론
(`:`)은 변수의 타입을 직접 명시하겠다고 러스트에게 알리는 것입니다. 러스트에는
몇 가지 내장 숫자 타입이 있습니다. 여기서 보이는 `u32`는 부호 없는 32비트 정수
입니다. 작은 양수에 대한 기본 선택으로 좋습니다. 다른 숫자 타입은
[3장][integers]<!-- ignore -->에서 배웁니다.

또한 이 예제 프로그램의 `u32` 주석과 `secret_number`와의 비교는, 러스트가
`secret_number`도 `u32`여야 한다고 추론하도록 만듭니다. 그래서 이제 비교는 같은
타입의 두 값 사이에서 이뤄지게 됩니다!

`parse` 메서드는 논리적으로 숫자로 변환할 수 있는 문자에만 동작하므로 쉽게 에러를
낼 수 있습니다. 예를 들어, 문자열이 `A👍%`를 담고 있다면, 그것을 숫자로 변환할
방법이 없습니다. 실패할 수 있기 때문에, `parse` 메서드는 `Result` 타입을 반환
합니다(앞서 [“`Result`로 잠재적 실패
다루기”](#handling-potential-failure-with-result)<!-- ignore -->에서 설명한
`read_line` 메서드가 하는 것과 비슷합니다). 우리는 이 `Result`를 다시 `expect`
메서드를 사용해 같은 방식으로 처리합니다. `parse`가 문자열에서 숫자를 만들어 내지
못해서 `Err` `Result` 배리언트를 반환한다면, `expect` 호출은 게임을 충돌시키고
우리가 넘겨준 메시지를 출력할 것입니다. `parse`가 문자열을 숫자로 성공적으로 변환
하면, `Result`의 `Ok` 배리언트를 반환할 것이고, `expect`는 그 `Ok` 값이 담고 있던,
우리가 원했던 숫자를 반환해 줍니다.

이제 프로그램을 실행해 봅시다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

좋습니다! 예측값 앞에 공백이 있었지만 프로그램은 여전히 사용자가 76을 예측했다고
파악했습니다. 프로그램을 몇 번 실행해 보며 서로 다른 종류의 입력에 대해 다른
동작이 나오는지 확인해 보세요. 숫자를 정확히 맞혀 보고, 너무 큰 숫자를 예측해
보고, 너무 작은 숫자도 예측해 보세요.

게임의 대부분이 동작하게 됐지만, 사용자는 단 한 번만 예측할 수 있습니다. 루프를
추가해 이를 바꿔 봅시다!

## 루프로 여러 번 예측하기

`loop` 키워드는 무한 루프를 만듭니다. 사용자에게 숫자를 맞힐 기회를 더 주기
위해 루프를 추가합니다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

보시다시피, 예측값 입력 프롬프트부터 그 이후의 모든 부분을 루프 안으로 옮겼습
니다. 루프 안에 들어간 줄들은 각각 네 칸씩 더 들여쓰기해야 한다는 점을 잊지
마시고, 프로그램을 다시 실행해 보세요. 이제 프로그램은 영원히 또 다른 예측값을
물어볼 것이고, 이는 실제로 새로운 문제를 만들어 냅니다. 사용자가 나갈 수 없어
보인다는 것이죠!

사용자는 언제든 <kbd>ctrl</kbd>-<kbd>C</kbd> 키보드 단축키로 프로그램을 중단시킬
수 있습니다. 하지만 이 지칠 줄 모르는 괴물에게서 벗어날 다른 방법도 있습니다.
앞서 [“예측값과 비밀 숫자
비교하기”](#comparing-the-guess-to-the-secret-number)<!-- ignore -->에서 `parse`
에 대해 다룬 내용에 언급되어 있듯이, 사용자가 숫자가 아닌 답을 입력하면 프로그램이
충돌합니다. 이를 이용해 사용자가 나갈 수 있게 해 줄 수 있습니다. 다음과 같이
말이죠.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

`quit`을 입력하면 게임이 종료되지만, 알아챘겠지만 숫자가 아닌 다른 어떤 입력을
넣어도 마찬가지로 종료됩니다. 좋게 봐도 이상적이지 않습니다. 우리는 정답을
맞혔을 때에도 게임이 멈추기를 원합니다.

### 정답 뒤에 종료하기

사용자가 승리했을 때 게임이 종료되도록, `break` 문을 추가해 프로그램을 수정해
봅시다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

`You win!` 다음에 `break` 줄을 추가하면, 사용자가 비밀 숫자를 정확히 맞혔을 때
프로그램이 루프를 빠져나오게 됩니다. 루프를 빠져나오는 것은 곧 프로그램을 종료
하는 것과 같습니다. 루프가 `main`의 마지막 부분이기 때문입니다.

### 잘못된 입력 처리하기

게임의 동작을 더 다듬기 위해, 사용자가 숫자가 아닌 값을 입력했을 때 프로그램을
충돌시키지 않고, 그 입력을 무시해서 사용자가 계속 예측할 수 있게 만들어 봅시다.
`guess`를 `String`에서 `u32`로 변환하는 줄을 Listing 2-5처럼 고치면 이를 할 수
있습니다.

<Listing number="2-5" file-name="src/main.rs" caption="숫자가 아닌 예측값은 무시하고 프로그램을 충돌시키는 대신 다시 예측값을 묻기">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

에러 발생 시 충돌시키는 대신 처리하는 방향으로 넘어가기 위해, `expect` 호출을
`match` 표현식으로 바꿉니다. `parse`는 `Result` 타입을 반환하고, `Result`는 배리
언트 `Ok`와 `Err`를 가지는 열거형임을 기억하세요. `cmp` 메서드의 `Ordering`
결과에 대해 그랬던 것처럼 여기서도 `match` 표현식을 사용합니다.

`parse`가 문자열을 숫자로 성공적으로 변환할 수 있다면, 결과로 얻은 숫자를 담은
`Ok` 값을 반환합니다. 그 `Ok` 값은 첫 번째 갈래의 패턴과 일치할 것이고, `match`
표현식은 `parse`가 만들어 `Ok` 값에 넣어 둔 `num` 값을 그대로 반환합니다. 그 숫자는
우리가 만드는 새 `guess` 변수, 바로 우리가 원하던 자리에 들어가게 됩니다.

`parse`가 문자열을 숫자로 변환하지 *못하면*, 에러에 대한 더 많은 정보를 담은 `Err`
값을 반환합니다. `Err` 값은 첫 번째 `match` 갈래의 `Ok(num)` 패턴과 일치하지
않지만, 두 번째 갈래의 `Err(_)` 패턴과는 일치합니다. 밑줄 `_`은 모든 것을 받는
캐치올(catch-all) 값입니다. 이 예에서는 내부에 어떤 정보가 있든 상관없이 모든
`Err` 값과 일치시키겠다고 말하는 것입니다. 그래서 프로그램은 두 번째 갈래의
코드인 `continue`를 실행할 것이고, 이는 프로그램이 `loop`의 다음 반복으로 넘어가
또 다른 예측값을 묻도록 합니다. 따라서 사실상 프로그램은 `parse`가 마주칠 수
있는 모든 에러를 무시하게 됩니다!

이제 프로그램의 모든 것이 기대한 대로 동작해야 합니다. 시도해 봅시다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

훌륭합니다! 아주 작은 마지막 조정 하나로 숫자 맞히기 게임을 완성합니다. 프로그램이
여전히 비밀 숫자를 출력한다는 점을 떠올려 보세요. 테스트에는 유용했지만 게임을
망치기 때문에, 비밀 숫자를 출력하는 `println!`을 지우겠습니다. Listing 2-6이
최종 코드를 보여 줍니다.

<Listing number="2-6" file-name="src/main.rs" caption="완성된 숫자 맞히기 게임 코드">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

이 시점에서 여러분은 숫자 맞히기 게임을 성공적으로 만들었습니다. 축하드립니다!

## 요약

이 프로젝트는 `let`, `match`, 함수, 외부 크레이트 사용 등 많은 러스트 개념을 실습
으로 소개해 보는 시간이었습니다. 이어지는 몇 개의 장에서 이 개념들을 더 자세히
배웁니다. 3장은 변수, 데이터 타입, 함수처럼 대부분의 프로그래밍 언어가 가진
개념들을 다루며, 이를 러스트에서 어떻게 사용하는지 보여 줍니다. 4장은 러스트가
다른 언어와 구분되도록 해 주는 기능인 소유권을 탐구합니다. 5장은 구조체와 메서드
문법을, 6장은 열거형이 어떻게 동작하는지를 설명합니다.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
