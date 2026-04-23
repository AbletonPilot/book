## `Result`를 사용한 복구 가능한 에러

대부분의 에러는 프로그램을 완전히 멈춰야 할 만큼 심각하지는 않습니다. 때로는 함수가
실패할 때 쉽게 해석하고 대응할 수 있는 이유 때문에 실패합니다. 예를 들어, 파일을
열려고 했는데 파일이 존재하지 않아서 그 연산이 실패했다면, 프로세스를 종료하는
대신 파일을 만들고 싶을 수 있습니다.

2장의 [“`Result`로 잠재적 실패 다루기”][handle_failure]<!-- ignore -->에서 `Result`
열거형이 다음과 같이 두 개의 배리언트 `Ok`와 `Err`로 정의되어 있다는 점을 떠올려
보세요.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T`와 `E`는 제네릭 타입 매개변수입니다. 제네릭은 10장에서 더 자세히 다룹니다. 지금
알아야 할 것은, `T`가 성공한 경우 `Ok` 배리언트 안에서 반환될 값의 타입을 나타
내고, `E`가 실패한 경우 `Err` 배리언트 안에서 반환될 에러의 타입을 나타낸다는
것입니다. `Result`가 이런 제네릭 타입 매개변수를 가지기 때문에, 반환하고자 하는
성공 값과 에러 값이 서로 다를 수 있는 많은 서로 다른 상황에서 `Result` 타입과 그
위에 정의된 함수들을 사용할 수 있습니다.

함수가 실패할 수 있기 때문에 `Result` 값을 반환하는 함수를 호출해 봅시다. Listing
9-3에서는 파일을 열어 봅니다.

<Listing number="9-3" file-name="src/main.rs" caption="파일 열기">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

`File::open`의 반환 타입은 `Result<T, E>`입니다. 제네릭 매개변수 `T`는 `File::open`
의 구현에서 성공 값의 타입인 `std::fs::File`로 채워져 있습니다. 이는 파일 핸들
입니다. 에러 값에 쓰이는 `E`의 타입은 `std::io::Error`입니다. 이 반환 타입은
`File::open` 호출이 성공해서 우리가 읽거나 쓸 수 있는 파일 핸들을 반환할 수도
있다는 뜻입니다. 함수 호출은 실패할 수도 있습니다. 예를 들어, 파일이 존재하지
않거나 파일에 접근할 권한이 없을 수 있습니다. `File::open` 함수는 성공했는지 실패
했는지 알려 주면서 동시에 파일 핸들이든 에러 정보든 둘 중 하나를 우리에게 제공할
방법이 필요합니다. 이 정보가 정확히 `Result` 열거형이 전달하는 것입니다.

`File::open`이 성공한 경우 변수 `greeting_file_result`에 있는 값은 파일 핸들을
담은 `Ok` 인스턴스가 됩니다. 실패한 경우 `greeting_file_result`의 값은 어떤 종류의
에러가 일어났는지에 대한 더 많은 정보를 담은 `Err` 인스턴스가 됩니다.

`File::open`이 반환하는 값에 따라 다른 동작을 취하도록 Listing 9-3의 코드에 내용을
추가해야 합니다. Listing 9-4는 6장에서 논의한 기본 도구인 `match` 표현식을 사용해
`Result`를 처리하는 한 가지 방법을 보여 줍니다.

<Listing number="9-4" file-name="src/main.rs" caption="반환될 수 있는 `Result` 배리언트들을 처리하기 위한 `match` 표현식 사용하기">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

`Option` 열거형처럼, `Result` 열거형과 그 배리언트들이 프렐류드에 의해 스코프로
가져와져 있다는 점에 유의하세요. 그래서 `match` 갈래에서 `Ok`와 `Err` 배리언트 앞에
`Result::`를 지정할 필요가 없습니다.

결과가 `Ok`일 때, 이 코드는 `Ok` 배리언트에서 내부 `file` 값을 꺼내 반환합니다.
그런 다음 그 파일 핸들 값을 변수 `greeting_file`에 할당합니다. `match` 이후에는
읽기나 쓰기를 위해 그 파일 핸들을 사용할 수 있습니다.

`match`의 다른 갈래는 `File::open`에서 `Err` 값을 얻은 경우를 처리합니다. 이 예
에서는 `panic!` 매크로를 호출하기로 선택했습니다. 현재 디렉터리에 *hello.txt*
라는 파일이 없고 이 코드를 실행하면, `panic!` 매크로의 다음과 같은 출력을 볼 수
있습니다.

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

늘 그렇듯, 이 출력은 무엇이 잘못되었는지 정확히 알려 줍니다.

### 서로 다른 에러에 대해 매칭하기

Listing 9-4의 코드는 `File::open`이 실패한 이유가 무엇이든 `panic!`합니다. 그러나
실패 이유에 따라 다른 동작을 취하고 싶다고 해 봅시다. 파일이 존재하지 않아서
`File::open`이 실패했다면 파일을 만들고 새 파일의 핸들을 반환하고 싶습니다. 어떤
다른 이유(예: 파일을 열 권한이 없음)로 `File::open`이 실패했다면, 코드가 Listing
9-4에서 그랬던 것처럼 `panic!`하기를 원합니다. 이를 위해 Listing 9-5에 보이듯
내부 `match` 표현식을 추가합니다.

<Listing number="9-5" file-name="src/main.rs" caption="서로 다른 종류의 에러를 서로 다른 방식으로 처리하기">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

`File::open`이 `Err` 배리언트 안에 반환하는 값의 타입은 `io::Error`이며, 표준
라이브러리가 제공하는 구조체입니다. 이 구조체에는 `io::ErrorKind` 값을 얻기 위해
호출할 수 있는 `kind` 메서드가 있습니다. 표준 라이브러리가 제공하는 `io::ErrorKind`
열거형은 `io` 연산에서 발생할 수 있는 서로 다른 종류의 에러를 나타내는 배리언트들
을 가집니다. 우리가 사용하고 싶은 배리언트는 `ErrorKind::NotFound`로, 열려는
파일이 아직 존재하지 않음을 나타냅니다. 그래서 `greeting_file_result`에 매칭
하면서 `error.kind()`에도 내부 매칭을 합니다.

내부 match에서 확인하고 싶은 조건은, `error.kind()`가 반환한 값이 `ErrorKind`
열거형의 `NotFound` 배리언트인지 여부입니다. 만약 그렇다면 `File::create`로 파일
을 만들려고 시도합니다. 그러나 `File::create`도 실패할 수 있기 때문에, 내부
`match` 표현식에 두 번째 갈래가 필요합니다. 파일을 만들 수 없을 때에는 다른 에러
메시지가 출력됩니다. 외부 `match`의 두 번째 갈래는 그대로이므로, 파일이 없는 에러
를 제외한 어떤 에러에서도 프로그램은 패닉합니다.

> #### `Result<T, E>`에 `match`를 사용하는 것에 대한 대안
>
> `match`가 많네요! `match` 표현식은 매우 유용하지만 매우 원시적이기도 합니다.
> 13장에서는 `Result<T, E>`에 정의된 많은 메서드와 함께 사용되는 클로저에 대해
> 배우게 됩니다. 코드에서 `Result<T, E>` 값을 처리할 때, 이 메서드들을 사용하면
> `match`를 사용하는 것보다 더 간결할 수 있습니다.
>
> 예를 들어, 다음은 Listing 9-5와 같은 로직을 클로저와 `unwrap_or_else` 메서드를
> 사용해 쓰는 또 다른 방법입니다.
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
>
> 이 코드는 Listing 9-5와 같은 동작이지만, `match` 표현식을 하나도 담고 있지
> 않고 읽기에 더 깔끔합니다. 13장을 읽은 뒤 이 예제로 돌아와 표준 라이브러리
> 문서에서 `unwrap_or_else` 메서드를 찾아보세요. 에러를 다룰 때, 이런 많은 메서드
> 들이 거대하고 중첩된 `match` 표현식을 정리해 줄 수 있습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="shortcuts-for-panic-on-error-unwrap-and-expect"></a>

#### 에러 시 패닉을 위한 지름길

`match`를 사용하는 것은 충분히 잘 동작하지만, 다소 장황할 수 있고 의도를 늘 잘
전달하지는 않습니다. `Result<T, E>` 타입에는 다양하고 더 구체적인 작업을 위한
여러 헬퍼 메서드가 정의되어 있습니다. `unwrap` 메서드는 Listing 9-4에서 우리가
작성한 `match` 표현식과 똑같이 구현된 지름길 메서드입니다. `Result` 값이 `Ok`
배리언트라면 `unwrap`은 `Ok` 안의 값을 반환합니다. `Result`가 `Err` 배리언트라면
`unwrap`은 우리 대신 `panic!` 매크로를 호출합니다. 다음은 실제로 동작하는 `unwrap`
의 예입니다.

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

*hello.txt* 파일 없이 이 코드를 실행하면, `unwrap` 메서드가 수행하는 `panic!`
호출의 에러 메시지를 보게 됩니다.

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

비슷하게, `expect` 메서드는 `panic!` 에러 메시지도 선택할 수 있게 해 줍니다.
`unwrap` 대신 `expect`를 사용하고 좋은 에러 메시지를 제공하면 의도를 전달할 수
있고, 패닉의 원인을 추적하기 더 쉬워집니다. `expect`의 문법은 다음과 같이 생겼
습니다.

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

`expect`를 `unwrap`과 같은 방식으로 사용합니다. 파일 핸들을 반환하거나 `panic!`
매크로를 호출하는 것입니다. `expect`가 `panic!` 호출에서 사용하는 에러 메시지는,
`unwrap`이 사용하는 기본 `panic!` 메시지가 아니라, 우리가 `expect`에 전달하는
매개변수가 됩니다. 모습은 다음과 같습니다.

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

프로덕션 품질의 코드에서는, 대부분의 러스타시안이 `unwrap` 대신 `expect`를 선택해
연산이 항상 성공할 것으로 기대되는 이유에 대한 더 많은 맥락을 제공합니다. 그렇게
하면 여러분의 가정이 틀렸음이 드러나더라도 디버깅에 쓸 더 많은 정보를 갖게 됩니다.

### 에러 전파하기

함수의 구현이 실패할 수 있는 어떤 것을 호출할 때, 함수 자체에서 에러를 처리하는
대신 호출하는 코드로 에러를 반환해 그쪽이 무엇을 할지 결정하게 할 수 있습니다. 이를
에러를 *전파(propagating)*한다고 합니다. 이는 호출하는 코드에 더 많은 제어를
주는데, 그곳에는 여러분의 코드 맥락에서 가질 수 있는 것보다 에러 처리 방식을 지시
할 수 있는 정보나 로직이 더 많을 수도 있습니다.

예를 들어, Listing 9-6은 파일에서 사용자 이름을 읽는 함수를 보여 줍니다. 파일이
존재하지 않거나 읽을 수 없다면, 이 함수는 그 에러를 함수를 호출한 코드로 반환
합니다.

<Listing number="9-6" file-name="src/main.rs" caption="`match`를 사용해 호출하는 코드로 에러를 반환하는 함수">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

이 함수는 훨씬 더 짧게 쓸 수 있지만, 에러 처리를 탐구하기 위해 많은 부분을 수동
으로 하면서 시작하고 끝에 가서 더 짧은 방법을 보여 드리겠습니다. 먼저 함수의 반환
타입부터 살펴봅시다. `Result<String, io::Error>`입니다. 이는 함수가 `Result<T, E>`
타입의 값을 반환한다는 뜻이며, 제네릭 매개변수 `T`는 구체적 타입 `String`으로,
제네릭 타입 `E`는 구체적 타입 `io::Error`로 채워져 있습니다.

이 함수가 아무 문제 없이 성공하면, 이 함수를 호출하는 코드는 이 함수가 파일에서
읽은 `username`인 `String`을 담은 `Ok` 값을 받게 됩니다. 이 함수가 어떤 문제를
마주치면, 호출하는 코드는 문제가 무엇이었는지에 대한 더 많은 정보를 담은 `io::Error`
인스턴스를 담은 `Err` 값을 받게 됩니다. 이 함수의 반환 타입으로 `io::Error`를
선택한 이유는, 이 함수 본문에서 호출하는 실패할 수 있는 두 연산인 `File::open`
함수와 `read_to_string` 메서드 모두에서 반환되는 에러 값의 타입이기 때문입니다.

함수 본문은 `File::open` 함수를 호출하는 것으로 시작합니다. 그런 다음 Listing 9-4의
`match`와 비슷한 `match`로 `Result` 값을 처리합니다. `File::open`이 성공하면,
패턴 변수 `file`의 파일 핸들이 가변 변수 `username_file`의 값이 되고 함수는 계속
진행됩니다. `Err` 경우에는 `panic!`을 호출하는 대신 `return` 키워드를 사용해 함수
에서 일찍 반환하고, 이제 패턴 변수 `e`에 있는 `File::open`의 에러 값을 이 함수의
에러 값으로 호출하는 코드에 돌려줍니다.

그래서 `username_file`에 파일 핸들이 있다면, 함수는 변수 `username`에 새 `String`
을 만들고 `username_file`의 파일 핸들에 `read_to_string` 메서드를 호출해 파일의
내용을 `username`에 읽어 들입니다. `read_to_string` 메서드도 `File::open`이 성공
했더라도 실패할 수 있기 때문에 `Result`를 반환합니다. 그래서 그 `Result`를
처리하기 위한 또 다른 `match`가 필요합니다. `read_to_string`이 성공하면 우리 함수는
성공한 것이고, 파일에서 읽어 이제 `username`에 있는 사용자 이름을 `Ok`로 감싸
반환합니다. `read_to_string`이 실패하면, `File::open`의 반환값을 처리한 `match`
에서 에러 값을 반환했던 것과 같은 방식으로 에러 값을 반환합니다. 그러나 이것이
함수의 마지막 표현식이므로 명시적으로 `return`이라고 적을 필요는 없습니다.

이 코드를 호출하는 코드는 그 뒤에 사용자 이름을 담은 `Ok` 값이나 `io::Error`를
담은 `Err` 값 중 하나를 받아 처리하게 됩니다. 그 값들로 무엇을 할지는 호출하는
코드가 결정합니다. 호출하는 코드가 `Err` 값을 받으면, 예를 들어 `panic!`을 호출해
프로그램을 크래시시키거나, 기본 사용자 이름을 사용하거나, 파일이 아닌 다른 곳에서
사용자 이름을 조회할 수 있습니다. 호출하는 코드가 실제로 하려는 일에 대한 정보가
충분하지 않으므로, 모든 성공 또는 에러 정보를 위로 전파해서 그쪽이 적절히 처리
하게 합니다.

러스트에서는 이런 에러 전파 패턴이 너무나 흔하기 때문에, 러스트는 이를 더 쉽게
만드는 물음표 연산자 `?`를 제공합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-shortcut-for-propagating-errors-the--operator"></a>

#### `?` 연산자 지름길

Listing 9-7은 Listing 9-6과 같은 기능을 하는 `read_username_from_file`의 구현을
보여 주지만, 이 구현은 `?` 연산자를 사용합니다.

<Listing number="9-7" file-name="src/main.rs" caption="`?` 연산자로 호출하는 코드에 에러를 반환하는 함수">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

`Result` 값 뒤에 붙은 `?`는, Listing 9-6에서 `Result` 값을 처리하기 위해 정의한
`match` 표현식과 거의 같은 방식으로 동작하도록 정의되어 있습니다. `Result`의 값이
`Ok`라면 `Ok` 안의 값이 이 표현식에서 반환되고 프로그램은 계속됩니다. 값이 `Err`
라면, `return` 키워드를 사용한 것처럼 `Err`가 전체 함수에서 반환되어 에러 값이
호출하는 코드로 전파됩니다.

Listing 9-6의 `match` 표현식과 `?` 연산자가 하는 일에는 차이가 있습니다. `?` 연산
자가 호출된 에러 값은 표준 라이브러리의 `From` 트레이트에 정의된, 한 타입의 값을
다른 타입으로 변환하는 데 쓰이는 `from` 함수를 거쳐갑니다. `?` 연산자가 `from`
함수를 호출하면, 받은 에러 타입이 현재 함수의 반환 타입에 정의된 에러 타입으로
변환됩니다. 이는 함수가 실패할 수 있는 많은 방식을 하나의 에러 타입으로 표현하기
위해 함수가 하나의 에러 타입을 반환할 때 유용합니다. 각 부분이 여러 다른 이유로
실패할 수 있더라도 말이죠.

예를 들어, Listing 9-7의 `read_username_from_file` 함수를, 우리가 정의하는 커스텀
에러 타입 `OurError`를 반환하도록 바꿀 수 있습니다. `io::Error`로부터 `OurError`
인스턴스를 만들기 위해 `impl From<io::Error> for OurError`도 정의한다면,
`read_username_from_file` 본문의 `?` 연산자 호출들이 함수에 추가 코드를 더하지
않고도 `from`을 호출해 에러 타입을 변환합니다.

Listing 9-7의 맥락에서, `File::open` 호출 끝의 `?`는 `Ok` 안의 값을 변수
`username_file`로 반환합니다. 에러가 발생하면 `?` 연산자는 전체 함수에서 일찍
반환해 `Err` 값을 호출하는 코드에 돌려줍니다. `read_to_string` 호출 끝의 `?`에도
같은 것이 적용됩니다.

`?` 연산자는 많은 보일러플레이트를 없애고 이 함수의 구현을 더 간단하게 만듭니다.
Listing 9-8에 보이듯 `?` 뒤에 메서드 호출을 바로 이어 체이닝해서 이 코드를 더
짧게 할 수도 있습니다.

<Listing number="9-8" file-name="src/main.rs" caption="`?` 연산자 뒤에 메서드 호출 체이닝하기">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

`username`의 새 `String` 생성을 함수 시작 부분으로 옮겼습니다. 그 부분은 변하지
않았습니다. 변수 `username_file`을 만드는 대신, `read_to_string` 호출을
`File::open("hello.txt")?`의 결과에 바로 체이닝했습니다. 여전히 `read_to_string`
호출 끝에 `?`가 있고, 여전히 `File::open`과 `read_to_string`이 모두 성공하면
에러를 반환하는 대신 `username`을 담은 `Ok` 값을 반환합니다. 기능은 여전히 Listing
9-6, 9-7과 같습니다. 이것은 단지 더 다르고 더 편안한 작성 방식일 뿐입니다.

Listing 9-9는 `fs::read_to_string`을 사용해 이것을 훨씬 더 짧게 만드는 방법을 보여
줍니다.

<Listing number="9-9" file-name="src/main.rs" caption="파일을 열어 읽는 대신 `fs::read_to_string` 사용하기">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

파일을 문자열로 읽는 것은 꽤 흔한 연산이므로, 표준 라이브러리는 편리한
`fs::read_to_string` 함수를 제공합니다. 이 함수는 파일을 열고, 새 `String`을 만들고,
파일의 내용을 읽어 그 `String`에 넣고 반환합니다. 물론 `fs::read_to_string`을
사용하면 모든 에러 처리를 설명할 기회가 없으므로, 먼저 더 긴 방식으로 했습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="where-the--operator-can-be-used"></a>

#### `?` 연산자는 어디에서 사용할 수 있나

`?` 연산자는 그 `?`가 사용되는 값과 호환되는 반환 타입을 가진 함수에서만 사용할
수 있습니다. 이는 `?` 연산자가 Listing 9-6에서 정의한 `match` 표현식과 같은 방식
으로, 함수에서 값을 일찍 반환하도록 정의되어 있기 때문입니다. Listing 9-6에서
`match`는 `Result` 값을 사용하고 있었고, 일찍 반환하는 갈래는 `Err(e)` 값을 반환
했습니다. 이 `return`과 호환되려면 함수의 반환 타입이 `Result`여야 합니다.

Listing 9-10에서는, `?`를 사용하는 값의 타입과 호환되지 않는 반환 타입을 가진
`main` 함수에서 `?` 연산자를 사용할 때 어떤 에러가 나는지 살펴봅시다.

<Listing number="9-10" file-name="src/main.rs" caption="`()`를 반환하는 `main` 함수에서 `?`를 사용하려는 시도는 컴파일되지 않습니다.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

이 코드는 파일을 여는데, 이는 실패할 수 있습니다. `?` 연산자는 `File::open`이
반환하는 `Result` 값을 따르지만, 이 `main` 함수는 반환 타입이 `Result`가 아닌
`()`입니다. 이 코드를 컴파일하면 다음 에러 메시지를 받습니다.

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

이 에러는 `?` 연산자를 `Result`, `Option` 또는 `FromResidual`을 구현하는 다른
타입을 반환하는 함수에서만 쓸 수 있음을 지적합니다.

이 에러를 고치려면 두 가지 선택지가 있습니다. 한 가지 선택지는, 이를 막는 제약이
없는 한 함수의 반환 타입을 여러분이 `?` 연산자를 사용하는 값과 호환되도록 바꾸는
것입니다. 다른 선택지는 `match` 또는 `Result<T, E>` 메서드 중 하나를 사용해
적절한 어떤 방식으로든 `Result<T, E>`를 처리하는 것입니다.

에러 메시지는 또한 `?`를 `Option<T>` 값과도 함께 쓸 수 있음을 언급했습니다.
`Result`에서 `?`를 쓰는 것과 마찬가지로, `Option`에서는 `Option`을 반환하는
함수에서만 `?`를 쓸 수 있습니다. `Option<T>`에서 호출했을 때의 `?` 연산자의 동작은
`Result<T, E>`에서 호출했을 때의 동작과 비슷합니다. 값이 `None`이면, 그 지점에서
함수에서 `None`이 일찍 반환됩니다. 값이 `Some`이면 `Some` 안의 값이 그 표현식의
결과값이 되고 함수는 계속됩니다. Listing 9-11에는 주어진 텍스트의 첫 줄의 마지막
문자를 찾는 함수의 예가 있습니다.

<Listing number="9-11" caption="`Option<T>` 값에 `?` 연산자 사용하기">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

이 함수는 거기에 문자가 있을 가능성도 있고 없을 가능성도 있기 때문에 `Option<char>`
를 반환합니다. 이 코드는 `text` 문자열 슬라이스 인자를 받아 그 위에 `lines` 메서드
를 호출하는데, 이는 문자열의 줄에 대한 이터레이터를 반환합니다. 이 함수는 첫 줄을
살피려 하므로, 이터레이터에서 첫 값을 얻기 위해 `next`를 호출합니다. `text`가 빈
문자열이면 이 `next` 호출은 `None`을 반환하고, 그 경우 우리는 `?`를 사용해 멈추고
`last_char_of_first_line`에서 `None`을 반환합니다. `text`가 빈 문자열이 아니면
`next`는 `text`의 첫 줄에 대한 문자열 슬라이스를 담은 `Some` 값을 반환합니다.

`?`가 문자열 슬라이스를 꺼내 주고, 그 문자열 슬라이스에 `chars`를 호출해 그 문자
들의 이터레이터를 얻을 수 있습니다. 이 첫 줄의 마지막 문자에 관심이 있으므로,
이터레이터의 마지막 아이템을 반환하기 위해 `last`를 호출합니다. 이는 첫 줄이
빈 문자열일 가능성이 있기 때문에 `Option`입니다. 예를 들어 `"\nhi"`처럼 `text`가
빈 줄로 시작하지만 다른 줄에는 문자가 있는 경우입니다. 그러나 첫 줄에 마지막
문자가 있다면 그것은 `Some` 배리언트로 반환됩니다. 중간의 `?` 연산자는 이 로직을
간결하게 표현할 수 있게 해 주며, 이 함수를 한 줄로 구현할 수 있게 합니다. `Option`
에 `?` 연산자를 사용할 수 없었다면 더 많은 메서드 호출이나 `match` 표현식으로 이
로직을 구현해야 했을 것입니다.

`Result`를 반환하는 함수에서 `Result`에 대해 `?`를 쓸 수 있고, `Option`을 반환
하는 함수에서 `Option`에 대해 `?`를 쓸 수 있지만, 섞어 쓸 수는 없다는 점에 유의
하세요. `?` 연산자는 `Result`를 `Option`이나 그 반대로 자동 변환해 주지 않습니다.
그런 경우에는 `Result`의 `ok` 메서드나 `Option`의 `ok_or` 메서드 같은 메서드로
명시적으로 변환할 수 있습니다.

지금까지 우리가 사용한 모든 `main` 함수는 `()`를 반환했습니다. `main` 함수는
실행 가능한 프로그램의 진입점이자 종료점이기 때문에 특별하며, 프로그램이 기대대로
동작하도록 반환 타입에 제한이 있습니다.

다행히도 `main`은 `Result<(), E>`를 반환할 수도 있습니다. Listing 9-12에는 Listing
9-10의 코드가 있지만, `main`의 반환 타입을 `Result<(), Box<dyn Error>>`로 바꾸고
끝에 반환값 `Ok(())`를 추가했습니다. 이 코드는 이제 컴파일됩니다.

<Listing number="9-12" file-name="src/main.rs" caption="`main`이 `Result<(), E>`를 반환하도록 바꾸면 `Result` 값에 `?` 연산자를 사용할 수 있습니다.">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

`Box<dyn Error>` 타입은 트레이트 객체이며, 18장의 [“트레이트 객체로 공유 동작
추상화하기”][trait-objects]<!-- ignore -->에서 이야기할 것입니다. 지금은
`Box<dyn Error>`를 “어떤 종류의 에러든”으로 읽을 수 있습니다. 에러 타입이
`Box<dyn Error>`인 `main` 함수에서 `Result` 값에 `?`를 사용하는 것은 허용됩니다.
어떤 `Err` 값이든 일찍 반환될 수 있게 해 주기 때문입니다. 이 `main` 함수의 본문은
오직 `std::io::Error` 타입의 에러만 반환하겠지만, `Box<dyn Error>`를 지정함으로써,
다른 에러를 반환하는 코드가 `main` 본문에 추가되어도 이 시그니처는 계속 올바르게
유지됩니다.

`main` 함수가 `Result<(), E>`를 반환하면, `main`이 `Ok(())`를 반환할 때 실행 파일은
값 `0`으로 종료되고, `main`이 `Err` 값을 반환할 때는 0이 아닌 값으로 종료됩니다.
C로 작성된 실행 파일은 종료할 때 정수를 반환합니다. 성공적으로 종료하는 프로그램은
정수 `0`을 반환하고, 에러가 난 프로그램은 `0`이 아닌 어떤 정수를 반환합니다.
러스트도 이 관례와 호환되도록 실행 파일에서 정수를 반환합니다.

`main` 함수는 [`std::process::Termination`
트레이트][termination]<!-- ignore -->를 구현하는 어떤 타입이든 반환할 수 있는데,
이 트레이트는 `ExitCode`를 반환하는 `report` 함수를 담고 있습니다. 여러분의 타입에
`Termination` 트레이트를 구현하는 방법에 대한 더 많은 정보는 표준 라이브러리
문서를 참고하세요.

이제 `panic!`을 호출하거나 `Result`를 반환하는 세부 사항을 논의했으니, 어떤 경우에
어느 쪽을 쓰는 것이 적절한지 결정하는 주제로 돌아가 봅시다.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[termination]: ../std/process/trait.Termination.html
