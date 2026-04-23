## 이터레이터로 연속된 항목 처리하기

이터레이터 패턴은 일련의 항목에 대해 차례로 어떤 작업을 수행할 수 있게 해
줍니다. 이터레이터는 각 항목을 순회하는 로직과, 언제 순회가 끝나는지를 결정하는
책임을 집니다. 이터레이터를 사용하면 그런 로직을 직접 다시 구현하지 않아도
됩니다.

러스트에서 이터레이터는 _게으릅니다(lazy)_. 즉 이터레이터를 소비해 소진시키는
메서드를 호출하기 전까지는 아무 효과도 없습니다. 예를 들어 Listing 13-10의
코드는 `Vec<T>`에 정의된 `iter` 메서드를 호출해 벡터 `v1`의 항목에 대한
이터레이터를 만듭니다. 이 코드 자체는 아무 유용한 일도 하지 않습니다.

<Listing number="13-10" file-name="src/main.rs" caption="이터레이터 만들기">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

이터레이터는 `v1_iter` 변수에 저장됩니다. 이터레이터를 만든 뒤에는 여러 가지
방식으로 사용할 수 있습니다. Listing 3-5에서는 `for` 루프로 배열을 순회하면서
각 항목에 어떤 코드를 실행했습니다. 내부적으로 이는 이터레이터를 암묵적으로
만들어 소비했지만, 지금까지는 그 과정이 정확히 어떻게 동작하는지 넘어갔습니다.

Listing 13-11의 예제에서는 이터레이터 생성을 `for` 루프에서의 이터레이터
사용과 분리합니다. `v1_iter`의 이터레이터로 `for` 루프가 호출되면, 이터레이터
의 각 요소가 루프의 한 반복에 사용되어 각 값을 출력합니다.

<Listing number="13-11" file-name="src/main.rs" caption="`for` 루프에서 이터레이터 사용하기">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

표준 라이브러리에 이터레이터가 제공되지 않는 언어에서는 같은 기능을 쓰려면
인덱스 0에서 변수를 시작해 그 변수로 벡터를 인덱싱해 값을 얻고, 벡터의 전체
항목 수에 도달할 때까지 루프에서 변수 값을 증가시키는 식으로 작성해야 할
것입니다.

이터레이터는 이 모든 로직을 대신 처리해 주어, 실수할 수 있는 반복 코드를
줄여 줍니다. 벡터처럼 인덱싱할 수 있는 데이터 구조뿐 아니라, 다양한 종류의
시퀀스에 같은 로직을 사용할 수 있는 유연성을 줍니다. 이터레이터가 그것을
어떻게 하는지 살펴봅시다.

### `Iterator` 트레이트와 `next` 메서드

모든 이터레이터는 표준 라이브러리에 정의된 `Iterator`라는 트레이트를 구현합니다.
이 트레이트의 정의는 다음과 같습니다.

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

이 정의가 몇 가지 새로운 문법을 사용한다는 점에 유의하세요. `type Item`과
`Self::Item`은 이 트레이트와 연관 타입을 정의하는 것입니다. 연관 타입은
20장에서 자세히 다룹니다. 지금은 이 코드가 `Iterator` 트레이트를 구현하려면
`Item` 타입도 정의해야 하며, 이 `Item` 타입이 `next` 메서드의 반환 타입에
사용된다는 것만 알면 됩니다. 다시 말해 `Item` 타입은 이터레이터에서 반환되는
타입이 됩니다.

`Iterator` 트레이트는 구현자에게 단 하나의 메서드만 정의하도록 요구합니다.
`next` 메서드입니다. `next` 메서드는 이터레이터의 항목을 한 번에 하나씩
`Some`에 감싸 반환하고, 순회가 끝나면 `None`을 반환합니다.

이터레이터에 직접 `next` 메서드를 호출할 수 있습니다. Listing 13-12는 벡터
에서 만든 이터레이터에 `next`를 반복 호출했을 때 어떤 값이 반환되는지 보여
줍니다.

<Listing number="13-12" file-name="src/lib.rs" caption="이터레이터의 `next` 메서드 호출하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

`v1_iter`를 가변(mutable)으로 만들어야 했음에 유의하세요. 이터레이터에서
`next` 메서드를 호출하면, 이터레이터가 시퀀스의 어디에 있는지 추적하기 위해
사용하는 내부 상태가 바뀝니다. 다시 말해 이 코드는 이터레이터를 _소비_, 즉
소진시킵니다. `next` 호출 하나는 이터레이터에서 항목 하나를 먹어 치웁니다.
`for` 루프를 사용할 때는 `v1_iter`를 가변으로 만들 필요가 없었는데, 루프가
`v1_iter`의 소유권을 가져가고 내부적으로 가변으로 만들었기 때문입니다.

또한 `next` 호출에서 얻는 값은 벡터의 값에 대한 불변 참조임에 유의하세요.
`iter` 메서드는 불변 참조에 대한 이터레이터를 만듭니다. `v1`의 소유권을
가져가서 소유된 값을 반환하는 이터레이터를 만들고 싶다면 `iter` 대신
`into_iter`를 호출할 수 있습니다. 마찬가지로 가변 참조에 대해 순회하고 싶
다면 `iter` 대신 `iter_mut`를 호출할 수 있습니다.

### 이터레이터를 소비하는 메서드

`Iterator` 트레이트에는 표준 라이브러리가 제공하는 기본 구현이 있는 여러
가지 메서드가 있습니다. 이 메서드들에 대해서는 `Iterator` 트레이트의 표준
라이브러리 API 문서를 보면 알 수 있습니다. 그중 일부 메서드는 정의에서
`next` 메서드를 호출합니다. 그래서 `Iterator` 트레이트를 구현할 때 `next`
메서드를 구현해야 하는 것입니다.

`next`를 호출하는 메서드들은 _소비 어댑터(consuming adapters)_ 라고 부릅니다.
그것을 호출하면 이터레이터를 소진시키기 때문입니다. 한 예로 `sum` 메서드는
이터레이터의 소유권을 가져가서 `next`를 반복 호출해 항목을 순회하며
이터레이터를 소비합니다. 순회하면서 각 항목을 누적 합에 더하고, 순회가
완료되면 합계를 반환합니다. Listing 13-13에는 `sum` 메서드 사용을 보여
주는 테스트가 있습니다.

<Listing number="13-13" file-name="src/lib.rs" caption="이터레이터의 모든 항목의 합을 얻기 위해 `sum` 메서드 호출하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

`sum`은 우리가 그것을 호출한 이터레이터의 소유권을 가져가므로, `sum` 호출
이후에는 `v1_iter`를 사용할 수 없습니다.

### 다른 이터레이터를 만들어 내는 메서드

_이터레이터 어댑터(iterator adapters)_ 는 `Iterator` 트레이트에 정의된
메서드 중 이터레이터를 소비하지 않는 것들입니다. 대신 원래 이터레이터의
어떤 측면을 바꿔서 다른 이터레이터를 만들어 냅니다.

Listing 13-14는 이터레이터 어댑터 메서드 `map`을 호출하는 예를 보여 줍니다.
`map`은 항목을 순회할 때 각 항목에 대해 호출할 클로저를 받습니다. `map`
메서드는 수정된 항목을 만들어 내는 새 이터레이터를 반환합니다. 여기의 클로저
는 벡터의 각 항목을 1씩 증가시키는 새 이터레이터를 만듭니다.

<Listing number="13-14" file-name="src/main.rs" caption="이터레이터 어댑터 `map`을 호출해 새 이터레이터 만들기">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

그러나 이 코드는 경고를 만들어 냅니다.

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Listing 13-14의 코드는 아무 일도 하지 않습니다. 우리가 지정한 클로저는 호출
되지 않습니다. 경고가 그 이유를 상기시켜 줍니다. 이터레이터 어댑터는 게으르고,
여기서 이터레이터를 소비해야 합니다.

이 경고를 고치고 이터레이터를 소비하기 위해, Listing 12-1에서 `env::args`와
함께 사용한 `collect` 메서드를 쓰겠습니다. 이 메서드는 이터레이터를 소비해
그 결과 값을 컬렉션 데이터 타입으로 모읍니다.

Listing 13-15에서는 `map` 호출에서 반환된 이터레이터를 순회한 결과를 벡터로
모읍니다. 이 벡터는 원래 벡터의 각 항목을 1씩 증가시킨 값들을 담게 됩니다.

<Listing number="13-15" file-name="src/main.rs" caption="`map` 메서드를 호출해 새 이터레이터를 만든 다음 `collect`로 새 이터레이터를 소비해 벡터 만들기">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

`map`은 클로저를 받으므로, 각 항목에 대해 수행하고 싶은 어떤 연산이든 지정할
수 있습니다. 이것은 `Iterator` 트레이트가 제공하는 순회 동작을 재사용하면서도
어떤 동작을 사용자화할 수 있게 해 주는 클로저의 좋은 예입니다.

이터레이터 어댑터에 대한 호출을 여러 개 연결해서 복잡한 동작을 읽기 좋게
수행할 수 있습니다. 그러나 모든 이터레이터는 게으르므로, 이터레이터 어댑터
호출의 결과를 얻으려면 소비 어댑터 메서드 중 하나를 호출해야 합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-closures-that-capture-their-environment"></a>

### 환경을 캡처하는 클로저

많은 이터레이터 어댑터가 클로저를 인수로 받으며, 이터레이터 어댑터의 인수로
보통 지정하는 클로저는 환경을 캡처하는 클로저입니다.

이 예제에서는 클로저를 받는 `filter` 메서드를 사용하겠습니다. 클로저는
이터레이터에서 항목을 받아 `bool`을 반환합니다. 클로저가 `true`를 반환하면
`filter`가 만들어 내는 순회에 값이 포함됩니다. 클로저가 `false`를 반환하면
값이 포함되지 않습니다.

Listing 13-16에서는 환경에서 `shoe_size` 변수를 캡처하는 클로저와 함께
`filter`를 사용해 `Shoe` 구조체 인스턴스의 컬렉션을 순회합니다. 지정된 크기의
신발만 반환할 것입니다.

<Listing number="13-16" file-name="src/lib.rs" caption="`shoe_size`를 캡처하는 클로저와 함께 `filter` 메서드 사용하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

`shoes_in_size` 함수는 매개변수로 신발 벡터와 신발 크기를 받아 소유권을
가져갑니다. 지정된 크기의 신발만 담은 벡터를 반환합니다.

`shoes_in_size`의 본문에서는 `into_iter`를 호출해 벡터의 소유권을 가져가는
이터레이터를 만듭니다. 그런 다음 `filter`를 호출해 그 이터레이터를, 클로저가
`true`를 반환하는 요소만 담은 새 이터레이터로 적응시킵니다.

클로저는 환경에서 `shoe_size` 매개변수를 캡처해 그 값을 각 신발의 크기와
비교하고, 지정된 크기의 신발만 유지합니다. 마지막으로 `collect`를 호출하면
적응된 이터레이터가 반환한 값들을 벡터로 모아, 함수가 그것을 반환합니다.

테스트는 `shoes_in_size`를 호출했을 때 우리가 지정한 값과 같은 크기의 신발
만 돌려받음을 보여 줍니다.
