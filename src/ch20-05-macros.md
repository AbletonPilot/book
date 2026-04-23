## 매크로

이 책 전반에 걸쳐 `println!` 같은 매크로를 사용해 왔지만, 매크로가 무엇이고 어떻게 동작하는지 충분히 살펴보지는 않았습니다. _매크로_ 라는 용어는 러스트의 한 무리의 기능들을 가리킵니다. `macro_rules!`로 만드는 선언적 매크로(declarative macros)와 세 가지 종류의 절차적 매크로(procedural macros)입니다.

- 구조체와 열거형에 사용되는 `derive` 어트리뷰트로 추가되는 코드를 명시하는 사용자 정의 `#[derive]` 매크로
- 어떤 항목에든 사용할 수 있는 사용자 정의 어트리뷰트를 정의하는 어트리뷰트 유사 매크로
- 함수 호출처럼 보이지만 인수로 명시된 토큰들에 동작하는 함수 유사 매크로

각각을 차례로 이야기하겠지만, 그 전에 함수가 이미 있는데 왜 매크로가 필요한지부터 살펴봅시다.

### 매크로와 함수의 차이

기본적으로 매크로는 다른 코드를 작성하는 코드를 작성하는 한 방식이며, 이를 _메타프로그래밍(metaprogramming)_ 이라고 합니다. 부록 C에서는 다양한 트레이트의 구현을 자동으로 생성해 주는 `derive` 어트리뷰트를 논의합니다. 책 전반에 걸쳐 `println!`과 `vec!` 매크로도 사용해 왔습니다. 이 모든 매크로들은 _확장(expand)_ 되어, 우리가 수동으로 작성한 것보다 더 많은 코드를 만들어 냅니다.

메타프로그래밍은 작성하고 유지해야 하는 코드의 양을 줄이는 데 유용한데, 이는 함수의 역할 중 하나이기도 합니다. 그러나 매크로에는 함수가 가지지 못한 추가적인 능력이 있습니다.

함수 시그니처는 함수가 가지는 매개변수의 개수와 타입을 선언해야 합니다. 반면 매크로는 가변 개수의 매개변수를 받을 수 있습니다. `println!("hello")`처럼 인수 하나로 호출할 수도 있고 `println!("hello {}", name)`처럼 두 인수로 호출할 수도 있습니다. 또한 매크로는 컴파일러가 코드의 의미를 해석하기 전에 확장되므로, 예를 들어 매크로는 주어진 타입에 트레이트를 구현할 수 있습니다. 함수는 런타임에 호출되고 트레이트는 컴파일 시점에 구현되어야 하므로 함수는 그렇게 할 수 없습니다.

함수 대신 매크로를 구현하는 단점은 매크로 정의가 함수 정의보다 더 복잡하다는 점입니다. 러스트 코드를 작성하는 러스트 코드를 작성하기 때문입니다. 이러한 간접성 때문에, 매크로 정의는 일반적으로 함수 정의보다 읽고 이해하고 유지하기 더 어렵습니다.

매크로와 함수의 또 다른 중요한 차이는, 함수는 어디에서나 정의하고 어디에서나 호출할 수 있는 반면, 매크로는 파일에서 호출하기 _전에_ 정의하거나 스코프로 가져와야 한다는 것입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="declarative-macros-with-macro_rules-for-general-metaprogramming"></a>

### 일반 메타프로그래밍을 위한 선언적 매크로

러스트에서 가장 널리 사용되는 형태의 매크로는 _선언적 매크로(declarative macro)_ 입니다. 이를 “예시에 의한 매크로(macros by example)”, “`macro_rules!` 매크로”, 또는 그냥 “매크로”라고 부르기도 합니다. 본질적으로 선언적 매크로는 러스트 `match` 표현식과 비슷한 것을 작성할 수 있게 해 줍니다. 6장에서 논의했듯, `match` 표현식은 표현식을 받아, 표현식의 결과 값을 패턴들과 비교하고, 매칭되는 패턴과 연관된 코드를 실행하는 제어 구조입니다. 매크로 또한 값을 특정 코드와 연관된 패턴들과 비교합니다. 이 상황에서 값은 매크로에 전달된 리터럴 러스트 소스 코드이고, 패턴은 그 소스 코드의 구조와 비교되며, 매칭되었을 때 각 패턴과 연관된 코드가 매크로에 전달된 코드를 대체합니다. 이 모든 일은 컴파일 동안 일어납니다.

매크로를 정의하기 위해 `macro_rules!` 구문을 사용합니다. `vec!` 매크로가 어떻게 정의되어 있는지를 보면서 `macro_rules!` 사용 방법을 살펴봅시다. 8장에서는 특정 값들로 새 벡터를 만들기 위해 `vec!` 매크로를 어떻게 사용하는지 다루었습니다. 예를 들어, 다음 매크로는 세 정수를 담는 새 벡터를 만듭니다.

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

`vec!` 매크로를 사용해 두 정수의 벡터나 다섯 문자열 슬라이스의 벡터를 만들 수도 있을 것입니다. 함수로는 같은 일을 할 수 없을 것입니다. 미리 값들의 개수나 타입을 알 수 없기 때문입니다.

Listing 20-35는 `vec!` 매크로를 약간 단순화한 정의를 보여 줍니다.

<Listing number="20-35" file-name="src/lib.rs" caption="`vec!` 매크로 정의의 단순화된 버전">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> 참고: 표준 라이브러리의 실제 `vec!` 매크로 정의에는 미리 정확한 양의 메모리를 할당하는 코드가 포함되어 있습니다. 그 코드는 최적화로, 예시를 더 단순하게 만들기 위해 여기서는 포함하지 않았습니다.

`#[macro_export]` 어노테이션은 매크로가 정의된 크레이트가 스코프로 들어올 때마다 이 매크로가 사용 가능해야 함을 나타냅니다. 이 어노테이션이 없다면 매크로는 스코프로 들어올 수 없습니다.

그런 다음 `macro_rules!`와 우리가 정의하는 매크로의 이름(느낌표 _없이_)으로 매크로 정의를 시작합니다. 이 경우 이름은 `vec`이며, 매크로 정의의 본문을 나타내는 중괄호가 따라옵니다.

`vec!` 본문의 구조는 `match` 표현식의 구조와 비슷합니다. 여기서는 패턴 `( $( $x:expr ),* )`를 가진 갈래 하나가 있고, 그 뒤에 `=>`와 이 패턴과 연관된 코드 블록이 옵니다. 패턴이 매칭되면 연관된 코드 블록이 방출됩니다. 이것이 이 매크로의 유일한 패턴이므로, 매칭되는 유효한 방법은 단 하나뿐입니다. 다른 어떤 패턴도 오류를 일으킬 것입니다. 더 복잡한 매크로는 갈래가 둘 이상 있을 것입니다.

매크로 정의에서 유효한 패턴 문법은, 매크로 패턴이 값이 아니라 러스트 코드 구조와 매칭되기 때문에 19장에서 다룬 패턴 문법과 다릅니다. Listing 20-29의 패턴 조각들이 무엇을 의미하는지 따라가 봅시다. 매크로의 전체 패턴 문법은 [러스트 참조 문서][ref]를 참고하세요.

먼저, 전체 패턴을 감싸기 위해 괄호 한 쌍을 사용합니다. 패턴과 매칭되는 러스트 코드를 담을 매크로 시스템의 변수를 선언하기 위해 달러 기호(`$`)를 사용합니다. 달러 기호는 이것이 일반 러스트 변수와 달리 매크로 변수임을 분명히 합니다. 다음으로, 대체 코드에서 사용하기 위해 괄호 안의 패턴과 매칭되는 값을 캡처하는 괄호 한 쌍이 옵니다. `$()` 안에는 `$x:expr`이 있는데, 이는 어떤 러스트 표현식과도 매칭되며 그 표현식에 `$x`라는 이름을 줍니다.

`$()` 다음의 쉼표는 `$()` 안의 코드와 매칭되는 코드의 각 인스턴스 사이에 리터럴 쉼표 구분자가 나타나야 함을 나타냅니다. `*`는 `*` 앞에 있는 무엇이든 0개 이상 매칭되는 패턴임을 명시합니다.

`vec![1, 2, 3];`으로 이 매크로를 호출하면, `$x` 패턴은 세 표현식 `1`, `2`, `3`과 세 번 매칭됩니다.

이제 이 갈래와 연관된 코드 본문의 패턴을 살펴봅시다. `$()*` 안의 `temp_vec.push()`는 패턴이 매칭되는 횟수에 따라 패턴에서 `$()`와 매칭되는 부분 각각에 대해 0번 이상 생성됩니다. `$x`는 매칭된 각 표현식으로 대체됩니다. `vec![1, 2, 3];`으로 이 매크로를 호출하면, 이 매크로 호출을 대체하는 생성된 코드는 다음과 같을 것입니다.

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

어떤 타입의 인수를 몇 개든 받을 수 있고, 명시된 요소들을 담은 벡터를 만드는 코드를 생성할 수 있는 매크로를 정의했습니다.

매크로 작성 방법에 대해 더 알고 싶다면, 온라인 문서나 Daniel Keep이 시작하고 Lukas Wirth가 이어 가는 [“The Little Book of Rust Macros”][tlborm] 같은 다른 자료를 참고하세요.

### 어트리뷰트로부터 코드를 생성하는 절차적 매크로

매크로의 두 번째 형태는 절차적 매크로입니다. 이는 함수처럼 동작합니다(그리고 일종의 절차입니다). _절차적 매크로(procedural macros)_ 는 선언적 매크로처럼 패턴에 매칭하고 코드를 다른 코드로 대체하는 대신, 어떤 코드를 입력으로 받아 그 코드에 대해 동작하고 어떤 코드를 출력으로 만들어 냅니다. 절차적 매크로의 세 가지 종류는 사용자 정의 `derive`, 어트리뷰트 유사, 함수 유사이며, 모두 비슷한 방식으로 동작합니다.

절차적 매크로를 만들 때는 정의가 특별한 크레이트 타입을 가진 자기만의 크레이트에 있어야 합니다. 이는 미래에는 없애기를 바라는 복잡한 기술적 이유 때문입니다. Listing 20-36에서는 절차적 매크로를 정의하는 방법을 보여 주는데, `some_attribute`는 특정 매크로 종류를 사용한다는 자리표시자입니다.

<Listing number="20-36" file-name="src/lib.rs" caption="절차적 매크로 정의의 한 예">

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

절차적 매크로를 정의하는 함수는 `TokenStream`을 입력으로 받고 `TokenStream`을 출력으로 만들어 냅니다. `TokenStream` 타입은 러스트와 함께 포함되는 `proc_macro` 크레이트가 정의하며, 토큰들의 시퀀스를 나타냅니다. 이것이 매크로의 핵심입니다. 매크로가 동작 중인 소스 코드는 입력 `TokenStream`을 구성하고, 매크로가 만들어 내는 코드는 출력 `TokenStream`입니다. 함수에는 또한 우리가 만드는 절차적 매크로의 종류를 명시하는 어트리뷰트가 붙어 있습니다. 같은 크레이트에 여러 종류의 절차적 매크로를 가질 수 있습니다.

절차적 매크로의 다양한 종류를 살펴봅시다. 사용자 정의 `derive` 매크로부터 시작하여, 다른 형태들이 다른 점들을 작은 차이를 들어 설명하겠습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="how-to-write-a-custom-derive-macro"></a>

### 사용자 정의 `derive` 매크로

`hello_macro`라는 크레이트를 만들고, `hello_macro`라는 연관 함수 하나를 가진 `HelloMacro`라는 트레이트를 정의하겠습니다. 사용자가 자신의 각 타입에 대해 `HelloMacro` 트레이트를 구현하게 만드는 대신, 절차적 매크로를 제공해 사용자가 자신의 타입에 `#[derive(HelloMacro)]`를 어노테이트해 `hello_macro` 함수의 기본 구현을 얻을 수 있게 할 것입니다. 기본 구현은 `Hello, Macro! My name is TypeName!`을 출력할 텐데, 여기서 `TypeName`은 이 트레이트가 정의된 타입의 이름입니다. 다시 말해, 우리는 다른 프로그래머가 우리 크레이트를 사용해 Listing 20-37과 같은 코드를 작성할 수 있게 해 주는 크레이트를 작성할 것입니다.

<Listing number="20-37" file-name="src/main.rs" caption="우리 절차적 매크로를 사용할 때 우리 크레이트의 사용자가 작성할 수 있게 될 코드">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

이 코드는 우리가 끝마치고 나면 `Hello, Macro! My name is Pancakes!`를 출력할 것입니다. 첫 단계는 다음과 같이 새 라이브러리 크레이트를 만드는 것입니다.

```console
$ cargo new hello_macro --lib
```

다음으로 Listing 20-38에서 `HelloMacro` 트레이트와 그 연관 함수를 정의합니다.

<Listing file-name="src/lib.rs" number="20-38" caption="`derive` 매크로와 함께 사용할 단순한 트레이트">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

트레이트와 그 함수를 가지고 있습니다. 이 시점에서 우리 크레이트 사용자는 Listing 20-39처럼 트레이트를 구현해 원하는 기능을 얻을 수 있습니다.

<Listing number="20-39" file-name="src/main.rs" caption="사용자가 `HelloMacro` 트레이트의 수동 구현을 작성한 모습">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

그러나 `hello_macro`와 함께 사용하고 싶은 각 타입에 대해 구현 블록을 작성해야 할 것입니다. 우리는 사용자가 이 작업을 해야 하는 수고를 덜어 주고 싶습니다.

또한 트레이트가 구현된 타입의 이름을 출력할 기본 구현으로 `hello_macro` 함수를 아직 제공할 수 없습니다. 러스트는 리플렉션 능력을 가지고 있지 않으므로, 런타임에 타입의 이름을 조회할 수 없기 때문입니다. 컴파일 시점에 코드를 생성할 매크로가 필요합니다.

다음 단계는 절차적 매크로를 정의하는 것입니다. 이 글을 쓰는 시점에서, 절차적 매크로는 자기만의 크레이트에 있어야 합니다. 결국 이 제한은 풀릴 수 있습니다. 크레이트와 매크로 크레이트를 구성하는 관례는 다음과 같습니다. `foo`라는 크레이트의 경우, 사용자 정의 `derive` 절차적 매크로 크레이트는 `foo_derive`라고 부릅니다. `hello_macro` 프로젝트 안에 `hello_macro_derive`라는 새 크레이트를 시작합시다.

```console
$ cargo new hello_macro_derive --lib
```

두 크레이트는 긴밀하게 관련되어 있으므로, `hello_macro` 크레이트의 디렉터리 안에 절차적 매크로 크레이트를 만듭니다. `hello_macro`의 트레이트 정의를 변경하면, `hello_macro_derive`의 절차적 매크로 구현도 변경해야 할 것입니다. 두 크레이트는 별도로 게시해야 할 것이며, 이 크레이트들을 사용하는 프로그래머들은 둘 다 의존성으로 추가하고 둘 다 스코프로 가져와야 할 것입니다. 대신 `hello_macro` 크레이트가 `hello_macro_derive`를 의존성으로 사용해 절차적 매크로 코드를 다시 내보내게 할 수도 있을 것입니다. 그러나 우리가 프로젝트를 구성한 방식은, `derive` 기능을 원하지 않는 프로그래머도 `hello_macro`를 사용할 수 있게 합니다.

`hello_macro_derive` 크레이트를 절차적 매크로 크레이트로 선언해야 합니다. 또한 곧 보겠지만 `syn`과 `quote` 크레이트의 기능이 필요하므로, 그것들을 의존성으로 추가해야 합니다. `hello_macro_derive`의 _Cargo.toml_ 파일에 다음을 추가합니다.

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

절차적 매크로 정의를 시작하기 위해, Listing 20-40의 코드를 `hello_macro_derive` 크레이트의 _src/lib.rs_ 파일에 넣습니다. 이 코드는 `impl_hello_macro` 함수의 정의를 추가하기 전까지는 컴파일되지 않을 것입니다.

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="대부분의 절차적 매크로 크레이트가 러스트 코드를 처리하기 위해 필요할 코드">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

코드를 `TokenStream` 파싱을 책임지는 `hello_macro_derive` 함수와 구문 트리를 변환하는 책임을 지는 `impl_hello_macro` 함수로 분리했음에 주목하세요. 이렇게 하면 절차적 매크로를 작성하기가 더 편리해집니다. 외부 함수(이 경우 `hello_macro_derive`)의 코드는 여러분이 보거나 만드는 거의 모든 절차적 매크로 크레이트에서 같을 것입니다. 내부 함수(이 경우 `impl_hello_macro`)의 본문에 명시하는 코드는 여러분의 절차적 매크로의 목적에 따라 다를 것입니다.

세 새로운 크레이트가 등장했습니다. `proc_macro`, [`syn`][syn]<!-- ignore -->, [`quote`][quote]<!-- ignore -->입니다. `proc_macro` 크레이트는 러스트와 함께 제공되므로, _Cargo.toml_ 의존성에 추가할 필요가 없었습니다. `proc_macro` 크레이트는 우리 코드에서 러스트 코드를 읽고 조작할 수 있게 해 주는 컴파일러의 API입니다.

`syn` 크레이트는 문자열에서 러스트 코드를 파싱해 우리가 연산을 수행할 수 있는 자료 구조로 만들어 줍니다. `quote` 크레이트는 `syn` 자료 구조를 다시 러스트 코드로 변환합니다. 이러한 크레이트들은 우리가 다루고 싶을 어떤 종류의 러스트 코드든 파싱하는 것을 훨씬 더 단순하게 만들어 줍니다. 러스트 코드를 위한 완전한 파서를 작성하는 것은 단순한 작업이 아닙니다.

`hello_macro_derive` 함수는 우리 라이브러리의 사용자가 어떤 타입에 `#[derive(HelloMacro)]`를 명시할 때 호출됩니다. 여기서 `hello_macro_derive` 함수에 `proc_macro_derive`를 어노테이트하고 우리 트레이트 이름과 일치하는 `HelloMacro`라는 이름을 명시했기 때문에 가능합니다. 이는 대부분의 절차적 매크로가 따르는 관례입니다.

`hello_macro_derive` 함수는 먼저 `input`을 `TokenStream`에서 그 다음 우리가 해석하고 연산을 수행할 수 있는 자료 구조로 변환합니다. 여기서 `syn`이 등장합니다. `syn`의 `parse` 함수는 `TokenStream`을 받아 파싱된 러스트 코드를 나타내는 `DeriveInput` 구조체를 반환합니다. Listing 20-41은 `struct Pancakes;` 문자열을 파싱해서 얻는 `DeriveInput` 구조체의 관련 부분들을 보여 줍니다.

<Listing number="20-41" caption="Listing 20-37에서 매크로의 어트리뷰트가 있는 코드를 파싱했을 때 얻는 `DeriveInput` 인스턴스">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

이 구조체의 필드들은 우리가 파싱한 러스트 코드가 `Pancakes`라는 `ident`(_식별자_, 즉 이름)를 가진 단위 구조체임을 보여 줍니다. 이 구조체에는 모든 종류의 러스트 코드를 설명하는 더 많은 필드가 있습니다. 자세한 정보는 [`DeriveInput`에 대한 `syn` 문서][syn-docs]를 확인하세요.

곧 `impl_hello_macro` 함수를 정의할 텐데, 거기에서 우리가 포함하고 싶은 새 러스트 코드를 만들게 됩니다. 그 전에 우리 `derive` 매크로의 출력 또한 `TokenStream`임에 주목하세요. 반환된 `TokenStream`은 우리 크레이트 사용자가 작성하는 코드에 추가되므로, 그들이 자신의 크레이트를 컴파일하면, 수정된 `TokenStream`에서 우리가 제공한 추가 기능을 얻게 됩니다.

여기서 `syn::parse` 함수에 대한 호출이 실패하면 `hello_macro_derive` 함수가 패닉하도록 `unwrap`을 호출하고 있다는 점을 알아챘을 수도 있습니다. `proc_macro_derive` 함수는 절차적 매크로 API에 부합하기 위해 `Result`가 아니라 `TokenStream`을 반환해야 하므로, 우리 절차적 매크로가 오류 시 패닉하는 것이 필요합니다. 이 예제는 `unwrap`을 사용해 단순화했습니다. 운영 코드에서는 `panic!`이나 `expect`를 사용해 무엇이 잘못되었는지에 대한 더 구체적인 오류 메시지를 제공해야 합니다.

이제 어노테이트된 러스트 코드를 `TokenStream`에서 `DeriveInput` 인스턴스로 바꾸는 코드를 가졌으니, Listing 20-42에서 보이듯 어노테이트된 타입에 `HelloMacro` 트레이트를 구현하는 코드를 생성합시다.

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="파싱된 러스트 코드를 사용해 `HelloMacro` 트레이트 구현하기">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

`ast.ident`를 사용해 어노테이트된 타입의 이름(식별자)을 담은 `Ident` 구조체 인스턴스를 얻습니다. Listing 20-41의 구조체는 Listing 20-37의 코드에 대해 `impl_hello_macro` 함수를 실행하면 얻는 `ident`가 `"Pancakes"`라는 값을 가지는 `ident` 필드를 가지게 됨을 보여 줍니다. 따라서 Listing 20-42의 `name` 변수는 출력하면 Listing 20-37의 구조체 이름인 문자열 `"Pancakes"`가 되는 `Ident` 구조체 인스턴스를 담게 됩니다.

`quote!` 매크로는 우리가 반환하고 싶은 러스트 코드를 정의할 수 있게 해 줍니다. 컴파일러는 `quote!` 매크로 실행의 직접적인 결과와는 다른 무언가를 기대하므로, 그것을 `TokenStream`으로 변환해야 합니다. `into` 메서드를 호출해 이 중간 표현을 소비하고 요구되는 `TokenStream` 타입의 값을 반환하도록 합니다.

`quote!` 매크로는 또한 매우 멋진 템플릿 메커니즘을 제공합니다. `#name`을 입력하면 `quote!`가 변수 `name`의 값으로 그것을 대체합니다. 일반 매크로가 동작하는 방식과 비슷한 반복도 일부 할 수 있습니다. 자세한 소개는 [`quote` 크레이트의 문서][quote-docs]를 확인하세요.

우리는 우리 절차적 매크로가 사용자가 어노테이트한 타입에 우리 `HelloMacro` 트레이트의 구현을 생성하기를 원합니다. `#name`을 사용해 그 타입을 얻을 수 있습니다. 트레이트 구현에는 `hello_macro` 함수 하나가 있고, 그 본문에는 우리가 제공하고자 하는 기능, 즉 `Hello, Macro! My name is`를 출력한 뒤 어노테이트된 타입의 이름을 출력하는 기능이 들어 있습니다.

여기서 사용된 `stringify!` 매크로는 러스트에 내장되어 있습니다. `1 + 2` 같은 러스트 표현식을 받아, 컴파일 시점에 그 표현식을 `"1 + 2"` 같은 문자열 리터럴로 변환합니다. 이는 표현식을 평가한 다음 그 결과를 `String`으로 변환하는 매크로인 `format!`이나 `println!`과 다릅니다. `#name` 입력이 문자 그대로 출력할 표현식일 가능성이 있으므로 `stringify!`를 사용합니다. 또한 `stringify!`를 사용하면 `#name`을 컴파일 시점에 문자열 리터럴로 변환함으로써 할당을 줄일 수 있습니다.

이 시점에서 `cargo build`는 `hello_macro`와 `hello_macro_derive` 모두에서 성공적으로 완료되어야 합니다. 이 크레이트들을 Listing 20-37의 코드에 연결해 절차적 매크로의 동작을 보겠습니다! _projects_ 디렉터리에 `cargo new pancakes`를 사용해 새 바이너리 프로젝트를 만듭니다. `pancakes` 크레이트의 _Cargo.toml_ 에 `hello_macro`와 `hello_macro_derive`를 의존성으로 추가해야 합니다. 여러분이 자신의 `hello_macro`와 `hello_macro_derive` 버전을 [crates.io](https://crates.io/)<!-- ignore -->에 게시하고 있다면, 그것들은 일반 의존성이 될 것입니다. 그렇지 않다면 다음과 같이 `path` 의존성으로 명시할 수 있습니다.

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

Listing 20-37의 코드를 _src/main.rs_ 에 넣고 `cargo run`을 실행합니다. `Hello, Macro! My name is Pancakes!`가 출력될 것입니다. 절차적 매크로의 `HelloMacro` 트레이트 구현이 `pancakes` 크레이트가 그것을 구현할 필요 없이 포함되었습니다. `#[derive(HelloMacro)]`가 트레이트 구현을 추가했습니다.

다음으로, 다른 종류의 절차적 매크로가 사용자 정의 `derive` 매크로와 어떻게 다른지 살펴봅시다.

### 어트리뷰트 유사 매크로

어트리뷰트 유사 매크로는 사용자 정의 `derive` 매크로와 비슷하지만, `derive` 어트리뷰트를 위한 코드를 생성하는 대신 새로운 어트리뷰트를 만들 수 있게 해 줍니다. 또한 더 유연합니다. `derive`는 구조체와 열거형에만 동작하지만, 어트리뷰트는 함수 같은 다른 항목에도 적용될 수 있습니다. 다음은 어트리뷰트 유사 매크로를 사용하는 예입니다. 웹 애플리케이션 프레임워크를 사용할 때 함수에 어노테이트하는 `route`라는 어트리뷰트가 있다고 합시다.

```rust,ignore
#[route(GET, "/")]
fn index() {
```

이 `#[route]` 어트리뷰트는 프레임워크가 절차적 매크로로 정의할 것입니다. 매크로 정의 함수의 시그니처는 다음과 같이 보일 것입니다.

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

여기서는 `TokenStream` 타입의 매개변수가 두 개 있습니다. 첫 번째는 어트리뷰트의 내용, 즉 `GET, "/"` 부분입니다. 두 번째는 어트리뷰트가 붙은 항목의 본문, 이 경우 `fn index() {}`와 함수 본문의 나머지입니다.

그 외에는, 어트리뷰트 유사 매크로는 사용자 정의 `derive` 매크로와 같은 방식으로 동작합니다. `proc-macro` 크레이트 타입의 크레이트를 만들고 원하는 코드를 생성하는 함수를 구현합니다!

### 함수 유사 매크로

함수 유사 매크로는 함수 호출처럼 보이는 매크로를 정의합니다. `macro_rules!` 매크로와 비슷하게, 함수보다 더 유연합니다. 예를 들어 알 수 없는 수의 인수를 받을 수 있습니다. 그러나 `macro_rules!` 매크로는 앞의 [“일반 메타프로그래밍을 위한 선언적 매크로”][decl]<!-- ignore --> 절에서 논의한 매치 유사 문법을 사용해서만 정의할 수 있습니다. 함수 유사 매크로는 `TokenStream` 매개변수를 받고, 정의에서 다른 두 종류의 절차적 매크로처럼 러스트 코드를 사용해 그 `TokenStream`을 조작합니다. 함수 유사 매크로의 한 예는 다음과 같이 호출될 수 있는 `sql!` 매크로입니다.

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

이 매크로는 그 안의 SQL 문을 파싱하고 문법적으로 올바른지 검사할 텐데, 이는 `macro_rules!` 매크로가 할 수 있는 것보다 훨씬 더 복잡한 처리입니다. `sql!` 매크로는 다음과 같이 정의될 것입니다.

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

이 정의는 사용자 정의 `derive` 매크로의 시그니처와 비슷합니다. 괄호 안의 토큰들을 받아서 우리가 생성하고자 했던 코드를 반환합니다.

## 요약

휴! 이제 자주 사용하지는 않을지도 모르지만, 매우 특정한 상황에서 사용 가능하다는 것을 알게 될 러스트 기능들이 도구함에 들어왔습니다. 오류 메시지의 제안이나 다른 사람의 코드에서 마주쳤을 때 이러한 개념과 문법을 알아볼 수 있도록 몇 가지 복잡한 주제를 소개했습니다. 이 장을 해결책으로 안내하는 참고 자료로 사용하세요.

다음으로는, 이 책 전반에 걸쳐 논의한 모든 것을 실천에 옮기고 또 한 번 프로젝트를 진행하겠습니다!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
