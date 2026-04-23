## 패턴을 사용할 수 있는 모든 곳

패턴은 러스트 곳곳에 등장하며, 의식하지 못한 채 이미 많이 사용해 왔습니다! 이 절에서는 패턴이 유효한 모든 위치를 다룹니다.

### `match` 갈래(arms)

6장에서 논의했듯이, `match` 표현식의 갈래에서 패턴을 사용합니다. 형식적으로 `match` 표현식은 `match` 키워드, 매칭할 값, 그리고 패턴과 그 값이 해당 갈래의 패턴과 매칭될 때 실행할 표현식으로 구성된 하나 이상의 매치 갈래로 정의됩니다. 다음과 같은 형태입니다.

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre><code>match <em>VALUE</em> {
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
}</code></pre>

예를 들어, 다음은 변수 `x`의 `Option<i32>` 값에 대해 매칭하는 Listing 6-5의 `match` 표현식입니다.

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

이 `match` 표현식의 패턴은 각 화살표 왼쪽의 `None`과 `Some(i)`입니다.

`match` 표현식의 한 가지 요건은, `match` 표현식의 값에 대한 모든 가능성을 빠짐없이 처리해야 한다는 의미에서 _완전(exhaustive)_ 해야 한다는 것입니다. 모든 가능성을 다루었음을 보장하는 한 가지 방법은 마지막 갈래에 모든 것을 포괄하는 패턴을 두는 것입니다. 예를 들어, 어떤 값이라도 매칭하는 변수 이름은 결코 매칭에 실패하지 않으므로 남은 모든 경우를 다룹니다.

특정 패턴 `_`는 무엇이든 매칭하지만 변수에 결코 바인딩되지 않으므로, 종종 마지막 매치 갈래에 사용됩니다. `_` 패턴은 예를 들어 명시되지 않은 값을 무시하고 싶을 때 유용할 수 있습니다. 이 장 뒤편의 [“패턴에서 값 무시하기”][ignoring-values-in-a-pattern]<!-- ignore --> 절에서 `_` 패턴을 더 자세히 다룹니다.

### `let` 문

이 장 이전에는 `match`와 `if let`에서 패턴을 사용하는 것에 대해서만 명시적으로 논의했지만, 사실 `let` 문을 비롯한 다른 곳에서도 패턴을 사용해 왔습니다. 예를 들어, `let`을 사용하는 다음의 단순한 변수 할당을 봅시다.

```rust
let x = 5;
```

이런 형태의 `let` 문을 사용할 때마다, 의식하지 못했더라도 패턴을 사용해 온 것입니다! 좀 더 형식적으로 `let` 문은 다음과 같이 생겼습니다.

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre>
<code>let <em>PATTERN</em> = <em>EXPRESSION</em>;</code>
</pre>

`let x = 5;`처럼 PATTERN 자리에 변수 이름이 들어가는 문에서, 변수 이름은 그저 패턴의 특별히 단순한 한 형태입니다. 러스트는 표현식을 패턴과 비교한 뒤 패턴에서 발견하는 모든 이름에 값을 할당합니다. 그래서 `let x = 5;` 예시에서 `x`는 “여기에 매칭되는 것을 변수 `x`에 바인딩하라”는 의미의 패턴입니다. `x`라는 이름이 패턴 전체이므로, 이 패턴은 사실상 “값이 무엇이든 모든 것을 변수 `x`에 바인딩하라”는 의미입니다.

`let`의 패턴 매칭 측면을 더 명확히 보기 위해, 튜플을 분해하는 데 `let`과 함께 패턴을 사용하는 Listing 19-1을 살펴봅시다.


<Listing number="19-1" caption="튜플을 분해해 한 번에 세 개의 변수를 만들기 위해 패턴 사용하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

여기서는 튜플을 패턴에 매칭합니다. 러스트는 값 `(1, 2, 3)`을 패턴 `(x, y, z)`와 비교하고, 그 값이 패턴과 매칭됨을 봅니다—즉, 둘 다 요소의 개수가 같음을 보고 `1`을 `x`에, `2`를 `y`에, `3`을 `z`에 바인딩합니다. 이 튜플 패턴을 그 안에 세 개의 개별 변수 패턴을 중첩한 것으로 생각할 수 있습니다.

만약 패턴의 요소 개수가 튜플의 요소 개수와 일치하지 않으면 전체 타입이 일치하지 않게 되어 컴파일 오류가 발생합니다. 예를 들어, Listing 19-2는 세 요소짜리 튜플을 두 개의 변수로 분해하려는 시도를 보여 주는데, 이는 동작하지 않습니다.

<Listing number="19-2" caption="튜플의 요소 개수와 변수의 수가 일치하지 않는 잘못된 패턴 구성">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

이 코드를 컴파일하려고 하면 다음과 같은 타입 오류가 발생합니다.

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

이 오류를 고치려면 [“패턴에서 값 무시하기”][ignoring-values-in-a-pattern]<!-- ignore --> 절에서 보게 될 `_`나 `..`을 사용해 튜플의 값 중 하나 이상을 무시할 수 있습니다. 패턴의 변수가 너무 많은 것이 문제라면, 변수 수가 튜플의 요소 수와 같아지도록 변수를 제거해 타입을 맞추면 됩니다.

### 조건부 `if let` 표현식

6장에서는 단 하나의 경우만 매칭하는 `match`와 동일한 의미를 더 짧게 쓰는 방법으로 주로 `if let` 표현식을 다뤘습니다. 선택적으로 `if let`은 `if let`의 패턴이 매칭되지 않을 때 실행할 코드를 담은 대응되는 `else`를 가질 수 있습니다.

Listing 19-3은 `if let`, `else if`, `else if let` 표현식을 섞어 쓰는 것도 가능함을 보여 줍니다. 이렇게 하면 패턴과 비교할 값을 단 하나만 표현할 수 있는 `match` 표현식보다 더 많은 유연성을 얻을 수 있습니다. 또한 러스트는 일련의 `if let`, `else if`, `else if let` 갈래의 조건들이 서로 관련 있을 것을 요구하지도 않습니다.

Listing 19-3의 코드는 여러 조건에 대한 일련의 검사를 바탕으로 배경을 어떤 색으로 할지 결정합니다. 이 예제에서는 실제 프로그램이라면 사용자 입력에서 받았을 법한 값들을 하드코딩한 변수들을 만들었습니다.

<Listing number="19-3" file-name="src/main.rs" caption="`if let`, `else if`, `else if let`, `else` 섞어 쓰기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

사용자가 좋아하는 색을 지정하면 그 색이 배경으로 사용됩니다. 좋아하는 색이 지정되지 않았고 오늘이 화요일이면 배경색은 초록색입니다. 그 외에, 사용자가 자신의 나이를 문자열로 지정했고 그것을 숫자로 성공적으로 파싱할 수 있으면, 색은 그 숫자 값에 따라 보라색이나 주황색이 됩니다. 이런 조건이 모두 적용되지 않으면 배경색은 파란색입니다.

이러한 조건 구조는 복잡한 요구사항을 지원할 수 있게 해 줍니다. 여기에 있는 하드코딩된 값들에서 이 예제는 `Using purple as the background color`를 출력합니다.

`if let` 또한 `match` 갈래와 같은 방식으로, 기존 변수를 가리는(shadow) 새로운 변수를 도입할 수 있음을 볼 수 있습니다. `if let Ok(age) = age` 줄은 `Ok` 변형 안의 값을 담는 새로운 `age` 변수를 도입하여, 기존 `age` 변수를 가립니다. 이는 `if age > 30` 조건을 그 블록 안에 두어야 함을 의미합니다. 이 두 조건을 `if let Ok(age) = age && age > 30`처럼 결합할 수는 없습니다. 30과 비교하려는 새로운 `age`는 중괄호로 새로운 스코프가 시작되기 전까지는 유효하지 않습니다.

`if let` 표현식 사용의 단점은 컴파일러가 완전성을 검사하지 않는다는 점입니다. 반면 `match` 표현식에서는 검사합니다. 마지막 `else` 블록을 생략해 일부 경우를 처리하지 않더라도, 컴파일러는 가능한 로직 버그에 대해 우리에게 경고하지 않을 것입니다.

### `while let` 조건부 반복

`if let`과 비슷한 형태로, `while let` 조건부 반복은 패턴이 계속 매칭되는 한 `while` 반복을 실행할 수 있게 해 줍니다. Listing 19-4에서는 스레드 사이에 보내진 메시지를 기다리는 `while let` 반복을 보여 주는데, 이번에는 `Option` 대신 `Result`를 검사합니다.

<Listing number="19-4" caption="`rx.recv()`가 `Ok`를 반환하는 한 값들을 출력하기 위해 `while let` 반복 사용하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

이 예제는 `1`, `2`, 그 다음 `3`을 출력합니다. `recv` 메서드는 채널의 수신 측에서 첫 번째 메시지를 꺼내 `Ok(value)`를 반환합니다. 16장에서 `recv`를 처음 보았을 때는 오류를 직접 언랩하거나 `for` 반복을 사용해 이터레이터로서 상호작용했습니다. 그러나 Listing 19-4에서 보이듯, 송신자가 존재하는 한 `recv` 메서드는 메시지가 도착할 때마다 `Ok`를 반환하다가 송신측이 끊기면 `Err`를 만들어 내므로, `while let`도 사용할 수 있습니다.

### `for` 반복

`for` 반복에서 `for` 키워드 바로 뒤에 오는 값은 패턴입니다. 예를 들어 `for x in y`에서 `x`가 패턴입니다. Listing 19-5는 `for` 반복의 일부로 튜플을 분해, 즉 쪼개기 위해 `for` 반복에서 패턴을 어떻게 사용하는지 보여 줍니다.


<Listing number="19-5" caption="튜플을 분해하기 위해 `for` 반복에서 패턴 사용하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

Listing 19-5의 코드는 다음을 출력합니다.


```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

`enumerate` 메서드를 사용해 이터레이터를 변환함으로써, 값과 그 값에 대한 인덱스를 튜플로 만들어 산출합니다. 처음 산출되는 값은 튜플 `(0, 'a')`입니다. 이 값이 패턴 `(index, value)`에 매칭될 때, `index`는 `0`이고 `value`는 `'a'`가 되어 출력의 첫 줄을 인쇄합니다.


### 함수 매개변수

함수 매개변수도 패턴이 될 수 있습니다. Listing 19-6은 `i32` 타입의 매개변수 `x` 하나를 받는 `foo`라는 함수를 선언하는 코드인데, 지금쯤이면 익숙해 보일 것입니다.

<Listing number="19-6" caption="매개변수에 패턴을 사용하는 함수 시그니처">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

`x` 부분이 패턴입니다! `let`에서 했던 것처럼 함수 인자에서 튜플을 패턴에 매칭할 수 있습니다. Listing 19-7은 함수에 튜플을 전달하면서 그 안의 값들을 분리합니다.

<Listing number="19-7" file-name="src/main.rs" caption="튜플을 분해하는 매개변수를 가진 함수">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

이 코드는 `Current location: (3, 5)`를 출력합니다. 값 `&(3, 5)`가 패턴 `&(x, y)`와 매칭되어, `x`는 값 `3`이 되고 `y`는 값 `5`가 됩니다.

13장에서 논의했듯 클로저는 함수와 비슷하므로, 함수 매개변수 목록과 같은 방식으로 클로저 매개변수 목록에서도 패턴을 사용할 수 있습니다.

이 시점에서 패턴을 사용하는 여러 방법을 보았지만, 패턴은 사용할 수 있는 모든 곳에서 같은 방식으로 동작하는 것은 아닙니다. 어떤 곳에서는 반박 불가 패턴이어야 하고, 다른 상황에서는 반박 가능 패턴일 수 있습니다. 다음으로 이 두 개념을 다룹니다.

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
