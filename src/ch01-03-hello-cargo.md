## Hello, Cargo!

Cargo는 러스트의 빌드 시스템이자 패키지 관리자입니다. 대부분의 러스타시안은 이
도구로 러스트 프로젝트를 관리하는데, Cargo가 코드 빌드, 코드가 의존하는 라이브러리의
다운로드, 그리고 그 라이브러리들의 빌드와 같은 많은 일을 대신 처리해 주기 때문입니다.
(여러분의 코드가 필요로 하는 라이브러리를 *의존성(dependencies)*이라고 부릅니다.)

지금까지 작성한 것처럼 가장 단순한 러스트 프로그램에는 아무 의존성도 없습니다.
“Hello, world!” 프로젝트를 Cargo로 만들었다면, Cargo에서 코드 빌드를 담당하는
부분만 사용했을 것입니다. 더 복잡한 러스트 프로그램을 작성할수록 의존성을 추가하게
될 텐데, 프로젝트를 Cargo로 시작한다면 의존성을 훨씬 손쉽게 추가할 수 있습니다.

러스트 프로젝트의 절대 다수가 Cargo를 사용하기 때문에, 이 책의 나머지 부분도
여러분이 Cargo를 사용한다고 가정합니다. [“설치”][installation]<!-- ignore --> 절에서
설명한 공식 설치 프로그램을 사용했다면 Cargo는 러스트와 함께 설치됩니다. 다른
방법으로 러스트를 설치했다면, 터미널에 다음 명령을 입력해 Cargo가 설치되어 있는지
확인하세요.

```console
$ cargo --version
```

버전 번호가 보인다면 설치되어 있는 것입니다! `command not found`와 같은 에러가
보인다면, Cargo를 별도로 설치하는 방법을 확인하기 위해 여러분이 사용한 설치 방법의
문서를 살펴보세요.

### Cargo로 프로젝트 만들기

Cargo로 새 프로젝트를 만들고, 원래의 “Hello, world!” 프로젝트와 어떻게 다른지
살펴봅시다. *projects* 디렉터리(혹은 코드를 보관하기로 한 곳)로 이동합니다. 그런 다음
어떤 운영체제에서든 다음을 실행하세요.

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

첫 번째 명령은 *hello_cargo*라는 이름의 새 디렉터리와 프로젝트를 생성합니다. 우리가
프로젝트 이름을 *hello_cargo*로 지정했고, Cargo는 같은 이름의 디렉터리 안에 자신의
파일들을 만듭니다.

*hello_cargo* 디렉터리로 들어가 파일 목록을 확인해 보세요. Cargo가 우리를 위해 두
개의 파일과 한 개의 디렉터리, 즉 *Cargo.toml* 파일과 *main.rs* 파일이 들어 있는 *src*
디렉터리를 생성해 둔 것을 볼 수 있습니다.

또한 Cargo는 *.gitignore* 파일과 함께 새 Git 저장소도 초기화해 둡니다. 이미 존재하는
Git 저장소 안에서 `cargo new`를 실행하면 Git 관련 파일은 생성되지 않습니다. `cargo new
--vcs=git`을 사용해 이 동작을 덮어쓸 수 있습니다.

> 참고: Git은 널리 쓰이는 버전 관리 시스템입니다. `--vcs` 플래그를 사용해 `cargo
> new`가 다른 버전 관리 시스템을 사용하도록 하거나 버전 관리 시스템을 쓰지 않도록
> 변경할 수 있습니다. 사용 가능한 옵션은 `cargo new --help`를 실행해 보세요.

좋아하는 텍스트 에디터로 *Cargo.toml*을 열어 보세요. Listing 1-2의 코드와 비슷한
내용이 보일 것입니다.

<Listing number="1-2" file-name="Cargo.toml" caption="`cargo new`가 생성한 *Cargo.toml*의 내용">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

이 파일은 [*TOML*][toml]<!-- ignore --> (*Tom’s Obvious, Minimal Language*)
형식이며, Cargo의 설정 형식입니다.

첫 번째 줄인 `[package]`는 이후의 문장들이 패키지를 설정하고 있음을 나타내는 섹션
제목입니다. 이 파일에 더 많은 정보를 추가하면서 다른 섹션도 추가하게 됩니다.

다음 세 줄은 Cargo가 프로그램을 컴파일하기 위해 필요한 설정 정보, 즉 이름, 버전,
그리고 사용할 러스트 에디션을 지정합니다. `edition` 키에 대해서는 [부록
E][appendix-e]<!-- ignore -->에서 이야기하겠습니다.

마지막 줄인 `[dependencies]`는 프로젝트의 의존성을 나열하는 섹션의 시작입니다.
러스트에서는 코드 패키지를 *크레이트(crate)*라고 부릅니다. 이 프로젝트에서는 다른
크레이트가 필요하지 않지만, 2장의 첫 프로젝트에서는 필요하게 될 테니 그때 이
dependencies 섹션을 사용할 것입니다.

이제 *src/main.rs*를 열어 살펴보세요.

<span class="filename">파일명: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo가 Listing 1-1에서 우리가 작성한 것과 똑같은 “Hello, world!” 프로그램을
생성해 두었습니다! 지금까지 우리 프로젝트와 Cargo가 생성한 프로젝트의 차이점은,
Cargo가 코드를 *src* 디렉터리에 두고 있다는 것과 최상위 디렉터리에 *Cargo.toml*
설정 파일이 있다는 것뿐입니다.

Cargo는 소스 파일이 *src* 디렉터리 안에 위치할 것이라고 예상합니다. 최상위 프로젝트
디렉터리는 README 파일, 라이선스 정보, 설정 파일, 그리고 코드와 직접적으로 관련
없는 그 외의 것들을 위한 공간입니다. Cargo를 사용하면 프로젝트를 정리하는 데
도움이 됩니다. 모든 것에 자기 자리가 있고, 모든 것이 제자리에 있죠.

“Hello, world!” 프로젝트처럼 Cargo를 사용하지 않고 시작한 프로젝트가 있더라도
Cargo를 사용하는 프로젝트로 변환할 수 있습니다. 프로젝트 코드를 *src* 디렉터리로
옮기고, 적절한 *Cargo.toml* 파일을 만들면 됩니다. *Cargo.toml* 파일을 손쉽게
얻는 한 가지 방법은 `cargo init`을 실행하는 것인데, 이 명령이 파일을 자동으로
생성해 줍니다.

### Cargo 프로젝트 빌드하고 실행하기

이제 Cargo로 “Hello, world!” 프로그램을 빌드하고 실행할 때 무엇이 다른지 살펴봅시다!
*hello_cargo* 디렉터리에서 다음 명령을 입력해 프로젝트를 빌드하세요.

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

이 명령은 현재 디렉터리가 아니라 *target/debug/hello_cargo*(윈도우에서는
*target\debug\hello_cargo.exe*)에 실행 파일을 생성합니다. 기본 빌드는 디버그
빌드이기 때문에, Cargo는 바이너리를 *debug*라는 이름의 디렉터리에 넣습니다.
실행 파일은 다음 명령으로 실행할 수 있습니다.

```console
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
```

모든 것이 잘 된다면 터미널에 `Hello, world!`가 출력되어야 합니다. `cargo build`를
처음 실행하면 Cargo는 최상위에 *Cargo.lock*이라는 새 파일도 만듭니다. 이 파일은
프로젝트의 의존성에 대한 정확한 버전을 추적합니다. 이 프로젝트에는 의존성이 없으므로
파일 내용이 조금 단출합니다. 이 파일은 수동으로 바꿀 필요가 전혀 없습니다. Cargo가
내용을 관리해 줍니다.

방금 `cargo build`로 프로젝트를 빌드하고 `./target/debug/hello_cargo`로 실행해
봤는데, `cargo run`으로 코드를 컴파일하고 이어서 생성된 실행 파일까지 한 명령으로
실행할 수도 있습니다.

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

`cargo run`을 사용하는 편이 `cargo build`를 실행하고 이어서 바이너리의 전체
경로를 입력하는 것보다 훨씬 편리합니다. 그래서 대부분의 개발자가 `cargo run`을
사용합니다.

이번에는 Cargo가 `hello_cargo`를 컴파일하고 있다는 출력이 보이지 않은 점에
주목하세요. Cargo는 파일이 변경되지 않았다는 사실을 파악하고 다시 빌드하지 않고
바이너리만 실행했습니다. 만약 소스 코드를 수정했다면 Cargo가 프로젝트를 다시 빌드한
뒤에 실행했을 것이고, 다음과 같은 출력을 보았을 것입니다.

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo는 `cargo check`라는 명령도 제공합니다. 이 명령은 코드가 컴파일되는지만 빠르게
확인하며, 실행 파일은 생성하지 않습니다.

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

왜 실행 파일을 만들고 싶지 않을 수 있을까요? `cargo check`는 실행 파일을 만드는
단계를 건너뛰기 때문에 `cargo build`보다 훨씬 빠른 경우가 많습니다. 코드를 작성하는
동안 지속적으로 점검하고 싶다면, `cargo check`를 사용해 프로젝트가 여전히 컴파일
되는지 빠르게 확인할 수 있습니다! 그래서 많은 러스타시안이 프로그램을 작성하면서
`cargo check`를 주기적으로 실행하고, 실행 파일을 쓸 준비가 되었을 때에만 `cargo
build`를 실행합니다.

지금까지 Cargo에 대해 배운 내용을 정리해 봅시다.

- `cargo new`로 프로젝트를 생성할 수 있습니다.
- `cargo build`로 프로젝트를 빌드할 수 있습니다.
- `cargo run`으로 프로젝트를 한 번에 빌드하고 실행할 수 있습니다.
- `cargo check`로 바이너리를 만들지 않고 컴파일 오류를 확인하는 빌드를 할 수
  있습니다.
- Cargo는 빌드 결과물을 코드와 같은 디렉터리가 아니라 *target/debug* 디렉터리에
  저장합니다.

Cargo를 쓰는 또 다른 장점은, 어떤 운영체제에서 작업하든 명령이 동일하다는
점입니다. 따라서 이 시점부터는 리눅스와 맥OS, 그리고 윈도우에 대한 별도의 안내를
제공하지 않겠습니다.

### 릴리스를 위한 빌드

프로젝트를 최종적으로 릴리스할 준비가 되었다면, `cargo build --release`를 사용해
최적화를 적용해 컴파일할 수 있습니다. 이 명령은 *target/debug*가 아니라
*target/release*에 실행 파일을 만듭니다. 최적화는 러스트 코드를 더 빠르게 실행시켜
주지만, 최적화를 켜면 프로그램이 컴파일되는 데 걸리는 시간이 늘어납니다. 서로 다른
두 프로파일이 존재하는 이유가 바로 이것입니다. 하나는 빠르고 자주 빌드하고 싶은
개발용이고, 다른 하나는 반복해서 다시 빌드하지 않고 가능한 한 빨리 실행되기를
바라는, 사용자에게 전달할 최종 프로그램을 빌드하는 용도입니다. 코드의 실행 시간을
벤치마킹할 때에는, 반드시 `cargo build --release`로 빌드하고 *target/release*에
있는 실행 파일로 벤치마킹하세요.

<!-- Old headings. Do not remove or links may break. -->
<a id="cargo-as-convention"></a>

### Cargo의 관례 활용하기

단순한 프로젝트에서는 Cargo가 그냥 `rustc`를 쓰는 것에 비해 큰 가치를 주지
않지만, 프로그램이 복잡해질수록 진가를 발휘합니다. 프로그램이 여러 파일로
커지거나 의존성이 필요해지면, 빌드를 Cargo가 조율하게 두는 편이 훨씬 쉽습니다.

`hello_cargo` 프로젝트는 단순하지만, 이미 러스트로 일할 때 앞으로 자주 쓰게 될
도구들을 대부분 활용하고 있습니다. 실제로 기존 프로젝트에 작업을 이어 가려면, 다음
명령들로 Git을 이용해 코드를 체크아웃하고, 해당 프로젝트 디렉터리로 이동한 뒤
빌드할 수 있습니다.

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Cargo에 대한 추가 정보는 [Cargo 문서][cargo]를 확인해 보세요.

## 요약

여러분은 이미 러스트 여정의 훌륭한 출발선에 서 있습니다! 이 장에서는 다음을
배웠습니다.

- `rustup`으로 러스트의 최신 안정 버전 설치하기
- 새로운 러스트 버전으로 업데이트하기
- 로컬에 설치된 문서 열기
- `rustc`를 직접 사용해 “Hello, world!” 프로그램 작성하고 실행하기
- Cargo의 관례를 따라 새 프로젝트를 만들고 실행하기

이제 좀 더 본격적인 프로그램을 만들어 러스트 코드를 읽고 쓰는 데 익숙해지기 좋은
시점입니다. 그래서 2장에서는 숫자 맞히기 게임 프로그램을 만들어 보겠습니다. 일반적인
프로그래밍 개념이 러스트에서 어떻게 동작하는지를 먼저 배우고 싶다면, 3장을 먼저
읽은 뒤 2장으로 돌아와도 좋습니다.

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
