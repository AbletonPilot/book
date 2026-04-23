## 카고 워크스페이스

12장에서 바이너리 크레이트와 라이브러리 크레이트를 포함하는 패키지를 만들었
습니다. 프로젝트가 발전하면서 라이브러리 크레이트가 계속 커지고, 패키지를 여러
라이브러리 크레이트로 더 분리하고 싶어질 수 있습니다. 카고는 함께 개발되는
여러 관련 패키지를 관리하는 데 도움이 되는 _워크스페이스(workspaces)_ 라는
기능을 제공합니다.

### 워크스페이스 만들기

_워크스페이스_ 는 같은 _Cargo.lock_ 과 출력 디렉터리를 공유하는 패키지들의
집합입니다. 워크스페이스를 사용해 프로젝트를 만들어 봅시다 — 사소한 코드를
사용해 워크스페이스의 구조에 집중할 수 있게 하겠습니다. 워크스페이스를 구조화
하는 방법은 여러 가지이지만, 흔한 방법 한 가지만 보여 주겠습니다. 바이너리
하나와 라이브러리 두 개를 포함하는 워크스페이스를 만들 것입니다. 주요 기능을
제공할 바이너리는 두 라이브러리에 의존할 것입니다. 한 라이브러리는 `add_one`
함수를, 다른 라이브러리는 `add_two` 함수를 제공할 것입니다. 이 세 크레이트는
모두 같은 워크스페이스의 일부가 됩니다. 워크스페이스를 위한 새 디렉터리를
만드는 것부터 시작합시다.

```console
$ mkdir add
$ cd add
```

다음으로, _add_ 디렉터리에 워크스페이스 전체를 구성할 _Cargo.toml_ 파일을
만듭니다. 이 파일에는 `[package]` 섹션이 없습니다. 대신 워크스페이스에 멤버를
추가할 수 있게 해 주는 `[workspace]` 섹션으로 시작합니다. 또한 워크스페이스에서
카고의 최신 리졸버 알고리즘을 사용하기 위해 `resolver` 값을 `"3"`으로 설정합니다.

<span class="filename">파일명: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

다음으로 _add_ 디렉터리 안에서 `cargo new`를 실행해 `adder` 바이너리 크레이트를
만듭니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

워크스페이스 안에서 `cargo new`를 실행하면 새로 만들어진 패키지가 다음처럼
워크스페이스 _Cargo.toml_ 의 `[workspace]` 정의의 `members` 키에 자동으로
추가됩니다.

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

이 시점에 `cargo build`를 실행해 워크스페이스를 빌드할 수 있습니다. _add_
디렉터리의 파일은 다음과 같이 생겼을 것입니다.

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

워크스페이스는 최상위 수준에 컴파일된 산출물이 놓일 _target_ 디렉터리 하나를
가집니다. `adder` 패키지는 자체 _target_ 디렉터리를 가지지 않습니다. _adder_
디렉터리 안에서 `cargo build`를 실행하더라도, 컴파일된 산출물은 _add/adder/target_
이 아니라 여전히 _add/target_ 에 놓입니다. 카고는 워크스페이스의 크레이트들이
서로 의존하도록 되어 있기 때문에 워크스페이스의 _target_ 디렉터리를 이렇게
구조화합니다. 각 크레이트가 자체 _target_ 디렉터리를 가진다면, 각 크레이트가
자체 _target_ 디렉터리에 산출물을 놓기 위해 워크스페이스의 다른 크레이트
각각을 다시 컴파일해야 할 것입니다. 하나의 _target_ 디렉터리를 공유함으로써
크레이트들은 불필요한 재빌드를 피할 수 있습니다.

### 워크스페이스에 두 번째 패키지 만들기

다음으로 워크스페이스에 또 다른 멤버 패키지를 만들어 `add_one`이라고 부르겠
습니다. `add_one`이라는 새 라이브러리 크레이트를 생성하세요.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

최상위 _Cargo.toml_ 의 `members` 목록에 이제 _add_one_ 경로가 포함됩니다.

<span class="filename">파일명: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

_add_ 디렉터리에는 이제 다음 디렉터리와 파일들이 있어야 합니다.

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

_add_one/src/lib.rs_ 파일에 `add_one` 함수를 추가하겠습니다.

<span class="filename">파일명: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

이제 바이너리를 가진 `adder` 패키지가 라이브러리를 가진 `add_one` 패키지에
의존하도록 할 수 있습니다. 먼저 _adder/Cargo.toml_ 에 `add_one`에 대한 경로
의존성을 추가해야 합니다.

<span class="filename">파일명: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

카고는 워크스페이스의 크레이트들이 서로 의존한다고 가정하지 않으므로, 의존
관계에 대해 명시적이어야 합니다.

다음으로 `adder` 크레이트에서 (`add_one` 크레이트의) `add_one` 함수를 사용해
봅시다. _adder/src/main.rs_ 파일을 열고 Listing 14-7처럼 `add_one` 함수를
호출하도록 `main` 함수를 바꾸세요.

<Listing number="14-7" file-name="adder/src/main.rs" caption="`adder` 크레이트에서 `add_one` 라이브러리 크레이트 사용하기">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

최상위 _add_ 디렉터리에서 `cargo build`를 실행해 워크스페이스를 빌드해 봅시다!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

_add_ 디렉터리에서 바이너리 크레이트를 실행하려면, `-p` 인수와 패키지 이름으로
실행할 워크스페이스의 패키지를 `cargo run`에 지정할 수 있습니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

이는 `add_one` 크레이트에 의존하는 _adder/src/main.rs_ 의 코드를 실행합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="depending-on-an-external-package-in-a-workspace"></a>

### 외부 패키지에 의존하기

워크스페이스에는 각 크레이트 디렉터리에 _Cargo.lock_ 이 있는 것이 아니라
최상위에 단 하나의 _Cargo.lock_ 파일만 있다는 점에 유의하세요. 이는 모든
크레이트가 모든 의존성의 같은 버전을 사용하도록 보장합니다. _adder/Cargo.toml_
과 _add_one/Cargo.toml_ 파일에 `rand` 패키지를 추가하면, 카고는 그 둘을
`rand`의 한 버전으로 해결하고 하나의 _Cargo.lock_ 에 기록합니다. 워크스페이스
의 모든 크레이트가 같은 의존성을 사용하도록 하면 크레이트들이 항상 서로
호환됨을 의미합니다. `add_one` 크레이트에서 `rand` 크레이트를 사용할 수 있도록
_add_one/Cargo.toml_ 파일의 `[dependencies]` 섹션에 `rand` 크레이트를
추가합시다.

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">파일명: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

이제 _add_one/src/lib.rs_ 파일에 `use rand;`를 추가할 수 있고, _add_ 디렉터리
에서 `cargo build`를 실행해 전체 워크스페이스를 빌드하면 `rand` 크레이트를
가져와 컴파일합니다. 스코프로 가져온 `rand`를 참조하지 않으므로 경고가 하나
나올 것입니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

최상위 _Cargo.lock_ 은 이제 `add_one`이 `rand`에 의존한다는 정보를 포함합니다.
그러나 `rand`가 워크스페이스 어딘가에서 사용되더라도, 다른 크레이트들의
_Cargo.toml_ 에도 `rand`를 추가하지 않으면 그 크레이트들에서는 사용할 수
없습니다. 예를 들어 `adder` 패키지의 _adder/src/main.rs_ 파일에 `use rand;`를
추가하면 오류가 납니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

이를 고치려면 `adder` 패키지의 _Cargo.toml_ 파일을 편집해 `rand`가 의존성
임을 표시하세요. `adder` 패키지를 빌드하면 _Cargo.lock_ 의 `adder` 의존성
목록에 `rand`가 추가되지만, `rand`의 추가 사본은 다운로드되지 않습니다. 카고
는 워크스페이스의 어떤 패키지의 어떤 크레이트든 `rand`의 호환되는 버전을
지정하기만 하면 모두 같은 버전을 사용하도록 보장해, 공간을 절약하고 워크스
페이스의 크레이트들이 서로 호환됨을 보장합니다.

워크스페이스의 크레이트들이 같은 의존성의 호환되지 않는 버전을 지정하면,
카고는 각각을 해결하지만 가능한 한 적은 버전을 해결하려고 시도합니다.

### 워크스페이스에 테스트 추가하기

또 다른 개선으로, `add_one` 크레이트 내에서 `add_one::add_one` 함수의 테스트를
추가합시다.

<span class="filename">파일명: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

이제 최상위 _add_ 디렉터리에서 `cargo test`를 실행하세요. 이렇게 구조화된
워크스페이스에서 `cargo test`를 실행하면 워크스페이스의 모든 크레이트에 대한
테스트가 실행됩니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

출력의 첫 번째 섹션은 `add_one` 크레이트의 `it_works` 테스트가 통과했음을
보여 줍니다. 다음 섹션은 `adder` 크레이트에서 0개의 테스트가 발견됐음을
보여 주고, 마지막 섹션은 `add_one` 크레이트에서 0개의 문서 테스트가 발견
됐음을 보여 줍니다.

최상위 디렉터리에서 `-p` 플래그와 테스트하려는 크레이트의 이름을 지정해
워크스페이스의 특정 한 크레이트만 테스트할 수도 있습니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

이 출력은 `cargo test`가 `add_one` 크레이트에 대한 테스트만 실행하고 `adder`
크레이트 테스트는 실행하지 않았음을 보여 줍니다.

워크스페이스의 크레이트들을 [crates.io](https://crates.io/)<!-- ignore -->에
게시하려면 워크스페이스의 각 크레이트를 따로 게시해야 합니다. `cargo test`와
마찬가지로, `-p` 플래그와 게시하려는 크레이트의 이름을 지정해 워크스페이스의
특정 크레이트를 게시할 수 있습니다.

추가 연습으로, `add_one` 크레이트와 비슷한 방식으로 이 워크스페이스에
`add_two` 크레이트를 추가해 보세요!

프로젝트가 커지면 워크스페이스 사용을 고려해 보세요. 하나의 큰 덩어리 코드보다
더 작고 이해하기 쉬운 구성 요소로 작업할 수 있게 해 줍니다. 또한 크레이트들이
같이 자주 바뀐다면 워크스페이스에 크레이트들을 두면 크레이트 간 조율이 더
쉬워질 수 있습니다.
