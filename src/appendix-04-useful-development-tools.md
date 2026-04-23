## 부록 D: 유용한 개발 도구

이 부록에서는 러스트 프로젝트가 제공하는 몇 가지 유용한 개발 도구에 대해 이야기합니다. 자동 포매팅, 경고를 빠르게 적용하는 방법, 린터, IDE와의 통합을 살펴봅니다.

### `rustfmt`를 사용한 자동 포매팅

`rustfmt` 도구는 커뮤니티 코드 스타일에 따라 코드를 재포매팅합니다. 많은 협업 프로젝트는 러스트를 작성할 때 어떤 스타일을 사용할지에 대한 논쟁을 막기 위해 `rustfmt`를 사용합니다. 모두가 이 도구를 사용해 코드를 포매팅합니다.

러스트 설치는 기본적으로 `rustfmt`를 포함하므로, 시스템에 `rustfmt`와 `cargo-fmt` 프로그램이 이미 있을 것입니다. 이 두 명령은 `rustc`와 `cargo`와 유사합니다. `rustfmt`는 더 세밀한 제어를 가능하게 하고, `cargo-fmt`는 Cargo를 사용하는 프로젝트의 관례를 이해합니다. Cargo 프로젝트를 포매팅하려면 다음을 입력하세요.

```console
$ cargo fmt
```

이 명령을 실행하면 현재 크레이트의 모든 러스트 코드가 재포매팅됩니다. 이는 코드 스타일만 변경해야 하며 코드 의미는 바꾸지 않습니다. `rustfmt`에 대한 더 자세한 내용은 [문서][rustfmt]를 참고하세요.

### `rustfix`로 코드 수정하기

`rustfix` 도구는 러스트 설치에 포함되어 있으며, 여러분이 원하는 것이 분명한 명확한 수정 방법이 있는 컴파일러 경고를 자동으로 수정할 수 있습니다. 컴파일러 경고를 전에 본 적이 있을 것입니다. 예를 들어 다음 코드를 생각해 봅시다.

<span class="filename">파일명: src/main.rs</span>

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

여기서 변수 `x`를 가변으로 정의했지만, 실제로는 결코 변경하지 않습니다. 러스트는 이에 대해 경고합니다.

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

경고는 우리에게 `mut` 키워드를 제거하라고 제안합니다. `cargo fix` 명령을 실행하여 `rustfix` 도구를 사용해 그 제안을 자동으로 적용할 수 있습니다.

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

_src/main.rs_ 를 다시 보면 `cargo fix`가 코드를 변경한 것을 볼 수 있습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
fn main() {
    let x = 42;
    println!("{x}");
}
```

이제 변수 `x`는 불변이며 경고는 더 이상 나타나지 않습니다.

`cargo fix` 명령은 서로 다른 러스트 에디션 사이에서 코드를 전환하는 데도 사용할 수 있습니다. 에디션은 [부록 E][editions]<!-- ignore -->에서 다룹니다.

### Clippy로 더 많은 린트

Clippy 도구는 흔한 실수를 잡고 러스트 코드를 개선할 수 있도록 코드를 분석하는 린트 모음입니다. Clippy는 표준 러스트 설치에 포함되어 있습니다.

Cargo 프로젝트에서 Clippy의 린트를 실행하려면 다음을 입력하세요.

```console
$ cargo clippy
```

예를 들어, 이 프로그램이 하듯이 원주율 같은 수학 상수의 근삿값을 사용하는 프로그램을 작성한다고 가정해 봅시다.

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

이 프로젝트에서 `cargo clippy`를 실행하면 다음과 같은 오류가 발생합니다.

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

이 오류는 러스트에 이미 더 정밀한 `PI` 상수가 정의되어 있으며, 그 상수를 사용하면 프로그램이 더 정확해진다고 알려 줍니다. 그런 다음 `PI` 상수를 사용하도록 코드를 변경하게 됩니다.

다음 코드는 Clippy에서 어떤 오류나 경고도 일으키지 않습니다.

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Clippy에 대한 더 자세한 내용은 [문서][clippy]를 참고하세요.

### `rust-analyzer`를 사용한 IDE 통합

IDE 통합을 돕기 위해, 러스트 커뮤니티는 [`rust-analyzer`][rust-analyzer]<!-- ignore --> 사용을 권장합니다. 이 도구는 IDE와 프로그래밍 언어가 서로 통신하기 위한 명세인 [Language Server Protocol][lsp]<!-- ignore -->을 말하는 컴파일러 중심의 유틸리티 모음입니다. [Visual Studio Code용 Rust analyzer 플러그인][vscode] 같은 다른 클라이언트도 `rust-analyzer`를 사용할 수 있습니다.

설치 지침은 `rust-analyzer` 프로젝트의 [홈페이지][rust-analyzer]<!-- ignore -->를 방문한 다음, 특정 IDE에서 언어 서버 지원을 설치하세요. IDE가 자동 완성, 정의로 이동, 인라인 오류 같은 기능을 갖게 됩니다.

[rustfmt]: https://github.com/rust-lang/rustfmt
[editions]: appendix-05-editions.md
[clippy]: https://github.com/rust-lang/rust-clippy
[rust-analyzer]: https://rust-analyzer.github.io
[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer
