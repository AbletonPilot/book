## 변수와 가변성

[“변수에 값 저장하기”][storing-values-with-variables]<!-- ignore --> 절에서
언급했듯, 변수는 기본적으로 불변(immutable)입니다. 이는 러스트가 제공하는 안전성과
손쉬운 동시성을 활용하는 방식으로 코드를 작성하도록 러스트가 여러분을 슬쩍 밀어
주는 여러 장치 중 하나입니다. 하지만 변수를 가변(mutable)으로 만드는 선택지도
여전히 가지고 있습니다. 러스트가 왜 불변성을 선호하도록 장려하는지, 그리고 그것을
어떻게 하는지, 또 언제는 왜 가변을 선택하고 싶을 수 있는지 살펴봅시다.

변수가 불변이면, 어떤 값이 이름에 한 번 바인딩된 뒤에는 그 값을 바꿀 수 없습니다.
이를 보여 주기 위해, *projects* 디렉터리에서 `cargo new variables`로 *variables*
라는 이름의 새 프로젝트를 생성해 보세요.

그런 다음, 새로 만든 *variables* 디렉터리에서 *src/main.rs*를 열고 그 내용을 아직
컴파일되지 않는 다음 코드로 바꾸세요.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

`cargo run`으로 저장하고 프로그램을 실행하세요. 다음 출력에서 볼 수 있듯 불변성에
관한 에러 메시지를 받게 될 것입니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

이 예제는 컴파일러가 프로그램의 에러를 찾는 데 어떻게 도움을 주는지 보여 줍니다.
컴파일러 에러는 때론 답답할 수 있지만, 실제로는 여러분의 프로그램이 아직 원하는
바를 안전하게 수행하고 있지 못하다는 것을 의미할 뿐입니다. 여러분이 실력 없는
프로그래머라는 뜻이 *아닙니다*! 경험 많은 러스타시안도 여전히 컴파일러 에러를
받습니다.

에러 메시지 `` cannot assign twice to immutable variable `x` ``를 받은 이유는,
불변 변수 `x`에 두 번째 값을 할당하려고 했기 때문입니다.

불변으로 지정된 값을 바꾸려 했을 때 컴파일 타임 에러가 발생한다는 사실은 중요합니다.
바로 이런 상황이 버그로 이어질 수 있기 때문입니다. 코드의 한 부분은 어떤 값이 절대
바뀌지 않는다는 가정하에 동작하고, 다른 부분에서 그 값을 바꾼다면, 첫 번째 부분은
설계된 대로 동작하지 않을 수 있습니다. 특히 두 번째 코드가 *어쩌다 한 번씩만* 값을
바꿀 때, 이런 유형의 버그의 원인은 사후에 추적하기가 어렵습니다. 러스트 컴파일러는
여러분이 어떤 값이 바뀌지 않는다고 선언하면 정말 바뀌지 않도록 보장해 주기 때문에,
여러분이 직접 그것을 추적할 필요가 없습니다. 덕분에 코드를 추론하기가 더 쉬워집니다.

그렇지만 가변성은 매우 유용할 수 있고, 코드를 작성하는 것을 더 편리하게 만들어
주기도 합니다. 변수는 기본적으로 불변이지만, [2장][storing-values-with-variables]
<!-- ignore -->에서 했던 것처럼 변수 이름 앞에 `mut`를 붙여 가변으로 만들 수 있습
니다. `mut`를 추가하는 것은 또한 이 변수의 값이 코드의 다른 부분에서 바뀔 것이라는
의도를 미래의 코드 독자에게 전달해 주기도 합니다.

예를 들어, *src/main.rs*를 다음과 같이 바꿔 봅시다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

이제 프로그램을 실행하면 다음과 같은 결과를 얻습니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

`mut`를 사용하면 `x`에 바인딩된 값을 `5`에서 `6`으로 바꿀 수 있게 됩니다. 가변성을
사용할지 여부를 결정하는 것은 결국 여러분의 몫이며, 그 상황에서 무엇이 가장
명확하다고 생각하는지에 달려 있습니다.

<!-- Old headings. Do not remove or links may break. -->
<a id="constants"></a>

### 상수 선언하기

불변 변수와 마찬가지로, *상수(constant)*는 이름에 바인딩되고 바꿀 수 없는 값이지만,
상수와 변수 사이에는 몇 가지 차이점이 있습니다.

우선, 상수에는 `mut`를 사용할 수 없습니다. 상수는 단순히 기본적으로 불변인 것이
아니라, 항상 불변입니다. 상수는 `let` 키워드 대신 `const` 키워드로 선언하고, 값의
타입을 *반드시* 명시해야 합니다. 타입과 타입 명시에 대해서는 다음 절
[“데이터 타입”][data-types]<!-- ignore -->에서 다룰 예정이니, 지금은 세부 사항에
대해 걱정하지 마세요. 타입을 항상 명시해야 한다는 것만 알아 두세요.

상수는 전역 스코프를 포함해 어떤 스코프에서도 선언할 수 있으며, 코드의 여러 부분이
알아야 하는 값에 유용합니다.

마지막 차이점은, 상수는 런타임에만 계산될 수 있는 값의 결과가 아니라 오직 상수
표현식으로만 지정할 수 있다는 것입니다.

다음은 상수 선언의 예입니다.

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

상수의 이름은 `THREE_HOURS_IN_SECONDS`이고, 값은 60(1분의 초)에 60(1시간의 분)과
3(이 프로그램에서 세고 싶은 시간의 수)을 곱한 결과로 설정됩니다. 상수에 대한
러스트의 네이밍 관례는 모두 대문자로 쓰고 단어 사이는 밑줄로 구분하는 것입니다.
컴파일러는 컴파일 타임에 제한된 연산들을 평가할 수 있으므로, 우리는 이 상수를
10,800이라는 값으로 바로 설정하는 대신, 더 이해하고 검증하기 쉬운 방식으로 값을
적어 둘 수 있습니다. 상수를 선언할 때 어떤 연산이 사용 가능한지에 대한 자세한
정보는 [러스트 레퍼런스의 상수 평가(constant evaluation) 절][const-eval]을 참고
하세요.

상수는 선언된 스코프 안에서 프로그램이 실행되는 동안 내내 유효합니다. 이 속성 덕분에
상수는 게임에서 어떤 플레이어든 얻을 수 있는 최대 점수나, 빛의 속도처럼 애플리케이션
도메인의 값 중 프로그램의 여러 부분이 알아야 할 법한 값에 유용합니다.

프로그램 전반에서 사용되는 하드코딩된 값에 상수로 이름을 붙이는 것은, 그 값의
의미를 이후의 코드 유지보수자에게 전달하는 데 유용합니다. 또한 앞으로 하드코딩된
값을 업데이트해야 할 때 바꿔야 할 곳을 코드 내의 한 지점으로 좁혀 주기도 합니다.

### 섀도잉

[2장][comparing-the-guess-to-the-secret-number]<!-- ignore -->의 숫자 맞히기 게임
튜토리얼에서 보았듯, 이전 변수와 같은 이름을 가진 새 변수를 선언할 수 있습니다.
러스타시안들은 이를 두고 첫 번째 변수가 두 번째 변수에 의해 *섀도잉(shadowed)*
된다고 말합니다. 즉, 해당 변수의 이름을 사용할 때 컴파일러가 보게 되는 것은 두
번째 변수라는 뜻입니다. 결과적으로 두 번째 변수가 첫 번째 변수를 가리며, 자신이
다시 섀도잉되거나 스코프가 끝나기 전까지 해당 변수 이름의 모든 사용을 자기에게로
가져갑니다. 다음과 같이 같은 변수 이름을 사용하고 `let` 키워드의 사용을 반복해
변수를 섀도잉할 수 있습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

이 프로그램은 먼저 `x`를 값 `5`에 바인딩합니다. 그런 다음 `let x =`를 반복하여 원래
값을 받고 `1`을 더해 새 변수 `x`를 만들어 `x`의 값이 `6`이 되도록 합니다. 그런
다음, 중괄호로 만들어진 내부 스코프 안에서, 세 번째 `let` 문이 `x`를 다시 섀도잉해
이전 값에 `2`를 곱해 `x`에 `12`라는 값을 주는 새 변수를 만듭니다. 해당 스코프가
끝나면 내부의 섀도잉도 끝나고, `x`는 다시 `6`이 됩니다. 이 프로그램을 실행하면
다음과 같은 출력이 나옵니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

섀도잉은 변수를 `mut`로 표시하는 것과 다릅니다. `let` 키워드 없이 실수로 이 변수에
다시 할당하려고 하면 컴파일 타임 에러가 발생하기 때문입니다. `let`을 사용하면,
값을 몇 번 변환한 뒤에도 변환이 모두 끝난 시점에 변수는 불변으로 둘 수 있습니다.

`mut`와 섀도잉의 또 다른 차이점은, `let` 키워드를 다시 사용하면 사실상 새 변수를
만들어 내는 것이기 때문에, 같은 이름을 재사용하면서 값의 타입을 바꿀 수 있다는
점입니다. 예를 들어, 어떤 텍스트 사이에 공백 문자를 몇 개 넣고 싶은지 사용자에게
공백 문자로 보여 달라고 요청한 뒤, 그 입력을 숫자로 저장하고 싶다고 해 봅시다.

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

첫 번째 `spaces` 변수는 문자열 타입이고, 두 번째 `spaces` 변수는 숫자 타입입니다.
이렇게 섀도잉은 `spaces_str`과 `spaces_num` 같은 서로 다른 이름을 생각해 내야 하는
수고를 덜어 주고, 대신 더 간단한 이름 `spaces`를 재사용할 수 있게 해 줍니다. 그러나
여기서 `mut`를 사용하려고 하면, 다음과 같이 컴파일 타임 에러가 발생합니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

에러는 변수의 타입을 변경할 수 없다는 내용을 알려 줍니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

이제 변수가 어떻게 동작하는지 살펴봤으니, 변수가 가질 수 있는 더 다양한 데이터
타입을 살펴봅시다.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html
