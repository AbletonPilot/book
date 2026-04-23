## 흐름 제어

어떤 조건이 `true`인지에 따라 어떤 코드를 실행하는 능력과, 어떤 조건이 `true`인
동안 어떤 코드를 반복해서 실행하는 능력은 대부분의 프로그래밍 언어에서 기본적인
구성 요소입니다. 러스트 코드의 실행 흐름을 제어하도록 해 주는 가장 일반적인 구성
요소는 `if` 표현식과 루프입니다.

### `if` 표현식

`if` 표현식은 조건에 따라 코드를 분기할 수 있게 해 줍니다. 여러분은 조건을 제시
하고 다음과 같이 말합니다. “이 조건이 맞으면 이 코드 블록을 실행하라. 조건이 맞지
않으면 이 코드 블록을 실행하지 말라.”

`if` 표현식을 살펴보기 위해 *projects* 디렉터리에 *branches*라는 이름의 새 프로젝트를
만드세요. *src/main.rs* 파일에 다음을 입력합니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

모든 `if` 표현식은 `if` 키워드로 시작하고, 그 뒤에 조건이 붙습니다. 이 경우 조건은
변수 `number`의 값이 5보다 작은지 여부를 확인합니다. 조건이 `true`일 때 실행할
코드 블록은 조건 바로 뒤 중괄호 안에 둡니다. `if` 표현식에서 조건에 딸려 있는
코드 블록은 때로 *갈래(arm)*라고 부르기도 합니다. 2장의 [“예측값과 비밀 숫자
비교하기”][comparing-the-guess-to-the-secret-number]<!-- ignore --> 절에서 다뤘던
`match` 표현식의 갈래와 같은 의미입니다.

선택적으로 `else` 표현식도 포함시킬 수 있습니다. 여기서는 그렇게 했는데, 조건이
`false`로 평가될 경우 실행할 대체 코드 블록을 프로그램에 제공하기 위함입니다.
`else` 표현식을 제공하지 않고 조건이 `false`이면, 프로그램은 그냥 `if` 블록을
건너뛰고 다음 코드로 이동합니다.

이 코드를 실행해 보세요. 다음과 같은 출력이 나와야 합니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

이제 `number`의 값을 조건이 `false`가 되는 값으로 바꿔서 어떻게 되는지 봅시다.

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

프로그램을 다시 실행해서 출력을 보세요.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

이 코드에서 조건은 *반드시* `bool`이어야 한다는 점에도 주목할 만합니다. 조건이
`bool`이 아니면 에러가 납니다. 예를 들어, 다음 코드를 실행해 봅시다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

이번에는 `if` 조건이 `3`이라는 값으로 평가되고, 러스트는 에러를 던집니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

에러는 러스트가 `bool`을 기대했는데 정수를 받았다는 내용을 알려 줍니다. 루비나
자바스크립트 같은 언어와 달리, 러스트는 불리언이 아닌 타입을 자동으로 불리언으로
변환하려고 하지 않습니다. 명시적이어야 하며, `if`에는 항상 불리언을 조건으로
제공해야 합니다. 예를 들어 숫자가 `0`이 아닐 때에만 `if` 코드 블록이 실행되도록
하려면, `if` 표현식을 다음과 같이 바꿀 수 있습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

이 코드를 실행하면 `number was something other than zero`가 출력됩니다.

#### `else if`로 여러 조건 다루기

`if`와 `else`를 `else if` 표현식으로 조합해서 여러 조건을 다룰 수 있습니다. 예를
들면 다음과 같습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

이 프로그램은 네 가지 가능한 경로를 가지고 있습니다. 실행하면 다음과 같은 출력을
보게 됩니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

이 프로그램이 실행될 때, 각 `if` 표현식을 순서대로 확인하고 조건이 `true`로 평가
되는 첫 번째 본문을 실행합니다. 6이 2로 나누어 떨어지더라도 `number is divisible
by 2`라는 출력이 보이지 않고, `else` 블록의 `number is not divisible by 4, 3, or 2`
텍스트도 보이지 않는다는 점에 유의하세요. 러스트는 처음으로 `true`인 조건의 블록만
실행하며, 하나를 찾으면 나머지는 검사조차 하지 않기 때문입니다.

`else if` 표현식을 너무 많이 사용하면 코드가 복잡해질 수 있으므로, 두 개 이상이라면
코드를 리팩터링하고 싶을 수 있습니다. 6장에서는 이런 경우를 위한 강력한 러스트의
분기 구조인 `match`를 설명합니다.

#### `let` 문에서 `if` 사용하기

`if`는 표현식이기 때문에, Listing 3-2처럼 `let` 문의 오른쪽에 사용해 그 결과를 변수에
할당할 수 있습니다.

<Listing number="3-2" file-name="src/main.rs" caption="`if` 표현식의 결과를 변수에 할당하기">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

`number` 변수는 `if` 표현식의 결과에 따라 값에 바인딩됩니다. 이 코드를 실행해서
어떻게 되는지 봅시다.

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

코드 블록은 그 안의 마지막 표현식으로 평가된다는 점과, 숫자 자체도 표현식이라는
점을 기억하세요. 이 경우 `if` 전체 표현식의 값은 어느 코드 블록이 실행되는지에
따라 달라집니다. 이는 `if`의 각 갈래에서 결과로 나올 수 있는 값들이 같은 타입이어야
한다는 뜻입니다. Listing 3-2에서는 `if` 갈래와 `else` 갈래의 결과가 모두 `i32`
정수였습니다. 타입이 일치하지 않으면 다음 예시처럼 에러가 발생합니다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

이 코드를 컴파일하려고 하면 에러가 납니다. `if`와 `else` 갈래는 호환되지 않는 값
타입을 가지고 있고, 러스트는 프로그램에서 문제의 위치를 정확히 알려 줍니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

`if` 블록의 표현식은 정수로 평가되고 `else` 블록의 표현식은 문자열로 평가됩니다.
이는 동작하지 않습니다. 변수는 단일 타입을 가져야 하고, 러스트는 컴파일 타임에
`number` 변수의 타입이 무엇인지 확정적으로 알아야 하기 때문입니다. `number`의
타입을 아는 것은 컴파일러가 `number`를 사용하는 모든 곳에서 그 타입이 유효한지
검증할 수 있게 해 줍니다. 만약 `number`의 타입이 런타임에야 결정된다면 러스트는
그렇게 할 수 없을 것이고, 어떤 변수에 대해 여러 가정 타입을 추적해야 한다면 컴파일러
가 더 복잡해지고 코드에 대한 보장이 줄어들 것입니다.

### 루프로 반복하기

코드 블록을 한 번 이상 실행하는 것은 유용한 경우가 많습니다. 이를 위해 러스트는
여러 *루프(loop)*를 제공하며, 루프 본문 안의 코드를 끝까지 실행한 뒤 곧바로 다시
처음으로 돌아옵니다. 루프를 가지고 실험해 보기 위해 *loops*라는 이름의 새 프로젝
트를 만들어 봅시다.

러스트에는 세 종류의 루프가 있습니다. `loop`, `while`, `for`입니다. 각각을 써
봅시다.

#### `loop`로 코드 반복하기

`loop` 키워드는 러스트에게 코드 블록을 영원히, 또는 명시적으로 멈추라고 말할
때까지 계속 반복해서 실행하라고 알립니다.

예시로, *loops* 디렉터리의 *src/main.rs* 파일을 다음과 같이 바꿔 봅시다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

이 프로그램을 실행하면, 프로그램을 직접 멈출 때까지 `again!`이 계속 출력되는 것을
볼 수 있습니다. 대부분의 터미널은 계속 도는 루프에 빠진 프로그램을 중단시키기 위해
<kbd>ctrl</kbd>-<kbd>C</kbd> 키보드 단축키를 지원합니다. 한 번 해 봅시다.

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

`^C` 기호는 <kbd>ctrl</kbd>-<kbd>C</kbd>를 누른 지점을 나타냅니다.

`^C` 뒤에 `again!`이라는 단어가 한 번 더 찍힐 수도 있고 아닐 수도 있습니다. 인터럽트
신호를 받은 시점에 코드가 루프의 어느 위치에 있었는지에 따라 달라집니다.

다행히 러스트는 코드로 루프를 빠져나갈 방법도 제공합니다. 루프 안에 `break` 키워드를
두어 프로그램에게 언제 루프 실행을 멈출지 알려 줄 수 있습니다. 2장의 [“정답 뒤에
종료하기”][quitting-after-a-correct-guess]<!-- ignore --> 절에서 숫자 맞히기 게임에서
정답을 맞힌 사용자가 게임에서 이겼을 때 프로그램을 종료하기 위해 이를 사용했던
것을 떠올려 보세요.

숫자 맞히기 게임에서는 `continue`도 사용했는데, 이는 루프에서 이번 반복의 남은
코드를 건너뛰고 다음 반복으로 넘어가라고 프로그램에게 알립니다.

#### 루프에서 값 반환하기

`loop`의 한 가지 용도는, 스레드가 작업을 완료했는지 확인하는 것처럼 실패할 수
있다는 것을 아는 연산을 다시 시도하는 것입니다. 또한 해당 연산의 결과를 루프에서
나머지 코드로 전달해야 할 수도 있습니다. 이를 위해 루프를 멈추는 데 사용하는
`break` 표현식 뒤에 반환하려는 값을 추가할 수 있습니다. 그러면 그 값이 루프 밖으로
반환되어 다음처럼 사용할 수 있게 됩니다.

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

루프 전에 `counter`라는 이름의 변수를 선언하고 `0`으로 초기화합니다. 그런 다음
루프에서 반환된 값을 담을 `result`라는 변수를 선언합니다. 루프의 매 반복마다
`counter` 변수에 `1`을 더하고, `counter`가 `10`인지 확인합니다. `10`이 되면
`break` 키워드를 값 `counter * 2`와 함께 사용합니다. 루프 뒤에 세미콜론으로 `result`
에 값을 할당하는 문을 마칩니다. 마지막으로 `result`의 값을 출력하는데, 이 경우
`20`입니다.

루프 안에서 `return`을 할 수도 있습니다. `break`는 현재 루프만 종료하는 반면,
`return`은 항상 현재 함수를 종료합니다.

<!-- Old headings. Do not remove or links may break. -->
<a id="loop-labels-to-disambiguate-between-multiple-loops"></a>

#### 루프 레이블로 모호함 없애기

루프 안에 루프가 있을 때, `break`와 `continue`는 그 시점에 가장 안쪽 루프에
적용됩니다. 선택적으로 루프에 *루프 레이블(loop label)*을 지정할 수 있고, 이를
`break`나 `continue`와 함께 사용해 해당 키워드들이 가장 안쪽 루프 대신 레이블이
붙은 루프에 적용되도록 지정할 수 있습니다. 루프 레이블은 작은따옴표 하나로 시작
해야 합니다. 다음은 두 개의 중첩된 루프가 있는 예시입니다.

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

바깥쪽 루프에는 `'counting_up` 레이블이 있고 0부터 2까지 셉니다. 레이블이 없는
안쪽 루프는 10부터 9까지 셉니다. 레이블을 지정하지 않은 첫 번째 `break`는 안쪽
루프만 빠져나갑니다. `break 'counting_up;` 문은 바깥쪽 루프를 빠져나갑니다. 이
코드는 다음을 출력합니다.

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

<!-- Old headings. Do not remove or links may break. -->
<a id="conditional-loops-with-while"></a>

#### `while`로 조건부 루프를 간결하게 만들기

프로그램은 종종 루프 안에서 조건을 평가해야 합니다. 조건이 `true`인 동안 루프가
실행됩니다. 조건이 `true`가 아니게 되면 프로그램은 `break`를 호출해 루프를 멈춥
니다. `loop`, `if`, `else`, `break`의 조합으로 이런 동작을 구현할 수도 있습니다.
원한다면 지금 프로그램에서 한 번 시도해 봐도 좋습니다. 하지만 이 패턴은 아주
흔해서, 러스트는 이를 위한 내장 언어 구조인 `while` 루프를 가지고 있습니다.
Listing 3-3에서는 `while`을 사용해 프로그램을 세 번 반복하며 매번 카운트다운을
하고, 루프가 끝난 뒤에는 메시지를 출력하고 종료합니다.

<Listing number="3-3" file-name="src/main.rs" caption="조건이 `true`로 평가되는 동안 코드를 실행하는 `while` 루프 사용">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

이 구조는 `loop`, `if`, `else`, `break`를 사용할 때 필요한 많은 중첩을 제거해 주며,
더 명확합니다. 조건이 `true`로 평가되는 동안 코드가 실행되고, 그렇지 않으면 루프를
빠져나옵니다.

#### `for`로 컬렉션 순회하기

`while` 구조를 사용해 배열 같은 컬렉션의 요소를 순회할 수도 있습니다. 예를 들어,
Listing 3-4의 루프는 배열 `a`의 각 요소를 출력합니다.

<Listing number="3-4" file-name="src/main.rs" caption="`while` 루프로 컬렉션의 각 요소 순회하기">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

여기서 코드는 배열의 요소를 하나씩 세어 나갑니다. 인덱스 `0`에서 시작해 배열의
마지막 인덱스에 도달할 때까지(즉, `index < 5`가 더 이상 `true`가 아닐 때까지)
루프가 돕니다. 이 코드를 실행하면 배열의 모든 요소를 출력합니다.

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

기대한 대로 배열의 다섯 값 모두가 터미널에 나타납니다. `index`가 어느 시점에
`5`라는 값이 되더라도, 배열에서 여섯 번째 값을 가져오려 시도하기 전에 루프가 실행을
멈춥니다.

그러나 이 접근 방식은 에러가 나기 쉽습니다. 인덱스 값이나 테스트 조건이 잘못되면
프로그램이 패닉을 일으킬 수 있습니다. 예를 들어 배열 `a`의 정의를 네 개의 요소로
바꾸면서 조건을 `while index < 4`로 업데이트하는 것을 잊으면 코드가 패닉을 일으킬
것입니다. 또한 이 방식은 느리기도 한데, 컴파일러가 루프의 매 반복마다 인덱스가
배열 범위 안에 있는지 조건부 검사를 수행하는 런타임 코드를 추가하기 때문입니다.

더 간결한 대안으로, `for` 루프를 사용해 컬렉션의 각 항목에 대해 어떤 코드를 실행할
수 있습니다. `for` 루프는 Listing 3-5의 코드처럼 생겼습니다.

<Listing number="3-5" file-name="src/main.rs" caption="`for` 루프로 컬렉션의 각 요소 순회하기">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

이 코드를 실행하면 Listing 3-4와 같은 출력을 보게 됩니다. 더 중요한 것은, 코드의
안전성을 높였고 배열의 끝을 넘어가거나 충분히 멀리 가지 못해 일부 항목을 놓칠
위험을 제거했다는 점입니다. 또한 `for` 루프에서 생성된 기계어 코드가 매 반복마다
인덱스를 배열 길이와 비교할 필요가 없기 때문에 더 효율적일 수도 있습니다.

`for` 루프를 사용하면 배열의 값 수를 바꿨을 때 Listing 3-4의 방식처럼 다른 코드를
바꿔야 한다는 것을 기억할 필요가 없습니다.

`for` 루프의 안전성과 간결함 덕분에, 러스트에서 `for` 루프는 가장 일반적으로 쓰이는
루프 구조입니다. 심지어 Listing 3-3의 `while` 루프를 사용한 카운트다운 예시처럼
특정 횟수만큼 어떤 코드를 실행하고 싶을 때에도, 대부분의 러스타시안은 `for` 루프를
사용할 것입니다. 그 방법은 한 숫자에서 시작해 다른 숫자 앞까지의 모든 숫자를 순서
대로 생성해 주는, 표준 라이브러리가 제공하는 `Range`를 사용하는 것입니다.

다음은 `for` 루프와, 아직 이야기하지 않은 다른 메서드인 `rev`를 사용해 범위를
뒤집어서 만든 카운트다운의 모습입니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

이 코드가 조금 더 좋아 보이지 않나요?

## 요약

해내셨습니다! 이번 장은 분량이 꽤 있는 장이었습니다. 변수, 스칼라 및 복합 데이터
타입, 함수, 주석, `if` 표현식, 루프에 대해 배웠습니다! 이 장에서 다룬 개념들로
연습해 보기 위해 다음과 같은 프로그램들을 만들어 봅시다.

- 화씨와 섭씨 사이의 온도를 변환하기.
- *n*번째 피보나치 수 생성하기.
- 크리스마스 캐럴 “The Twelve Days of Christmas”의 가사를, 노래의 반복을 활용해
  출력하기.

다음으로 넘어갈 준비가 되면, 다른 프로그래밍 언어에는 흔히 *존재하지 않는* 러스트의
개념, 즉 소유권에 대해 이야기하겠습니다.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
