## Crates.io에 크레이트 게시하기

우리는 프로젝트의 의존성으로 [crates.io](https://crates.io/)<!-- ignore -->의
패키지를 사용해 봤지만, 여러분의 패키지를 게시하여 다른 사람들과 코드를 공유할
수도 있습니다. [crates.io](https://crates.io/)<!-- ignore -->의 크레이트
레지스트리는 여러분 패키지의 소스 코드를 배포하므로 주로 오픈 소스 코드를
호스팅합니다.

러스트와 카고에는 여러분이 게시한 패키지를 사람들이 더 쉽게 찾고 사용할 수
있도록 하는 기능이 있습니다. 이 기능들에 관해 이야기한 다음 패키지를 게시하는
방법을 설명하겠습니다.

### 유용한 문서화 주석 작성하기

패키지를 정확히 문서화하면 다른 사용자가 패키지를 어떻게, 언제 사용해야 할지
아는 데 도움이 되므로, 문서를 쓰는 데 시간을 투자할 가치가 있습니다. 3장에서는
슬래시 두 개 `//`로 러스트 코드에 주석을 다는 법을 이야기했습니다. 러스트에는
HTML 문서를 생성하는, 편리하게도 _문서화 주석(documentation comment)_ 이라고
부르는 특별한 종류의 주석도 있습니다. 이 HTML은 크레이트가 어떻게 _구현_ 되는지
보다는 크레이트를 어떻게 _사용_ 하는지를 알고 싶은 프로그래머를 대상으로 한,
공개 API 항목의 문서화 주석 내용을 보여 줍니다.

문서화 주석은 슬래시 두 개가 아닌 세 개 `///`를 사용하며, 텍스트 서식을 위한
마크다운 표기법을 지원합니다. 문서화 주석은 그것이 문서화하는 항목 바로 앞에
둡니다. Listing 14-1은 `my_crate`라는 크레이트의 `add_one` 함수에 대한 문서화
주석을 보여 줍니다.

<Listing number="14-1" file-name="src/lib.rs" caption="함수에 대한 문서화 주석">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

여기서는 `add_one` 함수가 무엇을 하는지 설명하고, `Examples`라는 제목의 섹션을
시작한 다음, `add_one` 함수 사용법을 보여 주는 코드를 제공합니다. `cargo doc`을
실행해 이 문서화 주석으로부터 HTML 문서를 생성할 수 있습니다. 이 명령은
러스트와 함께 배포되는 `rustdoc` 도구를 실행해 생성된 HTML 문서를 _target/doc_
디렉터리에 넣습니다.

편의를 위해 `cargo doc --open`을 실행하면 현재 크레이트 문서(그리고 크레이트의
모든 의존성 문서)의 HTML을 빌드한 다음 결과를 웹 브라우저에서 엽니다. `add_one`
함수로 이동하면 그림 14-1에 보이는 것처럼 문서화 주석의 텍스트가 어떻게
렌더링되는지 볼 수 있습니다.

<img alt="`my_crate`의 `add_one` 함수에 대한 렌더링된 HTML 문서" src="img/trpl14-01.png" class="center" />

<span class="caption">그림 14-1: `add_one` 함수에 대한 HTML 문서</span>

#### 자주 사용되는 섹션

Listing 14-1에서는 `# Examples` 마크다운 제목을 사용해 HTML에 “Examples”라는
제목의 섹션을 만들었습니다. 다음은 크레이트 작성자가 문서에서 흔히 사용하는
다른 섹션들입니다.

- **Panics**: 문서화 대상 함수가 패닉할 수 있는 시나리오입니다. 프로그램이
  패닉하기를 원치 않는 호출자는 이런 상황에서 함수를 호출하지 않도록 해야
  합니다.
- **Errors**: 함수가 `Result`를 반환하는 경우, 발생할 수 있는 오류의 종류와
  어떤 조건이 그런 오류를 반환하게 할 수 있는지 설명하면, 호출자가 서로 다른
  종류의 오류를 서로 다르게 처리하는 코드를 작성하는 데 도움이 될 수 있습니다.
- **Safety**: 함수 호출이 `unsafe`이면(안전하지 않음은 20장에서 논의합니다),
  함수가 왜 안전하지 않은지 설명하고 호출자가 지켜야 할 불변식을 다루는 섹션이
  있어야 합니다.

대부분의 문서화 주석에 이 모든 섹션이 필요하지는 않지만, 사용자가 알고 싶어
할 코드의 측면을 상기시키는 좋은 체크리스트입니다.

#### 테스트로서의 문서화 주석

문서화 주석에 예제 코드 블록을 추가하면 라이브러리 사용법을 보여 주는 데
도움이 되고, 추가로 보너스가 있습니다. `cargo test`를 실행하면 문서의 코드
예제가 테스트로 실행됩니다! 예제가 있는 문서보다 좋은 것은 없지만, 문서 작성
이후 코드가 바뀌어서 동작하지 않는 예제보다 나쁜 것은 없습니다. Listing 14-1의
`add_one` 함수 문서로 `cargo test`를 실행하면, 테스트 결과에 다음과 같은
섹션이 보일 것입니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

이제 함수나 예제를 바꿔서 예제의 `assert_eq!`가 패닉하도록 한 뒤 다시
`cargo test`를 실행하면, 문서 테스트가 예제와 코드가 서로 어긋났음을 잡아내는
것을 볼 수 있습니다!

<!-- Old headings. Do not remove or links may break. -->

<a id="commenting-contained-items"></a>

#### 포함하는 항목에 대한 주석

`//!` 스타일의 문서 주석은 주석 _뒤에 오는_ 항목이 아니라 주석을 _담고 있는_
항목에 문서를 추가합니다. 보통 이 문서 주석은 크레이트 루트 파일(관례상
_src/lib.rs_) 안이나 모듈 안에서 크레이트 또는 모듈 전체를 문서화하는 데
사용합니다.

예를 들어 `add_one` 함수를 담고 있는 `my_crate` 크레이트의 목적을 설명하는
문서를 추가하려면, Listing 14-2처럼 _src/lib.rs_ 파일의 시작에 `//!`로 시작
하는 문서화 주석을 추가합니다.

<Listing number="14-2" file-name="src/lib.rs" caption="`my_crate` 크레이트 전체에 대한 문서">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

`//!`로 시작하는 마지막 줄 뒤에 어떤 코드도 없음에 유의하세요. 주석을 `///`가
아니라 `//!`로 시작했기 때문에, 이 주석 뒤에 오는 항목이 아니라 이 주석을
담고 있는 항목을 문서화합니다. 이 경우 그 항목은 크레이트 루트인 _src/lib.rs_
파일입니다. 이 주석들은 크레이트 전체를 설명합니다.

`cargo doc --open`을 실행하면, 이 주석들은 그림 14-2처럼 `my_crate`의 문서
첫 페이지에서 크레이트의 공개 항목 목록 위쪽에 표시됩니다.

항목 내부의 문서화 주석은 특히 크레이트와 모듈을 설명하는 데 유용합니다.
컨테이너의 전체 목적을 설명해 사용자가 크레이트의 구성을 이해하는 데 도움이
되도록 사용하세요.

<img alt="크레이트 전체에 대한 주석이 포함된 렌더링된 HTML 문서" src="img/trpl14-02.png" class="center" />

<span class="caption">그림 14-2: 크레이트 전체를 설명하는 주석을 포함한
`my_crate`의 렌더링된 문서</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="exporting-a-convenient-public-api-with-pub-use"></a>

### 편리한 공개 API 내보내기

크레이트를 게시할 때 공개 API의 구조는 중요한 고려 사항입니다. 여러분의
크레이트를 사용하는 사람들은 여러분만큼 구조에 익숙하지 않고, 크레이트에 큰
모듈 계층이 있다면 사용하려는 부분을 찾기 어려워할 수 있습니다.

7장에서는 `pub` 키워드로 항목을 공개하는 방법과 `use` 키워드로 항목을 스코프로
가져오는 방법을 다뤘습니다. 그러나 크레이트를 개발하는 여러분에게 자연스러운
구조가 사용자에게는 그리 편리하지 않을 수 있습니다. 구조체를 여러 수준의
계층으로 구성하고 싶을 수도 있지만, 여러분이 계층 깊숙이에 정의한 타입을
사용하고 싶은 사람들은 그 타입이 존재한다는 사실을 알아내는 데 어려움을 겪을
수 있습니다. 또한 `use my_crate::UsefulType;` 대신
`use my_crate::some_module::another_module::UsefulType;`을 입력해야 해서
짜증이 날 수도 있습니다.

좋은 소식은 구조가 다른 라이브러리에서 사용하기에 편리하지 _않다고_ 해서
내부 구성을 재배치할 필요는 없다는 것입니다. 대신 `pub use`를 사용해 항목을
재내보내기(re-export)하여, 여러분의 비공개 구조와는 다른 공개 구조를 만들 수
있습니다. _재내보내기_ 는 한 위치의 공개 항목을 마치 다른 위치에 정의된 것처럼
그 다른 위치에서 공개로 만드는 것입니다.

예를 들어 예술 개념을 모델링하기 위해 `art`라는 라이브러리를 만들었다고 해
봅시다. 이 라이브러리 안에는 두 개의 모듈이 있습니다. `PrimaryColor`와
`SecondaryColor`라는 두 열거형을 담은 `kinds` 모듈과, `mix`라는 함수를 담은
`utils` 모듈이죠. Listing 14-3처럼요.

<Listing number="14-3" file-name="src/lib.rs" caption="`kinds`와 `utils` 모듈로 항목이 구성된 `art` 라이브러리">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

그림 14-3은 `cargo doc`이 이 크레이트에 대해 생성한 문서의 첫 페이지가 어떻게
생겼는지 보여 줍니다.

<img alt="`kinds`와 `utils` 모듈을 나열하는 `art` 크레이트의 렌더링된 문서" src="img/trpl14-03.png" class="center" />

<span class="caption">그림 14-3: `kinds`와 `utils` 모듈을 나열하는 `art`
문서의 첫 페이지</span>

`PrimaryColor`와 `SecondaryColor` 타입이 첫 페이지에 나열되지 않으며, `mix`
함수도 마찬가지임에 유의하세요. 그들을 보려면 `kinds`와 `utils`를 클릭해야
합니다.

이 라이브러리에 의존하는 또 다른 크레이트는 현재 정의된 모듈 구조를 지정해
`art`의 항목을 스코프로 가져오는 `use` 구문이 필요합니다. Listing 14-4는
`art` 크레이트의 `PrimaryColor`와 `mix` 항목을 사용하는 크레이트의 예를 보여
줍니다.

<Listing number="14-4" file-name="src/main.rs" caption="내부 구조가 그대로 내보내진 `art` 크레이트의 항목을 사용하는 크레이트">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

`art` 크레이트를 사용하는 Listing 14-4 코드의 작성자는 `PrimaryColor`가
`kinds` 모듈에 있고 `mix`가 `utils` 모듈에 있음을 알아내야 했습니다. `art`
크레이트의 모듈 구조는 `art` 크레이트를 사용하는 사람보다 `art` 크레이트를
개발하는 사람에게 더 관련 있습니다. 내부 구조는 `art` 크레이트 사용법을
이해하려는 사람에게 유용한 정보를 담고 있지 않으며, 오히려 개발자들이 어디를
보아야 할지 알아내고 `use` 구문에 모듈 이름을 지정해야 하기 때문에 혼란을
유발합니다.

공개 API에서 내부 구성을 제거하기 위해, Listing 14-5처럼 Listing 14-3의 `art`
크레이트 코드를 수정해 `pub use` 구문을 추가해서 최상위 수준에서 항목을
재내보내기할 수 있습니다.

<Listing number="14-5" file-name="src/lib.rs" caption="항목을 재내보내기하는 `pub use` 구문 추가하기">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

이제 이 크레이트에 대해 `cargo doc`이 생성하는 API 문서는 그림 14-4처럼
첫 페이지에 재내보내기를 나열하고 링크합니다. `PrimaryColor`와 `SecondaryColor`
타입, 그리고 `mix` 함수를 더 쉽게 찾을 수 있게 해 줍니다.

<img alt="재내보내기가 첫 페이지에 포함된 `art` 크레이트의 렌더링된 문서" src="img/trpl14-04.png" class="center" />

<span class="caption">그림 14-4: 재내보내기를 나열하는 `art` 문서의 첫
페이지</span>

`art` 크레이트 사용자는 Listing 14-4처럼 Listing 14-3의 내부 구조를 여전히
보고 사용할 수 있고, Listing 14-6처럼 Listing 14-5의 더 편리한 구조를 사용할
수도 있습니다.

<Listing number="14-6" file-name="src/main.rs" caption="`art` 크레이트의 재내보내진 항목을 사용하는 프로그램">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

중첩된 모듈이 많은 경우, `pub use`로 최상위 수준에서 타입을 재내보내기하는
것은 크레이트를 사용하는 사람들의 경험에 상당한 차이를 만들 수 있습니다.
`pub use`의 또 다른 흔한 쓰임새는 현재 크레이트에서 의존성의 정의를 재내보내기
하여 그 크레이트의 정의를 여러분 크레이트의 공개 API의 일부로 만드는 것입니다.

유용한 공개 API 구조를 만드는 것은 과학이라기보다 예술에 가까우며, 사용자에게
가장 잘 맞는 API를 찾기 위해 반복할 수 있습니다. `pub use`를 선택하면 크레이트
내부를 구성하는 방식에 유연성을 주고, 그 내부 구조를 사용자에게 보여 주는
것과 분리할 수 있습니다. 여러분이 설치한 크레이트 중 일부의 코드를 살펴보며
내부 구조가 공개 API와 다른지 확인해 보세요.

### Crates.io 계정 설정하기

크레이트를 게시하려면 먼저 [crates.io](https://crates.io/)<!-- ignore -->에
계정을 만들고 API 토큰을 얻어야 합니다. 그러려면 [crates.io](https://crates.io/)<!-- ignore -->
홈페이지에 방문해 GitHub 계정으로 로그인하세요. (현재는 GitHub 계정이 필수
이지만, 향후 다른 계정 생성 방식도 지원될 수 있습니다.) 로그인한 후
[https://crates.io/me/](https://crates.io/me/)<!-- ignore -->의 계정 설정에
방문해 API 키를 가져오세요. 그런 다음 `cargo login` 명령을 실행하고 프롬프트
가 나오면 API 키를 붙여 넣습니다. 다음처럼요.

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

이 명령은 카고에 API 토큰을 알리고 로컬 _~/.cargo/credentials.toml_ 에
저장합니다. 이 토큰은 비밀임에 유의하세요. 아무에게도 공유하지 마세요. 어떤
이유로든 공유하게 된다면 즉시 취소하고 [crates.io](https://crates.io/)<!-- ignore -->
에서 새 토큰을 생성해야 합니다.

### 새 크레이트에 메타데이터 추가하기

게시하고 싶은 크레이트가 있다고 해 봅시다. 게시하기 전에 크레이트 _Cargo.toml_
파일의 `[package]` 섹션에 메타데이터를 추가해야 합니다.

크레이트에는 고유한 이름이 필요합니다. 로컬에서 크레이트를 작업하는 동안에는
원하는 어떤 이름이든 지을 수 있습니다. 하지만 [crates.io](https://crates.io/)<!-- ignore -->
의 크레이트 이름은 선착순으로 할당됩니다. 어떤 크레이트 이름이 이미 사용되면
다른 누구도 그 이름으로 크레이트를 게시할 수 없습니다. 크레이트를 게시하기
전에 사용하고 싶은 이름을 검색해 보세요. 이미 사용되었다면 다른 이름을 찾아서
게시를 위해 _Cargo.toml_ 파일의 `[package]` 섹션 `name` 필드를 새 이름으로
편집해야 합니다. 다음과 같이요.

<span class="filename">파일명: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

고유한 이름을 골랐더라도 이 시점에 `cargo publish`를 실행해 크레이트를 게시
하려고 하면 경고와 오류를 얻게 됩니다.

<!-- manual-regeneration
Create a new package with an unregistered name, making no further modifications
  to the generated package, so it is missing the description and license fields.
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

이는 중요한 정보 일부가 빠져 있기 때문에 오류가 발생합니다. 사람들이 여러분의
크레이트가 무엇을 하는지와 어떤 조건에서 사용할 수 있는지 알 수 있도록
설명(description)과 라이선스(license)가 필요합니다. _Cargo.toml_ 에 한두
문장의 설명을 추가하세요. 검색 결과에서 크레이트와 함께 표시됩니다. `license`
필드에는 _라이선스 식별자 값(license identifier value)_ 을 주어야 합니다.
[리눅스 재단의 Software Package Data Exchange(SPDX)][spdx]에 이 값으로 사용
할 수 있는 식별자들이 나열되어 있습니다. 예를 들어 MIT 라이선스로 크레이트를
라이선스했음을 지정하려면 `MIT` 식별자를 추가하세요.

<span class="filename">파일명: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

SPDX에 없는 라이선스를 사용하려면 그 라이선스의 텍스트를 파일에 담고, 프로젝트
에 그 파일을 포함한 뒤 `license` 키 대신 `license-file`을 사용해 그 파일
이름을 지정해야 합니다.

어떤 라이선스가 여러분의 프로젝트에 적합한지에 대한 지침은 이 책의 범위를
벗어납니다. 러스트 커뮤니티의 많은 사람들은 러스트와 같은 방식, 즉 `MIT OR
Apache-2.0`의 이중 라이선스로 프로젝트를 라이선스합니다. 이 관행은 여러 라이
선스를 `OR`로 구분해 지정해 프로젝트에 여러 라이선스를 둘 수도 있음을 보여
줍니다.

고유 이름, 버전, 설명, 라이선스가 추가된 게시 준비가 된 프로젝트의
_Cargo.toml_ 파일은 다음과 같이 보일 수 있습니다.

<span class="filename">파일명: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[카고 문서](https://doc.rust-lang.org/cargo/)에는 다른 사람들이 여러분의
크레이트를 더 쉽게 발견하고 사용할 수 있도록 지정할 수 있는 다른 메타데이터가
설명되어 있습니다.

### Crates.io에 게시하기

이제 계정을 만들고, API 토큰을 저장하고, 크레이트 이름을 정하고, 필수 메타
데이터를 지정했으니 게시 준비가 되었습니다! 크레이트를 게시하면 특정 버전이
[crates.io](https://crates.io/)<!-- ignore -->에 업로드되어 다른 사람들이
사용할 수 있게 됩니다.

조심하세요. 게시는 _영구적_ 입니다. 그 버전은 절대 덮어쓸 수 없으며, 특정한
상황이 아니면 코드를 삭제할 수 없습니다. Crates.io의 주요 목표 중 하나는 코드의
영구 아카이브 역할을 해, [crates.io](https://crates.io/)<!-- ignore -->의
크레이트에 의존하는 모든 프로젝트의 빌드가 계속 동작하도록 하는 것입니다.
버전 삭제를 허용하면 그 목표를 달성할 수 없게 됩니다. 그러나 여러분이 게시할
수 있는 크레이트 버전의 수에는 제한이 없습니다.

다시 `cargo publish` 명령을 실행해 보세요. 이제 성공할 것입니다.

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
    Packaged 6 files, 1.2KiB (895.0B compressed)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
    Uploaded guessing_game v0.1.0 to registry `crates-io`
note: waiting for `guessing_game v0.1.0` to be available at registry
`crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published guessing_game v0.1.0 at registry `crates-io`
```

축하합니다! 러스트 커뮤니티에 여러분의 코드를 공유했으며, 이제 누구나 쉽게
여러분의 크레이트를 프로젝트 의존성으로 추가할 수 있습니다.

### 기존 크레이트의 새 버전 게시하기

크레이트를 변경하고 새 버전을 릴리스할 준비가 되었다면, _Cargo.toml_ 파일에
지정된 `version` 값을 바꾸고 다시 게시하세요. 여러분이 한 변경의 종류에 따라
적절한 다음 버전 번호를 정하려면 [시맨틱 버저닝 규칙][semver]을 사용하세요.
그런 다음 `cargo publish`를 실행해 새 버전을 업로드하세요.

<!-- Old headings. Do not remove or links may break. -->

<a id="removing-versions-from-cratesio-with-cargo-yank"></a>
<a id="deprecating-versions-from-cratesio-with-cargo-yank"></a>

### Crates.io에서 버전 사용 중단하기

크레이트의 이전 버전을 제거할 수는 없지만, 향후 프로젝트가 그 버전을 새
의존성으로 추가하지 못하게 막을 수는 있습니다. 이는 어떤 이유로든 크레이트
버전이 망가졌을 때 유용합니다. 그런 상황에서 카고는 크레이트 버전 양크(yank)를
지원합니다.

버전을 _양크(yank)_ 하면 새 프로젝트가 그 버전에 의존하지 못하게 막으면서,
그 버전에 의존하는 모든 기존 프로젝트는 계속 작업할 수 있게 합니다. 본질적
으로 양크는 _Cargo.lock_ 을 가진 모든 프로젝트는 망가지지 않음을 의미하며,
이후 생성되는 _Cargo.lock_ 파일은 양크된 버전을 사용하지 않습니다.

크레이트 버전을 양크하려면 이전에 게시한 크레이트의 디렉터리에서 `cargo yank`
를 실행하고 양크하려는 버전을 지정하세요. 예를 들어 `guessing_game`이라는
크레이트의 1.0.1 버전을 게시했고 이를 양크하고 싶다면, `guessing_game`의
프로젝트 디렉터리에서 다음을 실행합니다.

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

명령에 `--undo`를 추가하면 양크를 취소하고 프로젝트가 다시 그 버전에 의존하기
시작할 수 있게 할 수도 있습니다.

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

양크는 어떤 코드도 삭제하지 _않습니다_. 예를 들어 실수로 업로드한 비밀을
삭제할 수는 없습니다. 그런 일이 있다면 그 비밀을 즉시 재설정해야 합니다.

[spdx]: https://spdx.org/licenses/
[semver]: https://semver.org/
