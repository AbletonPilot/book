<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

## 공유된 동작을 추상화하기 위한 트레이트 객체 사용

8장에서 벡터의 한 가지 한계로 한 가지 타입의 요소만 저장할 수 있다는 점을 언급했습니다. Listing 8-9에서는 정수, 부동소수점 수, 텍스트를 담을 수 있는 변형들을 가진 `SpreadsheetCell` 열거형을 정의해 우회 방법을 만들었습니다. 이 덕분에 셀마다 다른 종류의 데이터를 저장하면서도 한 행의 셀들을 표현하는 벡터를 가질 수 있었습니다. 이는 코드를 컴파일할 때 알 수 있는 고정된 타입 집합으로 항목을 서로 교체해야 할 때 매우 좋은 해법입니다.

그러나 때때로 우리는 라이브러리 사용자가 특정 상황에서 유효한 타입 집합을 확장할 수 있게 해 주고 싶습니다. 이를 어떻게 달성할 수 있는지 보이기 위해, 항목 목록을 순회하면서 각 항목에 대해 `draw` 메서드를 호출해 화면에 그리는 예제 그래픽 사용자 인터페이스(GUI) 도구를 만들어 보겠습니다. 이는 GUI 도구에서 흔한 기법입니다. GUI 라이브러리의 구조를 담은 `gui`라는 라이브러리 크레이트를 만들 것입니다. 이 크레이트에는 사람들이 사용할 수 있는 `Button`이나 `TextField` 같은 몇 가지 타입이 포함될 수 있습니다. 또한 `gui` 사용자들은 그릴 수 있는 자신만의 타입을 만들고 싶어 할 것입니다. 예를 들어 한 프로그래머는 `Image`를 추가할 수 있고, 다른 프로그래머는 `SelectBox`를 추가할 수 있습니다.

라이브러리를 작성하는 시점에는 다른 프로그래머들이 만들고 싶어 할 모든 타입을 알고 정의할 수 없습니다. 그러나 `gui`가 다양한 타입의 여러 값을 추적해야 하고, 그렇게 다른 타입의 값들 각각에 대해 `draw` 메서드를 호출해야 한다는 것은 알고 있습니다. `draw` 메서드를 호출했을 때 정확히 무슨 일이 일어날지 알 필요는 없고, 그 값에 우리가 호출할 수 있는 메서드가 있다는 것만 알면 됩니다.

상속이 있는 언어에서 이를 하려면, `draw`라는 메서드를 가진 `Component`라는 클래스를 정의할 수도 있을 것입니다. 그러면 `Button`, `Image`, `SelectBox` 같은 다른 클래스들이 `Component`를 상속해 `draw` 메서드를 물려받을 것입니다. 각각은 자신의 동작을 정의하기 위해 `draw` 메서드를 재정의할 수 있고, 프레임워크는 모든 타입을 마치 `Component` 인스턴스인 것처럼 취급해 그 위에서 `draw`를 호출할 수 있을 것입니다. 그러나 러스트에는 상속이 없으므로, 사용자들이 라이브러리와 호환되는 새로운 타입을 만들 수 있도록 `gui` 라이브러리를 구성할 다른 방법이 필요합니다.

### 공통된 동작을 위한 트레이트 정의하기

`gui`가 가졌으면 하는 동작을 구현하기 위해, `draw`라는 메서드 하나를 가진 `Draw`라는 트레이트를 정의하겠습니다. 그런 다음 트레이트 객체를 받는 벡터를 정의할 수 있습니다. _트레이트 객체_ 는 우리가 지정한 트레이트를 구현하는 어떤 타입의 인스턴스와, 런타임에 그 타입의 트레이트 메서드를 조회하는 데 사용되는 테이블 모두를 가리킵니다. 트레이트 객체는 참조나 `Box<T>` 스마트 포인터와 같은 일종의 포인터를 지정하고, 그 다음에 `dyn` 키워드를 쓰고, 마지막으로 관련 트레이트를 지정해 만듭니다. (트레이트 객체가 왜 포인터를 사용해야 하는지에 대해서는 20장의 [“동적 크기 타입과 `Sized` 트레이트”][dynamically-sized]<!-- ignore --> 절에서 이야기하겠습니다.) 트레이트 객체는 제네릭이나 구체 타입 대신에 사용할 수 있습니다. 트레이트 객체를 사용하는 어디에서든, 러스트의 타입 시스템은 그 문맥에서 사용된 어떤 값이라도 트레이트 객체의 트레이트를 구현한다는 것을 컴파일 시점에 보장해 줍니다. 따라서 컴파일 시점에 가능한 모든 타입을 알 필요가 없습니다.

러스트에서는 다른 언어의 객체와 구분하기 위해 구조체와 열거형을 “객체”라 부르지 않는다고 말한 바 있습니다. 구조체나 열거형에서는 구조체 필드의 데이터와 `impl` 블록의 동작이 분리되어 있는 반면, 다른 언어들에서는 데이터와 동작이 하나의 개념으로 결합된 것을 흔히 객체라고 부릅니다. 트레이트 객체는 다른 언어의 객체와 달리 트레이트 객체에 데이터를 추가할 수 없다는 점에서 차이가 있습니다. 트레이트 객체는 다른 언어의 객체만큼 일반적으로 유용하지는 않습니다. 트레이트 객체의 구체적인 목적은 공통된 동작에 대한 추상화를 가능하게 하는 것입니다.

Listing 18-3은 `draw`라는 메서드 하나를 가진 `Draw`라는 트레이트를 정의하는 방법을 보여 줍니다.

<Listing number="18-3" file-name="src/lib.rs" caption="`Draw` 트레이트의 정의">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

이 문법은 10장에서 트레이트를 정의하는 방법에 대한 논의에서 익숙하실 것입니다. 다음으로 새로운 문법이 등장합니다. Listing 18-4는 `components`라는 벡터를 담는 `Screen`이라는 구조체를 정의합니다. 이 벡터는 `Box<dyn Draw>` 타입인데, 이는 트레이트 객체이며, `Draw` 트레이트를 구현하는 `Box` 안의 어떤 타입이든 대신할 수 있는 자리표시자입니다.

<Listing number="18-4" file-name="src/lib.rs" caption="`Draw` 트레이트를 구현하는 트레이트 객체들의 벡터를 담는 `components` 필드를 가진 `Screen` 구조체의 정의">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

`Screen` 구조체에 Listing 18-5에서 보이듯 자신의 `components` 각각에 대해 `draw` 메서드를 호출하는 `run`이라는 메서드를 정의합니다.

<Listing number="18-5" file-name="src/lib.rs" caption="각 컴포넌트에 대해 `draw` 메서드를 호출하는 `Screen`의 `run` 메서드">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

이는 트레이트 바운드를 가진 제네릭 타입 매개변수를 사용하는 구조체를 정의하는 것과는 다르게 동작합니다. 제네릭 타입 매개변수는 한 번에 단 하나의 구체 타입으로만 치환될 수 있는 반면, 트레이트 객체는 런타임에 트레이트 객체 자리에 여러 구체 타입이 채워질 수 있게 해 줍니다. 예를 들어 Listing 18-6처럼 제네릭 타입과 트레이트 바운드를 사용해 `Screen` 구조체를 정의할 수도 있었습니다.

<Listing number="18-6" file-name="src/lib.rs" caption="제네릭과 트레이트 바운드를 사용한 `Screen` 구조체와 `run` 메서드의 대안적 구현">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

이는 `Screen` 인스턴스가 모두 `Button` 타입이거나 모두 `TextField` 타입인 컴포넌트 목록을 가지도록 우리를 제한합니다. 항상 동질적인 컬렉션만 다룰 것이라면, 정의가 컴파일 시점에 구체 타입을 사용하도록 단형화되므로 제네릭과 트레이트 바운드를 사용하는 편이 낫습니다.

반면, 트레이트 객체를 사용하는 방식에서는 하나의 `Screen` 인스턴스가 `Box<Button>`과 `Box<TextField>` 둘 다 포함된 `Vec<T>`를 담을 수 있습니다. 이것이 어떻게 동작하는지 살펴본 다음, 런타임 성능 측면에서의 영향에 대해 이야기하겠습니다.

### 트레이트 구현하기

이제 `Draw` 트레이트를 구현하는 몇 가지 타입을 추가하겠습니다. `Button` 타입을 제공할 것입니다. 다시 한번 말하지만, 실제로 GUI 라이브러리를 구현하는 것은 이 책의 범위를 넘어서므로 `draw` 메서드의 본문에는 유용한 구현이 없습니다. 구현이 어떤 모습일지 상상해 보면, `Button` 구조체는 Listing 18-7에서 보이듯 `width`, `height`, `label` 필드를 가질 수 있을 것입니다.

<Listing number="18-7" file-name="src/lib.rs" caption="`Draw` 트레이트를 구현하는 `Button` 구조체">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

`Button`의 `width`, `height`, `label` 필드는 다른 컴포넌트의 필드와 다를 것입니다. 예를 들어 `TextField` 타입은 그와 같은 필드에 더해 `placeholder` 필드를 가질 수도 있습니다. 화면에 그리려는 각 타입은 `Draw` 트레이트를 구현하지만, 그 특정 타입을 어떻게 그릴지 정의하기 위해 `draw` 메서드 안에서 서로 다른 코드를 사용합니다. 여기에서는 `Button`이 그러한 예입니다(앞서 언급했듯이 실제 GUI 코드는 없습니다). 예를 들어 `Button` 타입은 사용자가 버튼을 클릭했을 때 어떤 일이 일어날지에 관한 메서드를 담은 추가 `impl` 블록을 가질 수도 있습니다. 이런 종류의 메서드는 `TextField` 같은 타입에는 적용되지 않을 것입니다.

우리 라이브러리를 사용하는 누군가가 `width`, `height`, `options` 필드를 가진 `SelectBox` 구조체를 구현하기로 결정했다면, Listing 18-8에서 보이듯 `SelectBox` 타입에도 `Draw` 트레이트를 구현할 것입니다.

<Listing number="18-8" file-name="src/main.rs" caption="`gui`를 사용하는 또 다른 크레이트가 `SelectBox` 구조체에 `Draw` 트레이트를 구현하는 모습">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

이제 라이브러리 사용자는 `Screen` 인스턴스를 만드는 자신의 `main` 함수를 작성할 수 있습니다. 그 `Screen` 인스턴스에 `SelectBox`와 `Button`을 각각 `Box<T>`에 넣어 트레이트 객체로 만든 다음 추가할 수 있습니다. 그런 다음 `Screen` 인스턴스에서 `run` 메서드를 호출하면, 각 컴포넌트에 대해 `draw`가 호출됩니다. Listing 18-9는 이 구현을 보여 줍니다.

<Listing number="18-9" file-name="src/main.rs" caption="같은 트레이트를 구현하는 서로 다른 타입의 값들을 저장하기 위해 트레이트 객체 사용하기">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

라이브러리를 작성할 때 누군가 `SelectBox` 타입을 추가할지 알지 못했지만, `SelectBox`가 `Draw` 트레이트를 구현하므로(즉 `draw` 메서드를 구현하므로) 우리의 `Screen` 구현은 새로운 타입에 대해 동작하고 그것을 그릴 수 있었습니다.

이러한 개념—값의 구체 타입이 아니라 그 값이 응답하는 메시지에만 관심을 갖는다는 개념—은 동적 타입 언어의 _덕 타이핑(duck typing)_ 개념과 비슷합니다. 오리처럼 걷고 오리처럼 꽥꽥거리면 그것은 분명 오리다! Listing 18-5의 `Screen`에 있는 `run` 구현에서, `run`은 각 컴포넌트의 구체 타입이 무엇인지 알 필요가 없습니다. 컴포넌트가 `Button`의 인스턴스인지 `SelectBox`의 인스턴스인지 검사하지 않고, 그저 컴포넌트에서 `draw` 메서드를 호출할 뿐입니다. `components` 벡터의 값 타입을 `Box<dyn Draw>`로 지정함으로써, `Screen`이 `draw` 메서드를 호출할 수 있는 값들을 필요로 한다고 정의한 것입니다.

트레이트 객체와 러스트의 타입 시스템을 사용해 덕 타이핑을 사용하는 코드와 비슷한 코드를 작성할 때의 장점은, 어떤 값이 특정 메서드를 구현했는지 런타임에 검사할 필요가 전혀 없고, 어떤 값이 메서드를 구현하지 않았는데도 호출하면 발생할 오류를 걱정할 필요도 없다는 것입니다. 트레이트 객체에 필요한 트레이트를 값들이 구현하지 않으면 러스트는 우리의 코드를 컴파일하지 않습니다.

예를 들어 Listing 18-10은 컴포넌트로 `String`을 가진 `Screen`을 만들려고 하면 무슨 일이 일어나는지 보여 줍니다.

<Listing number="18-10" file-name="src/main.rs" caption="트레이트 객체의 트레이트를 구현하지 않는 타입을 사용하려는 시도">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

`String`이 `Draw` 트레이트를 구현하지 않으므로 다음 오류가 나옵니다.

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

이 오류는 우리가 `Screen`에 의도하지 않은 무언가를 전달하고 있어 다른 타입을 전달해야 하거나, 아니면 `Screen`이 그것에 대해 `draw`를 호출할 수 있도록 `String`에 `Draw`를 구현해야 한다는 것을 알려 줍니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="trait-objects-perform-dynamic-dispatch"></a>

### 동적 디스패치 수행

10장의 [“제네릭을 사용하는 코드의 성능”][performance-of-code-using-generics]<!-- ignore --> 절에서 컴파일러가 제네릭에 대해 수행하는 단형화 과정에 대해 논의했던 것을 떠올려 보세요. 컴파일러는 우리가 제네릭 타입 매개변수 자리에 사용한 각 구체 타입에 대해 함수와 메서드의 비제네릭 구현을 생성합니다. 단형화의 결과로 생성되는 코드는 _정적 디스패치(static dispatch)_ 를 수행하는데, 이는 컴파일러가 컴파일 시점에 어떤 메서드를 호출하는지 아는 경우입니다. 이는 _동적 디스패치(dynamic dispatch)_ 와 대비되는데, 동적 디스패치는 컴파일러가 컴파일 시점에 어떤 메서드를 호출하는지 알 수 없는 경우입니다. 동적 디스패치의 경우, 컴파일러는 런타임에 어떤 메서드를 호출할지 알게 될 코드를 만들어 냅니다.

트레이트 객체를 사용할 때 러스트는 반드시 동적 디스패치를 사용해야 합니다. 컴파일러는 트레이트 객체를 사용하는 코드와 함께 사용될 수 있는 모든 타입을 알지 못하므로, 어떤 타입에 구현된 어떤 메서드를 호출해야 할지 알지 못합니다. 대신 런타임에 러스트는 트레이트 객체 안의 포인터를 사용해 어떤 메서드를 호출할지 알아냅니다. 이러한 조회는 정적 디스패치에서는 발생하지 않는 런타임 비용을 수반합니다. 동적 디스패치는 또한 컴파일러가 메서드의 코드를 인라인하는 선택을 막아 일부 최적화를 막으며, 러스트에는 동적 디스패치를 사용할 수 있는 곳과 없는 곳에 대한 규칙이 있는데 이를 _dyn 호환성(dyn compatibility)_ 이라고 부릅니다. 그 규칙들은 이 논의의 범위를 넘어서지만, [참조 문서][dyn-compatibility]<!-- ignore -->에서 더 자세히 읽어 볼 수 있습니다. 그러나 우리는 Listing 18-5에서 작성한 코드에서 추가적인 유연성을 얻었고 Listing 18-9에서 그것을 지원할 수 있었으므로, 고려해 볼 만한 트레이드오프입니다.

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
