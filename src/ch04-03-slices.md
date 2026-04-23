## 슬라이스 타입

*슬라이스(slice)*는 [컬렉션](ch08-00-common-collections.md)<!-- ignore --> 안의
연속된 요소 시퀀스를 참조할 수 있게 해 줍니다. 슬라이스는 참조의 한 종류이므로
소유권을 갖지 않습니다.

간단한 프로그래밍 문제를 하나 봅시다. 공백으로 구분된 단어들의 문자열을 받아, 그
문자열에서 찾은 첫 단어를 반환하는 함수를 작성하세요. 함수가 문자열에서 공백을
찾지 못하면, 문자열 전체가 하나의 단어라는 뜻이므로 문자열 전체를 반환해야 합니다.

> 참고: 슬라이스를 소개하기 위해 이 절에서는 ASCII만을 가정합니다. UTF-8 처리에
> 대한 더 철저한 논의는 8장의 [“문자열로 UTF-8 인코딩된 텍스트 저장하기”][strings]
> <!-- ignore --> 절에 있습니다.

슬라이스 없이 이 함수의 시그니처를 어떻게 쓸지 살펴보며, 슬라이스가 해결할 문제를
이해해 봅시다.

```rust,ignore
fn first_word(s: &String) -> ?
```

`first_word` 함수는 `&String` 타입의 매개변수를 가집니다. 소유권이 필요 없으니
이것은 괜찮습니다. (관용적인 러스트에서는 함수가 자신의 인자의 소유권을 가져가야
할 필요가 없는 한 가져가지 않으며, 그 이유는 앞으로 진행하면서 분명해질 것입니다.)
하지만 무엇을 반환해야 할까요? 문자열의 *일부*를 이야기할 수 있는 마땅한 방법이
없습니다. 그러나 공백으로 표시되는 단어의 끝 인덱스를 반환할 수는 있습니다. Listing
4-7처럼 시도해 봅시다.

<Listing number="4-7" file-name="src/main.rs" caption="`String` 매개변수의 바이트 인덱스 값을 반환하는 `first_word` 함수">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

`String`을 요소별로 훑어보며 값이 공백인지 확인해야 하므로, `as_bytes` 메서드를
사용해 `String`을 바이트 배열로 변환하겠습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

다음으로, `iter` 메서드로 바이트 배열에 대한 이터레이터(iterator)를 만듭니다.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

이터레이터에 대해서는 [13장][ch13]<!-- ignore -->에서 더 자세히 다룹니다. 지금은
`iter`가 컬렉션의 각 요소를 반환하는 메서드이고, `enumerate`는 `iter`의 결과를
감싸 각 요소를 튜플의 일부로 반환한다는 정도만 알아 두세요. `enumerate`가 반환
하는 튜플의 첫 번째 요소는 인덱스이고, 두 번째 요소는 해당 요소에 대한 참조입니다.
이는 직접 인덱스를 계산하는 것보다 조금 더 편리합니다.

`enumerate` 메서드는 튜플을 반환하기 때문에, 패턴을 사용해 그 튜플을 구조 분해할
수 있습니다. 패턴에 대해서는 [6장][ch6]<!-- ignore -->에서 더 다룹니다. `for`
루프에서 튜플의 인덱스에 해당하는 자리에는 `i`를, 튜플 안의 단일 바이트에 해당하는
자리에는 `&item`을 두는 패턴을 지정합니다. `.iter().enumerate()`로부터 요소의
참조를 받기 때문에, 패턴에서 `&`를 사용합니다.

`for` 루프 안에서, 바이트 리터럴 문법을 사용해 공백을 나타내는 바이트를 찾습니다.
공백을 찾으면 그 위치를 반환합니다. 그렇지 않으면 `s.len()`을 사용해 문자열의
길이를 반환합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

이제 우리는 문자열에서 첫 단어의 끝 인덱스를 알아낼 수단을 가졌지만, 문제가 있습
니다. 우리는 `usize`를 그 자체로 반환하고 있는데, 이 값은 `&String`의 맥락에서만
의미 있는 숫자입니다. 다시 말해, 이 값은 `String`과 별개의 값이기 때문에 앞으로도
유효할 것이라는 보장이 없습니다. Listing 4-7의 `first_word` 함수를 사용하는
Listing 4-8의 프로그램을 봅시다.

<Listing number="4-8" file-name="src/main.rs" caption="`first_word` 함수의 결과를 저장한 뒤 `String` 내용을 변경하기">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

이 프로그램은 아무 에러 없이 컴파일되며, `s.clear()` 호출 뒤에 `word`를 사용해도
마찬가지입니다. `word`는 `s`의 상태와 전혀 연결되어 있지 않기 때문에, `word`는
여전히 값 `5`를 담고 있습니다. 그 `5`를 변수 `s`와 함께 사용해 첫 단어를 뽑아
내려고 할 수는 있지만, 우리가 `5`를 `word`에 저장한 이후 `s`의 내용이 바뀌었기
때문에 이는 버그가 될 것입니다.

`word`에 있는 인덱스가 `s`의 데이터와 어긋나지 않는지 걱정해야 하는 것은 번거롭고
에러가 나기 쉽습니다! `second_word` 함수를 작성한다면 이 인덱스들을 관리하는 일은
더 취약해집니다. 그 시그니처는 다음과 같은 모습이 될 것입니다.

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

이제 시작 인덱스*와* 끝 인덱스를 추적하고 있고, 특정 상태의 데이터로부터 계산
되었지만 그 상태와 전혀 연결되어 있지 않은 값들이 더 많아졌습니다. 서로 동기화
되어야 하는 세 개의 무관한 변수가 떠다니는 셈입니다.

다행히 러스트에는 이 문제를 해결할 방법이 있습니다. 바로 문자열 슬라이스입니다.

### 문자열 슬라이스

*문자열 슬라이스(string slice)*는 `String`의 요소들의 연속된 시퀀스에 대한 참조
이며, 다음과 같이 생겼습니다.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

`hello`는 전체 `String`에 대한 참조가 아니라, `[0..5]` 부분으로 지정된, `String`의
일부에 대한 참조입니다. 슬라이스는 대괄호 안의 범위를 사용해서
`[starting_index..ending_index]` 형태로 만듭니다. 여기서 _`starting_index`_ 는
슬라이스의 첫 위치이고, _`ending_index`_ 는 슬라이스의 마지막 위치보다 하나 큰
값입니다. 내부적으로 슬라이스 자료 구조는 시작 위치와 슬라이스의 길이를 저장하며,
이 길이는 _`ending_index`_ 에서 _`starting_index`_ 를 뺀 값에 해당합니다. 따라서
`let world = &s[6..11];`의 경우 `world`는 `s`의 인덱스 6 바이트를 가리키는 포인터와
길이 값 `5`를 가지는 슬라이스가 됩니다.

Figure 4-7이 이를 그림으로 보여 줍니다.

<img alt="세 개의 표: 한 표는 s의 스택 데이터를 나타내며, 힙에 있는 문자열 데이터
&quot;hello world&quot; 표의 인덱스 0 바이트를 가리킵니다. 세 번째 표는 슬라이스 world의
스택 데이터를 나타내며, 길이 값 5를 가지고 힙 데이터 표의 바이트 6을 가리킵니다."
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-7: `String`의 일부를 가리키는 문자열
슬라이스</span>

러스트의 `..` 범위 문법에서는, 인덱스 0에서 시작하려면 두 점 앞의 값을 생략할 수
있습니다. 즉, 다음 두 가지는 같습니다.

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

같은 맥락에서, 슬라이스가 `String`의 마지막 바이트를 포함한다면 뒤의 숫자를 생략할
수 있습니다. 즉, 다음 두 가지는 같습니다.

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

두 값 모두 생략해서 전체 문자열의 슬라이스를 만들 수도 있습니다. 따라서 다음은
같습니다.

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> 참고: 문자열 슬라이스의 범위 인덱스는 유효한 UTF-8 문자 경계에서 나타나야 합
> 니다. 여러 바이트 문자 중간에서 문자열 슬라이스를 만들려고 하면, 프로그램이
> 에러와 함께 종료됩니다.

이 모든 내용을 바탕으로, 슬라이스를 반환하도록 `first_word`를 다시 작성해 봅시다.
“문자열 슬라이스”를 의미하는 타입은 `&str`로 씁니다.

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

Listing 4-7에서와 같은 방식으로, 공백이 처음 나타나는 위치를 찾아 단어의 끝
인덱스를 얻습니다. 공백을 찾으면, 문자열의 시작과 공백의 인덱스를 시작과 끝 인덱스
로 사용해 문자열 슬라이스를 반환합니다.

이제 `first_word`를 호출하면, 기저 데이터에 묶인 하나의 값을 돌려받습니다. 이
값은 슬라이스의 시작 지점을 가리키는 참조와 슬라이스의 요소 개수로 이뤄집니다.

`second_word` 함수에서도 슬라이스를 반환하는 것이 잘 동작합니다.

```rust,ignore
fn second_word(s: &String) -> &str {
```

이제 우리는 훨씬 망치기 어려운, 단순한 API를 가지게 되었습니다. 컴파일러가 `String`
안으로 들어가는 참조가 유효하도록 보장해 주기 때문입니다. 첫 단어의 끝 인덱스를
얻은 뒤 문자열을 비워서 인덱스가 유효하지 않게 되었던 Listing 4-8 프로그램의
버그를 기억하시나요? 그 코드는 논리적으로는 잘못되었지만 즉각적인 에러를 드러
내지는 않았습니다. 비워진 문자열에 첫 단어 인덱스를 계속 사용하려 하면 문제가
이후에 드러날 뿐이었죠. 슬라이스는 이 버그를 불가능하게 만들며, 코드에 문제가
있음을 훨씬 더 일찍 알려 줍니다. 슬라이스 버전의 `first_word`를 사용하면 컴파일
타임 에러가 발생합니다.

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

다음은 컴파일러 에러입니다.

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

빌림 규칙을 떠올리면, 어떤 것에 대한 불변 참조를 가지고 있으면 가변 참조도 동시에
가질 수는 없었습니다. `clear`는 `String`을 잘라 내야 하므로 가변 참조를 얻어야
합니다. `clear` 호출 뒤의 `println!`은 `word`의 참조를 사용하므로, 그 시점에도
불변 참조가 여전히 활성 상태여야 합니다. 러스트는 `clear`의 가변 참조와 `word`의
불변 참조가 같은 시점에 존재하는 것을 허용하지 않으며, 컴파일이 실패합니다. 러스트는
API를 사용하기 쉽게 만들어 주는 것에 그치지 않고, 이런 종류의 에러 부류 전체를
컴파일 타임에 제거해 주기도 하는 것입니다!

<!-- Old headings. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### 문자열 리터럴은 슬라이스

문자열 리터럴이 바이너리 안에 저장된다고 이야기했던 것을 떠올려 보세요. 이제
슬라이스를 알게 되었으니, 문자열 리터럴을 제대로 이해할 수 있습니다.

```rust
let s = "Hello, world!";
```

여기서 `s`의 타입은 `&str`입니다. 바이너리의 해당 특정 지점을 가리키는 슬라이스
이죠. 이것이 바로 문자열 리터럴이 불변인 이유이기도 합니다. `&str`은 불변
참조입니다.

#### 매개변수로서의 문자열 슬라이스

리터럴과 `String` 값 모두의 슬라이스를 만들 수 있다는 사실로부터, `first_word`에
대한 또 하나의 개선점이 나옵니다. 바로 그 시그니처입니다.

```rust,ignore
fn first_word(s: &String) -> &str {
```

좀 더 경험 많은 러스타시안은 대신 Listing 4-9에 보이는 시그니처를 쓸 것입니다. 이
시그니처는 `&String` 값과 `&str` 값 모두에 같은 함수를 사용할 수 있게 해 주기
때문입니다.

<Listing number="4-9" caption="`s` 매개변수의 타입으로 문자열 슬라이스를 사용해 `first_word` 함수 개선하기">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

문자열 슬라이스가 있다면 그것을 바로 전달할 수 있습니다. `String`이 있다면 그
`String`의 슬라이스나 `String`에 대한 참조를 전달할 수 있습니다. 이런 유연성은
15장의 [“함수와 메서드에서의 Deref 강제 변환
사용”][deref-coercions]<!-- ignore --> 절에서 다룰 Deref 강제 변환(deref
coercion) 기능을 활용합니다.

함수가 `String`의 참조 대신 문자열 슬라이스를 받도록 정의하면, 어떤 기능도 잃지
않으면서 API를 더 일반적이고 유용하게 만들 수 있습니다.

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### 그 외의 슬라이스

짐작하시겠지만, 문자열 슬라이스는 문자열에 특화된 것입니다. 하지만 더 일반적인
슬라이스 타입도 있습니다. 다음 배열을 봅시다.

```rust
let a = [1, 2, 3, 4, 5];
```

문자열의 일부를 참조하고 싶은 것처럼, 배열의 일부를 참조하고 싶을 수도 있습니다.
다음과 같이 할 수 있습니다.

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

이 슬라이스는 `&[i32]` 타입을 가집니다. 첫 요소에 대한 참조와 길이를 저장한다는
점에서 문자열 슬라이스와 똑같이 동작합니다. 이런 종류의 슬라이스를 다른 모든
종류의 컬렉션에도 사용하게 될 것입니다. 이 컬렉션들은 8장에서 벡터를 이야기할
때 자세히 다룹니다.

## 요약

소유권, 빌림, 슬라이스라는 개념은 러스트 프로그램에서 컴파일 타임에 메모리
안전성을 보장합니다. 러스트는 다른 시스템 프로그래밍 언어들과 마찬가지로 메모리
사용에 대한 제어권을 제공합니다. 하지만 데이터의 소유자가 스코프를 벗어날 때
그 데이터를 자동으로 정리해 준다는 것은, 이 제어권을 얻기 위해 추가 코드를 작성
하고 디버깅하지 않아도 된다는 뜻입니다.

소유권은 러스트의 다른 많은 부분이 동작하는 방식에도 영향을 주므로, 이 개념들에
대해 책의 나머지 부분에서도 계속 이야기할 것입니다. 5장으로 넘어가 `struct`로
데이터 조각들을 묶는 것을 살펴봅시다.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#using-deref-coercions-in-functions-and-methods
