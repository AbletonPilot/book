## 명령행 인수 받기

여느 때처럼 `cargo new`로 새 프로젝트를 만듭시다. 시스템에 이미 있을 수 있는
`grep` 도구와 구별하기 위해 프로젝트 이름을 `minigrep`으로 하겠습니다.

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

첫 번째 작업은 `minigrep`이 두 개의 명령행 인수, 즉 파일 경로와 검색할 문자열을
받도록 하는 것입니다. 즉 `cargo run`으로 프로그램을 실행하되 이어지는 인수가
`cargo`가 아닌 우리 프로그램의 것임을 알리는 하이픈 두 개, 검색할 문자열,
검색할 파일 경로를 다음과 같이 전달할 수 있도록 하고 싶습니다.

```console
$ cargo run -- searchstring example-filename.txt
```

지금은 `cargo new`가 생성한 프로그램이 우리가 주는 인수를 처리할 수 없습니다.
[crates.io](https://crates.io/)의 기존 라이브러리 중에는 명령행 인수를 받는
프로그램 작성을 돕는 것들이 있지만, 지금 이 개념을 배우는 중이니 직접 이
기능을 구현해 봅시다.

### 인수 값 읽기

우리가 전달하는 명령행 인수의 값을 `minigrep`이 읽을 수 있게 하려면, 러스트
표준 라이브러리가 제공하는 `std::env::args` 함수가 필요합니다. 이 함수는
`minigrep`에 전달된 명령행 인수의 이터레이터를 반환합니다. 이터레이터는
[13장][ch13]<!-- ignore -->에서 전반적으로 다룰 것입니다. 지금은 이터레이터에
관해 두 가지만 알면 됩니다. 이터레이터는 일련의 값을 생성하며, 이터레이터에
`collect` 메서드를 호출하면 이터레이터가 생성하는 모든 요소를 담은 벡터 같은
컬렉션으로 만들 수 있습니다.

Listing 12-1의 코드는 `minigrep` 프로그램이 전달된 명령행 인수를 모두 읽어
벡터로 모으도록 합니다.

<Listing number="12-1" file-name="src/main.rs" caption="명령행 인수를 벡터로 모아 출력하기">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

먼저 `use` 구문으로 `std::env` 모듈을 스코프로 가져와서 그 안의 `args` 함수를
사용할 수 있게 합니다. `std::env::args` 함수가 두 단계의 모듈 안에 중첩되어
있음에 유의하세요. [7장][ch7-idiomatic-use]<!-- ignore -->에서 다룬 것처럼,
원하는 함수가 두 단계 이상의 모듈 안에 있을 때는 함수가 아니라 부모 모듈을
스코프로 가져오기로 했습니다. 그렇게 하면 `std::env`의 다른 함수들도 쉽게
사용할 수 있습니다. 또한 `use std::env::args`를 추가하고 그냥 `args`로 함수를
호출하는 것보다 모호함이 덜합니다. 왜냐하면 `args`는 현재 모듈에 정의된
함수로 오해되기 쉽기 때문입니다.

> ### `args` 함수와 유효하지 않은 유니코드
>
> 인수 중 하나라도 유효하지 않은 유니코드를 포함하면 `std::env::args`는
> 패닉함에 유의하세요. 프로그램이 유효하지 않은 유니코드를 포함한 인수를
> 받아야 한다면 `std::env::args_os`를 대신 사용하세요. 이 함수는 `String` 값
> 대신 `OsString` 값을 만들어 내는 이터레이터를 반환합니다. 여기서는 단순함을
> 위해 `std::env::args`를 선택했는데, `OsString` 값은 플랫폼마다 다르고 `String`
> 값보다 다루기가 더 복잡하기 때문입니다.

`main`의 첫 줄에서 `env::args`를 호출하고, 바로 이어서 `collect`를 사용해
이터레이터가 만들어 내는 모든 값을 담은 벡터로 바꿉니다. `collect` 함수는 여러
종류의 컬렉션을 만드는 데 사용할 수 있으므로, `args`의 타입을 명시적으로
애너테이트해 문자열의 벡터를 원함을 지정합니다. 러스트에서 타입을 애너테이트
해야 할 일은 매우 드물지만, `collect`는 러스트가 어떤 종류의 컬렉션을 원하는지
추론할 수 없기 때문에 자주 애너테이트해야 하는 함수 중 하나입니다.

마지막으로 디버그 매크로를 사용해 벡터를 출력합니다. 인수 없이, 그리고 인수
두 개와 함께 코드를 실행해 봅시다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

벡터의 첫 번째 값이 `"target/debug/minigrep"` 으로 우리 바이너리의 이름임에
유의하세요. 이것은 C에서 인수 목록의 동작과 일치하며, 프로그램이 실행 중에
자신을 호출하는 데 사용된 이름을 사용할 수 있게 해 줍니다. 메시지에 출력하고
싶거나 프로그램 호출 시 사용된 명령행 별칭에 따라 프로그램의 동작을 바꾸고
싶을 때 프로그램 이름에 접근할 수 있는 것은 종종 편리합니다. 하지만 이 장의
목적상 이것은 무시하고 우리가 필요로 하는 두 인수만 저장하겠습니다.

### 인수 값 변수에 저장하기

프로그램은 현재 명령행 인수로 지정된 값에 접근할 수 있습니다. 이제 두 인수의
값을 변수에 저장해, 프로그램의 나머지 부분에서 그 값들을 사용할 수 있게 해야
합니다. Listing 12-2에서 그렇게 합니다.

<Listing number="12-2" file-name="src/main.rs" caption="쿼리 인수와 파일 경로 인수를 담을 변수 만들기">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

벡터를 출력했을 때 보았듯이 프로그램 이름이 `args[0]`의 첫 번째 값을 차지하므로
인수는 인덱스 1부터 시작합니다. `minigrep`이 받는 첫 번째 인수는 검색할 문자열
이므로, 첫 번째 인수에 대한 참조를 `query` 변수에 넣습니다. 두 번째 인수는
파일 경로이므로, 두 번째 인수에 대한 참조를 `file_path` 변수에 넣습니다.

코드가 의도대로 동작함을 확인하기 위해 이 변수들의 값을 임시로 출력합니다.
`test`와 `sample.txt` 인수로 이 프로그램을 다시 실행해 봅시다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

좋습니다, 프로그램이 동작합니다! 필요한 인수 값들이 올바른 변수에 저장되고
있습니다. 나중에는 사용자가 아무 인수도 주지 않는 경우 등 잠재적 오류 상황을
다루기 위한 오류 처리를 추가하겠습니다. 지금은 그 상황은 무시하고, 대신
파일 읽기 기능을 추가하는 데 집중해 봅시다.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
