## `RefCell<T>`와 내부 가변성 패턴

_내부 가변성(interior mutability)_ 은 불변 참조가 있는 데이터라도 변경할 수
있게 해 주는 러스트의 디자인 패턴입니다. 보통 이 동작은 빌림 규칙에 의해
허용되지 않습니다. 데이터를 변경하기 위해 이 패턴은 자료 구조 내부에 `unsafe`
코드를 사용해 변경과 빌림을 관장하는 러스트의 일반 규칙을 구부립니다. 안전
하지 않은 코드는 컴파일러에게 우리가 컴파일러의 검사에 의존하는 대신 수동으로
규칙을 검사하고 있음을 알리는 것입니다. 안전하지 않은 코드는 20장에서 더
논의합니다.

내부 가변성 패턴을 사용하는 타입은 컴파일러가 보장할 수 없더라도 런타임에
빌림 규칙이 지켜질 것을 우리가 보장할 수 있을 때만 사용할 수 있습니다. 관련된
`unsafe` 코드는 그런 다음 안전한 API로 감싸지며, 바깥 타입은 여전히 불변
입니다.

내부 가변성 패턴을 따르는 `RefCell<T>` 타입을 살펴보며 이 개념을 탐구해 봅시다.

<!-- Old headings. Do not remove or links may break. -->

<a id="enforcing-borrowing-rules-at-runtime-with-refcellt"></a>

### 런타임에 빌림 규칙 강제하기

`Rc<T>`와 달리 `RefCell<T>` 타입은 그것이 담는 데이터에 대한 단일 소유권을
나타냅니다. 그렇다면 `Box<T>` 같은 타입과 `RefCell<T>`의 차이는 무엇일까요?
4장에서 배운 빌림 규칙을 떠올려 보세요.

- 어떤 시점에든 하나의 가변 참조 _또는_ 얼마든지 많은 수의 불변 참조를 가질
  수 있습니다(둘 다는 안 됩니다).
- 참조는 항상 유효해야 합니다.

참조와 `Box<T>`에서는 빌림 규칙의 불변식이 컴파일 타임에 강제됩니다.
`RefCell<T>`에서는 이 불변식이 _런타임에_ 강제됩니다. 참조에서는 이 규칙을
어기면 컴파일러 오류가 납니다. `RefCell<T>`에서는 이 규칙을 어기면 프로그램이
패닉하고 종료됩니다.

빌림 규칙을 컴파일 타임에 검사하는 것의 장점은 개발 과정 초기에 오류가 잡히고,
모든 분석이 미리 완료되므로 런타임 성능에 영향이 없다는 점입니다. 이런 이유로
빌림 규칙을 컴파일 타임에 검사하는 것은 대부분의 경우 최선의 선택이며, 그래서
러스트의 기본입니다.

빌림 규칙을 대신 런타임에 검사하는 것의 장점은 컴파일 타임 검사로는 허용되지
않았을 특정 메모리 안전 시나리오가 허용된다는 점입니다. 러스트 컴파일러 같은
정적 분석은 본래 보수적입니다. 코드의 어떤 속성은 코드를 분석해 감지하는 것이
불가능합니다. 가장 유명한 예는 정지 문제(Halting Problem)로, 이 책의 범위를
벗어나지만 연구할 가치가 있는 흥미로운 주제입니다.

일부 분석이 불가능하므로, 러스트 컴파일러가 소유권 규칙을 코드가 준수하는지
확신할 수 없다면 정확한 프로그램도 거부할 수 있습니다. 이런 식으로 보수적
입니다. 러스트가 잘못된 프로그램을 받아들였다면, 사용자는 러스트가 제공하는
보장을 신뢰할 수 없을 것입니다. 반면 러스트가 정확한 프로그램을 거부하면
프로그래머는 불편해지겠지만 재앙적인 일은 일어나지 않습니다. 여러분의 코드가
빌림 규칙을 따른다고 확신하지만 컴파일러가 이를 이해하고 보장할 수 없을 때
`RefCell<T>` 타입이 유용합니다.

`Rc<T>`와 마찬가지로 `RefCell<T>`도 단일 스레드 시나리오에서만 사용할 수
있으며, 다중 스레드 맥락에서 사용하려 하면 컴파일 타임 오류가 납니다. 16장
에서 다중 스레드 프로그램에서 `RefCell<T>` 기능을 얻는 방법을 이야기합니다.

`Box<T>`, `Rc<T>`, `RefCell<T>`를 선택하는 이유를 요약하면 다음과 같습니다.

- `Rc<T>`는 같은 데이터의 여러 소유자를 가능하게 합니다. `Box<T>`와 `RefCell<T>`
  는 단일 소유자를 가집니다.
- `Box<T>`는 컴파일 타임에 검사되는 불변 또는 가변 빌림을 허용합니다. `Rc<T>`
  는 컴파일 타임에 검사되는 불변 빌림만 허용합니다. `RefCell<T>`는 런타임에
  검사되는 불변 또는 가변 빌림을 허용합니다.
- `RefCell<T>`는 런타임에 검사되는 가변 빌림을 허용하므로, `RefCell<T>`가
  불변이더라도 `RefCell<T>` 내부의 값을 변경할 수 있습니다.

불변 값 내부의 값을 변경하는 것이 내부 가변성 패턴입니다. 내부 가변성이 유용한
상황을 살펴보고, 이것이 어떻게 가능한지 알아봅시다.

<!-- Old headings. Do not remove or links may break. -->

<a id="interior-mutability-a-mutable-borrow-to-an-immutable-value"></a>

### 내부 가변성 사용하기

빌림 규칙의 결과로, 불변 값이 있을 때 그것을 가변으로 빌릴 수 없습니다. 예를
들어 이 코드는 컴파일되지 않습니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

이 코드를 컴파일하려 하면 다음 오류가 납니다.

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

그러나 값이 자신의 메서드에서 스스로를 변경하면서 다른 코드에는 불변으로
보이면 유용한 상황이 있습니다. 값의 메서드 밖의 코드는 값을 변경할 수 없을
것입니다. `RefCell<T>`를 사용하는 것은 내부 가변성을 갖는 한 가지 방법이지만,
`RefCell<T>`가 빌림 규칙을 완전히 우회하는 것은 아닙니다. 컴파일러의 빌림
검사기는 이 내부 가변성을 허용하고, 빌림 규칙은 대신 런타임에 검사됩니다.
규칙을 어기면 컴파일러 오류 대신 `panic!`이 발생합니다.

`RefCell<T>`로 불변 값을 변경하고, 그것이 왜 유용한지 볼 수 있는 실용적
예제를 살펴봅시다.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-use-case-for-interior-mutability-mock-objects"></a>

#### 목 객체로 테스트하기

가끔 테스트 동안 특정 동작을 관찰하고 올바르게 구현되었는지 단언하기 위해
프로그래머가 어떤 타입 대신 다른 타입을 사용합니다. 이 자리 표시자 타입을
_테스트 더블(test double)_ 이라고 합니다. 영화 제작에서 스턴트 대역, 즉 특히
까다로운 장면을 찍기 위해 배우를 대신해 사람이 들어서는 것을 떠올리면 됩니다.
테스트 더블은 우리가 테스트를 실행할 때 다른 타입을 대신합니다. _목 객체(mock
object)_ 는 테스트 중에 일어난 일을 기록하는 테스트 더블의 특정 종류로, 올바른
동작이 일어났음을 단언할 수 있게 해 줍니다.

러스트에는 다른 언어들이 객체를 가지는 것과 같은 의미의 객체가 없으며, 러스트
는 일부 다른 언어들처럼 표준 라이브러리에 목 객체 기능이 내장되어 있지
않습니다. 그러나 목 객체와 같은 목적을 수행하는 구조체를 만들 수는 확실히
있습니다.

테스트할 시나리오는 이렇습니다. 값을 최댓값과 비교해 추적하고, 현재 값이
최댓값에 얼마나 가까운지에 따라 메시지를 보내는 라이브러리를 만들겠습니다.
이 라이브러리는 예를 들어 사용자가 허용된 API 호출 수 할당량을 추적하는 데
사용될 수 있습니다.

우리 라이브러리는 값이 최댓값에 얼마나 가까운지 추적하는 기능과 어떤 시점에
어떤 메시지가 와야 하는지만 제공합니다. 우리 라이브러리를 사용하는 애플리케이션
은 메시지를 보내는 메커니즘을 제공해야 합니다. 애플리케이션은 메시지를 사용자
에게 직접 보여 주거나, 이메일을 보내거나, 문자 메시지를 보내거나, 다른 무언가를
할 수 있습니다. 라이브러리는 그 세부 사항을 알 필요가 없습니다. 필요한 것은
우리가 제공할 `Messenger`라는 트레이트를 구현하는 무언가뿐입니다. Listing
15-20이 라이브러리 코드를 보여 줍니다.

<Listing number="15-20" file-name="src/lib.rs" caption="값이 최댓값에 얼마나 가까운지 추적하고 특정 수준에서 경고하는 라이브러리">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

이 코드의 중요한 부분 중 하나는 `Messenger` 트레이트에 `self`에 대한 불변
참조와 메시지 텍스트를 받는 `send`라는 메서드가 있다는 점입니다. 이 트레이트
는 우리 목 객체가 실제 객체와 같은 방식으로 사용되도록 구현해야 하는 인터페이스
입니다. 또 다른 중요한 부분은 `LimitTracker`의 `set_value` 메서드 동작을
테스트하고 싶다는 점입니다. `value` 매개변수로 무엇을 전달할지 바꿀 수는
있지만, `set_value`는 단언할 것을 반환하지 않습니다. `Messenger` 트레이트를
구현하는 무언가와 특정 `max` 값으로 `LimitTracker`를 만들고 `value`에 여러
수를 전달하면, 메신저가 적절한 메시지를 보내도록 지시받는지 말할 수 있어야
합니다.

`send`를 호출할 때 이메일이나 문자 메시지를 보내는 대신, 보내라고 지시받은
메시지를 추적만 하는 목 객체가 필요합니다. 목 객체의 새 인스턴스를 만들고,
그 목 객체를 사용하는 `LimitTracker`를 만들고, `LimitTracker`의 `set_value`
메서드를 호출한 뒤, 목 객체가 우리가 기대한 메시지를 가지고 있는지 확인할
수 있습니다. Listing 15-21은 그렇게 할 목 객체의 구현 시도를 보여 주지만
빌림 검사기가 허용하지 않습니다.

<Listing number="15-21" file-name="src/lib.rs" caption="빌림 검사기가 허용하지 않는 `MockMessenger` 구현 시도">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

이 테스트 코드는 보내라고 지시받은 메시지를 추적하기 위해 `String` 값들의
`Vec`를 담는 `sent_messages` 필드를 가진 `MockMessenger` 구조체를 정의합니다.
또한 빈 메시지 목록으로 시작하는 새 `MockMessenger` 값을 편리하게 만들기 위한
연관 함수 `new`도 정의합니다. 그런 다음 `MockMessenger`를 `LimitTracker`에
줄 수 있도록 `MockMessenger`에 대해 `Messenger` 트레이트를 구현합니다. `send`
메서드의 정의에서는 매개변수로 전달된 메시지를 받아 `MockMessenger`의
`sent_messages` 목록에 저장합니다.

테스트에서는 `LimitTracker`에 `value`를 `max` 값의 75%보다 큰 값으로 설정
하라고 지시할 때 무슨 일이 일어나는지 테스트합니다. 먼저 빈 메시지 목록으로
시작할 새 `MockMessenger`를 만듭니다. 그런 다음 새 `LimitTracker`를 만들어
그 새 `MockMessenger`에 대한 참조와 `max` 값 `100`을 줍니다. `LimitTracker`의
`set_value` 메서드를 100의 75%보다 큰 `80` 값으로 호출합니다. 그런 다음
`MockMessenger`가 추적 중인 메시지 목록에 이제 메시지가 하나 있어야 한다고
단언합니다.

그러나 이 테스트에는 다음에 보이는 한 가지 문제가 있습니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

`send` 메서드가 `self`에 대한 불변 참조를 받기 때문에 `MockMessenger`를 변경해
메시지를 추적할 수 없습니다. 또한 `impl` 메서드와 트레이트 정의 모두에 `&mut
self`를 사용하라는 오류 텍스트의 제안을 받아들일 수도 없습니다. 테스트만을
위해 `Messenger` 트레이트를 바꾸고 싶지는 않기 때문입니다. 대신 우리 테스트
코드가 기존 설계와 함께 올바르게 동작하게 할 방법을 찾아야 합니다.

이것은 내부 가변성이 도움이 될 수 있는 상황입니다! `sent_messages`를
`RefCell<T>` 안에 저장하면, `send` 메서드가 `sent_messages`를 변경해 본
메시지를 저장할 수 있습니다. Listing 15-22가 그 모습을 보여 줍니다.

<Listing number="15-22" file-name="src/lib.rs" caption="바깥 값은 불변으로 간주되는 동안 내부 값을 변경하기 위해 `RefCell<T>` 사용하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

`sent_messages` 필드는 이제 `Vec<String>`이 아니라 `RefCell<Vec<String>>`
타입입니다. `new` 함수에서 빈 벡터를 새 `RefCell<Vec<String>>` 인스턴스로
감쌉니다.

`send` 메서드 구현에서, 첫 번째 매개변수는 여전히 `self`의 불변 빌림이며
이는 트레이트 정의와 일치합니다. `self.sent_messages`의 `RefCell<Vec<String>>`
에 `borrow_mut`를 호출해 `RefCell<Vec<String>>` 내부 값, 즉 벡터에 대한
가변 참조를 얻습니다. 그런 다음 벡터에 대한 가변 참조에 `push`를 호출해
테스트 중에 보낸 메시지를 추적할 수 있습니다.

마지막으로 단언에서도 변경이 필요합니다. 내부 벡터에 몇 개의 항목이 있는지
보기 위해 `RefCell<Vec<String>>`에 `borrow`를 호출해 벡터에 대한 불변 참조를
얻습니다.

이제 `RefCell<T>` 사용 방법을 보았으니, 어떻게 동작하는지 자세히 들어가 봅시다!

<!-- Old headings. Do not remove or links may break. -->

<a id="keeping-track-of-borrows-at-runtime-with-refcellt"></a>

#### 런타임에 빌림 추적하기

불변 참조와 가변 참조를 만들 때 우리는 각각 `&`와 `&mut` 문법을 사용합니다.
`RefCell<T>`에서는 `RefCell<T>`에 속하는 안전한 API의 일부인 `borrow`와
`borrow_mut` 메서드를 사용합니다. `borrow` 메서드는 스마트 포인터 타입
`Ref<T>`를, `borrow_mut`는 스마트 포인터 타입 `RefMut<T>`를 반환합니다. 두
타입 모두 `Deref`를 구현하므로 일반 참조처럼 다룰 수 있습니다.

`RefCell<T>`는 현재 활성화된 `Ref<T>`와 `RefMut<T>` 스마트 포인터의 수를
추적합니다. 우리가 `borrow`를 호출할 때마다 `RefCell<T>`는 활성 불변 빌림의
카운트를 늘립니다. `Ref<T>` 값이 스코프를 벗어나면 불변 빌림의 카운트가 1
감소합니다. 컴파일 타임 빌림 규칙과 마찬가지로, `RefCell<T>`는 어느 시점에든
많은 수의 불변 빌림 또는 하나의 가변 빌림을 가질 수 있게 해 줍니다.

이 규칙을 어기려 하면, 참조에서 얻을 컴파일러 오류 대신 `RefCell<T>`의 구현이
런타임에 패닉합니다. Listing 15-23은 Listing 15-22의 `send` 구현 수정을 보여
줍니다. `RefCell<T>`가 런타임에 이를 막는 것을 보여 주기 위해, 같은 스코프
에서 두 개의 가변 빌림이 활성화되도록 일부러 시도합니다.

<Listing number="15-23" file-name="src/lib.rs" caption="같은 스코프에 두 개의 가변 참조를 만들어 `RefCell<T>`가 패닉하는 것을 보기">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

`borrow_mut`로부터 반환된 `RefMut<T>` 스마트 포인터를 위한 변수 `one_borrow`
를 만듭니다. 그런 다음 같은 방식으로 변수 `two_borrow`에 또 다른 가변 빌림을
만듭니다. 이는 허용되지 않는, 같은 스코프의 두 가변 참조를 만듭니다. 우리
라이브러리의 테스트를 실행하면, Listing 15-23의 코드는 오류 없이 컴파일되지만
테스트는 실패합니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

코드가 `already borrowed: BorrowMutError` 메시지와 함께 패닉했음에 유의하세요.
이것이 `RefCell<T>`가 런타임에 빌림 규칙 위반을 처리하는 방식입니다.

여기서처럼 컴파일 타임이 아니라 런타임에 빌림 오류를 잡도록 선택하는 것은,
코드의 실수를 개발 과정의 더 늦은 시점에 — 코드가 프로덕션에 배포된 후일
수도 있습니다 — 찾을 수 있다는 뜻입니다. 또한 컴파일 타임이 아니라 런타임에
빌림을 추적한 결과로 코드에 작은 런타임 성능 벌점이 듭니다. 그러나 `RefCell<T>`
를 사용하면 불변 값만 허용되는 맥락에서 본 메시지를 추적하도록 자신을 변경
할 수 있는 목 객체를 작성할 수 있습니다. 그 트레이드오프에도 불구하고
`RefCell<T>`를 사용해 일반 참조가 제공하는 것보다 더 많은 기능을 얻을 수
있습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>
<a id="allowing-multiple-owners-of-mutable-data-with-rct-and-refcellt"></a>

### `Rc<T>`와 `RefCell<T>`로 가변 데이터의 여러 소유자 허용하기

`RefCell<T>`를 사용하는 일반적 방법은 `Rc<T>`와 조합하는 것입니다. `Rc<T>`는
일부 데이터의 여러 소유자를 가질 수 있게 해 주지만, 그 데이터에 대한 불변
접근만 제공한다는 점을 떠올려 보세요. `RefCell<T>`를 담는 `Rc<T>`가 있으면
여러 소유자를 가질 수 있고 _동시에_ 변경 가능한 값을 얻을 수 있습니다!

예를 들어 Listing 15-18의 콘스 리스트 예제에서 `Rc<T>`를 사용해 여러 리스트가
다른 리스트의 소유권을 공유할 수 있게 했던 것을 떠올려 보세요. `Rc<T>`는
불변 값만 담으므로, 리스트를 만들고 나면 그 안의 값을 바꿀 수 없습니다. 리스트
의 값을 바꿀 수 있는 능력을 위해 `RefCell<T>`를 추가해 봅시다. Listing 15-24는
`Cons` 정의에 `RefCell<T>`를 사용하면 모든 리스트에 저장된 값을 수정할 수
있음을 보여 줍니다.

<Listing number="15-24" file-name="src/main.rs" caption="변경 가능한 `List`를 만들기 위해 `Rc<RefCell<i32>>` 사용하기">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

`Rc<RefCell<i32>>`의 인스턴스인 값을 만들어 `value`라는 변수에 저장해서 나중에
직접 접근할 수 있게 합니다. 그런 다음 `value`를 담는 `Cons` 배리언트로
`a`에 `List`를 만듭니다. `value`의 소유권을 `a`로 이전하거나 `a`가 `value`
로부터 빌리지 않도록, `a`와 `value` 둘 다 내부 `5` 값의 소유권을 갖도록
`value`를 복제해야 합니다.

그런 다음 `b`와 `c` 리스트를 만들 때 둘 다 `a`를 참조할 수 있도록 `a` 리스트를
`Rc<T>`로 감쌉니다. Listing 15-18에서 했던 것과 같습니다.

`a`, `b`, `c` 리스트를 만든 뒤, `value`의 값에 10을 더하고 싶습니다. `value`
에 `borrow_mut`를 호출해서 이를 수행합니다. 5장의 [“`->` 연산자는 어디에
있나요?”][wheres-the---operator]<!-- ignore -->에서 논의한 자동 역참조 기능을
사용해 `Rc<T>`를 내부 `RefCell<T>` 값으로 역참조합니다. `borrow_mut` 메서드는
`RefMut<T>` 스마트 포인터를 반환하고, 그것에 역참조 연산자를 사용해 내부 값을
바꿉니다.

`a`, `b`, `c`를 출력하면 모두 `5`가 아니라 수정된 값 `15`를 가진다는 것을 볼
수 있습니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

이 기법은 꽤 근사합니다! `RefCell<T>`를 사용함으로써 겉보기에는 불변인 `List`
값을 가집니다. 하지만 내부 가변성에 대한 접근을 제공하는 `RefCell<T>`의 메서드
를 사용해, 필요할 때 데이터를 수정할 수 있습니다. 빌림 규칙의 런타임 검사는
데이터 경합으로부터 우리를 보호하며, 우리 자료 구조에서 이 유연성을 위해 속도
를 조금 거래하는 것이 가끔은 가치 있습니다. `RefCell<T>`는 다중 스레드 코드
에서 동작하지 않음에 유의하세요! `Mutex<T>`가 `RefCell<T>`의 스레드 안전
버전이며, 16장에서 `Mutex<T>`를 논의합니다.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
