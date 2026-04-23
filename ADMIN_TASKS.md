# 관리 작업(Administrative Tasks)

이 문서는 저장소를 관리하는 사람이 가끔 수행해야 하는 유지보수 작업을 잊지
않도록 정리한 참고 자료입니다.

## `rustc` 버전 업데이트

- 어차피 전부 다시 컴파일할 예정이므로 `target` 디렉터리를 먼저 삭제하세요.
- `.github/workflows/main.yml`의 버전 번호를 변경합니다.
- `rust-toolchain`의 버전 번호를 변경합니다. 이 파일의 버전은 로컬에서 `rustup`으로
  사용하는 버전도 함께 바꿔 줍니다.
- `src/title-page.md`의 버전 번호를 변경합니다.
- `./tools/update-rustc.sh`를 실행합니다(이 스크립트가 무엇을 하는지에 대한 자세한
  내용은 스크립트 안의 주석을 확인하세요).
- 변경 사항(git이 보고하는 파일 변경 내역)과 그 효과(`tmp/book-before`, `tmp/book-after`
  안의 파일)를 살펴보고, 적절해 보이면 커밋합니다.
- `manual-regeneration`을 grep으로 찾아, 스크립트로 재생성할 수 없는 출력 결과를
  수동으로 업데이트하라는 지시 사항을 따라 반영합니다.

## 모든 리스팅(listing)의 `edition` 업데이트

모든 리스팅의 `Cargo.toml`에 있는 `edition = "[year]"` 메타데이터를 업데이트하려면,
`./tools/update-editions.sh` 스크립트를 실행하세요. diff를 확인해서 업데이트가
합리적인지, 특히 업데이트에 따라 본문도 수정이 필요한지 여부를 점검한 뒤 커밋합니다.

## mdBook 설정의 `edition` 업데이트

`book.toml`과 `nostarch/book.toml`을 열어 `[rust]` 테이블의 `edition` 값을 새
에디션으로 설정합니다.

## 리스팅의 새 버전 배포

이제 모든 리스팅이 포함된 완성된 프로젝트를 `.tar` 파일로 묶어 [GitHub
Releases](https://github.com/rust-lang/book/releases)에 제공합니다. 편집
내용이나 러스트 및 `rustfmt` 업데이트로 인해 코드 변경이 생겼을 때, 새 릴리스
아티팩트를 만드는 방법은 다음과 같습니다.

- 릴리스용 git 태그를 만들어 GitHub에 push 하거나, GitHub UI에서 [새 릴리스
  초안 작성](https://github.com/rust-lang/book/releases/new) 페이지로 가서 기존
  태그 선택 대신 새 태그를 입력하는 방식으로 태그를 새로 만듭니다.
- `cargo run --bin release_listings`를 실행하면 `tmp/listings.tar.gz`가
  생성됩니다.
- GitHub UI의 릴리스 초안에 `tmp/listings.tar.gz`를 업로드합니다.
- 릴리스를 게시(publish)합니다.

## 새 리스팅 추가

모든 리스팅에 `rustfmt`를 실행하고, 컴파일러 업데이트에 따라 출력을 갱신하며, 리스팅에
대응하는 완전한 프로젝트를 포함한 릴리스 아티팩트를 생성하는 스크립트가 제대로
동작하도록, 아주 사소한 리스팅을 제외한 모든 리스팅은 별도의 파일로 추출되어 있어야
합니다. 방법은 다음과 같습니다.

- `listings` 디렉터리에서 새 리스팅이 들어갈 위치를 찾습니다.
  - 각 장마다 하나의 하위 디렉터리가 있습니다.
  - 번호가 매겨진 리스팅은 디렉터리 이름을 `listing-[chapter num]-[listing num]`
    형식으로 사용해야 합니다.
  - 번호가 없는 리스팅은 `no-listing-` 뒤에, 해당 장의 다른 번호 없는 리스팅들에
    대비한 위치를 나타내는 번호를 붙이고, 그 뒤에 찾는 사람이 원하는 코드를 쉽게
    알아볼 수 있도록 짧은 설명을 덧붙입니다.
  - 코드의 *출력*을 보여 주기 위해서만 사용되는 리스팅(예: "만약 y 대신 x라고
    작성했다면 이런 컴파일러 오류가 발생합니다"라고 하지만 실제로는 x 코드를 싣지
    않는 경우)은 `output-only-` 뒤에 해당 장 내 다른 출력 전용 리스팅들에 대비한
    위치를 나타내는 번호를 붙이고, 그 뒤에 저자나 기여자가 찾는 코드를 쉽게 알아볼
    수 있도록 짧은 설명을 덧붙여야 합니다.
  - **주변 리스팅의 번호를 적절히 조정하는 것을 잊지 마세요!**
- 해당 디렉터리에 완성된 Cargo 프로젝트를 만듭니다. `cargo new`를 사용하거나 다른
  리스팅을 복사해 시작점으로 삼을 수 있습니다.
- 완전하게 동작하는 예제가 되도록 필요한 주변 코드까지 포함하여 코드를 추가합니다.
- 파일 안의 일부 코드만 보여 주고 싶다면, 앵커 주석(`// ANCHOR: some_tag`과
  `// ANCHOR_END: some_tag`)으로 보여 줄 부분을 표시합니다.
- 러스트 코드의 경우, 본문 내 코드 블록 안에서
  `{{#rustdoc_include [filename:some_tag]}}` 지시자를 사용합니다. `rustdoc_include`
  지시자는 화면에 표시되지 않는 코드도 `mdbook test`가 `rustdoc`에 전달할 수 있게
  해 줍니다.
- 그 외의 경우에는 `{{#include [filename:some_tag]}}` 지시자를 사용합니다.
- 본문에서 명령 실행 결과도 함께 보여 주고 싶다면, 리스팅 디렉터리에 `output.txt`
  파일을 다음과 같이 만듭니다.
  - `cargo run`이나 `cargo test` 같은 명령을 실행하고, 출력 전체를 복사합니다.
  - 새 `output.txt` 파일을 만들고, 첫 줄에 `$ [실행한 명령어]`를 적습니다.
  - 방금 복사한 출력을 붙여 넣습니다.
  - `./tools/update-rustc.sh`를 실행하면 컴파일러 출력에 대한 정규화(normalization)가
    수행됩니다.
  - `{{#include [filename]}}` 지시자로 본문에 출력을 포함시킵니다.
  - `output.txt`를 git에 추가하고 커밋합니다.
- 어떤 이유로 스크립트로 생성할 수 없는 출력(예: 사용자 입력이나 웹 요청과 같은
  외부 이벤트 때문에)을 보여 주고 싶다면, 출력을 본문 안에 직접(inline) 유지하되,
  `manual-regeneration`이라는 주석과 함께 인라인 출력을 수동으로 갱신하는 방법을
  함께 적어 두세요.
- 어떤 예제가 의도적으로 파싱되지 않는 경우처럼 `rustfmt`의 포맷팅 시도조차 원하지
  않는다면, 해당 리스팅 디렉터리에 `rustfmt-ignore` 파일을 만들고, 포맷팅하지 않는
  이유를 그 파일의 내용으로 적어 두세요(언젠가 수정될 수 있는 rustfmt 버그인
  경우를 대비하기 위함).

## 어떤 변경이 렌더링된 책에 미치는 영향 확인하기

예를 들어 `mdbook` 업데이트나 파일 포함 방식 변경 등의 효과를 확인하려면 다음과
같이 하세요.

- 테스트할 변경을 적용하기 전에 `mdbook build -d tmp/book-before`를 실행해 빌드된
  책을 생성합니다.
- 테스트할 변경을 적용한 뒤 `mdbook build -d tmp/book-after`를 실행합니다.
- `./tools/megadiff.sh`를 실행합니다.
- `tmp/book-before`와 `tmp/book-after`에 남아 있는 파일에 차이가 있는 것이며,
  원하는 diff 도구로 직접 확인할 수 있습니다.

## No Starch용 마크다운 파일 생성

- `./tools/nostarch.sh`를 실행합니다.
- 스크립트가 `nostarch` 디렉터리에 생성한 파일을 간단히(spot check) 검토합니다.
- 편집 작업을 시작하는 시점이라면 git에 커밋합니다.

## diff 용 마크다운을 docx에서 생성하기

- docx 파일을 `tmp/chapterXX.docx` 로 저장합니다.
- Word에서 검토 탭으로 가서 "모든 변경 내용 적용 및 변경 내용 추적 중지(Accept
  all changes and stop tracking)"를 선택합니다.
- docx를 다시 저장하고 Word를 닫습니다.
- `./tools/doc-to-md.sh`를 실행합니다.
- 이 스크립트는 `nostarch/chapterXX.md` 를 생성합니다. 필요하다면
  `tools/doc-to-md.xsl`의 XSL을 조정한 뒤 `./tools/doc-to-md.sh`를 다시
  실행하세요.

## Graphviz dot 생성

책의 일부 다이어그램에는 [Graphviz](http://graphviz.org/)를 사용합니다. 해당
파일들의 소스는 `dot` 디렉터리에 있습니다. 예를 들어 `dot/trpl04-01.dot` 파일을
`svg`로 변환하려면 다음을 실행하세요.

```bash
$ dot dot/trpl04-01.dot -Tsvg > src/img/trpl04-01.svg
```

생성된 SVG에서 `svg` 요소의 `width`와 `height` 속성을 제거하고, `viewBox`
속성을 `0.00 0.00 1000.00 1000.00` 또는 이미지가 잘리지 않는 다른 값으로
설정합니다.

## GitHub Pages에 미리보기 게시

작업 중인 내용을 미리 보여 주기 위해 GitHub Pages에 게시하기도 합니다. 권장
절차는 다음과 같습니다.

- `pip install ghp-import`(또는 [pipx][pipx]를 사용해 `pipx install ghp-import`)를
  실행해 `ghp-import` 도구를 설치합니다.
- 프로젝트 루트에서 `tools/generate-preview.sh`를 실행합니다.

[pipx]: https://pipx.pypa.io/stable/#install-pipx
