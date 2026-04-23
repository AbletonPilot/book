## 데이터 타입

러스트의 모든 값은 특정한 *데이터 타입(data type)*을 가집니다. 이를 통해 러스트는
어떤 종류의 데이터가 지정되고 있는지를 알고, 그 데이터를 어떻게 다룰지 파악합니다.
여기서는 데이터 타입의 두 하위 집합을 살펴보겠습니다. 스칼라(scalar)와 복합
(compound)입니다.

러스트는 *정적 타입(statically typed)* 언어라는 점을 기억해 두세요. 이는 컴파일 타임에
모든 변수의 타입을 알고 있어야 한다는 뜻입니다. 컴파일러는 보통 값과 그 값의
사용 방식을 바탕으로 우리가 어떤 타입을 사용하고자 하는지 추론할 수 있습니다.
2장의 [“예측값과 비밀 숫자
비교하기”][comparing-the-guess-to-the-secret-number]<!-- ignore --> 절에서 `parse`로
`String`을 숫자 타입으로 변환했을 때처럼, 여러 타입이 가능한 경우에는 다음과 같이
타입 명시(annotation)를 반드시 추가해야 합니다.

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

앞의 코드에서 보인 `: u32` 타입 명시를 추가하지 않으면, 러스트는 다음과 같은
에러를 표시합니다. 이는 컴파일러가 어떤 타입을 사용하려는지 알기 위해 우리로부터
더 많은 정보를 얻어야 한다는 뜻입니다.

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

다른 데이터 타입에서는 또 다른 타입 명시를 보게 될 것입니다.

### 스칼라 타입

*스칼라(scalar)* 타입은 하나의 값을 나타냅니다. 러스트에는 네 가지 기본 스칼라
타입이 있습니다. 정수, 부동소수점 수, 불리언, 문자입니다. 다른 프로그래밍 언어에서
이미 본 적이 있을지도 모릅니다. 러스트에서 이들이 어떻게 동작하는지 바로 살펴
봅시다.

#### 정수 타입

*정수(integer)*는 소수부가 없는 숫자입니다. 2장에서는 정수 타입 `u32`를 하나
사용했습니다. 이 타입 선언은, 연관된 값이 32비트의 공간을 차지하는 부호 없는
정수여야 한다는 것을 나타냅니다(부호 있는 정수 타입은 `u` 대신 `i`로 시작합니다).
Table 3-1은 러스트의 내장 정수 타입을 보여 줍니다. 이 배리언트 중 어떤 것이든
정수 값의 타입을 선언하는 데 사용할 수 있습니다.

<span class="caption">Table 3-1: 러스트의 정수 타입</span>

| 길이    | 부호 있음 | 부호 없음 |
| ------- | -------- | --------- |
| 8-bit   | `i8`     | `u8`      |
| 16-bit  | `i16`    | `u16`     |
| 32-bit  | `i32`    | `u32`     |
| 64-bit  | `i64`    | `u64`     |
| 128-bit | `i128`   | `u128`    |
| 아키텍처 의존 | `isize` | `usize`  |

각 배리언트는 부호 있는 것과 부호 없는 것 중 하나이며, 명시적인 크기를 가집니다.
*부호 있음(signed)*과 *부호 없음(unsigned)*은 숫자가 음수가 될 수 있는지 여부를
의미합니다. 즉, 숫자에 부호가 함께 있어야 하는지(부호 있음), 아니면 항상 양수이기
때문에 부호 없이 표현될 수 있는지(부호 없음)를 말합니다. 종이에 숫자를 쓰는 것과
비슷합니다. 부호가 중요할 때는 양수 기호나 음수 기호와 함께 숫자를 표시하지만,
숫자가 양수라고 안전하게 가정할 수 있을 때는 부호 없이 표시합니다. 부호 있는 숫자는
[2의 보수(two's complement)][twos-complement]<!-- ignore --> 표현을 사용해
저장됩니다.

각 부호 있는 배리언트는 −(2<sup>n − 1</sup>)부터 2<sup>n − 1</sup> − 1까지의
숫자를 저장할 수 있습니다. 여기서 *n*은 해당 배리언트가 사용하는 비트 수입니다.
따라서 `i8`은 −(2<sup>7</sup>)부터 2<sup>7</sup> − 1까지, 즉 −128부터 127까지의
숫자를 저장할 수 있습니다. 부호 없는 배리언트는 0부터 2<sup>n</sup> − 1까지의
숫자를 저장할 수 있으므로, `u8`은 0부터 2<sup>8</sup> − 1, 즉 0부터 255까지의
숫자를 저장할 수 있습니다.

또한 `isize`와 `usize` 타입은 프로그램이 실행되는 컴퓨터의 아키텍처에 따라 달라
집니다. 64비트 아키텍처에서는 64비트이고, 32비트 아키텍처에서는 32비트입니다.

정수 리터럴은 Table 3-2에 나온 어떤 형식으로도 쓸 수 있습니다. 여러 숫자 타입이
될 수 있는 숫자 리터럴은 `57u8`처럼 타입을 지정하기 위한 타입 접미사를 허용
합니다. 숫자 리터럴은 읽기 쉽도록 시각적인 구분자로 `_`도 사용할 수 있습니다.
예를 들어 `1_000`은 `1000`으로 지정한 것과 같은 값을 가집니다.

<span class="caption">Table 3-2: 러스트의 정수 리터럴</span>

| 숫자 리터럴          | 예시          |
| -------------------- | ------------- |
| 10진수               | `98_222`      |
| 16진수               | `0xff`        |
| 8진수                | `0o77`        |
| 2진수                | `0b1111_0000` |
| 바이트(`u8` 전용)    | `b'A'`        |

그렇다면 어떤 정수 타입을 사용할지 어떻게 알 수 있을까요? 확신이 없다면, 러스트의
기본값이 대체로 좋은 출발점입니다. 정수 타입은 기본적으로 `i32`입니다. `isize`나
`usize`를 사용하는 주된 상황은 어떤 종류의 컬렉션을 인덱싱할 때입니다.

> ##### 정수 오버플로
>
> `u8` 타입의 변수가 있고, 이 변수는 0부터 255 사이의 값을 담을 수 있다고 해
> 봅시다. 변수를 그 범위를 벗어난 값, 예를 들어 256으로 바꾸려 하면, *정수
> 오버플로(integer overflow)*가 발생합니다. 정수 오버플로는 두 가지 동작 중 하나를
> 유발할 수 있습니다. 디버그 모드로 컴파일할 때 러스트는 정수 오버플로 검사를 포함
> 시키며, 이 동작이 발생하면 런타임에 프로그램이 *패닉(panic)*을 일으킵니다. 러스트는
> 프로그램이 에러와 함께 종료될 때 *패닉*이라는 용어를 사용합니다. 패닉에 대해서는
> 9장의 [“`panic!`을 사용한 복구 불가능한 에러”][unrecoverable-errors-with-panic]
> <!-- ignore --> 절에서 더 자세히 다룹니다.
>
> `--release` 플래그로 릴리스 모드에서 컴파일할 때, 러스트는 패닉을 유발하는 정수
> 오버플로 검사를 포함시키지 *않습니다*. 대신, 오버플로가 발생하면 러스트는 *2의
> 보수 래핑(two’s complement wrapping)*을 수행합니다. 요약하자면, 타입이 담을 수
> 있는 최댓값을 넘는 값은 해당 타입이 담을 수 있는 최솟값으로 “래핑(wrap around)”
> 됩니다. `u8`의 경우, 값 256은 0이 되고, 값 257은 1이 되는 식입니다. 프로그램이
> 패닉을 일으키지는 않지만, 변수는 여러분이 기대했던 값이 아닐 가능성이 높은 값을
> 가지게 됩니다. 정수 오버플로의 래핑 동작에 의존하는 것은 오류로 간주됩니다.
>
> 오버플로의 가능성을 명시적으로 처리하기 위해 표준 라이브러리가 원시 수치 타입에
> 제공하는 다음의 메서드 계열을 사용할 수 있습니다.
>
> - `wrapping_add`처럼 `wrapping_*` 메서드로 모든 모드에서 래핑합니다.
> - `checked_*` 메서드는 오버플로가 있으면 `None` 값을 반환합니다.
> - `overflowing_*` 메서드는 값과, 오버플로가 있었는지 여부를 나타내는 불리언을
>   반환합니다.
> - `saturating_*` 메서드는 타입의 최솟값이나 최댓값으로 포화시킵니다.

#### 부동소수점 타입

러스트는 *부동소수점 수(floating-point number)*, 즉 소수점이 있는 숫자를 위한 두
가지 원시 타입도 가지고 있습니다. 러스트의 부동소수점 타입은 `f32`와 `f64`로, 각각
32비트와 64비트 크기입니다. 기본 타입은 `f64`인데, 현대 CPU에서는 `f32`와 대략
비슷한 속도이지만 더 높은 정밀도를 가질 수 있기 때문입니다. 모든 부동소수점 타입은
부호 있는 타입입니다.

다음은 부동소수점 수를 실제로 보여 주는 예입니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

부동소수점 수는 IEEE-754 표준에 따라 표현됩니다.

#### 수치 연산

러스트는 모든 숫자 타입에 대해 여러분이 기대할 만한 기본 수학 연산을 지원합니다.
덧셈, 뺄셈, 곱셈, 나눗셈, 나머지입니다. 정수 나눗셈은 0에 가까운 방향으로
잘려서 가장 가까운 정수가 됩니다. 다음 코드는 `let` 문에서 각 수치 연산을 어떻게
사용하는지 보여 줍니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

이 문장들의 각 표현식은 수학 연산자를 사용하고 하나의 값으로 평가되어, 그 값이
변수에 바인딩됩니다. 러스트가 제공하는 모든 연산자의 목록은 [부록 B][appendix_b]
<!-- ignore -->에 있습니다.

#### 불리언 타입

대부분의 다른 프로그래밍 언어와 마찬가지로, 러스트의 불리언 타입은 `true`와
`false`라는 두 가지 값만 가질 수 있습니다. 불리언은 크기가 1바이트입니다. 러스트의
불리언 타입은 `bool`로 지정합니다. 예를 들면 다음과 같습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

불리언 값을 사용하는 주요 방법은 `if` 표현식과 같은 조건문을 통해서입니다. 러스트
에서 `if` 표현식이 어떻게 동작하는지는 [“흐름 제어”][control-flow]<!-- ignore -->
절에서 다룹니다.

#### 문자 타입

러스트의 `char` 타입은 이 언어의 가장 기본적인 알파벳 타입입니다. 다음은 `char`
값을 선언하는 예시입니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

`char` 리터럴은 큰따옴표를 사용하는 문자열 리터럴과 달리 작은따옴표로 지정한다는
점에 유의하세요. 러스트의 `char` 타입은 4바이트 크기이고 유니코드 스칼라 값
(Unicode scalar value)을 표현합니다. 즉, ASCII보다 훨씬 많은 것을 표현할 수 있다는
뜻입니다. 악센트가 있는 문자, 한국어·중국어·일본어 문자, 이모지, 영폭(zero-width)
공백 모두 러스트에서 유효한 `char` 값입니다. 유니코드 스칼라 값의 범위는 `U+0000`
부터 `U+D7FF`까지, 그리고 `U+E000`부터 `U+10FFFF`까지 (양 끝 포함) 입니다. 그러나
유니코드에서 “문자(character)”는 사실 개념적으로 딱 떨어지지 않기 때문에, 사람의
직관상 “문자”라고 여기는 것이 러스트의 `char`와 일치하지 않을 수 있습니다. 이
주제는 8장의 [“문자열로 UTF-8 인코딩된 텍스트 저장하기”][strings]<!-- ignore --> 절
에서 자세히 다룹니다.

### 복합 타입

*복합 타입(compound types)*은 여러 값을 하나의 타입으로 묶을 수 있습니다. 러스트
에는 두 가지 원시 복합 타입이 있습니다. 튜플과 배열입니다.

#### 튜플 타입

*튜플(tuple)*은 다양한 타입의 여러 값을 하나의 복합 타입으로 묶는 일반적인 방법
입니다. 튜플은 고정된 길이를 가집니다. 한 번 선언되면 크기를 늘리거나 줄일 수
없습니다.

튜플은 괄호 안에 쉼표로 구분된 값 목록을 써서 만듭니다. 튜플의 각 위치는 타입을
가지며, 튜플 안의 서로 다른 값들의 타입이 모두 같을 필요는 없습니다. 이 예제에는
선택적인 타입 명시도 함께 넣어 두었습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

변수 `tup`은 튜플 전체에 바인딩됩니다. 튜플이 하나의 복합 요소로 간주되기 때문
입니다. 튜플에서 개별 값을 꺼내려면, 다음처럼 패턴 매칭을 사용해 튜플 값을 구조
분해(destructure)할 수 있습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

이 프로그램은 먼저 튜플을 만들어 변수 `tup`에 바인딩합니다. 그런 다음 `let`과
패턴을 사용해 `tup`을 받아 이를 `x`, `y`, `z`라는 세 개의 개별 변수로 나눕니다.
하나의 튜플을 세 부분으로 쪼개기 때문에 이를 *구조 분해(destructuring)*라고
부릅니다. 마지막으로, 프로그램은 `y`의 값인 `6.4`를 출력합니다.

또한 튜플 요소는 접근하고자 하는 값의 인덱스 앞에 마침표(`.`)를 붙여서 직접 접근
할 수도 있습니다. 예를 들면 다음과 같습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

이 프로그램은 튜플 `x`를 만든 뒤, 각 요소를 해당 인덱스로 접근합니다. 대부분의
프로그래밍 언어와 마찬가지로, 튜플의 첫 인덱스는 0입니다.

아무 값도 없는 튜플에는 *유닛(unit)*이라는 특별한 이름이 붙어 있습니다. 이 값과
그 대응 타입 모두 `()`로 쓰며, 빈 값 또는 빈 반환 타입을 나타냅니다. 표현식이 다른
값을 반환하지 않으면 암묵적으로 유닛 값을 반환합니다.

#### 배열 타입

여러 값을 한데 모아 두는 또 다른 방법은 *배열(array)*입니다. 튜플과 달리, 배열의
모든 요소는 같은 타입을 가져야 합니다. 다른 일부 언어의 배열과 달리, 러스트의
배열은 고정된 길이를 가집니다.

배열의 값들은 다음과 같이 대괄호 안에 쉼표로 구분된 목록으로 씁니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

배열은 지금까지 본 다른 타입들처럼 데이터를 힙이 아닌 스택에 할당하고자 할 때(스택과
힙에 대해서는 [4장][stack-and-heap]<!-- ignore -->에서 더 이야기합니다), 혹은 항상
고정된 수의 요소를 갖도록 보장하고자 할 때 유용합니다. 하지만 배열은 벡터 타입만큼
유연하지는 않습니다. 벡터는 내용이 힙에 저장되기 때문에 크기가 커지거나 줄어들
수 *있는*, 표준 라이브러리가 제공하는 유사한 컬렉션 타입입니다. 배열과 벡터 중
어떤 것을 써야 할지 확신이 서지 않는다면, 벡터를 사용하는 편이 맞을 가능성이
높습니다. 벡터는 [8장][vectors]<!-- ignore -->에서 더 자세히 다룹니다.

그러나 요소 수가 바뀔 필요가 없다는 것이 확실할 때에는 배열이 더 유용합니다.
예를 들어 프로그램에서 달(month)의 이름을 사용한다면, 항상 12개의 요소를 가진다는
것을 알고 있기 때문에 벡터가 아닌 배열을 사용하게 될 것입니다.

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

배열의 타입은 대괄호 안에 각 요소의 타입, 세미콜론, 그리고 배열의 요소 수를 적는
방식으로 씁니다.

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

여기서 `i32`는 각 요소의 타입입니다. 세미콜론 뒤의 숫자 `5`는 배열이 다섯 개의
요소를 포함한다는 것을 나타냅니다.

또한 초기값을 지정하고 뒤이어 세미콜론과 대괄호 안에 배열의 길이를 적으면, 각
요소를 같은 값으로 초기화한 배열을 만들 수도 있습니다.

```rust
let a = [3; 5];
```

이름이 `a`인 배열은 처음에 모두 값 `3`으로 설정된 `5`개의 요소를 가지게 됩니다.
이는 `let a = [3, 3, 3, 3, 3];`이라고 쓴 것과 같지만 더 간결한 방식입니다.

<!-- Old headings. Do not remove or links may break. -->
<a id="accessing-array-elements"></a>

#### 배열 요소 접근

배열은 스택에 할당될 수 있는, 크기가 알려진 고정된 메모리 한 덩어리입니다. 배열의
요소에는 다음과 같이 인덱싱으로 접근할 수 있습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

이 예시에서 `first`라는 변수는 배열의 인덱스 `[0]`에 있는 값이 `1`이므로 `1`을
얻습니다. `second`라는 변수는 배열의 인덱스 `[1]`에서 `2`를 얻습니다.

#### 잘못된 배열 요소 접근

배열의 끝을 넘어선 요소에 접근하려고 하면 어떤 일이 벌어지는지 살펴봅시다. 2장의
숫자 맞히기 게임과 비슷하게, 다음 코드를 실행해 사용자로부터 배열 인덱스를 받는다고
해 봅시다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

이 코드는 성공적으로 컴파일됩니다. `cargo run`으로 이 코드를 실행하고 `0`, `1`,
`2`, `3`, `4` 중 하나를 입력하면 배열의 해당 인덱스에 있는 값을 출력합니다. 대신
`10`처럼 배열의 끝을 넘어선 숫자를 입력하면 다음과 같은 출력을 보게 됩니다.

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

이 프로그램은 인덱싱 연산에서 잘못된 값을 사용하는 시점에 런타임 에러를 일으켰
습니다. 프로그램은 에러 메시지와 함께 종료되었고, 마지막 `println!` 문은 실행되지
않았습니다. 인덱싱으로 요소에 접근하려고 하면, 러스트는 여러분이 지정한 인덱스가
배열 길이보다 작은지 확인합니다. 인덱스가 길이 이상이면 러스트는 패닉을 일으킵니다.
이 검사는 런타임에 일어나야 합니다. 특히 이 경우에는 사용자가 이후에 코드를 실행할
때 어떤 값을 입력할지 컴파일러가 알 수 있는 방법이 전혀 없기 때문입니다.

이는 러스트의 메모리 안전성 원칙이 실제로 동작하는 예시입니다. 많은 저수준 언어
에서는 이런 검사가 이뤄지지 않아, 잘못된 인덱스를 제공하면 유효하지 않은 메모리에
접근하게 될 수 있습니다. 러스트는 메모리 접근을 허용하고 실행을 이어가는 대신
즉시 종료함으로써, 이런 종류의 에러로부터 여러분을 보호합니다. 9장에서는 러스트의
에러 처리와, 패닉도 일으키지 않고 유효하지 않은 메모리 접근도 허용하지 않는 읽기
쉽고 안전한 코드를 작성하는 방법을 더 다룹니다.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md
