# The Rust Programming Language (한국어 번역 2026.04.24)

![Build Status](https://github.com/rust-lang/book/workflows/CI/badge.svg)

이 저장소는 *The Rust Programming Language* 책의 소스를 담고 있습니다.

[이 책은 No Starch Press에서 종이책 형태로도 구매할 수 있습니다.][nostarch]

[nostarch]: https://nostarch.com/rust-programming-language-2nd-edition

온라인에서 무료로 읽을 수도 있습니다. 러스트의 최신 [stable], [beta], [nightly]
릴리스와 함께 배포되는 버전을 확인하세요. 해당 릴리스들은 업데이트 주기가 길기 때문에,
저장소에서는 이미 수정된 이슈가 릴리스 버전에는 아직 반영되어 있지 않을 수 있다는 점에
유의해 주세요.

[stable]: https://doc.rust-lang.org/stable/book/
[beta]: https://doc.rust-lang.org/beta/book/
[nightly]: https://doc.rust-lang.org/nightly/book/

책에 등장하는 모든 코드 예제의 소스만 내려받고 싶다면 [releases]를 참고하세요.

[releases]: https://github.com/rust-lang/book/releases

## 요구사항

책을 빌드하려면 [mdBook]이 필요합니다. 가능하면 rust-lang/rust가 [이
파일][rust-mdbook]에서 사용하는 것과 동일한 버전을 사용하세요. 다음 명령으로
설치할 수 있습니다.

[mdBook]: https://github.com/rust-lang/mdBook
[rust-mdbook]: https://github.com/rust-lang/rust/blob/HEAD/src/tools/rustbook/Cargo.toml

```bash
$ cargo install mdbook --locked --version <version_num>
```

## 빌드

책을 빌드하려면 다음을 실행하세요.

```bash
$ mdbook build
```

결과물은 `book` 하위 디렉터리에 생성됩니다. 확인하려면 웹 브라우저에서 열어 보세요.

_Firefox:_

```bash
$ firefox book/index.html                       # Linux
$ open -a "Firefox" book/index.html             # OS X
$ Start-Process "firefox.exe" .\book\index.html # Windows (PowerShell)
$ start firefox.exe .\book\index.html           # Windows (Cmd)
```

_Chrome:_

```bash
$ google-chrome book/index.html                 # Linux
$ open -a "Google Chrome" book/index.html       # OS X
$ Start-Process "chrome.exe" .\book\index.html  # Windows (PowerShell)
$ start chrome.exe .\book\index.html            # Windows (Cmd)
```

테스트를 실행하려면 다음과 같이 하세요.

```bash
$ cd packages/trpl
$ mdbook test --library-path packages/trpl/target/debug/deps
```

## 기여하기

여러분의 도움을 환영합니다! 어떤 종류의 기여를 받고 있는지는
[CONTRIBUTING.md][contrib]를 참고하세요.

[contrib]: https://github.com/rust-lang/book/blob/main/CONTRIBUTING.md

이 책은 [인쇄본][nostarch]으로도 출판되고 있으며, 온라인 버전을 가능한 한 인쇄본과
가깝게 유지하고자 하기 때문에 이슈나 풀 리퀘스트 대응이 일반 프로젝트보다 오래 걸릴
수 있습니다.

지금까지는 [러스트 에디션](https://doc.rust-lang.org/edition-guide/)에 맞춰
대규모 개정 작업을 진행해 왔습니다. 그 사이사이에는 오류 수정만 반영합니다. 여러분의
이슈나 풀 리퀘스트가 엄밀한 의미의 오류 수정이 아니라면, 다음 대규모 개정 작업이
진행될 때까지 보류될 수 있습니다(몇 달에서 몇 년 단위로 걸릴 수 있습니다). 양해
부탁드립니다!

### 번역

번역 작업에 도움을 주실 분을 환영합니다! 현재 진행 중인 작업에 참여하려면
[Translations] 레이블을 살펴보세요. 새로운 언어로 번역을 시작하려면 새 이슈를
열어 주세요. 여러 언어를 본 저장소에 병합하기 전에 [mdbook support]를 기다리고
있지만, 그 전에도 자유롭게 시작하셔도 괜찮습니다!

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang/mdBook/issues/5

## 맞춤법 검사

소스 파일의 맞춤법 오류를 확인하려면 `ci` 디렉터리에 있는 `spellcheck.sh`
스크립트를 사용할 수 있습니다. 이 스크립트는 유효한 단어 사전을 필요로 하며,
`ci/dictionary.txt`에 제공되어 있습니다. 스크립트가 오탐(예: `BTreeMap`처럼
스크립트가 유효하지 않다고 판단하는 단어 사용)을 보고한다면, 해당 단어를
`ci/dictionary.txt`에 추가해 주세요(일관성을 위해 정렬된 순서를 유지합니다).
