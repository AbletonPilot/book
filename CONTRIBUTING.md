# 기여하기

여러분의 도움을 환영합니다! 이 책에 관심을 가져 주셔서 감사합니다.

## 어디를 수정해야 하나요

모든 편집은 `src` 디렉터리에서 이루어져야 합니다.

`nostarch` 디렉터리는 인쇄판 출판사에 편집 내용을 전달하기 위한 스냅샷을 담고
있습니다. 이 스냅샷 파일들은 어떤 수정 사항이 출판사에 전달되었는지를 반영하기
때문에, No Starch에 편집 내용을 전달할 때에만 업데이트됩니다. **`nostarch`
디렉터리의 파일을 변경하는 풀 리퀘스트는 제출하지 말아 주세요. 닫히게 됩니다.**

저장소 내 러스트 코드의 표준 포매팅은 [`rustfmt`][rustfmt]로, 마크다운 소스와
러스트 이외의 코드에 대한 표준 포매팅은 [`dprint`][dprint]로 적용합니다.

[rustfmt]: https://github.com/rust-lang/rustfmt
[dprint]: https://dprint.dev

러스트 툴체인이 설치되어 있다면 `rustfmt`는 일반적으로 이미 설치되어 있을 것입니다.
어떤 이유로 `rustfmt`가 없다면 다음 명령으로 추가할 수 있습니다.

```sh
rustup component add rustfmt
```

`dprint`를 설치하려면 다음 명령을 실행하거나,

```sh
cargo install dprint
```

`dprint` 웹사이트의 [instructions][install-dprint]를 따라 설치할 수 있습니다.

[install-dprint]: https://dprint.dev/install/

러스트 코드를 포맷팅하려면 `rustfmt <path to file>`을 실행하면 되고, 그 외의
파일을 포맷팅하려면 `dprint fmt <path to file>`을 실행하면 됩니다. 많은 텍스트
에디터에서도 `rustfmt`와 `dprint`를 네이티브로 지원하거나 확장 기능을 통해
지원합니다.

## 수정 사항 확인하기

이 책은 러스트의 릴리스 트레인을 따라갑니다. 그러므로
https://doc.rust-lang.org/stable/book 에서 어떤 문제를 발견하셨다면, 이
저장소의 `main` 브랜치에서는 이미 수정되어 있지만 nightly → beta → stable 까지
아직 반영되지 않았을 수 있습니다. 이슈를 신고하시기 전에 본 저장소의 `main`
브랜치를 먼저 확인해 주세요.

특정 파일의 히스토리를 살펴보면, 어떤 이슈가 이미 수정되었는지 아닌지 확인하는 데에도
도움이 됩니다.

새 이슈를 신고하거나 새 PR을 열기 전에, 열려 있거나 닫힌 이슈 및 PR도 함께 검색해
주시기 바랍니다.

## 라이선스

이 저장소는 러스트 본체와 동일한 MIT/Apache2 라이선스를 따릅니다. 각 라이선스의
전문은 저장소의 `LICENSE-*` 파일에서 확인할 수 있습니다.

## 행동 강령

러스트 프로젝트에는 본 서브 프로젝트를 포함한 모든 프로젝트에 적용되는 [행동
강령](http://rust-lang.org/policies/code-of-conduct)이 있습니다. 존중해 주세요!

## 기대치에 대한 안내

이 책은 [인쇄본][nostarch]으로도 출판되고 있고, 가능하면 온라인 버전을 인쇄본과
가깝게 유지하려고 하기 때문에, 이슈나 풀 리퀘스트에 대한 대응이 일반 프로젝트보다
오래 걸릴 수 있습니다.

[nostarch]: https://nostarch.com/rust-programming-language-2nd-edition

지금까지는 [러스트 에디션](https://doc.rust-lang.org/edition-guide/)에 맞춰
대규모 개정 작업을 진행해 왔습니다. 그 사이사이에는 오류 수정만 반영합니다. 여러분의
이슈나 풀 리퀘스트가 엄밀한 의미의 오류 수정이 아니라면, 다음 대규모 개정 작업이
진행될 때까지 보류될 수 있습니다(몇 달에서 몇 년 단위로 걸릴 수 있습니다). 양해
부탁드립니다!

## 도움이 필요한 작업(Help wanted)

책을 대량으로 읽거나 쓰는 작업이 아닌 형태로 기여하고 싶다면, [E-help-wanted
라벨이 붙은 열려 있는 이슈][help-wanted]를 살펴보세요. 본문, 러스트 코드, 프런트엔드
코드, 또는 책을 좀 더 효율적이거나 개선되게 해 줄 셸 스크립트에 대한 작은 수정
작업들이 포함되어 있습니다!

[help-wanted]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3AE-help-wanted

## 번역

번역 작업에 도움을 주실 분을 환영합니다! 현재 진행 중인 작업에 참여하려면
[Translations] 라벨을 살펴보세요. 새로운 언어로 번역을 시작하려면 새 이슈를 열어
주세요. 여러 언어를 본 저장소에 병합하기 전에 [mdbook support]를 기다리고
있지만, 그 전에도 자유롭게 시작하셔도 괜찮습니다!

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang/mdBook/issues/5
