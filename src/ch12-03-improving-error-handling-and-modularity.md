## 모듈성과 오류 처리 개선을 위한 리팩터링

프로그램을 개선하기 위해 프로그램의 구조와 잠재적 오류 처리 방식과 관련된 네
가지 문제를 고치겠습니다. 첫째, 우리 `main` 함수는 이제 두 가지 작업을
수행합니다. 인수를 파싱하고 파일을 읽는 일입니다. 프로그램이 커질수록 `main`
함수가 다루는 별개 작업의 수는 늘어날 것입니다. 함수가 책임을 더 가질수록,
추론하기가 더 어려워지고, 테스트하기가 더 어려워지며, 그 부분을 깨뜨리지 않고
변경하기가 더 어려워집니다. 각 함수가 하나의 작업에 책임을 지도록 기능을
분리하는 것이 가장 좋습니다.

이 문제는 두 번째 문제와도 연결됩니다. `query`와 `file_path`는 프로그램의
구성 변수이지만, `contents` 같은 변수는 프로그램의 로직을 수행하는 데
사용됩니다. `main`이 길어질수록 스코프로 가져와야 하는 변수가 더 많아지고,
스코프에 변수가 많을수록 각 변수의 용도를 추적하기가 더 어려워집니다. 구성
변수들을 하나의 구조로 묶어 그 용도를 명확히 하는 것이 가장 좋습니다.

세 번째 문제는 파일 읽기에 실패했을 때 오류 메시지를 출력하는 데 `expect`를
사용했는데, 오류 메시지가 그냥 `Should have been able to read the file`만
출력한다는 점입니다. 파일 읽기는 여러 가지 방식으로 실패할 수 있습니다. 예를
들어 파일이 없을 수도 있고, 파일을 열 권한이 없을 수도 있습니다. 현재는
상황과 관계없이 모든 경우에 동일한 오류 메시지를 출력하므로, 사용자에게
아무 정보도 주지 않습니다!

넷째, 오류를 처리하는 데 `expect`를 사용하는데, 사용자가 충분한 인수를 주지
않고 프로그램을 실행하면 러스트로부터 `index out of bounds` 오류를 받게 되며
이는 문제를 명확히 설명해 주지 않습니다. 모든 오류 처리 코드가 한 곳에 있어서,
오류 처리 로직이 바뀌어야 할 때 미래의 유지보수자가 참고할 곳이 한 곳만
되도록 하는 것이 가장 좋습니다. 오류 처리 코드를 한 곳에 모으면, 최종
사용자에게 의미 있는 메시지를 출력하고 있음도 보장할 수 있습니다.

이 네 가지 문제를 프로젝트 리팩터링으로 해결해 봅시다.

<!-- Old headings. Do not remove or links may break. -->

<a id="separation-of-concerns-for-binary-projects"></a>

### 바이너리 프로젝트에서의 관심사 분리

여러 작업의 책임을 `main` 함수에 할당하는 구성적 문제는 많은 바이너리
프로젝트에서 흔합니다. 그 결과 많은 러스트 프로그래머는 `main` 함수가 커지기
시작할 때 바이너리 프로그램의 별개 관심사를 분리하는 것이 유용하다고 느낍니다.
이 과정은 다음 단계로 구성됩니다.

- 프로그램을 _main.rs_ 파일과 _lib.rs_ 파일로 분리하고, 프로그램의 로직을
  _lib.rs_ 로 옮긴다.
- 명령행 파싱 로직이 작다면 `main` 함수에 남아 있어도 된다.
- 명령행 파싱 로직이 복잡해지기 시작하면, `main` 함수에서 다른 함수나 타입으로
  추출한다.

이 과정을 거친 후 `main` 함수에 남아야 할 책임은 다음으로 제한되어야 합니다.

- 인수 값을 가지고 명령행 파싱 로직 호출하기
- 기타 구성 설정하기
- _lib.rs_ 의 `run` 함수 호출하기
- `run`이 오류를 반환하면 그 오류 처리하기

이 패턴은 관심사 분리에 관한 것입니다. _main.rs_ 는 프로그램 실행을 다루고,
_lib.rs_ 는 해당 작업의 모든 로직을 다룹니다. `main` 함수를 직접 테스트할
수는 없으므로, 이 구조는 프로그램의 모든 로직을 `main` 함수 밖으로 옮김으로써
테스트할 수 있게 해 줍니다. `main` 함수에 남은 코드는 읽는 것만으로 정확성을
검증할 수 있을 만큼 충분히 작을 것입니다. 이 과정을 따라 프로그램을 재작업해
봅시다.

#### 인수 파서 추출하기

인수 파싱 기능을 `main`이 호출할 함수로 추출하겠습니다. Listing 12-5는 새
함수 `parse_config`를 호출하는 `main` 함수의 새 시작 부분을 보여 줍니다.
`parse_config`는 _src/main.rs_ 에 정의할 것입니다.

<Listing number="12-5" file-name="src/main.rs" caption="`main`에서 `parse_config` 함수 추출하기">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

여전히 명령행 인수를 벡터로 모으지만, `main` 함수 안에서 인덱스 1의 인수 값을
`query` 변수에, 인덱스 2의 인수 값을 `file_path` 변수에 할당하는 대신 벡터
전체를 `parse_config` 함수에 전달합니다. 그러면 `parse_config` 함수가 어떤
인수가 어떤 변수에 들어갈지 결정하는 로직을 갖고, `main`으로 값을 돌려보냅니다.
여전히 `main`에서 `query`와 `file_path` 변수를 만들지만, `main`은 더 이상
명령행 인수와 변수가 어떻게 대응되는지 결정하는 책임을 지지 않습니다.

이 작은 프로그램에서는 이 재작업이 과해 보일 수 있지만, 우리는 작고 점진적인
단계로 리팩터링하고 있습니다. 이 변경을 한 뒤에는 프로그램을 다시 실행해 인수
파싱이 여전히 동작하는지 확인하세요. 문제가 발생했을 때 원인을 빠르게 파악할
수 있도록 진행 상황을 자주 확인하는 것은 좋은 습관입니다.

#### 구성 값 묶기

`parse_config` 함수를 더 개선하기 위해 또 하나의 작은 단계를 밟을 수 있습니다.
지금은 튜플을 반환하지만, 곧바로 그 튜플을 다시 개별 부분으로 쪼개고 있습니다.
이는 아마도 아직 올바른 추상화를 갖추지 못했다는 신호입니다.

개선의 여지를 보여 주는 또 다른 지표는 `parse_config`의 `config` 부분인데,
이는 우리가 반환하는 두 값이 관련되어 있으며 하나의 구성 값의 일부임을
암시합니다. 현재는 두 값을 튜플로 묶는 것 외에는 이 의미를 데이터 구조에서
전달하지 않고 있습니다. 대신 두 값을 하나의 구조체에 넣고 각 구조체 필드에
의미 있는 이름을 부여하겠습니다. 그렇게 하면 이 코드의 미래 유지보수자가
서로 다른 값들이 어떻게 관련되고 그 용도가 무엇인지 더 쉽게 이해할 수 있습니다.

Listing 12-6은 `parse_config` 함수의 개선을 보여 줍니다.

<Listing number="12-6" file-name="src/main.rs" caption="`Config` 구조체의 인스턴스를 반환하도록 `parse_config` 리팩터링하기">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

`query`와 `file_path`라는 필드를 가지도록 정의된 `Config`라는 구조체를
추가했습니다. `parse_config`의 시그니처는 이제 `Config` 값을 반환함을 나타
냅니다. `parse_config`의 본문에서, 이전에는 `args`의 `String` 값들을 참조하는
문자열 슬라이스를 반환했지만 이제는 `Config`가 소유된 `String` 값들을 담도록
정의합니다. `main`의 `args` 변수는 인수 값들의 소유자이며, `parse_config` 함수
가 빌리도록만 해 주고 있으므로, `Config`가 `args`의 값들의 소유권을 가져가려
하면 러스트의 빌림 규칙을 어기게 됩니다.

`String` 데이터를 다룰 방법은 여러 가지입니다. 가장 쉬운 방법은, 비록 다소
비효율적이지만 값들에 `clone` 메서드를 호출하는 것입니다. 그러면 `Config`
인스턴스가 소유할 데이터의 완전한 복사가 이루어지는데, 이는 문자열 데이터에
대한 참조를 저장하는 것보다 더 많은 시간과 메모리를 사용합니다. 그러나 데이터를
복제하면, 참조의 라이프타임을 관리하지 않아도 되므로 코드가 매우 직관적이
됩니다. 이런 상황에서 약간의 성능을 포기하고 단순함을 얻는 것은 할 만한 절충
입니다.

> ### `clone` 사용의 트레이드오프
>
> 많은 러스타시안(Rustaceans)은 런타임 비용 때문에 소유권 문제를 고치기 위해
> `clone`을 쓰는 것을 피하려는 경향이 있습니다. [13장][ch13]<!-- ignore -->에서
> 이런 종류의 상황에서 더 효율적인 방법을 어떻게 사용하는지 배웁니다. 하지만
> 지금은 진척을 계속 내기 위해 몇 개의 문자열을 복사해도 괜찮습니다. 이
> 복사는 한 번만 이루어지고, 파일 경로와 쿼리 문자열은 매우 작기 때문입니다.
> 첫 시도에서 코드를 과도하게 최적화하려 하기보다, 약간 비효율적이더라도
> 동작하는 프로그램을 갖는 편이 낫습니다. 러스트 경험이 쌓이면 처음부터 가장
> 효율적인 해법으로 시작하기가 더 쉬워지겠지만, 지금은 `clone`을 호출하는
> 것도 완전히 괜찮습니다.

`parse_config`가 반환한 `Config` 인스턴스를 `config`라는 변수에 넣도록 `main`을
수정했고, 이전에 별도의 `query`와 `file_path` 변수를 사용하던 코드를 대신
`Config` 구조체의 필드를 사용하도록 수정했습니다.

이제 우리 코드는 `query`와 `file_path`가 관련되어 있고, 그 용도가 프로그램의
동작 방식을 구성하는 것임을 더 명확히 전달합니다. 이 값들을 사용하는 모든
코드는 그것들을 용도에 맞게 이름 붙인 필드에서 `config` 인스턴스를 통해 찾을
수 있음을 알게 됩니다.

#### `Config`용 생성자 만들기

지금까지 명령행 인수 파싱을 책임지는 로직을 `main`에서 추출해 `parse_config`
함수에 넣었습니다. 그렇게 하니 `query`와 `file_path` 값이 관련되어 있고, 그
관계가 코드에 전달되어야 함을 볼 수 있었습니다. 그런 다음 `query`와
`file_path`의 관련된 용도에 이름을 붙이고 `parse_config` 함수로부터 값들의
이름을 구조체 필드 이름으로 돌려받을 수 있도록 `Config` 구조체를 추가했습니다.

이제 `parse_config` 함수의 용도가 `Config` 인스턴스를 만드는 것이므로,
`parse_config`를 평범한 함수에서 `Config` 구조체와 연관된 `new`라는 함수로
바꿀 수 있습니다. 이렇게 바꾸면 코드가 더 관용적이 됩니다. `String` 같은 표준
라이브러리의 타입 인스턴스는 `String::new`를 호출해 만들 수 있습니다.
마찬가지로, `parse_config`를 `Config`와 연관된 `new` 함수로 바꾸면
`Config::new`를 호출해 `Config` 인스턴스를 만들 수 있게 됩니다. Listing 12-7
은 필요한 변경 사항을 보여 줍니다.

<Listing number="12-7" file-name="src/main.rs" caption="`parse_config`를 `Config::new`로 바꾸기">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

`parse_config`를 호출하던 `main`을 `Config::new`를 호출하도록 수정했습니다.
`parse_config`의 이름을 `new`로 바꾸고 `impl` 블록으로 옮겨, `new` 함수를
`Config`와 연관시켰습니다. 이 코드를 다시 컴파일해 동작을 확인해 보세요.

### 오류 처리 고치기

이제 오류 처리 개선에 착수하겠습니다. `args` 벡터의 인덱스 1이나 2의 값을
접근하려 할 때 벡터에 항목이 세 개 미만이면 프로그램이 패닉한다는 점을 기억
하세요. 인수 없이 프로그램을 실행해 보세요. 결과는 다음과 같을 것입니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

`index out of bounds: the len is 1 but the index is 1` 줄은 프로그래머를 위한
오류 메시지입니다. 대신 어떻게 해야 하는지 최종 사용자가 이해하는 데 도움이
되지 않습니다. 지금 고쳐 봅시다.

#### 오류 메시지 개선하기

Listing 12-8에서는 인덱스 1과 2에 접근하기 전에 슬라이스의 길이가 충분한지
검증하는 검사를 `new` 함수에 추가합니다. 슬라이스가 충분히 길지 않으면
프로그램은 패닉하고 더 나은 오류 메시지를 표시합니다.

<Listing number="12-8" file-name="src/main.rs" caption="인수 개수에 대한 검사 추가하기">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

이 코드는 Listing 9-13에서 작성한 [`Guess::new` 함수][ch9-custom-types]<!-- ignore -->
와 비슷합니다. 거기서는 `value` 인수가 유효한 값의 범위를 벗어날 때 `panic!`을
호출했습니다. 여기서는 값의 범위를 검사하는 대신 `args`의 길이가 최소 `3`인지
검사하고, 그 조건이 충족되었다는 가정 아래 함수의 나머지가 동작할 수 있습니다.
`args`의 항목이 세 개 미만이면 이 조건은 `true`가 되고, 프로그램을 즉시 종료
하기 위해 `panic!` 매크로를 호출합니다.

`new`의 이 몇 줄 추가와 함께 다시 인수 없이 프로그램을 실행해 오류가 어떻게
보이는지 확인해 봅시다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

이 출력이 더 낫습니다. 이제 합리적인 오류 메시지가 있습니다. 그러나 사용자에게
주고 싶지 않은 불필요한 정보도 함께 있습니다. 어쩌면 Listing 9-13에서 사용한
기법이 여기서는 최선이 아닐지도 모릅니다. [9장에서 논의했듯이][ch9-error-guidelines]<!-- ignore -->
`panic!` 호출은 사용 문제보다 프로그래밍 문제에 더 적합합니다. 대신 9장에서
배운 다른 기법인 [`Result` 반환하기][ch9-result]<!-- ignore -->를 사용하겠습니다.
`Result`는 성공이나 오류를 나타냅니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### `panic!` 호출 대신 `Result` 반환하기

대신 성공 시에는 `Config` 인스턴스를 담고, 오류 시에는 문제를 설명하는 값을
담는 `Result` 값을 반환할 수 있습니다. 또한 함수 이름을 `new`에서 `build`로
바꾸려고 합니다. 많은 프로그래머가 `new` 함수는 절대 실패하지 않는다고
기대하기 때문입니다. `Config::build`가 `main`과 통신할 때 `Result` 타입을
사용해 문제가 있음을 알릴 수 있습니다. 그런 다음 `main`을 수정해 `panic!`
호출이 일으키는 `thread 'main'`과 `RUST_BACKTRACE` 같은 주변 텍스트 없이,
사용자를 위한 더 실용적인 오류로 `Err` 배리언트를 변환할 수 있습니다.

Listing 12-9는 이제 우리가 `Config::build`라고 부를 함수의 반환 값에 가해야
할 변경과 `Result`를 반환하기 위한 함수 본문을 보여 줍니다. `main`도 다음
리스트에서 업데이트할 때까지는 컴파일되지 않는다는 점에 유의하세요.

<Listing number="12-9" file-name="src/main.rs" caption="`Config::build`에서 `Result` 반환하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

우리 `build` 함수는 성공 시에 `Config` 인스턴스를, 오류 시에 문자열 리터럴을
담은 `Result`를 반환합니다. 오류 값은 항상 `'static` 라이프타임을 가진 문자열
리터럴이 될 것입니다.

함수 본문에는 두 가지 변경을 가했습니다. 사용자가 충분한 인수를 주지 않았을
때 `panic!`을 호출하는 대신 이제 `Err` 값을 반환하고, `Config` 반환 값을 `Ok`
로 감쌌습니다. 이러한 변경은 함수를 새 타입 시그니처에 부합하도록 만듭니다.

`Config::build`가 `Err` 값을 반환하면, `main` 함수가 `build` 함수가 반환한
`Result` 값을 처리하고 오류 상황에서 프로세스를 더 깔끔하게 종료할 수 있게
됩니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="calling-confignew-and-handling-errors"></a>

#### `Config::build` 호출과 오류 처리

오류 상황을 처리하고 사용자 친화적인 메시지를 출력하려면, Listing 12-10처럼
`Config::build`가 반환하는 `Result`를 처리하도록 `main`을 수정해야 합니다.
또한 0이 아닌 오류 코드로 명령행 도구를 종료하는 책임도 `panic!`에서 가져와
손수 구현하겠습니다. 0이 아닌 종료 상태는 우리 프로그램을 호출한 프로세스에게
프로그램이 오류 상태로 종료되었음을 알리는 관례입니다.

<Listing number="12-10" file-name="src/main.rs" caption="`Config` 빌드가 실패하면 오류 코드와 함께 종료하기">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

이 리스트에서는 아직 자세히 다루지 않은 메서드인 `unwrap_or_else`를 사용했는데,
이는 표준 라이브러리가 `Result<T, E>`에 정의한 것입니다. `unwrap_or_else`를
사용하면 `panic!`이 아닌 사용자 정의 오류 처리를 정의할 수 있습니다. `Result`
가 `Ok` 값이면 이 메서드의 동작은 `unwrap`과 비슷해서, `Ok`가 감싸고 있는
내부 값을 반환합니다. 하지만 값이 `Err` 값이면 이 메서드는 클로저의 코드를
호출합니다. 클로저는 우리가 정의해 `unwrap_or_else`에 인수로 전달하는 익명
함수입니다. 클로저는 [13장][ch13]<!-- ignore -->에서 더 자세히 다룹니다.
지금은 `unwrap_or_else`가 `Err`의 내부 값(이 경우 Listing 12-9에서 추가한
정적 문자열 `"not enough arguments"`)을 수직 파이프 사이에 나타나는 `err`
인수로 우리 클로저에 전달한다는 것만 알면 됩니다. 그러면 클로저의 코드는
실행될 때 `err` 값을 사용할 수 있습니다.

표준 라이브러리의 `process`를 스코프로 가져오기 위해 새 `use` 줄을 추가했습니다.
오류 상황에서 실행되는 클로저의 코드는 단 두 줄입니다. `err` 값을 출력한 다음
`process::exit`를 호출합니다. `process::exit` 함수는 프로그램을 즉시 중지하고
종료 상태 코드로 전달된 숫자를 반환합니다. 이는 Listing 12-8에서 사용한
`panic!` 기반 처리와 비슷하지만, 이제는 모든 추가 출력을 얻지 않습니다. 한번
해 봅시다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

좋습니다! 이 출력은 사용자에게 훨씬 친화적입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="extracting-logic-from-the-main-function"></a>

### `main`에서 로직 추출하기

구성 파싱 리팩터링을 끝냈으므로, 이제 프로그램 로직으로 넘어가겠습니다.
[“바이너리 프로젝트에서의 관심사 분리”](#separation-of-concerns-for-binary-projects)<!-- ignore -->
에서 말했듯이, 구성 설정이나 오류 처리에 관여하지 않는 `main` 함수의 모든
로직을 담을 `run`이라는 함수를 추출하겠습니다. 끝나면 `main` 함수는 간결하고
읽기만 해도 검증할 수 있게 되며, 나머지 모든 로직에 대해 테스트를 작성할 수
있게 됩니다.

Listing 12-11은 `run` 함수를 추출하는 작고 점진적인 개선을 보여 줍니다.

<Listing number="12-11" file-name="src/main.rs" caption="나머지 프로그램 로직을 담는 `run` 함수 추출하기">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

`run` 함수는 이제 파일 읽기부터 시작해 `main`에 남아 있던 모든 로직을 담고
있습니다. `run` 함수는 `Config` 인스턴스를 인수로 받습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-errors-from-the-run-function"></a>

#### `run`에서 오류 반환하기

남아 있는 프로그램 로직을 `run` 함수로 분리했으니, Listing 12-9에서 `Config::build`
에 대해 한 것처럼 오류 처리를 개선할 수 있습니다. `expect`를 호출해 프로그램을
패닉시키는 대신, `run` 함수는 뭔가 잘못됐을 때 `Result<T, E>`를 반환할 것입니다.
이를 통해 오류 처리 로직을 사용자 친화적인 방식으로 `main`에 더 모을 수 있습니다.
Listing 12-12는 `run`의 시그니처와 본문에 가해야 할 변경을 보여 줍니다.

<Listing number="12-12" file-name="src/main.rs" caption="`run` 함수가 `Result`를 반환하도록 변경하기">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

여기서 중요한 세 가지 변경을 가했습니다. 첫째, `run` 함수의 반환 타입을
`Result<(), Box<dyn Error>>`로 바꿨습니다. 이 함수는 이전에 유닛 타입 `()`를
반환했고, `Ok` 경우의 반환 값으로 그 타입을 유지합니다.

오류 타입으로는 트레이트 객체 `Box<dyn Error>`를 사용했습니다(그리고 파일
맨 위의 `use` 구문으로 `std::error::Error`를 스코프로 가져왔습니다). 트레이트
객체는 [18장][ch18]<!-- ignore -->에서 다룹니다. 지금은 `Box<dyn Error>`가
함수가 `Error` 트레이트를 구현하는 어떤 타입을 반환하지만, 반환 값이 구체적
으로 어떤 타입이 될지는 명시할 필요가 없다는 의미임을 알면 됩니다. 이를 통해
서로 다른 오류 상황에서 서로 다른 타입의 오류 값을 반환할 수 있는 유연성이
생깁니다. `dyn` 키워드는 _dynamic_ 의 약자입니다.

둘째, [9장][ch9-question-mark]<!-- ignore -->에서 이야기한 대로 `expect` 호출을
`?` 연산자로 바꿨습니다. 오류 시 `panic!`하는 대신 `?`는 호출자가 처리하도록
현재 함수에서 오류 값을 반환합니다.

셋째, 이제 `run` 함수는 성공 시 `Ok` 값을 반환합니다. 시그니처에서 `run`
함수의 성공 타입을 `()`로 선언했으므로, 유닛 타입 값을 `Ok` 값으로 감싸야
합니다. 이 `Ok(())` 문법은 처음에는 약간 이상해 보일 수 있습니다. 그러나
이런 식으로 `()`를 쓰는 것은 `run`을 부작용을 위해서만 호출하며 필요한 값을
반환하지 않음을 나타내는 관용적 방법입니다.

이 코드를 실행하면 컴파일되지만 경고가 표시됩니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

러스트는 우리 코드가 `Result` 값을 무시했으며, 그 `Result` 값이 오류 발생을
나타낼 수 있다고 말합니다. 그러나 우리는 오류가 있었는지 확인하지 않고 있고,
컴파일러는 여기에 오류 처리 코드를 두려던 것 아니냐고 상기시켜 줍니다! 이제
그 문제를 바로잡읍시다.

#### `main`에서 `run`이 반환한 오류 처리하기

Listing 12-10에서 `Config::build`에 사용한 것과 비슷한 기법으로 오류를
검사하고 처리하겠습니다. 다만 약간의 차이가 있습니다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

`run`이 `Err` 값을 반환하는지 검사하고, 그렇다면 `process::exit(1)`을 호출하기
위해 `unwrap_or_else`가 아니라 `if let`을 사용합니다. `run` 함수는 `Config::build`
가 `Config` 인스턴스를 반환하는 것과 같은 방식으로 `unwrap`하고 싶은 값을
반환하지 않습니다. `run`은 성공 시 `()`를 반환하므로 오류 감지에만 신경쓰고,
언래핑된 값(`()`일 뿐)을 반환하기 위한 `unwrap_or_else`는 필요하지 않습니다.

두 경우 모두 `if let`과 `unwrap_or_else` 함수의 본문은 같습니다. 오류를 출력
하고 종료합니다.

### 라이브러리 크레이트로 코드 분리하기

`minigrep` 프로젝트가 지금까지 잘 흘러가고 있습니다! 이제 _src/main.rs_ 파일을
분리하고 일부 코드를 _src/lib.rs_ 파일에 넣겠습니다. 그러면 코드를 테스트할
수 있고 _src/main.rs_ 파일은 책임이 더 적어집니다.

_src/main.rs_ 가 아니라 _src/lib.rs_ 에 텍스트 검색을 책임지는 코드를
정의합시다. 그러면 우리(또는 우리 `minigrep` 라이브러리를 사용하는 다른
누군가)가 `minigrep` 바이너리뿐만 아니라 더 많은 맥락에서 검색 함수를 호출할
수 있게 됩니다.

먼저 Listing 12-13처럼 _src/lib.rs_ 에 `search` 함수 시그니처를 정의하고
본문에는 `unimplemented!` 매크로를 호출하게 하세요. 구현을 채울 때 시그니처를
더 자세히 설명하겠습니다.

<Listing number="12-13" file-name="src/lib.rs" caption="*src/lib.rs*에 `search` 함수 정의하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs}}
```

</Listing>

`search`를 라이브러리 크레이트의 공개 API의 일부로 지정하기 위해 함수 정의에
`pub` 키워드를 사용했습니다. 이제 바이너리 크레이트에서 사용할 수 있고 테스트
할 수 있는 라이브러리 크레이트를 갖게 되었습니다!

이제 _src/lib.rs_ 에 정의된 코드를 _src/main.rs_ 의 바이너리 크레이트 스코프로
가져와서 호출해야 합니다. Listing 12-14처럼 말이죠.

<Listing number="12-14" file-name="src/main.rs" caption="*src/main.rs*에서 `minigrep` 라이브러리 크레이트의 `search` 함수 사용하기">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

`use minigrep::search` 줄을 추가해 라이브러리 크레이트의 `search` 함수를
바이너리 크레이트 스코프로 가져옵니다. 그런 다음 `run` 함수에서 파일 내용을
출력하는 대신 `search` 함수를 호출하고 `config.query` 값과 `contents`를 인수로
전달합니다. 그 후 `run`은 `search`가 반환한, 쿼리와 일치한 각 줄을 출력하기
위해 `for` 루프를 사용합니다. 이 시점에 프로그램이 오류가 없다면 검색 결과만
출력하도록 쿼리와 파일 경로를 표시하던 `main` 함수의 `println!` 호출들도
제거하는 것이 좋습니다.

검색 함수가 결과를 내기 전에 모든 결과를 반환할 벡터에 모은다는 점에 유의하
세요. 이 구현은 큰 파일을 검색할 때 결과가 발견되는 즉시 출력되지 않으므로
결과 표시가 느릴 수 있습니다. 13장에서 이터레이터를 사용해 이를 고치는 가능한
방법을 논의할 것입니다.

휴! 일이 많았지만, 앞으로의 성공을 위한 기반을 마련했습니다. 이제 오류를
처리하기가 훨씬 쉽고, 코드를 더 모듈화했습니다. 이제부터 거의 모든 작업은
_src/lib.rs_ 에서 이루어집니다.

이 새롭게 얻은 모듈성을 활용해, 예전 코드에서는 어려웠지만 새 코드에서는
쉬운 무언가를 해 봅시다. 테스트를 작성할 것입니다!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
