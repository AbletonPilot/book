## 메서드

메서드는 함수와 비슷합니다. `fn` 키워드와 이름으로 선언하고, 매개변수와 반환값을
가질 수 있으며, 메서드가 다른 곳에서 호출될 때 실행되는 코드를 담고 있습니다.
함수와 달리 메서드는 구조체(또는 열거형, 혹은 트레이트 객체, 각각 [6장][enums]
<!-- ignore -->과 [18장][trait-objects]<!-- ignore -->에서 다룸)의 맥락 안에서
정의되며, 메서드의 첫 매개변수는 항상 `self`입니다. `self`는 메서드가 호출되고
있는 구조체의 인스턴스를 나타냅니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-methods"></a>

### 메서드 문법

`Rectangle` 인스턴스를 매개변수로 받는 `area` 함수를 `Rectangle` 구조체에 정의된
`area` 메서드로 바꿔 봅시다. Listing 5-13을 보세요.

<Listing number="5-13" file-name="src/main.rs" caption="`Rectangle` 구조체에 `area` 메서드 정의하기">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

`Rectangle`의 맥락 안에 함수를 정의하기 위해, `Rectangle`을 위한 `impl`(implementation,
구현) 블록을 시작합니다. 이 `impl` 블록 안의 모든 것은 `Rectangle` 타입과 연관
됩니다. 그런 다음 `area` 함수를 `impl` 중괄호 안으로 옮기고, 시그니처와 본문 전체에
걸쳐 첫 번째(이 경우에는 유일한) 매개변수를 `self`로 바꿉니다. `area` 함수를 호출
하고 `rect1`을 인자로 넘기던 `main`에서는, 이제 *메서드 문법(method syntax)*을
사용해 `Rectangle` 인스턴스에 `area` 메서드를 호출할 수 있습니다. 메서드 문법은
인스턴스 뒤에 옵니다. 점, 메서드 이름, 괄호, 그리고 어떤 인자를 덧붙입니다.

`area`의 시그니처에서 `rectangle: &Rectangle` 대신 `&self`를 사용합니다. `&self`는
사실 `self: &Self`의 축약입니다. `impl` 블록 안에서 `Self` 타입은 그 `impl` 블록이
대상으로 하는 타입의 별칭입니다. 메서드는 첫 번째 매개변수로 `Self` 타입의 `self`
라는 이름의 매개변수를 가져야 하므로, 러스트는 첫 매개변수 자리에서 이를 `self`
라는 이름만으로 축약할 수 있게 해 줍니다. `self` 축약 앞에도 여전히 `&`가 필요한
점에 유의하세요. 이는 `rectangle: &Rectangle`에서 했던 것과 마찬가지로, 이 메서드가
`Self` 인스턴스를 빌린다는 것을 나타내기 위함입니다. 메서드는 다른 어떤 매개변수
와도 마찬가지로, `self`의 소유권을 가져갈 수도 있고, 우리가 여기서 한 것처럼 `self`
를 불변으로 빌릴 수도 있고, `self`를 가변으로 빌릴 수도 있습니다.

여기서 `&self`를 선택한 이유는 함수 버전에서 `&Rectangle`을 쓴 것과 같습니다.
소유권을 가져가고 싶지도 않고, 구조체의 데이터를 읽기만 할 뿐 쓰기를 하고 싶은
것도 아닙니다. 메서드가 동작의 일부로 자신이 호출된 인스턴스를 바꾸고 싶다면,
첫 매개변수로 `&mut self`를 사용할 것입니다. 첫 매개변수로 그냥 `self`를 사용해
인스턴스의 소유권을 가져가는 메서드를 만드는 일은 드뭅니다. 이 기법은 보통
메서드가 `self`를 다른 무언가로 변환하고, 그 변환 후에 원본 인스턴스를 호출자가
사용하지 못하도록 막고 싶을 때 사용됩니다.

메서드를 함수 대신 사용하는 주된 이유는, 메서드 문법을 제공하고 모든 메서드
시그니처에서 `self`의 타입을 반복하지 않아도 된다는 점 외에도, 조직화 때문입니다.
타입의 인스턴스로 할 수 있는 모든 일을 하나의 `impl` 블록에 모아 두면, 우리가
제공하는 라이브러리의 여기저기에서 `Rectangle`의 기능을 찾아야 하는 미래의 사용자
에게 수고를 끼치지 않게 됩니다.

메서드에 구조체의 필드 중 하나와 같은 이름을 줄 수도 있다는 점에 유의하세요.
예를 들어, `Rectangle`에 `width`라는 이름의 메서드를 정의할 수 있습니다.

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

여기서는 인스턴스의 `width` 필드의 값이 `0`보다 크면 `true`를, 값이 `0`이면
`false`를 반환하도록 `width` 메서드를 만들었습니다. 같은 이름의 메서드 안에서 필드를
어떤 용도로든 사용할 수 있습니다. `main`에서 `rect1.width` 뒤에 괄호를 붙이면,
러스트는 우리가 `width` 메서드를 의미한다는 것을 압니다. 괄호를 사용하지 않으면,
러스트는 우리가 `width` 필드를 의미한다는 것을 압니다.

항상 그런 것은 아니지만, 메서드에 필드와 같은 이름을 부여할 때에는 메서드가 해당
필드의 값만 반환하고 그 외에는 아무것도 하지 않기를 원하는 경우가 많습니다. 이런
메서드를 *게터(getter)*라고 부르며, 러스트는 다른 일부 언어처럼 구조체 필드에
대해 이를 자동으로 구현해 주지는 않습니다. 게터가 유용한 이유는, 필드는 비공개로
하되 메서드는 공개로 해서, 타입의 공개 API의 일부로 그 필드에 읽기 전용 접근을
가능하게 할 수 있기 때문입니다. 공개(public)와 비공개(private)가 무엇인지, 그리고
필드나 메서드를 공개 또는 비공개로 지정하는 방법은 [7장][public]<!-- ignore -->
에서 다룹니다.

> ### `->` 연산자는 어디로 갔나요?
>
> C와 C++에서는 메서드 호출에 두 개의 서로 다른 연산자를 사용합니다. 객체에 직접
> 메서드를 호출하면 `.`을, 객체를 가리키는 포인터에 메서드를 호출해서 포인터를
> 먼저 역참조해야 할 때에는 `->`를 사용합니다. 다시 말해, `object`가 포인터일
> 때 `object->something()`은 `(*object).something()`과 비슷합니다.
>
> 러스트에는 `->` 연산자에 해당하는 것이 없습니다. 대신 러스트에는 *자동 참조 및
> 역참조(automatic referencing and dereferencing)*라는 기능이 있습니다. 메서드
> 호출은 러스트에서 이런 동작이 일어나는 몇 안 되는 자리 중 하나입니다.
>
> 동작 방식은 다음과 같습니다. `object.something()`으로 메서드를 호출할 때,
> 러스트는 `object`가 메서드의 시그니처에 맞도록 자동으로 `&`, `&mut`, 또는 `*`를
> 덧붙여 줍니다. 즉, 다음 두 가지는 같습니다.
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> 첫 번째 쪽이 훨씬 깔끔해 보입니다. 메서드는 명확한 수신자, 즉 `self`의 타입을
> 가지기 때문에 이 자동 참조 동작이 가능합니다. 수신자와 메서드 이름이 주어지면
> 러스트는 그 메서드가 읽기(`&self`)인지, 변경(`&mut self`)인지, 소비(`self`)
> 인지 확정적으로 알아낼 수 있습니다. 러스트가 메서드 수신자에 대해 빌림을
> 암묵적으로 만들어 준다는 사실은, 실전에서 소유권을 편안하게 쓸 수 있도록 만드
> 는 큰 요인입니다.

### 더 많은 매개변수를 가진 메서드

메서드를 연습해 보기 위해 `Rectangle` 구조체에 두 번째 메서드를 구현해 봅시다.
이번에는 `Rectangle` 인스턴스가 또 다른 `Rectangle` 인스턴스를 받아, 두 번째
`Rectangle`이 `self`(첫 번째 `Rectangle`) 안에 완전히 들어갈 수 있으면 `true`를,
그렇지 않으면 `false`를 반환하도록 하고 싶습니다. 즉, `can_hold` 메서드를 정의한
뒤에는 Listing 5-14에 보이는 프로그램을 작성할 수 있기를 원합니다.

<Listing number="5-14" file-name="src/main.rs" caption="아직 작성하지 않은 `can_hold` 메서드 사용하기">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

기대되는 출력은 다음과 같습니다. `rect2`의 두 치수 모두 `rect1`의 치수보다 작지만,
`rect3`은 `rect1`보다 너비가 크기 때문입니다.

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

메서드를 정의할 것이므로 `impl Rectangle` 블록 안에 넣을 것임을 알고 있습니다.
메서드 이름은 `can_hold`가 될 것이고, 매개변수로 다른 `Rectangle`의 불변 빌림을
받을 것입니다. 매개변수의 타입은 메서드를 호출하는 코드를 보면 알 수 있습니다.
`rect1.can_hold(&rect2)`는 `&rect2`를 전달하는데, 이는 `Rectangle`의 인스턴스인
`rect2`에 대한 불변 빌림입니다. 이는 합리적입니다. 우리는 `rect2`를 읽기만 하면
되기 때문입니다(쓰기가 필요하다면 가변 빌림이 필요했을 것입니다). 그리고 `can_hold`
메서드를 호출한 뒤에도 `rect2`를 다시 사용할 수 있도록, `main`이 `rect2`의 소유권을
유지하기를 원하기 때문입니다. `can_hold`의 반환값은 불리언이 될 것이고, 구현에서는
`self`의 너비와 높이가 각각 다른 `Rectangle`의 너비와 높이보다 큰지 확인합니다.
Listing 5-13의 `impl` 블록에 새로운 `can_hold` 메서드를 추가합시다. Listing
5-15에 나와 있습니다.

<Listing number="5-15" file-name="src/main.rs" caption="매개변수로 다른 `Rectangle` 인스턴스를 받는 `can_hold` 메서드를 `Rectangle`에 구현하기">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

Listing 5-14의 `main` 함수와 함께 이 코드를 실행하면 원하는 출력을 얻습니다.
메서드는 `self` 매개변수 뒤에 시그니처에 추가한 여러 매개변수를 받을 수 있으며,
그 매개변수들은 함수의 매개변수와 똑같이 동작합니다.

### 연관 함수

`impl` 블록 안에 정의된 모든 함수는 *연관 함수(associated function)*라고 부릅
니다. `impl` 뒤에 오는 타입과 연관되어 있기 때문입니다. `self`를 첫 매개변수로
가지지 않는 연관 함수도 정의할 수 있고(따라서 이는 메서드가 아닙니다), 이런
함수는 타입의 인스턴스가 필요 없이 동작하기 때문입니다. 우리는 이미 이런 함수
하나를 사용해 왔습니다. `String` 타입에 정의된 `String::from` 함수가 그것입니다.

메서드가 아닌 연관 함수는 구조체의 새 인스턴스를 반환하는 생성자에 자주 사용됩니다.
이런 함수의 이름은 `new`인 경우가 많지만, `new`는 특별한 이름이 아니며 언어에 내장
되어 있지도 않습니다. 예를 들어, 한 치수 매개변수를 받아 너비와 높이 모두에
사용하는 `square`라는 연관 함수를 제공해, 같은 값을 두 번 지정할 필요 없이 정사각형
`Rectangle`을 더 쉽게 만들 수 있게 할 수 있습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

반환 타입과 함수 본문의 `Self` 키워드는 `impl` 키워드 뒤에 오는 타입의 별칭이며,
이 경우에는 `Rectangle`입니다.

이 연관 함수를 호출할 때에는 구조체 이름과 함께 `::` 문법을 사용합니다.
`let sq = Rectangle::square(3);`이 그 예입니다. 이 함수는 구조체에 의해 네임스페이스
화됩니다. `::` 문법은 연관 함수와 모듈로 만든 네임스페이스 모두에 사용됩니다.
모듈은 [7장][modules]<!-- ignore -->에서 다룹니다.

### 여러 `impl` 블록

각 구조체는 여러 `impl` 블록을 가질 수 있습니다. 예를 들어, Listing 5-15는 각
메서드를 자신의 `impl` 블록에 담은 Listing 5-16의 코드와 동등합니다.

<Listing number="5-16" caption="Listing 5-15를 여러 `impl` 블록을 사용해 다시 쓰기">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

여기서 메서드들을 여러 `impl` 블록으로 분리할 이유는 없지만, 유효한 문법입니다.
여러 `impl` 블록이 유용한 사례는 10장에서 제네릭 타입과 트레이트를 이야기할 때
보게 됩니다.

## 요약

구조체를 사용하면 여러분의 도메인에 의미 있는 커스텀 타입을 만들 수 있습니다.
구조체를 사용하면 연관된 데이터 조각들을 서로 연결된 상태로 유지하고, 각 조각에
이름을 붙여 코드를 명확하게 할 수 있습니다. `impl` 블록에서는 타입과 연관된
함수를 정의할 수 있고, 메서드는 구조체 인스턴스가 가지는 동작을 지정할 수 있게
해 주는 연관 함수의 한 종류입니다.

하지만 커스텀 타입을 만드는 방법이 구조체만 있는 것은 아닙니다. 러스트의 열거형
기능으로 여러분의 도구 상자에 또 다른 도구를 추가해 봅시다.

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
