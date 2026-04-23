## 환경 변수 다루기

`minigrep` 바이너리에 추가 기능 하나를 더해 개선하겠습니다. 사용자가 환경
변수로 켤 수 있는, 대소문자 구분 없는 검색 옵션입니다. 이 기능을 명령행 옵션
으로 만들어 사용자가 매번 적용하고 싶을 때마다 입력하게 할 수도 있지만, 대신
환경 변수로 만들면 사용자가 환경 변수를 한 번 설정하기만 하면 해당 터미널
세션에서 모든 검색이 대소문자 구분 없이 이루어지도록 할 수 있습니다.

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-a-failing-test-for-the-case-insensitive-search-function"></a>

### 대소문자 구분 없는 검색에 대한 실패하는 테스트 작성하기

먼저 환경 변수에 값이 있을 때 호출될 `search_case_insensitive`라는 새 함수를
`minigrep` 라이브러리에 추가합니다. TDD 프로세스를 계속 따를 것이므로 첫
단계는 다시 실패하는 테스트를 작성하는 것입니다. 새 `search_case_insensitive`
함수에 대한 새 테스트를 추가하고, 두 테스트의 차이를 분명히 하기 위해 예전
테스트의 이름을 `one_result`에서 `case_sensitive`로 바꾸겠습니다. Listing
12-20처럼요.

<Listing number="12-20" file-name="src/lib.rs" caption="곧 추가할 대소문자 구분 없는 함수에 대한 새 실패 테스트 추가하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

예전 테스트의 `contents`도 편집했음에 유의하세요. 대문자 _D_ 를 사용한
`"Duct tape."`라는 텍스트 줄을 추가했는데, 이는 대소문자를 구분해서 검색할
때 쿼리 `"duct"`와 일치하지 않아야 합니다. 예전 테스트를 이런 식으로 바꾸면,
이미 구현한 대소문자 구분 검색 기능을 실수로 망가뜨리지 않도록 하는 데
도움이 됩니다. 이 테스트는 지금도 통과해야 하며, 대소문자 구분 없는 검색을
작업하는 동안에도 계속 통과해야 합니다.

대소문자 _구분 없는_ 검색을 위한 새 테스트는 쿼리로 `"rUsT"`를 사용합니다.
곧 추가할 `search_case_insensitive` 함수에서 쿼리 `"rUsT"`는 대문자 _R_ 을
가진 `"Rust:"`를 포함하는 줄과 일치해야 하고, 쿼리와 대소문자가 다른 `"Trust me."`
줄과도 일치해야 합니다. 이것이 우리의 실패 테스트이며, `search_case_insensitive`
함수를 아직 정의하지 않았기 때문에 컴파일에 실패합니다. Listing 12-16에서
`search` 함수에 했던 것과 비슷하게, 항상 빈 벡터를 반환하는 뼈대 구현을
추가해 테스트가 컴파일되고 실패하도록 해 봐도 좋습니다.

### `search_case_insensitive` 함수 구현하기

Listing 12-21에 보인 `search_case_insensitive` 함수는 `search` 함수와 거의
같습니다. 유일한 차이는 `query`와 각 `line`을 소문자로 만든다는 점입니다.
그러면 입력 인수의 대소문자가 어떻든, 줄이 쿼리를 포함하는지 확인할 때 같은
대소문자가 됩니다.

<Listing number="12-21" file-name="src/lib.rs" caption="비교 전에 쿼리와 줄을 소문자로 만드는 `search_case_insensitive` 함수 정의하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

먼저 `query` 문자열을 소문자로 만들고 원래 `query`를 가리는(shadowing) 같은
이름의 새 변수에 저장합니다. 쿼리에 `to_lowercase`를 호출하는 것은 사용자의
쿼리가 `"rust"`든, `"RUST"`든, `"Rust"`든, `"rUsT"`든 마치 `"rust"`인 것처럼
처리하고 대소문자 구분 없이 다루기 위해 필요합니다. `to_lowercase`는 기본
유니코드를 처리하지만 100% 정확하지는 않습니다. 실제 애플리케이션을 작성한다면
여기서 좀 더 많은 작업을 하고 싶겠지만, 이 절은 유니코드가 아니라 환경 변수
에 관한 것이므로 여기서는 이 정도로 두겠습니다.

`to_lowercase`를 호출하면 기존 데이터를 참조하는 것이 아니라 새 데이터를
만들어 내므로, `query`는 이제 문자열 슬라이스가 아니라 `String`이라는 점에
유의하세요. 예를 들어 쿼리가 `"rUsT"`라고 해 봅시다. 그 문자열 슬라이스에는
우리가 사용할 수 있는 소문자 `u`나 `t`가 포함되어 있지 않으므로, `"rust"`를
담은 새 `String`을 할당해야 합니다. 이제 `contains` 메서드에 인수로 `query`를
전달할 때는 앰퍼샌드(`&`)를 붙여야 합니다. `contains`의 시그니처가 문자열
슬라이스를 받도록 정의되어 있기 때문입니다.

다음으로, 모든 문자를 소문자로 만들기 위해 각 `line`에 `to_lowercase` 호출을
추가합니다. `line`과 `query`를 소문자로 변환했으므로, 이제 쿼리의 대소문자가
어떻든 일치를 찾을 수 있습니다.

이 구현이 테스트를 통과하는지 봅시다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

좋습니다! 통과했습니다. 이제 `run` 함수에서 새 `search_case_insensitive` 함수를
호출합시다. 먼저 대소문자 구분과 구분 없음 사이를 전환할 구성 옵션을 `Config`
구조체에 추가합니다. 이 필드를 어디에서도 초기화하지 않기 때문에 이 필드를
추가하면 컴파일러 오류가 발생합니다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

불(Boolean)을 담는 `ignore_case` 필드를 추가했습니다. 다음으로, Listing 12-22
처럼 `run` 함수가 `ignore_case` 필드의 값을 확인하고, 그에 따라 `search`
함수를 호출할지 `search_case_insensitive` 함수를 호출할지 결정하도록 해야
합니다. 이 코드도 아직 컴파일되지 않습니다.

<Listing number="12-22" file-name="src/main.rs" caption="`config.ignore_case`의 값에 따라 `search` 또는 `search_case_insensitive` 호출하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

마지막으로 환경 변수를 확인해야 합니다. 환경 변수를 다루기 위한 함수는 표준
라이브러리의 `env` 모듈에 있으며, 이미 _src/main.rs_ 상단에서 스코프에
들어와 있습니다. Listing 12-23처럼, `env` 모듈의 `var` 함수를 사용해
`IGNORE_CASE`라는 환경 변수에 어떤 값이 설정되어 있는지 확인하겠습니다.

<Listing number="12-23" file-name="src/main.rs" caption="`IGNORE_CASE`라는 환경 변수에 값이 있는지 확인하기">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

여기서 새 변수 `ignore_case`를 만듭니다. 값을 설정하려면 `env::var` 함수를
호출하고 `IGNORE_CASE` 환경 변수의 이름을 전달합니다. `env::var` 함수는
`Result`를 반환합니다. 환경 변수가 어떤 값으로든 설정되어 있으면 환경 변수의
값을 담은 성공 `Ok` 배리언트가 될 것입니다. 환경 변수가 설정되어 있지 않으면
`Err` 배리언트를 반환합니다.

환경 변수가 설정되었는지 확인하기 위해 `Result`의 `is_ok` 메서드를 사용합니다.
설정되어 있으면 프로그램이 대소문자 구분 없는 검색을 수행해야 함을 의미합니다.
`IGNORE_CASE` 환경 변수가 아무 값으로도 설정되어 있지 않으면 `is_ok`는 `false`
를 반환하고, 프로그램은 대소문자 구분 검색을 수행합니다. 환경 변수의 _값_
자체는 신경 쓰지 않고 설정되었는지 여부만 중요하므로, `unwrap`, `expect`, 또는
`Result`에서 본 다른 메서드 대신 `is_ok`를 검사합니다.

`ignore_case` 변수의 값을 `Config` 인스턴스에 전달해, Listing 12-22에서 구현한
대로 `run` 함수가 그 값을 읽어 `search_case_insensitive` 또는 `search` 중
무엇을 호출할지 결정할 수 있도록 합니다.

한번 실행해 봅시다! 먼저 환경 변수가 설정되지 않은 상태에서, 모든 소문자로
된 _to_ 라는 단어를 포함한 줄과 일치해야 하는 쿼리 `to`로 프로그램을 실행
합니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

여전히 잘 동작하는 것 같습니다! 이제 같은 쿼리 `to`로, 하지만 `IGNORE_CASE`를
`1`로 설정하여 프로그램을 실행합시다.

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

PowerShell을 사용 중이라면, 환경 변수 설정과 프로그램 실행을 별도의 명령으로
실행해야 합니다.

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

그러면 `IGNORE_CASE`가 셸 세션이 끝날 때까지 유지됩니다. `Remove-Item`
cmdlet으로 해제할 수 있습니다.

```console
PS> Remove-Item Env:IGNORE_CASE
```

대문자가 있을 수 있는, _to_ 를 포함한 줄들을 얻어야 합니다.

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

훌륭합니다, _To_ 를 포함한 줄도 함께 얻었습니다! 이제 우리 `minigrep` 프로그램
은 환경 변수로 제어되는 대소문자 구분 없는 검색을 수행할 수 있습니다. 이제
여러분은 명령행 인수 또는 환경 변수로 설정된 옵션을 관리하는 방법을 압니다.

어떤 프로그램은 같은 구성에 대해 인수 _와_ 환경 변수를 모두 허용합니다. 그런
경우 프로그램은 둘 중 하나가 우선권을 가진다고 결정합니다. 스스로 할 수 있는
또 하나의 연습으로, 명령행 인수나 환경 변수로 대소문자 구분을 제어해 보세요.
프로그램이 대소문자 구분과 대소문자 무시 중 하나씩이 서로 다르게 설정되어
실행될 때 명령행 인수와 환경 변수 중 어느 쪽이 우선할지 결정해 보세요.

`std::env` 모듈에는 환경 변수를 다루기 위한 더 많은 유용한 기능이 있습니다.
어떤 것이 있는지 문서를 확인해 보세요.
