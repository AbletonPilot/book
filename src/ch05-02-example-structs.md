## 구조체를 사용하는 예제 프로그램

구조체가 언제 쓰고 싶어지는지 이해하기 위해, 사각형의 넓이를 계산하는 프로그램을
작성해 봅시다. 개별 변수로 시작한 뒤, 구조체를 사용하는 형태로 프로그램을
리팩터링해 나가겠습니다.

Cargo로 *rectangles*라는 새 바이너리 프로젝트를 만듭시다. 이 프로젝트는 픽셀
단위로 지정된 사각형의 너비와 높이를 받아 사각형의 넓이를 계산할 것입니다. Listing
5-8은 프로젝트의 *src/main.rs*에서 정확히 그 일을 수행하는 한 방법을 짧은 프로그램
으로 보여 줍니다.

<Listing number="5-8" file-name="src/main.rs" caption="분리된 너비와 높이 변수로 지정된 사각형의 넓이 계산하기">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

이제 `cargo run`으로 이 프로그램을 실행해 봅시다.

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

이 코드는 `area` 함수에 각 치수를 건네 사각형의 넓이를 계산해 내는 데에는 성공
하지만, 코드를 더 명확하고 읽기 쉽게 만들 여지가 더 있습니다.

이 코드의 문제는 `area`의 시그니처에서 드러납니다.

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

`area` 함수는 한 사각형의 넓이를 계산해야 하지만, 우리가 작성한 함수는 두 개의
매개변수를 가지며, 프로그램 어디에서도 이 매개변수들이 서로 관련되어 있다는 것이
명확하지 않습니다. 너비와 높이를 함께 묶는 편이 더 읽기 쉽고 관리하기 쉬울 것
입니다. 그렇게 할 수 있는 한 가지 방법은 3장의 [“튜플 타입”][the-tuple-type]<!--
ignore --> 절에서 이미 논의했습니다. 바로 튜플을 사용하는 것입니다.

### 튜플로 리팩터링하기

Listing 5-9는 튜플을 사용하는 또 다른 버전의 프로그램을 보여 줍니다.

<Listing number="5-9" file-name="src/main.rs" caption="튜플로 사각형의 너비와 높이 지정하기">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

어떤 면에서 이 프로그램은 더 낫습니다. 튜플로 약간의 구조를 추가할 수 있고, 이제
인자도 하나만 넘기면 됩니다. 하지만 다른 면에서는 이 버전이 덜 명확합니다. 튜플은
요소에 이름을 붙이지 않기 때문에, 튜플의 부분에 인덱스로 접근해야 해서 계산이 덜
분명해집니다.

넓이를 계산할 때에는 너비와 높이를 뒤섞어도 상관없지만, 사각형을 화면에 그리고
싶다면 문제가 됩니다! `width`가 튜플 인덱스 `0`이고 `height`가 튜플 인덱스 `1`
이라는 점을 계속 기억하고 있어야 합니다. 다른 누군가가 우리 코드를 사용한다면,
이를 파악하고 기억하는 것은 더욱 어려울 것입니다. 코드로 데이터의 의미를 전달하지
않았기 때문에, 에러를 들여오기가 더 쉬워졌습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="refactoring-with-structs-adding-more-meaning"></a>

### 구조체로 리팩터링하기

우리는 구조체를 사용해 데이터에 라벨을 붙여 의미를 더합니다. Listing 5-10에 보이
듯이, 사용 중인 튜플을 전체에 이름을 붙이고 각 부분에도 이름을 붙인 구조체로
바꿀 수 있습니다.

<Listing number="5-10" file-name="src/main.rs" caption="`Rectangle` 구조체 정의">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

여기서는 구조체를 정의하고 `Rectangle`이라고 이름 붙였습니다. 중괄호 안에는 필드로
`width`와 `height`를 정의했고, 둘 다 `u32` 타입을 가집니다. 그런 다음 `main`에서
너비 `30`, 높이 `50`인 특정 `Rectangle` 인스턴스를 만들었습니다.

이제 `area` 함수는 하나의 매개변수로 정의됩니다. 이름은 `rectangle`이고, 타입은
구조체 `Rectangle` 인스턴스의 불변 빌림입니다. 4장에서 언급했듯, 구조체의 소유권을
가져가지 않고 빌리고 싶기 때문입니다. 이렇게 하면 `main`이 소유권을 유지하고
`rect1`을 계속 사용할 수 있습니다. 이것이 함수 시그니처와 함수 호출 지점에서 `&`를
사용하는 이유입니다.

`area` 함수는 `Rectangle` 인스턴스의 `width`와 `height` 필드에 접근합니다(빌려진
구조체 인스턴스의 필드에 접근하는 것은 필드 값을 이동시키지 않는다는 점에 유의
하세요. 이 때문에 구조체의 빌림을 자주 보게 됩니다). 이제 `area`의 함수 시그니처는
우리가 의미하는 바를 정확히 말해 줍니다. `Rectangle`의 `width`와 `height` 필드를
사용해 넓이를 계산한다고 말이죠. 이는 너비와 높이가 서로 연관되어 있다는 것을
전달해 주고, 튜플 인덱스 값 `0`과 `1`을 쓰는 대신 값들에 설명적인 이름을 부여
합니다. 명확성 측면에서 승리입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-useful-functionality-with-derived-traits"></a>

### 파생 트레이트로 유용한 기능 추가하기

프로그램을 디버깅하는 동안 `Rectangle` 인스턴스를 출력해서 모든 필드의 값을 볼 수
있다면 유용할 것입니다. Listing 5-11은 앞선 장들에서 사용했던 [`println!`
매크로][println]<!-- ignore -->를 사용해 이를 시도합니다. 그러나 이는 동작하지
않습니다.

<Listing number="5-11" file-name="src/main.rs" caption="`Rectangle` 인스턴스 출력 시도">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

이 코드를 컴파일하면 다음 핵심 메시지와 함께 에러가 납니다.

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

`println!` 매크로는 여러 종류의 포매팅을 할 수 있고, 기본적으로 중괄호는 `println!`
에게 `Display`로 알려진 포매팅을 사용하라고 알립니다. 이는 최종 사용자가 직접 소비
하도록 의도된 출력입니다. 지금까지 본 원시 타입들은 기본적으로 `Display`를 구현
합니다. `1`이나 다른 어떤 원시 타입을 사용자에게 보여 주고 싶은 방식은 하나뿐이기
때문이죠. 하지만 구조체의 경우, `println!`이 출력을 어떻게 포매팅해야 할지가 덜
명확합니다. 표시 가능성이 더 많기 때문이죠. 쉼표를 원하는지 아닌지? 중괄호를
출력할 것인지? 모든 필드를 다 보여 줘야 하는지? 이런 모호성 때문에, 러스트는 우리가
원하는 것을 추측하려 하지 않으며, 구조체는 `println!`과 `{}` 자리표시자에 사용할
수 있는 `Display` 구현을 기본 제공하지 않습니다.

에러를 계속 읽으면 다음의 도움이 되는 노트를 발견합니다.

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

시도해 봅시다! `println!` 매크로 호출은 이제 `println!("rect1 is {rect1:?}");`
처럼 생겼습니다. 중괄호 안에 지정자 `:?`를 넣으면, `println!`에게 `Debug`라는
출력 포맷을 사용하고 싶다는 것을 알립니다. `Debug` 트레이트는 개발자가 코드를
디버깅하는 동안 값의 내용을 볼 수 있도록, 개발자에게 유용한 방식으로 구조체를
출력할 수 있게 해 줍니다.

이 변경으로 코드를 컴파일해 봅시다. 이런! 여전히 에러가 납니다.

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

다시 컴파일러가 도움이 되는 노트를 줍니다.

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

러스트는 디버깅 정보를 출력하는 기능을 *실제로* 포함하고 있지만, 우리 구조체에
그 기능을 쓸 수 있게 하려면 명시적으로 옵트인해야 합니다. 그렇게 하려면 Listing
5-12에 보이듯이 구조체 정의 바로 앞에 외부 속성(outer attribute) `#[derive(Debug)]`
를 추가합니다.

<Listing number="5-12" file-name="src/main.rs" caption="`Debug` 트레이트를 파생하는 속성을 추가하고 디버그 포매팅을 사용해 `Rectangle` 인스턴스 출력하기">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

이제 프로그램을 실행하면 에러가 발생하지 않고, 다음 출력을 볼 수 있습니다.

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

좋습니다! 가장 예쁜 출력은 아니지만, 이 인스턴스의 모든 필드 값을 보여 주므로
디버깅에 확실히 도움이 됩니다. 더 큰 구조체를 가지게 되면, 조금 더 읽기 쉬운
출력이 유용합니다. 그런 경우에는 `println!` 문자열에서 `{:?}` 대신 `{:#?}`를
사용할 수 있습니다. 이 예에서 `{:#?}` 스타일을 사용하면 다음과 같은 출력이
나옵니다.

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

`Debug` 포맷으로 값을 출력하는 또 다른 방법은 [`dbg!` 매크로][dbg]<!-- ignore -->
를 사용하는 것입니다. 이 매크로는 (`println!`이 참조를 받는 것과는 달리) 표현식의
소유권을 가져와서, 여러분의 코드에서 그 `dbg!` 매크로 호출이 일어난 파일과 줄
번호를 그 표현식의 결과 값과 함께 출력하고, 그 값의 소유권을 다시 돌려줍니다.

> 참고: `dbg!` 매크로 호출은 표준 출력 콘솔 스트림(`stdout`)에 출력하는
> `println!`과 달리, 표준 에러 콘솔 스트림(`stderr`)에 출력합니다. `stderr`와
> `stdout`에 대해서는 [12장의 “에러를 표준 에러로 리다이렉트하기”][err]<!-- ignore
> --> 절에서 더 이야기합니다.

다음은 `width` 필드에 할당되는 값과, `rect1`에 있는 구조체 전체의 값에 관심이
있는 예입니다.

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

`30 * scale` 표현식 주위에 `dbg!`를 두고, `dbg!`가 표현식 값의 소유권을 다시
돌려주기 때문에 `width` 필드는 `dbg!` 호출이 없었을 때와 같은 값을 받게 됩니다.
`dbg!`가 `rect1`의 소유권을 가져가는 것은 원하지 않으므로, 다음 호출에서는 `rect1`의
참조를 사용합니다. 이 예의 출력은 다음과 같습니다.

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

첫 부분 출력은 *src/main.rs*의 10번째 줄에서 왔고, 거기서 `30 * scale` 표현식을
디버깅하고 있으며 그 결과 값은 `60`입니다(정수에 구현된 `Debug` 포매팅은 값만 출력
합니다). *src/main.rs*의 14번째 줄에 있는 `dbg!` 호출은 `&rect1`의 값을 출력하는데,
이는 `Rectangle` 구조체입니다. 이 출력은 `Rectangle` 타입의 예쁜 `Debug` 포매팅을
사용합니다. `dbg!` 매크로는 여러분의 코드가 무엇을 하고 있는지 파악하려고 할 때
정말 유용할 수 있습니다!

`Debug` 트레이트 외에도, 러스트는 우리가 커스텀 타입에 유용한 동작을 추가할 수
있는 여러 트레이트를 `derive` 속성과 함께 사용할 수 있도록 제공합니다. 해당 트레이트
들과 그 동작은 [부록 C][app-c]<!-- ignore -->에 나열되어 있습니다. 이 트레이트들을
커스텀 동작으로 구현하는 방법과 직접 트레이트를 만드는 방법은 10장에서 다룹니다.
`derive` 외에도 많은 속성이 있습니다. 자세한 내용은 [러스트 레퍼런스의 “Attributes”
절][attributes]을 참고하세요.

우리의 `area` 함수는 매우 특수합니다. 오직 사각형의 넓이만 계산합니다. 이 동작을
다른 어떤 타입에도 동작하지 않을 것이므로, 우리의 `Rectangle` 구조체와 더 밀접하게
묶어 두는 편이 도움이 될 것입니다. 이제 `area` 함수를 `Rectangle` 타입에 정의된
`area` 메서드로 바꾸며 이 코드를 리팩터링하는 방법을 살펴봅시다.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
