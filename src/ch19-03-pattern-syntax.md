## 패턴 문법

이 절에서는 패턴에서 유효한 모든 문법을 모으고, 각각을 왜 그리고 언제 사용하고 싶을지 논의합니다.

### 리터럴 매칭

6장에서 본 것처럼 패턴을 리터럴과 직접 매칭할 수 있습니다. 다음 코드가 몇 가지 예시를 보여 줍니다.

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

이 코드는 `x`의 값이 `1`이므로 `one`을 출력합니다. 이 문법은 코드가 어떤 특정한 구체 값을 받았을 때 어떤 동작을 취하게 하고 싶을 때 유용합니다.

### 이름 붙은 변수 매칭

이름 붙은 변수는 어떤 값이든 매칭하는 반박 불가 패턴이며, 이 책에서 여러 번 사용해 왔습니다. 그러나 `match`, `if let`, `while let` 표현식 안에서 이름 붙은 변수를 사용할 때는 주의해야 할 점이 있습니다. 이런 종류의 표현식 각각이 새로운 스코프를 시작하므로, 이런 표현식 안의 패턴 일부로 선언된 변수는 모든 변수가 그러하듯 그 구문 바깥에서 같은 이름을 가진 변수를 가립니다. Listing 19-11에서는 값이 `Some(5)`인 변수 `x`와 값이 `10`인 변수 `y`를 선언합니다. 그런 다음 값 `x`에 대한 `match` 표현식을 만듭니다. 매치 갈래의 패턴들과 끝의 `println!`을 보고, 이 코드를 실행하거나 더 읽어 내려가기 전에 무엇을 출력할지 직접 알아내려 해 보세요.

<Listing number="19-11" file-name="src/main.rs" caption="기존 변수 `y`를 가리는 새 변수를 도입하는 갈래를 가진 `match` 표현식">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

`match` 표현식이 실행될 때 무슨 일이 일어나는지 따라가 봅시다. 첫 번째 매치 갈래의 패턴은 정의된 `x` 값과 매칭되지 않으므로 코드는 계속 진행됩니다.

두 번째 매치 갈래의 패턴은 `Some` 값 안의 어떤 값이든 매칭하는 새로운 변수 `y`를 도입합니다. `match` 표현식 안의 새로운 스코프에 있으므로, 이는 처음에 값 `10`으로 선언했던 `y`가 아니라 새로운 `y` 변수입니다. 이 새로운 `y` 바인딩은 `Some` 안의 어떤 값이든 매칭할 것이며, 그것이 우리가 `x`에 가진 것입니다. 따라서 이 새로운 `y`는 `x`의 `Some` 안 값에 바인딩됩니다. 그 값은 `5`이므로, 그 갈래의 표현식이 실행되어 `Matched, y = 5`를 출력합니다.

만약 `x`가 `Some(5)`가 아니라 `None` 값이었다면, 처음 두 갈래의 패턴은 매칭되지 않았을 것이므로 값은 언더스코어에 매칭되었을 것입니다. 언더스코어 갈래의 패턴에서는 `x` 변수를 도입하지 않았으므로, 표현식의 `x`는 여전히 가려지지 않은 외부의 `x`입니다. 이 가상의 경우라면 `match`는 `Default case, x = None`을 출력했을 것입니다.

`match` 표현식이 끝나면 그 스코프가 끝나고, 그와 함께 내부의 `y`의 스코프도 끝납니다. 마지막 `println!`은 `at the end: x = Some(5), y = 10`을 만들어 냅니다.

기존 `y` 변수를 가리는 새 변수를 도입하는 대신 외부의 `x`와 `y` 값을 비교하는 `match` 표현식을 만들려면, 매치 가드 조건을 사용해야 할 것입니다. 매치 가드는 뒤의 [“매치 가드로 조건 추가하기”](#adding-conditionals-with-match-guards)<!-- ignore --> 절에서 다룹니다.

<!-- Old headings. Do not remove or links may break. -->
<a id="multiple-patterns"></a>

### 여러 패턴 매칭

`match` 표현식에서 패턴 _or_ 연산자인 `|` 문법을 사용해 여러 패턴을 매칭할 수 있습니다. 예를 들어, 다음 코드에서 `x`의 값을 매치 갈래에 매칭하는데, 그중 첫 번째 갈래는 _or_ 옵션을 가지고 있어 `x`의 값이 그 갈래의 값들 중 하나와 매칭되면 그 갈래의 코드가 실행됨을 의미합니다.


```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

이 코드는 `one or two`를 출력합니다.

### `..=`로 값의 범위 매칭

`..=` 문법은 포함적 값 범위에 매칭할 수 있게 해 줍니다. 다음 코드에서 패턴이 주어진 범위 내의 어떤 값과 매칭되면 그 갈래가 실행됩니다.

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

`x`가 `1`, `2`, `3`, `4`, `5` 중 하나라면 첫 번째 갈래가 매칭됩니다. 같은 의미를 표현하기 위해 `|` 연산자를 사용하는 것보다 여러 매치 값에 대해 이 문법이 더 편리합니다. `|`를 사용한다면 `1 | 2 | 3 | 4 | 5`라고 명시해야 할 것입니다. 예컨대 1과 1,000 사이의 어떤 숫자든 매칭하고 싶다면 범위를 명시하는 편이 훨씬 짧습니다!

컴파일러는 컴파일 시점에 범위가 비어 있지 않은지 검사하며, 러스트가 범위가 비어 있는지 아닌지를 알 수 있는 유일한 타입은 `char`와 수치 타입이므로 범위는 수치 또는 `char` 값과 함께만 허용됩니다.

다음은 `char` 값 범위를 사용하는 예시입니다.

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

러스트는 `'c'`가 첫 번째 패턴의 범위 안에 있음을 알 수 있으므로 `early ASCII letter`를 출력합니다.

### 분해(destructuring)로 값을 쪼개기

또한 패턴을 사용해 구조체, 열거형, 튜플을 분해하여 이러한 값들의 서로 다른 부분들을 사용할 수 있습니다. 각각을 살펴봅시다.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs"></a>

#### 구조체

Listing 19-12는 `let` 문에서 패턴을 사용해 쪼갤 수 있는, `x`와 `y`라는 두 필드를 가진 `Point` 구조체를 보여 줍니다.

<Listing number="19-12" file-name="src/main.rs" caption="구조체의 필드들을 별도의 변수로 분해">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

이 코드는 `p` 구조체의 `x`, `y` 필드 값과 매칭되는 변수 `a`, `b`를 만듭니다. 이 예제는 패턴의 변수 이름이 구조체의 필드 이름과 일치할 필요가 없음을 보여 줍니다. 그러나 어떤 변수가 어떤 필드에서 왔는지 더 쉽게 기억하기 위해 변수 이름을 필드 이름과 맞추는 것이 흔합니다. 이 흔한 사용법 때문에, 그리고 `let Point { x: x, y: y } = p;`라고 쓰는 것에 중복이 많아서, 러스트는 구조체 필드와 매칭되는 패턴을 위한 단축 표기를 제공합니다. 구조체 필드의 이름만 적으면 그 패턴에서 만들어지는 변수는 같은 이름을 갖게 됩니다. Listing 19-13은 Listing 19-12와 같은 방식으로 동작하지만, `let` 패턴에서 만들어지는 변수가 `a`와 `b`가 아니라 `x`와 `y`입니다.

<Listing number="19-13" file-name="src/main.rs" caption="구조체 필드 단축 표기를 사용한 구조체 필드 분해">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

이 코드는 `p` 변수의 `x`와 `y` 필드와 매칭되는 변수 `x`와 `y`를 만듭니다. 결과적으로 변수 `x`와 `y`는 `p` 구조체의 값을 담게 됩니다.

또한 모든 필드에 대해 변수를 만드는 대신, 구조체 패턴의 일부로 리터럴 값을 사용해 분해할 수도 있습니다. 그렇게 하면 일부 필드는 특정 값인지 검사하면서, 다른 필드는 분해해 변수로 만들 수 있습니다.

Listing 19-14에는 `Point` 값을 세 가지 경우로 분리하는 `match` 표현식이 있습니다. `x` 축 위에 직접 놓인 점(`y = 0`이면 참), `y` 축 위에 놓인 점(`x = 0`), 또는 어느 축에도 놓이지 않은 점입니다.

<Listing number="19-14" file-name="src/main.rs" caption="한 패턴에서 분해와 리터럴 값 매칭하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

첫 번째 갈래는 `y` 필드가 리터럴 `0`과 매칭되는지 명시함으로써 `x` 축 위에 있는 어떤 점이라도 매칭합니다. 이 패턴은 여전히 갈래의 코드에서 사용할 수 있는 `x` 변수를 만듭니다.

마찬가지로 두 번째 갈래는 `x` 필드가 `0`과 매칭되는지 명시하고 `y` 필드 값을 위한 변수 `y`를 만들어 `y` 축 위의 어떤 점이라도 매칭합니다. 세 번째 갈래는 어떤 리터럴도 명시하지 않으므로 다른 어떤 `Point`라도 매칭하며, `x`와 `y` 필드 모두에 대해 변수를 만듭니다.

이 예제에서 값 `p`는 `x`가 `0`을 담고 있으므로 두 번째 갈래에 매칭되고, 따라서 이 코드는 `On the y axis at 7`을 출력합니다.

`match` 표현식은 첫 번째로 매칭되는 패턴을 찾으면 더 이상 갈래를 검사하지 않는다는 점을 기억하세요. 그러므로 `Point { x: 0, y: 0 }`이 `x` 축과 `y` 축 모두에 있더라도 이 코드는 `On the x axis at 0`만 출력할 것입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-enums"></a>

#### 열거형

이 책에서는 열거형을 분해해 왔지만(예를 들어 6장의 Listing 6-5), 열거형을 분해하는 패턴이 그 열거형 안에 저장된 데이터가 정의된 방식에 대응한다는 점을 아직 명시적으로 논의한 적은 없습니다. 예시로, Listing 19-15에서는 Listing 6-2의 `Message` 열거형을 사용하고, 각 내부 값을 분해할 패턴을 가진 `match`를 작성합니다.

<Listing number="19-15" file-name="src/main.rs" caption="서로 다른 종류의 값을 담는 열거형 변형들 분해하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

이 코드는 `Change color to red 0, green 160, and blue 255`를 출력합니다. `msg`의 값을 바꿔 다른 갈래의 코드가 실행되도록 시도해 보세요.

`Message::Quit`처럼 데이터가 없는 열거형 변형의 경우 값을 더 이상 분해할 수 없습니다. 리터럴 `Message::Quit` 값에만 매칭할 수 있으며, 그 패턴에는 어떤 변수도 없습니다.

`Message::Move`처럼 구조체 형태의 열거형 변형에는, 구조체에 매칭하는 데 명시하는 패턴과 비슷한 패턴을 사용할 수 있습니다. 변형 이름 다음에 중괄호를 두고, 갈래의 코드에서 사용하기 위해 조각들을 쪼갤 수 있도록 변수와 함께 필드들을 나열합니다. 여기서는 Listing 19-13에서 했던 것처럼 단축 형태를 사용합니다.

`Message::Write`처럼 한 요소를 담는 튜플과 `Message::ChangeColor`처럼 세 요소를 담는 튜플을 가지는 튜플 형태의 열거형 변형에서는, 패턴이 튜플에 매칭하기 위해 명시하는 패턴과 비슷합니다. 패턴의 변수 수는 매칭하는 변형의 요소 수와 일치해야 합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-nested-structs-and-enums"></a>

#### 중첩된 구조체와 열거형

지금까지 우리의 예제는 모두 한 단계 깊이의 구조체나 열거형을 매칭하는 것이었지만, 매칭은 중첩된 항목에 대해서도 동작합니다! 예를 들어, Listing 19-16에서 보이듯 `ChangeColor` 메시지에서 RGB와 HSV 색을 지원하도록 Listing 19-15의 코드를 리팩터링할 수 있습니다.

<Listing number="19-16" caption="중첩된 열거형에 대한 매칭">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

`match` 표현식의 첫 번째 갈래의 패턴은 `Color::Rgb` 변형을 담고 있는 `Message::ChangeColor` 열거형 변형과 매칭한 다음, 패턴이 세 개의 내부 `i32` 값에 바인딩됩니다. 두 번째 갈래의 패턴 또한 `Message::ChangeColor` 열거형 변형에 매칭하지만, 내부 열거형은 `Color::Hsv`와 매칭됩니다. 두 개의 열거형이 관여하더라도 한 `match` 표현식에서 이러한 복잡한 조건을 명시할 수 있습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs-and-tuples"></a>

#### 구조체와 튜플

분해 패턴을 더욱 복잡한 방식으로 섞고, 매칭하고, 중첩할 수 있습니다. 다음 예제는 튜플 안에 구조체와 튜플을 중첩하고 모든 원시 값을 분해해 꺼내는 복잡한 분해를 보여 줍니다.

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

이 코드는 복잡한 타입을 그 구성 요소들로 쪼개서, 우리가 관심 있는 값을 따로 사용할 수 있게 해 줍니다.

패턴으로 분해하는 것은 구조체의 각 필드의 값처럼 값의 일부를 서로 분리해서 사용하기 위한 편리한 방법입니다.

### 패턴에서 값 무시하기

`match`의 마지막 갈래에서, 실제로 아무것도 하지 않지만 남은 모든 가능한 값을 다루는 포괄 패턴을 얻기 위해 패턴에서 값을 무시하는 것이 때때로 유용함을 보았습니다. 패턴에서 값 전체나 값의 일부를 무시하는 방법은 몇 가지 있습니다. (이미 본 적 있는) `_` 패턴을 사용하기, 다른 패턴 안에서 `_` 패턴을 사용하기, 언더스코어로 시작하는 이름 사용하기, 또는 값의 남은 부분을 무시하기 위해 `..` 사용하기입니다. 이러한 패턴 각각을 어떻게 그리고 왜 사용하는지 살펴봅시다.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-entire-value-with-_"></a>

#### `_`로 값 전체 무시하기

언더스코어를 어떤 값에든 매칭되지만 그 값에 바인딩되지 않는 와일드카드 패턴으로 사용해 왔습니다. 이는 `match` 표현식의 마지막 갈래로 특히 유용하지만, Listing 19-17에서 보이듯 함수 매개변수를 포함한 어떤 패턴에서도 사용할 수 있습니다.

<Listing number="19-17" file-name="src/main.rs" caption="함수 시그니처에서 `_` 사용하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

이 코드는 첫 번째 인자로 전달된 값 `3`을 완전히 무시하고, `This code only uses the y parameter: 4`를 출력합니다.

대부분의 경우 특정 함수 매개변수가 더 이상 필요 없다면, 사용하지 않는 매개변수가 포함되지 않도록 시그니처를 변경하게 될 것입니다. 함수 매개변수를 무시하는 것은 예를 들어 특정 타입 시그니처가 필요한 트레이트를 구현하지만 구현의 함수 본문에서는 그 매개변수 중 하나가 필요 없는 경우에 특히 유용할 수 있습니다. 그러면 이름을 사용했을 때처럼 사용되지 않은 함수 매개변수에 대한 컴파일러 경고를 받지 않게 됩니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### 중첩된 `_`로 값의 일부 무시하기

값의 일부만 무시하기 위해 다른 패턴 안에서도 `_`를 사용할 수 있습니다. 예를 들어 값의 일부만 검사하고 싶지만, 실행하려는 대응 코드에서 다른 부분은 쓸 일이 없을 때입니다. Listing 19-18은 설정 값을 관리하는 코드를 보여 줍니다. 비즈니스 요구사항은, 사용자가 설정의 기존 사용자 정의 값을 덮어쓸 수 없어야 하지만, 설정을 해제하거나 현재 해제된 상태라면 값을 부여할 수 있어야 한다는 것입니다.

<Listing number="19-18" caption="`Some` 안의 값을 사용할 필요가 없을 때 `Some` 변형에 매칭되는 패턴 안에서 언더스코어 사용">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

이 코드는 `Can't overwrite an existing customized value`를 출력한 뒤 `setting is Some(5)`를 출력합니다. 첫 번째 매치 갈래에서는 두 `Some` 변형 안의 값을 매칭하거나 사용할 필요는 없지만, `setting_value`와 `new_setting_value`가 둘 다 `Some` 변형인 경우를 검사해야 합니다. 그 경우 `setting_value`를 변경하지 않는 이유를 출력하고, 값은 변경되지 않습니다.

(다른 모든 경우, 즉 `setting_value` 또는 `new_setting_value` 중 하나라도 `None`인 경우) 두 번째 갈래의 `_` 패턴이 표현하듯, `new_setting_value`가 `setting_value`가 되도록 허용하고 싶습니다.

또한 한 패턴의 여러 곳에서 언더스코어를 사용해 특정 값을 무시할 수 있습니다. Listing 19-19는 다섯 항목짜리 튜플에서 두 번째와 네 번째 값을 무시하는 예시를 보여 줍니다.

<Listing number="19-19" caption="튜플의 여러 부분 무시하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

이 코드는 `Some numbers: 2, 8, 32`를 출력하고, 값 `4`와 `16`은 무시됩니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### 이름 앞에 `_`를 붙여 사용하지 않는 변수 무시하기

변수를 만들어 놓고 어디에서도 사용하지 않으면, 사용되지 않은 변수가 버그일 수 있으므로 러스트는 보통 경고를 발행합니다. 그러나 때로는 프로토타이핑 중이거나 프로젝트를 막 시작할 때처럼 아직 사용하지 않을 변수를 만들 수 있는 것이 유용합니다. 이런 상황에서는 변수 이름을 언더스코어로 시작함으로써 사용되지 않은 변수에 대해 러스트에게 경고하지 말라고 알려 줄 수 있습니다. Listing 19-20에서는 사용되지 않는 두 변수를 만들지만, 이 코드를 컴파일하면 그중 하나에 대해서만 경고를 받게 됩니다.

<Listing number="19-20" file-name="src/main.rs" caption="사용되지 않는 변수 경고를 피하기 위해 변수 이름을 언더스코어로 시작">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

여기서는 변수 `y`를 사용하지 않는 것에 대해 경고를 받지만, `_x`를 사용하지 않는 것에 대해서는 경고를 받지 않습니다.

`_`만 사용하는 것과 언더스코어로 시작하는 이름을 사용하는 것 사이에는 미묘한 차이가 있다는 점에 유의하세요. `_x` 문법은 여전히 값을 변수에 바인딩하지만, `_`는 전혀 바인딩하지 않습니다. 이 구분이 중요한 경우를 보이기 위해 Listing 19-21은 우리에게 오류를 줍니다.

<Listing number="19-21" caption="언더스코어로 시작하는 사용되지 않는 변수도 여전히 값을 바인딩하므로 값의 소유권을 가져갈 수 있습니다.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

`s` 값이 여전히 `_s`로 옮겨져 `s`를 다시 사용할 수 없게 되므로 오류를 받게 됩니다. 그러나 언더스코어 자체만 사용하는 것은 값에 결코 바인딩되지 않습니다. Listing 19-22는 `s`가 `_`로 옮겨지지 않으므로 어떤 오류도 없이 컴파일됩니다.

<Listing number="19-22" caption="언더스코어를 사용하면 값을 바인딩하지 않습니다.">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

이 코드는 `s`를 어떤 것에도 바인딩하지 않으므로 잘 동작합니다. 옮겨지지 않습니다.

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### `..`로 값의 남은 부분 무시하기

여러 부분으로 이루어진 값에 대해, `..` 문법을 사용하여 특정 부분만 사용하고 나머지는 무시할 수 있어, 무시하려는 각 값에 대해 언더스코어를 나열할 필요가 없게 해 줍니다. `..` 패턴은 패턴의 나머지 부분에서 명시적으로 매칭하지 않은 값의 어떤 부분이든 무시합니다. Listing 19-23에서는 3차원 공간의 좌표를 담는 `Point` 구조체를 가지고 있습니다. `match` 표현식에서 `x` 좌표만 다루고 `y`와 `z` 필드의 값은 무시하고 싶습니다.

<Listing number="19-23" caption="`..`을 사용해 `x`를 제외한 `Point`의 모든 필드 무시하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

`x` 값을 나열한 다음 그저 `..` 패턴을 포함시킵니다. 특히 필드가 많은 구조체에서 한두 개의 필드만 관련 있는 상황이라면, `y: _`와 `z: _`를 일일이 나열하는 것보다 빠릅니다.

`..` 문법은 필요한 만큼의 값으로 확장됩니다. Listing 19-24는 튜플과 함께 `..`을 사용하는 방법을 보여 줍니다.

<Listing number="19-24" file-name="src/main.rs" caption="튜플의 첫 값과 마지막 값만 매칭하고 나머지 모든 값은 무시하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

이 코드에서 첫 번째와 마지막 값은 `first`와 `last`로 매칭됩니다. `..`은 가운데의 모든 것에 매칭되어 무시합니다.

그러나 `..`의 사용은 모호하지 않아야 합니다. 어떤 값이 매칭 대상이고 어떤 값이 무시되어야 하는지 분명하지 않다면, 러스트는 우리에게 오류를 줍니다. Listing 19-25는 `..`을 모호하게 사용하는 예시이므로 컴파일되지 않습니다.

<Listing number="19-25" file-name="src/main.rs" caption="`..`을 모호한 방식으로 사용하려는 시도">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

이 예제를 컴파일하면 다음과 같은 오류가 나옵니다.

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

러스트는 `second`로 값을 매칭하기 전에 튜플의 몇 개의 값을 무시해야 하는지, 그리고 그 후에 몇 개의 값을 더 무시해야 하는지 알아낼 수 없습니다. 이 코드는 `2`를 무시하고 `4`를 `second`에 바인딩한 뒤 `8`, `16`, `32`를 무시하라는 의미일 수도 있고, `2`와 `4`를 무시하고 `second`를 `8`에 바인딩한 뒤 `16`과 `32`를 무시하라는 의미일 수도 있고, 그런 식입니다. 변수 이름 `second`는 러스트에게 어떤 특별한 의미도 없으므로, 이렇게 두 곳에서 `..`을 사용하는 것은 모호해서 컴파일 오류를 받게 됩니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="extra-conditionals-with-match-guards"></a>

### 매치 가드로 조건 추가하기

_매치 가드(match guard)_ 는 `match` 갈래에서 패턴 뒤에 명시되는 추가적인 `if` 조건으로, 그 갈래가 선택되려면 이 조건도 매칭되어야 합니다. 매치 가드는 패턴만으로 표현할 수 있는 것보다 더 복잡한 아이디어를 표현하는 데 유용합니다. 다만 `if let`이나 `while let` 표현식에서는 사용할 수 없고 `match` 표현식에서만 사용할 수 있다는 점에 유의하세요.

조건은 패턴에서 만들어진 변수를 사용할 수 있습니다. Listing 19-26은 첫 번째 갈래에 패턴 `Some(x)`와 함께 `if x % 2 == 0`(숫자가 짝수면 `true`가 됨)이라는 매치 가드를 가진 `match`를 보여 줍니다.

<Listing number="19-26" caption="패턴에 매치 가드 추가하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

이 예제는 `The number 4 is even`을 출력합니다. `num`이 첫 번째 갈래의 패턴과 비교될 때 `Some(4)`가 `Some(x)`와 매칭되므로 매칭됩니다. 그런 다음 매치 가드는 `x`를 2로 나눈 나머지가 0과 같은지 검사하고, 같으므로 첫 번째 갈래가 선택됩니다.

만약 `num`이 대신 `Some(5)`였다면, 5를 2로 나눈 나머지는 1이고 0과 같지 않으므로 첫 번째 갈래의 매치 가드는 `false`였을 것입니다. 그러면 러스트는 두 번째 갈래로 넘어가는데, 두 번째 갈래는 매치 가드가 없으므로 어떤 `Some` 변형이라도 매칭하므로 매칭됩니다.

`if x % 2 == 0` 조건을 패턴 안에 표현할 방법은 없으므로, 매치 가드가 이 로직을 표현할 능력을 줍니다. 이러한 추가적인 표현력의 단점은 매치 가드 표현식이 관련될 때 컴파일러가 완전성을 검사하려 시도하지 않는다는 점입니다.

Listing 19-11을 논의할 때, 우리의 패턴 가림 문제를 해결하기 위해 매치 가드를 사용할 수 있다고 언급했습니다. `match` 바깥의 변수를 사용하지 않고 `match` 표현식 안의 패턴에서 새 변수를 만들었던 것을 떠올려 보세요. 그 새 변수는 외부 변수의 값과 비교할 수 없음을 의미했습니다. Listing 19-27은 이 문제를 고치기 위해 매치 가드를 어떻게 사용할 수 있는지 보여 줍니다.

<Listing number="19-27" file-name="src/main.rs" caption="외부 변수와의 동등성 검사를 위한 매치 가드 사용">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

이제 이 코드는 `Default case, x = Some(5)`를 출력합니다. 두 번째 매치 갈래의 패턴은 외부 `y`를 가리는 새 변수 `y`를 도입하지 않으므로, 매치 가드에서 외부 `y`를 사용할 수 있습니다. 외부 `y`를 가리는 `Some(y)`로 패턴을 명시하지 않고 `Some(n)`으로 명시합니다. 이는 `match` 바깥에 `n` 변수가 없으므로 어떤 것도 가리지 않는 새 변수 `n`을 만듭니다.

매치 가드 `if n == y`는 패턴이 아니므로 새 변수를 도입하지 않습니다. 이 `y`는 그것을 가리는 새 `y`가 _아니라_ 외부의 `y`이며, `n`을 `y`와 비교함으로써 외부 `y`와 같은 값을 가진 값을 찾을 수 있습니다.

매치 가드에서 _or_ 연산자 `|`를 사용해 여러 패턴을 명시할 수도 있습니다. 매치 가드 조건은 모든 패턴에 적용됩니다. Listing 19-28은 `|`를 사용하는 패턴과 매치 가드를 결합할 때의 우선순위를 보여 줍니다. 이 예제에서 중요한 부분은, `if y` 매치 가드가 마치 `6`에만 적용되는 것처럼 보일 수 있지만 실제로는 `4`, `5`, _그리고_ `6` 모두에 적용된다는 점입니다.

<Listing number="19-28" caption="여러 패턴을 매치 가드와 결합하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

매치 조건은 `x`의 값이 `4`, `5`, `6` 중 하나와 같고 _또한_ `y`가 `true`일 때만 갈래가 매칭된다고 명시합니다. 이 코드를 실행하면 `x`가 `4`이므로 첫 번째 갈래의 패턴은 매칭되지만, 매치 가드 `if y`가 `false`이므로 첫 번째 갈래는 선택되지 않습니다. 코드는 두 번째 갈래로 넘어가 매칭되고, 이 프로그램은 `no`를 출력합니다. 그 이유는 `if` 조건이 마지막 값 `6`에만 적용되는 것이 아니라 패턴 `4 | 5 | 6` 전체에 적용되기 때문입니다. 다시 말해, 매치 가드와 패턴의 우선순위는 다음과 같이 동작합니다.

```text
(4 | 5 | 6) if y => ...
```

다음과 같지 않습니다.

```text
4 | 5 | (6 if y) => ...
```

코드를 실행해 보면 우선순위 동작이 분명해집니다. 만약 매치 가드가 `|` 연산자로 명시된 값 목록의 마지막 값에만 적용되었다면 갈래가 매칭되었을 것이고 프로그램은 `yes`를 출력했을 것입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="-bindings"></a>

### `@` 바인딩 사용

_at_ 연산자 `@`는 어떤 값을 패턴에 매칭하는지 검사하는 동시에 그 값을 담는 변수를 만들 수 있게 해 줍니다. Listing 19-29에서는 `Message::Hello`의 `id` 필드가 `3..=7` 범위 안에 있는지 검사하고 싶습니다. 또한 그 값을 변수 `id`에 바인딩해, 그 갈래에 연관된 코드에서 사용할 수 있게 하고 싶습니다.

<Listing number="19-29" caption="패턴에서 값을 검사하면서 동시에 그 값을 바인딩하기 위해 `@` 사용하기">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

이 예제는 `Found an id in range: 5`를 출력합니다. 범위 `3..=7` 앞에 `id @`를 명시함으로써, 그 값이 범위 패턴과 매칭되는지 검사함과 동시에 범위와 매칭된 값을 `id`라는 변수에 캡처하고 있습니다.

두 번째 갈래에서는 패턴에 범위만 명시되어 있고, 그 갈래에 연관된 코드에는 `id` 필드의 실제 값을 담는 변수가 없습니다. `id` 필드의 값은 10, 11, 또는 12 중 하나일 수 있지만, 그 패턴과 함께 가는 코드는 그것이 무엇인지 알지 못합니다. `id` 값을 변수에 저장하지 않았기 때문에 패턴 코드는 `id` 필드의 값을 사용할 수 없습니다.

마지막 갈래에서는 범위 없이 변수를 명시했으므로, 갈래의 코드에서 `id`라는 변수로 그 값을 사용할 수 있습니다. 그 이유는 구조체 필드 단축 표기를 사용했기 때문입니다. 그러나 처음 두 갈래에서 했던 것처럼 이 갈래에서는 `id` 필드의 값에 어떤 검사도 적용하지 않았습니다. 어떤 값이라도 이 패턴에 매칭됩니다.

`@`를 사용하면 한 패턴 안에서 값을 검사하면서 동시에 변수에 저장할 수 있습니다.

## 요약

러스트의 패턴은 서로 다른 종류의 데이터를 구분하는 데 매우 유용합니다. `match` 표현식에서 사용될 때, 러스트는 우리의 패턴이 모든 가능한 값을 다루도록 보장하며, 그렇지 않으면 프로그램은 컴파일되지 않습니다. `let` 문과 함수 매개변수의 패턴은 그러한 구문을 더 유용하게 만들어, 값을 더 작은 부분으로 분해하고 그 부분들을 변수에 할당할 수 있게 해 줍니다. 우리의 필요에 맞게 단순하거나 복잡한 패턴을 만들 수 있습니다.

다음으로 책의 끝에서 두 번째 장에서는 다양한 러스트 기능의 고급 측면들을 살펴보겠습니다.
