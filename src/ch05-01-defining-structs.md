## 구조체 정의 및 인스턴스화

구조체는 [“튜플 타입”][tuples]<!-- ignore --> 절에서 다룬 튜플과 비슷합니다. 둘
모두 여러 관련된 값을 담는다는 점에서 그렇습니다. 튜플처럼 구조체의 조각들도 서로
다른 타입이 될 수 있습니다. 튜플과 다른 점은, 구조체에서는 각 데이터 조각에 이름을
붙이기 때문에 값이 무엇을 의미하는지 명확해진다는 것입니다. 이 이름들이 추가된다는
것은 구조체가 튜플보다 유연하다는 뜻입니다. 인스턴스의 값을 지정하거나 접근하기
위해 데이터의 순서에 의존할 필요가 없습니다.

구조체를 정의하려면 `struct` 키워드를 입력하고 구조체 전체에 이름을 붙입니다.
구조체의 이름은 함께 묶인 데이터 조각들의 의미를 설명해야 합니다. 그런 다음 중괄호
안에 데이터 조각들의 이름과 타입을 정의하는데, 이를 *필드(field)*라고 부릅니다.
예를 들어, Listing 5-1은 사용자 계정에 관한 정보를 저장하는 구조체를 보여 줍니다.

<Listing number="5-1" file-name="src/main.rs" caption="`User` 구조체 정의">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

구조체를 정의한 뒤에 사용하려면, 각 필드에 구체적인 값을 지정해서 그 구조체의
*인스턴스(instance)*를 만듭니다. 인스턴스는 구조체 이름을 적고, 그 뒤에 중괄호를
덧붙인 다음, 중괄호 안에 _`key: value`_ 쌍을 작성해서 만듭니다. 여기서 키는 필드의
이름이고, 값은 그 필드에 저장하고 싶은 데이터입니다. 필드를 구조체에서 선언한 순서
그대로 지정할 필요는 없습니다. 다시 말해, 구조체 정의는 타입에 대한 일반적인
템플릿과 같고, 인스턴스는 특정 데이터로 그 템플릿을 채워 타입의 값을 만들어 냅니다.
예를 들어, Listing 5-2에 보이듯 특정 사용자를 선언할 수 있습니다.

<Listing number="5-2" file-name="src/main.rs" caption="`User` 구조체의 인스턴스 만들기">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

구조체에서 특정 값을 가져오려면 점 표기법을 사용합니다. 예를 들어, 이 사용자의
이메일 주소에 접근하려면 `user1.email`을 사용합니다. 인스턴스가 가변이라면 점 표기법
으로 특정 필드에 할당해서 값을 바꿀 수 있습니다. Listing 5-3은 가변 `User`
인스턴스의 `email` 필드 값을 바꾸는 방법을 보여 줍니다.

<Listing number="5-3" file-name="src/main.rs" caption="`User` 인스턴스의 `email` 필드 값 바꾸기">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

인스턴스 전체가 가변이어야 한다는 점에 유의하세요. 러스트는 특정 필드만 가변으로
표시하는 것을 허용하지 않습니다. 다른 표현식에서처럼, 함수 본문의 마지막 표현식으로
구조체의 새 인스턴스를 생성해 그 새 인스턴스를 암묵적으로 반환할 수 있습니다.

Listing 5-4는 주어진 이메일과 사용자 이름으로 `User` 인스턴스를 반환하는 `build_user`
함수를 보여 줍니다. `active` 필드는 값 `true`를 받고, `sign_in_count`는 값 `1`을
받습니다.

<Listing number="5-4" file-name="src/main.rs" caption="이메일과 사용자 이름을 받아 `User` 인스턴스를 반환하는 `build_user` 함수">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

함수 매개변수의 이름을 구조체 필드와 같은 이름으로 짓는 것은 합리적이지만, `email`
과 `username` 필드 이름과 변수를 반복해서 쓰는 일은 다소 번거롭습니다. 구조체에
필드가 더 많았다면, 각 이름을 반복하는 일은 더 성가셨을 것입니다. 다행히 편리한
축약 문법이 있습니다!

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### 필드 초기화 축약 문법 사용하기

Listing 5-4에서 매개변수 이름과 구조체 필드 이름이 완전히 같기 때문에, *필드 초기화
축약(field init shorthand)* 문법을 사용해 `build_user`를 다시 작성할 수 있습니다.
동작은 똑같지만 `username`과 `email`의 반복이 없는 버전입니다. Listing 5-5를
봅시다.

<Listing number="5-5" file-name="src/main.rs" caption="`username`과 `email` 매개변수의 이름이 구조체 필드와 같기 때문에 필드 초기화 축약 문법을 사용하는 `build_user` 함수">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

여기서는 `email`이라는 필드를 가진 `User` 구조체의 새 인스턴스를 만들고 있습니다.
`email` 필드의 값을 `build_user` 함수의 `email` 매개변수의 값으로 설정하고 싶은
것이죠. `email` 필드와 `email` 매개변수의 이름이 같기 때문에, `email: email`
대신 `email`만 쓰면 됩니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-instances-from-other-instances-with-struct-update-syntax"></a>

### 구조체 업데이트 문법으로 인스턴스 만들기

같은 타입의 다른 인스턴스의 값을 대부분 포함하되 일부만 바꾼 새 인스턴스를 만드는
일이 유용한 경우가 많습니다. *구조체 업데이트 문법(struct update syntax)*을 사용해
이를 할 수 있습니다.

먼저, Listing 5-6에서는 업데이트 문법 없이 일반적인 방식으로 `user2`에 새 `User`
인스턴스를 만드는 방법을 보여 줍니다. `email`에는 새 값을 설정하고, 나머지는
Listing 5-2에서 만든 `user1`의 값과 같게 사용합니다.

<Listing number="5-6" file-name="src/main.rs" caption="`user1`의 값을 하나만 제외하고 모두 사용해 새 `User` 인스턴스 만들기">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

Listing 5-7처럼 구조체 업데이트 문법을 사용하면 더 적은 코드로 같은 효과를 낼 수
있습니다. `..` 문법은 명시적으로 설정하지 않은 나머지 필드가 주어진 인스턴스의
필드 값과 같아야 함을 지정합니다.

<Listing number="5-7" file-name="src/main.rs" caption="`User` 인스턴스의 `email`에는 새 값을 설정하고 나머지 값은 `user1`의 값을 사용하는 구조체 업데이트 문법">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

Listing 5-7의 코드 또한 `user2`에 인스턴스를 만드는데, `email` 값은 다르지만
`username`, `active`, `sign_in_count` 필드는 `user1`의 값을 그대로 사용합니다.
`..user1`은 남은 필드들이 `user1`의 대응하는 필드의 값을 받아야 함을 지정하기 위해
마지막에 와야 하지만, 구조체 정의의 필드 순서와 상관없이 원하는 만큼의 필드를 원하는
순서로 직접 지정할 수 있습니다.

구조체 업데이트 문법은 할당과 같은 방식으로 `=`을 사용합니다. 이는 [“변수와 데이터가
이동(Move)으로 상호작용하기”][move]<!-- ignore --> 절에서 본 것처럼 데이터를
*이동*시키기 때문입니다. 이 예에서는 `user1`의 `username` 필드에 있는 `String`이
`user2`로 이동되었기 때문에, `user2`를 만든 뒤에는 `user1`을 더 이상 사용할 수
없습니다. 만약 `user2`에 `email`과 `username` 둘 다 새 `String` 값을 주어서 `user1`
에서는 `active`와 `sign_in_count` 값만 사용했다면, `user2`를 만든 뒤에도 `user1`은
여전히 유효했을 것입니다. `active`와 `sign_in_count`는 모두 `Copy` 트레이트를
구현하는 타입이므로, [“스택 전용 데이터: 복사(Copy)”][copy]<!-- ignore --> 절에서
다룬 동작이 적용되기 때문입니다. 이 예에서 `user1.email`은 여전히 사용할 수
있는데, 그 값이 `user1`에서 바깥으로 이동되지 않았기 때문입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-tuple-structs-without-named-fields-to-create-different-types"></a>

### 튜플 구조체로 다른 타입 만들기

러스트는 튜플과 비슷해 보이는 구조체인 *튜플 구조체(tuple struct)*도 지원합니다.
튜플 구조체는 구조체 이름이 제공하는 추가적인 의미를 가지지만, 필드에 이름이 연관
되어 있지 않습니다. 대신 필드의 타입만 가집니다. 튜플 구조체는 튜플 전체에 이름을
붙이고 그 튜플을 다른 튜플들과 구별되는 타입으로 만들고 싶을 때, 그리고 일반
구조체처럼 각 필드의 이름을 붙이는 것이 장황하거나 중복스러울 때 유용합니다.

튜플 구조체를 정의하려면 `struct` 키워드와 구조체 이름, 그리고 튜플에 들어갈
타입들을 나열합니다. 예를 들어, 여기서는 `Color`와 `Point`라는 두 튜플 구조체를
정의하고 사용합니다.

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

`black`과 `origin` 값은 서로 다른 튜플 구조체의 인스턴스이므로, 서로 다른 타입이
라는 점에 유의하세요. 여러분이 정의하는 각 구조체는 그 자체로 고유한 타입이며,
구조체 안의 필드들이 같은 타입을 가지고 있어도 마찬가지입니다. 예를 들어, `Color`
타입의 매개변수를 받는 함수는 `Point`를 인자로 받을 수 없습니다. 두 타입 모두 세
`i32` 값으로 이뤄져 있는데도 그렇습니다. 그 외에는 튜플 구조체 인스턴스도 튜플과
비슷해서 개별 조각으로 구조 분해할 수 있고, `.` 뒤에 인덱스를 붙여 개별 값에 접근할
수 있습니다. 튜플과 다른 점은, 튜플 구조체를 구조 분해할 때에는 구조체의 타입
이름을 써야 한다는 것입니다. 예를 들어, `origin` 점의 값을 `x`, `y`, `z`라는 이름의
변수로 구조 분해하려면 `let Point(x, y, z) = origin;`처럼 씁니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="unit-like-structs-without-any-fields"></a>

### 유닛 유사 구조체 정의하기

필드가 하나도 없는 구조체도 정의할 수 있습니다! 이런 구조체를 *유닛 유사
구조체(unit-like struct)*라고 부르며, 이는 [“튜플 타입”][tuples]<!-- ignore -->
절에서 언급한 유닛 타입 `()`와 비슷하게 동작하기 때문에 그런 이름이 붙었습니다.
유닛 유사 구조체는 어떤 타입에 트레이트를 구현해야 하지만 그 타입 자체에 저장하고
싶은 데이터가 없을 때 유용할 수 있습니다. 트레이트는 10장에서 다룹니다. 다음은
`AlwaysEqual`이라는 유닛 구조체를 선언하고 인스턴스화하는 예시입니다.

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

`AlwaysEqual`을 정의하려면 `struct` 키워드, 원하는 이름, 그리고 세미콜론을 사용
합니다. 중괄호나 괄호는 필요 없습니다! 그런 다음 정의한 이름을 사용해, 중괄호나
괄호 없이 `subject` 변수에 `AlwaysEqual`의 인스턴스를 얻을 수 있습니다. 나중에
이 타입에 대해 `AlwaysEqual`의 모든 인스턴스가 다른 어떤 타입의 모든 인스턴스와
항상 같다고 여겨지는 동작을 구현한다고 상상해 봅시다. 테스트 목적으로 알려진
결과를 갖도록 하기 위해서 말이죠. 그 동작을 구현하는 데 어떤 데이터도 필요하지
않을 것입니다! 트레이트를 정의하고, 유닛 유사 구조체를 포함한 어떤 타입에든 구현
하는 방법은 10장에서 볼 수 있습니다.

> ### 구조체 데이터의 소유권
>
> Listing 5-1의 `User` 구조체 정의에서는 `&str` 문자열 슬라이스 타입이 아니라
> 소유 문자열인 `String` 타입을 사용했습니다. 이는 의도적인 선택인데, 이 구조체의
> 각 인스턴스가 자신의 모든 데이터를 소유하게 하고, 그 데이터가 구조체 전체가
> 유효한 동안 유효하도록 하고 싶기 때문입니다.
>
> 구조체가 다른 누군가가 소유한 데이터에 대한 참조를 저장할 수도 있지만, 그렇게
> 하려면 10장에서 다룰 러스트 기능인 *라이프타임*을 사용해야 합니다. 라이프타임은
> 구조체가 참조하는 데이터가 구조체가 유효한 동안 유효하도록 보장합니다. 예를
> 들어 라이프타임을 지정하지 않고 구조체에 참조를 저장하려고 하면, 다음처럼
> *src/main.rs*에 작성한 코드는 동작하지 않습니다.
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> 컴파일러는 라이프타임 지정자(lifetime specifier)가 필요하다고 불평할 것입니다.
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> 10장에서는 구조체에 참조를 저장할 수 있도록 이런 에러를 고치는 방법을 다룹니다.
> 지금은 `&str` 같은 참조 대신 `String` 같은 소유 타입을 사용해 이런 에러를
> 해결하겠습니다.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
