## 함수

함수는 러스트 코드에서 아주 흔히 쓰입니다. 언어에서 가장 중요한 함수 중 하나는 이미
보셨을 것입니다. 바로 많은 프로그램의 진입점이 되는 `main` 함수입니다. 새 함수를
선언할 수 있게 해 주는 `fn` 키워드도 이미 보셨습니다.

러스트 코드에서는 함수와 변수 이름의 관례적인 스타일로 *스네이크 케이스(snake
case)*를 사용합니다. 스네이크 케이스에서는 모든 글자가 소문자이고 단어 사이를
밑줄로 구분합니다. 다음은 예시 함수 정의를 담은 프로그램입니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

러스트에서 함수는 `fn`을 쓰고 그 뒤에 함수 이름과 한 쌍의 괄호를 붙여 정의합니다.
중괄호는 컴파일러에게 함수 본문이 어디서 시작하고 끝나는지를 알려 줍니다.

정의한 함수는 그 이름 뒤에 한 쌍의 괄호를 붙여 호출할 수 있습니다. `another_function`
은 프로그램에 정의되어 있으므로 `main` 함수 안에서 호출할 수 있습니다. 소스 코드
에서 `another_function`을 `main` 함수 *뒤*에 정의했다는 점에 유의하세요. 앞에
정의해도 마찬가지로 잘 동작했을 것입니다. 러스트는 함수가 어디에 정의되어 있는지는
신경 쓰지 않습니다. 호출하는 쪽에서 볼 수 있는 스코프 어딘가에 정의되어 있기만
하면 됩니다.

함수를 더 살펴보기 위해 *functions*라는 이름의 새 바이너리 프로젝트를 시작해 봅시다.
앞서 본 `another_function` 예제를 *src/main.rs*에 넣고 실행해 보세요. 다음과 같은
출력이 보여야 합니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

줄들은 `main` 함수에 등장하는 순서대로 실행됩니다. 먼저 “Hello, world!” 메시지가
출력되고, 그다음 `another_function`이 호출되어 그 메시지가 출력됩니다.

### 매개변수

함수에 *매개변수(parameter)*를 갖도록 정의할 수 있습니다. 매개변수는 함수 시그니처의
일부인 특별한 변수입니다. 함수가 매개변수를 가지면, 그 매개변수에 대한 구체적인
값을 전달할 수 있습니다. 엄밀히 말해 그 구체적인 값들은 *인자(argument)*라고
부르지만, 일상적인 대화에서는 함수 정의의 변수이든 함수를 호출할 때 전달하는
구체적인 값이든 가리지 않고 *매개변수*와 *인자*라는 단어를 혼용해서 쓰는 경향이
있습니다.

다음은 매개변수를 추가한 버전의 `another_function`입니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

이 프로그램을 실행해 보세요. 다음과 같은 출력이 나와야 합니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

`another_function`의 선언에는 `x`라는 매개변수 하나가 있습니다. `x`의 타입은 `i32`로
지정되어 있습니다. `another_function`에 `5`를 전달하면, `println!` 매크로는 포맷
문자열에서 `x`를 담은 중괄호 쌍이 있던 자리에 `5`를 놓습니다.

함수 시그니처에서는 각 매개변수의 타입을 *반드시* 선언해야 합니다. 이는 러스트
설계의 의도적인 결정입니다. 함수 정의에 타입 명시를 요구하면, 컴파일러가 여러분이
의미한 타입을 알아내기 위해 코드의 다른 곳에서 타입 명시를 거의 필요로 하지 않게
됩니다. 또한 함수가 어떤 타입을 기대하는지 컴파일러가 알고 있으면, 더 도움이 되는
에러 메시지를 줄 수도 있습니다.

여러 매개변수를 정의할 때에는 다음처럼 쉼표로 매개변수 선언을 구분합니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

이 예시는 두 개의 매개변수를 가진 `print_labeled_measurement`라는 함수를 만듭니다.
첫 번째 매개변수는 이름이 `value`이고 `i32` 타입입니다. 두 번째는 이름이
`unit_label`이고 `char` 타입입니다. 이 함수는 `value`와 `unit_label`을 모두 포함한
텍스트를 출력합니다.

이 코드를 실행해 봅시다. *functions* 프로젝트의 *src/main.rs*에 있는 현재 프로그램을
앞의 예시로 바꾸고 `cargo run`으로 실행하세요.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

`value`에는 `5`를, `unit_label`에는 `'h'`를 값으로 전달해 함수를 호출했기 때문에,
프로그램 출력에 그 값들이 담겨 있습니다.

### 문(statement)과 표현식(expression)

함수 본문은 선택적으로 표현식으로 끝나는 일련의 문들로 이뤄집니다. 지금까지 다뤘던
함수들은 끝맺는 표현식을 포함하지 않았지만, 문의 일부로 표현식을 본 적은 있습니다.
러스트는 표현식 기반 언어이기 때문에, 이 구분을 이해하는 것이 중요합니다. 다른
언어들은 이런 구분을 하지 않기 때문에, 문과 표현식이 무엇이며 그 차이가 함수 본문에
어떤 영향을 미치는지 살펴봅시다.

- *문(statement)*은 어떤 동작을 수행하고 값을 반환하지 않는 명령입니다.
- *표현식(expression)*은 결과값으로 평가됩니다.

몇 가지 예를 살펴봅시다.

사실 우리는 이미 문과 표현식을 사용해 왔습니다. `let` 키워드로 변수를 만들고 값을
할당하는 것은 문입니다. Listing 3-1의 `let y = 6;`은 문입니다.

<Listing number="3-1" file-name="src/main.rs" caption="문 하나를 담고 있는 `main` 함수 선언">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

함수 정의 또한 문입니다. 앞의 예시 전체가 그 자체로 하나의 문입니다. (곧 보겠지만
함수 *호출*은 문이 아닙니다.)

문은 값을 반환하지 않습니다. 따라서 다음 코드처럼 `let` 문을 다른 변수에 할당할
수는 없습니다. 에러가 납니다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

이 프로그램을 실행하면 다음과 같은 에러가 발생합니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

`let y = 6` 문은 값을 반환하지 않으므로, `x`에 바인딩할 것이 없습니다. 이는 C나
루비 같은 다른 언어와 다른 점입니다. 그런 언어에서는 할당이 할당의 값을 반환하
므로, `x = y = 6`이라고 써서 `x`와 `y`가 모두 `6`을 가지도록 할 수 있습니다. 러스트
에서는 그렇게 동작하지 않습니다.

표현식은 값으로 평가되며, 여러분이 러스트에서 작성하게 될 코드 대부분을 이룹니다.
`5 + 6` 같은 수학 연산을 생각해 보세요. 이는 `11`이라는 값으로 평가되는 표현식
입니다. 표현식은 문의 일부가 될 수 있습니다. Listing 3-1의 `let y = 6;` 문에서
`6`은 값 `6`으로 평가되는 표현식입니다. 함수 호출도 표현식입니다. 매크로 호출도
표현식입니다. 예를 들어, 중괄호로 만드는 새 스코프 블록도 표현식입니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

다음 표현식은

```rust,ignore
{
    let x = 3;
    x + 1
}
```

이 경우 `4`로 평가되는 블록입니다. 그 값은 `let` 문의 일부로 `y`에 바인딩됩니다.
마지막의 `x + 1` 줄에 세미콜론이 없다는 점에 주목하세요. 이는 지금까지 본 대부분의
줄과 다릅니다. 표현식에는 끝맺는 세미콜론이 없습니다. 표현식의 끝에 세미콜론을
붙이면 그것을 문으로 바꾸게 되고, 그 결과 값을 반환하지 않게 됩니다. 다음에 함수
반환값과 표현식을 살펴볼 때 이 점을 기억해 두세요.

### 반환값이 있는 함수

함수는 자신을 호출한 코드로 값을 반환할 수 있습니다. 반환값에는 이름을 붙이지
않지만, 화살표(`->`) 뒤에 그 타입을 선언해야 합니다. 러스트에서 함수의 반환값은
함수 본문 블록의 마지막 표현식의 값과 동의어입니다. `return` 키워드와 값을 명시해
함수에서 일찍 반환할 수도 있지만, 대부분의 함수는 마지막 표현식을 암묵적으로
반환합니다. 다음은 값을 반환하는 함수의 예입니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

`five` 함수에는 함수 호출도, 매크로도, 심지어 `let` 문도 없습니다. 숫자 `5`만
달랑 있을 뿐입니다. 그리고 이것은 러스트에서 완전히 유효한 함수입니다. 함수의
반환 타입도 `-> i32`로 지정되어 있다는 점에 유의하세요. 이 코드를 실행해 봅시다.
출력은 다음과 같아야 합니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

`five`의 `5`는 함수의 반환값이고, 그래서 반환 타입이 `i32`입니다. 이 점을 좀 더
자세히 살펴봅시다. 중요한 두 가지가 있습니다. 첫째, `let x = five();` 줄은 함수의
반환값을 변수 초기화에 사용하고 있음을 보여 줍니다. `five` 함수가 `5`를 반환하므로
그 줄은 다음과 같습니다.

```rust
let x = 5;
```

둘째, `five` 함수는 매개변수가 없고 반환값의 타입을 정의하지만, 함수 본문은 세미
콜론 없이 외롭게 `5` 하나뿐입니다. 우리가 반환하고자 하는 값의 표현식이기 때문
입니다.

또 다른 예시를 보겠습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

이 코드를 실행하면 `The value of x is: 6`을 출력합니다. 하지만 `x + 1`이 있는 줄의
끝에 세미콜론을 붙여서 표현식을 문으로 바꾸면 어떤 일이 벌어질까요?

<span class="filename">파일명: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

이 코드를 컴파일하면 다음과 같이 에러가 발생합니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

메인 에러 메시지 `mismatched types`가 이 코드의 핵심 문제를 드러냅니다. `plus_one`
함수의 정의는 `i32`를 반환한다고 되어 있지만, 문은 값으로 평가되지 않으며, 이는
유닛 타입 `()`로 표현됩니다. 따라서 아무것도 반환되지 않고, 이는 함수 정의와
모순되어 에러로 이어집니다. 이 출력에서 러스트는 이 문제를 해결하는 데 도움이 될
법한 메시지를 함께 제공합니다. 바로 세미콜론을 제거하면 에러가 해결될 것이라는
제안입니다.
