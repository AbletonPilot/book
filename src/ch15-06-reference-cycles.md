## 참조 순환은 메모리를 누수할 수 있습니다

러스트의 메모리 안전 보장은 실수로 절대 정리되지 않는 메모리(_메모리 누수
(memory leak)_ 라고 부름)를 만드는 것을 어렵게 하지만 불가능하게 만들지는
않습니다. 메모리 누수를 완전히 방지하는 것은 러스트의 보장 중 하나가 아니며,
따라서 러스트에서 메모리 누수는 메모리 안전합니다. `Rc<T>`와 `RefCell<T>`를
사용하면 러스트가 메모리 누수를 허용한다는 것을 볼 수 있습니다. 항목들이 순환
으로 서로를 참조하는 참조를 만들 수 있습니다. 이는 메모리 누수를 일으키는데,
순환의 각 항목의 참조 카운트가 절대 0에 도달하지 않고 값이 절대 드롭되지 않기
때문입니다.

### 참조 순환 만들기

Listing 15-25의 `List` 열거형 정의와 `tail` 메서드부터 시작해, 참조 순환이
어떻게 일어날 수 있고 어떻게 방지할 수 있는지 살펴봅시다.

<Listing number="15-25" file-name="src/main.rs" caption="`Cons` 배리언트가 참조하는 대상을 수정할 수 있도록 `RefCell<T>`를 담는 콘스 리스트 정의">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs:here}}
```

</Listing>

Listing 15-5의 `List` 정의의 또 다른 변형을 사용하고 있습니다. `Cons` 배리언트
의 두 번째 요소는 이제 `RefCell<Rc<List>>`입니다. Listing 15-24에서 `i32`
값을 수정할 수 있게 했던 것과 달리, `Cons` 배리언트가 가리키는 `List` 값을
수정하고 싶다는 뜻입니다. `Cons` 배리언트가 있으면 두 번째 항목에 편리하게
접근할 수 있도록 `tail` 메서드도 추가했습니다.

Listing 15-26에서는 Listing 15-25의 정의를 사용하는 `main` 함수를 추가합니다.
이 코드는 `a`에 리스트를 만들고, `a`의 리스트를 가리키는 리스트를 `b`에
만듭니다. 그런 다음 `a`의 리스트를 수정해 `b`를 가리키도록 해서 참조 순환을
만듭니다. 이 과정 곳곳에 참조 카운트가 어떤지 보여 주는 `println!` 구문이
있습니다.

<Listing number="15-26" file-name="src/main.rs" caption="서로를 가리키는 두 `List` 값의 참조 순환 만들기">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

초기 리스트 `5, Nil`을 담는 `List` 값을 담는 `Rc<List>` 인스턴스를 변수 `a`에
만듭니다. 그런 다음 값 `10`을 담고 `a`의 리스트를 가리키는 또 다른 `List`
값을 담는 `Rc<List>` 인스턴스를 변수 `b`에 만듭니다.

`a`를 수정해 `Nil` 대신 `b`를 가리키도록 해 순환을 만듭니다. `tail` 메서드를
사용해 `a`의 `RefCell<Rc<List>>`에 대한 참조를 얻어 `link` 변수에 넣습니다.
그런 다음 `RefCell<Rc<List>>`의 `borrow_mut` 메서드를 사용해 내부 값을 `Nil`
값을 담는 `Rc<List>`에서 `b`의 `Rc<List>`로 바꿉니다.

마지막 `println!`을 잠시 주석 처리한 채로 이 코드를 실행하면 다음 출력을
얻습니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

`a`의 리스트가 `b`를 가리키도록 바꾸면 `a`와 `b`의 `Rc<List>` 인스턴스 참조
카운트는 둘 다 2가 됩니다. `main`의 끝에서 러스트는 변수 `b`를 드롭하고, `b`의
`Rc<List>` 인스턴스 참조 카운트는 2에서 1로 감소합니다. 참조 카운트가 0이
아니라 1이므로 이 시점에 `Rc<List>`가 힙에 가진 메모리는 드롭되지 않습니다.
그런 다음 러스트는 `a`를 드롭해 `a`의 `Rc<List>` 인스턴스 참조 카운트도 2
에서 1로 감소시킵니다. 다른 `Rc<List>` 인스턴스가 여전히 그것을 참조하므로,
이 인스턴스의 메모리도 드롭될 수 없습니다. 리스트에 할당된 메모리는 영원히
수거되지 않을 것입니다. 이 참조 순환을 시각화하기 위해 그림 15-4의 다이어그램
을 만들었습니다.

<img alt="'a'라는 레이블이 붙은 직사각형이 정수 5를 담는 직사각형을 가리킵니다. 'b'라는 레이블이 붙은 직사각형은 정수 10을 담는 직사각형을 가리킵니다. 5를 담은 직사각형은 10을 담은 직사각형을 가리키고, 10을 담은 직사각형은 다시 5를 담은 직사각형을 가리켜 순환을 만듭니다." src="img/trpl15-04.svg" class="center" />

<span class="caption">그림 15-4: 서로를 가리키는 리스트 `a`와 `b`의 참조
순환</span>

마지막 `println!`의 주석을 풀고 프로그램을 실행하면, 러스트는 `a`가 `b`를
가리키고 `b`가 `a`를 가리키고… 식으로 이 순환을 출력하려 하다가 스택을
오버플로합니다.

실제 프로그램과 비교하면 이 예제에서 참조 순환을 만드는 결과는 그리 심각하지
않습니다. 참조 순환을 만든 직후 프로그램이 끝납니다. 그러나 더 복잡한 프로그램
이 순환에서 많은 메모리를 할당하고 오랫동안 붙들고 있다면, 프로그램은 필요
이상의 메모리를 사용하고 사용 가능한 메모리를 다 쓰게 만들어 시스템을 압도할
수 있습니다.

참조 순환을 만드는 것은 쉽게 일어나는 일은 아니지만 불가능하지도 않습니다.
`Rc<T>` 값을 담는 `RefCell<T>` 값이나 내부 가변성과 참조 카운팅을 결합한
비슷한 중첩 조합이 있다면, 순환을 만들지 않도록 보장해야 합니다. 러스트가
그것을 잡아내리라 기대할 수 없습니다. 참조 순환을 만드는 것은 자동화된 테스트,
코드 리뷰, 기타 소프트웨어 개발 관행으로 최소화해야 하는 여러분 프로그램의
논리 버그일 것입니다.

참조 순환을 피하는 또 다른 해법은 어떤 참조는 소유권을 표현하고 어떤 참조는
그러지 않도록 자료 구조를 재구성하는 것입니다. 그 결과 어떤 소유 관계와 어떤
비소유 관계로 이루어진 순환을 가질 수 있고, 오직 소유 관계만 값을 드롭할
수 있는지에 영향을 줍니다. Listing 15-25에서는 `Cons` 배리언트가 항상 자신의
리스트를 소유하기를 원하므로 자료 구조 재구성은 불가능합니다. 비소유 관계가
참조 순환을 방지하는 적절한 방법인 경우를 보기 위해, 부모 노드와 자식 노드로
이루어진 그래프 예를 살펴봅시다.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### `Weak<T>`로 참조 순환 방지하기

지금까지 우리는 `Rc::clone`을 호출하면 `Rc<T>` 인스턴스의 `strong_count`가
증가하고, `Rc<T>` 인스턴스는 `strong_count`가 0일 때만 정리됨을 보여 왔습니다.
`Rc::downgrade`를 호출하고 `Rc<T>`에 대한 참조를 전달해 `Rc<T>` 인스턴스 내부
값에 대한 약한 참조를 만들 수도 있습니다. _강한 참조(strong reference)_ 는
`Rc<T>` 인스턴스의 소유권을 공유할 수 있는 방식입니다. _약한 참조(weak
reference)_ 는 소유 관계를 표현하지 않으며, 그 카운트는 `Rc<T>` 인스턴스가
언제 정리될지에 영향을 주지 않습니다. 약한 참조가 포함된 순환은 관여한 값들의
강한 참조 카운트가 0이 되면 깨지므로 참조 순환을 일으키지 않습니다.

`Rc::downgrade`를 호출하면 `Weak<T>` 타입의 스마트 포인터를 얻습니다.
`Rc::downgrade`를 호출하면 `Rc<T>` 인스턴스의 `strong_count`가 1 증가하는
대신 `weak_count`가 1 증가합니다. `Rc<T>` 타입은 `strong_count`와 비슷하게
얼마나 많은 `Weak<T>` 참조가 존재하는지 추적하기 위해 `weak_count`를 사용
합니다. 차이점은 `Rc<T>` 인스턴스가 정리되기 위해 `weak_count`가 0일 필요가
없다는 것입니다.

`Weak<T>`가 참조하는 값은 드롭되었을 수 있으므로, `Weak<T>`가 가리키는 값으로
무언가를 하려면 그 값이 여전히 존재하는지 확인해야 합니다. 이를 위해 `Weak<T>`
인스턴스에 `upgrade` 메서드를 호출하면 `Option<Rc<T>>`를 반환합니다. `Rc<T>`
값이 아직 드롭되지 않았다면 `Some` 결과를, 드롭되었다면 `None` 결과를 얻습니다.
`upgrade`가 `Option<Rc<T>>`를 반환하므로, 러스트는 `Some`과 `None` 경우가 모두
처리되도록 보장하며 유효하지 않은 포인터가 생기지 않습니다.

예시로 다음 항목만 아는 리스트 대신, 자식 항목 _과_ 부모 항목을 모두 아는
항목으로 이루어진 트리를 만들어 보겠습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-tree-data-structure-a-node-with-child-nodes"></a>

#### 트리 자료 구조 만들기

먼저 자식 노드를 아는 노드를 가진 트리를 만들어 봅시다. 자신의 `i32` 값과
자식 `Node` 값에 대한 참조를 담는 `Node`라는 구조체를 만들겠습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

우리는 `Node`가 자식들을 소유하기를 원하고, 트리의 각 `Node`에 직접 접근할 수
있도록 변수와 그 소유권을 공유하고 싶습니다. 이를 위해 `Vec<T>` 항목이
`Rc<Node>` 타입의 값이 되도록 정의합니다. 또한 어떤 노드가 다른 노드의 자식
인지 수정하고 싶으므로, `children`의 `Vec<Rc<Node>>` 주위에 `RefCell<T>`를
둡니다.

다음으로 우리 구조체 정의를 사용해, Listing 15-27처럼 값이 `3`이고 자식이
없는 `leaf`라는 `Node` 인스턴스 하나와, 값이 `5`이고 `leaf`를 자식 중 하나로
두는 `branch`라는 또 다른 인스턴스를 만듭니다.

<Listing number="15-27" file-name="src/main.rs" caption="자식이 없는 `leaf` 노드와 자식 중 하나로 `leaf`를 두는 `branch` 노드 만들기">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

`leaf`의 `Rc<Node>`를 복제해 `branch`에 저장합니다. 즉 `leaf`의 `Node`는 이제
두 소유자를 가집니다. `leaf`와 `branch`입니다. `branch.children`을 통해
`branch`에서 `leaf`로 갈 수 있지만, `leaf`에서 `branch`로 가는 방법은 없습니다.
`leaf`가 `branch`에 대한 참조를 가지지 않고 그들이 관련된다는 것을 모르기
때문입니다. 우리는 `leaf`가 `branch`가 자신의 부모임을 알기를 원합니다. 다음
에 그것을 하겠습니다.

#### 자식에서 부모로의 참조 추가하기

자식 노드가 자신의 부모를 알도록 만들려면, `Node` 구조체 정의에 `parent`
필드를 추가해야 합니다. 문제는 `parent`의 타입을 무엇으로 정할지 결정하는
것입니다. `Rc<T>`를 담을 수 없음을 압니다. 그것은 `leaf.parent`가 `branch`를
가리키고 `branch.children`이 `leaf`를 가리켜 `strong_count` 값이 절대 0이
되지 않는 참조 순환을 만들 것입니다.

관계를 다른 식으로 생각하면, 부모 노드는 자식을 소유해야 합니다. 부모 노드가
드롭되면 그 자식 노드도 드롭되어야 합니다. 그러나 자식은 부모를 소유해서는
안 됩니다. 자식 노드를 드롭해도 부모는 여전히 존재해야 합니다. 이것은 약한
참조를 위한 경우입니다!

그래서 `parent`의 타입으로 `Rc<T>` 대신 `Weak<T>`, 구체적으로
`RefCell<Weak<Node>>`를 사용하겠습니다. 이제 `Node` 구조체 정의는 다음과
같이 보입니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

노드는 자신의 부모 노드를 참조할 수 있지만 부모를 소유하지는 않습니다. Listing
15-28에서는 `leaf` 노드가 부모인 `branch`를 참조할 수 있도록 이 새 정의를
사용하도록 `main`을 업데이트합니다.

<Listing number="15-28" file-name="src/main.rs" caption="부모 노드 `branch`에 대한 약한 참조를 가진 `leaf` 노드">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

`leaf` 노드 생성은 `parent` 필드를 제외하면 Listing 15-27과 비슷합니다. `leaf`
는 처음에 부모가 없으므로, 비어 있는 새 `Weak<Node>` 참조 인스턴스를 만듭니다.

이 시점에 `upgrade` 메서드로 `leaf`의 부모에 대한 참조를 얻으려 하면 `None`
값을 얻습니다. 첫 번째 `println!` 구문의 출력에서 이를 볼 수 있습니다.

```text
leaf parent = None
```

`branch` 노드를 만들면 `branch`에도 부모 노드가 없으므로 `parent` 필드에 새
`Weak<Node>` 참조를 가집니다. 여전히 `leaf`를 `branch`의 자식 중 하나로
둡니다. `branch`에 `Node` 인스턴스가 생기면 `leaf`를 수정해 부모에 대한
`Weak<Node>` 참조를 줄 수 있습니다. `leaf`의 `parent` 필드의
`RefCell<Weak<Node>>`에 `borrow_mut` 메서드를 사용한 다음, `Rc::downgrade`
함수를 사용해 `branch`의 `Rc<Node>`로부터 `branch`에 대한 `Weak<Node>` 참조를
만듭니다.

`leaf`의 부모를 다시 출력하면, 이번에는 `branch`를 담은 `Some` 배리언트를
얻습니다. 이제 `leaf`는 자신의 부모에 접근할 수 있습니다! `leaf`를 출력할 때
도 Listing 15-26에서 스택 오버플로로 끝났던 순환을 피합니다. `Weak<Node>`
참조는 `(Weak)`로 출력됩니다.

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

무한 출력이 없다는 것은 이 코드가 참조 순환을 만들지 않았음을 나타냅니다.
`Rc::strong_count`와 `Rc::weak_count`를 호출해 얻는 값을 보고도 알 수 있습니다.

#### `strong_count`와 `weak_count`의 변화 시각화하기

내부 스코프를 새로 만들고 `branch` 생성을 그 스코프 안으로 옮겨서 `Rc<Node>`
인스턴스의 `strong_count`와 `weak_count` 값이 어떻게 변하는지 살펴봅시다.
그렇게 하면 `branch`가 생성되었을 때와 스코프를 벗어나 드롭될 때 어떤 일이
일어나는지 볼 수 있습니다. 수정은 Listing 15-29에 나와 있습니다.

<Listing number="15-29" file-name="src/main.rs" caption="내부 스코프에서 `branch`를 만들고 강한 참조와 약한 참조 카운트 조사하기">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

`leaf`가 만들어진 후, 그 `Rc<Node>`는 강한 카운트 1과 약한 카운트 0을 가집니다.
내부 스코프에서 `branch`를 만들고 `leaf`와 연관시키면, 카운트를 출력할 때
`branch`의 `Rc<Node>`는 강한 카운트 1과 약한 카운트 1을 가집니다(`leaf.parent`
가 `Weak<Node>`로 `branch`를 가리키기 때문). `leaf`의 카운트를 출력할 때는,
`branch`가 이제 `branch.children`에 저장된 `leaf`의 `Rc<Node>`의 복제본을
가지므로 강한 카운트 2를 가지지만, 약한 카운트는 여전히 0을 가지는 것을 볼
것입니다.

내부 스코프가 끝나면 `branch`는 스코프를 벗어나고 `Rc<Node>`의 강한 카운트가
0으로 감소해, 그 `Node`가 드롭됩니다. `leaf.parent`의 약한 카운트 1은 `Node`
가 드롭되는지 여부에 영향이 없으므로 메모리 누수가 생기지 않습니다!

스코프가 끝난 뒤 `leaf`의 부모에 접근하려 하면 다시 `None`을 얻습니다. 프로
그램의 끝에서 `leaf`의 `Rc<Node>`는 강한 카운트 1과 약한 카운트 0을 가집니다.
변수 `leaf`가 이제 다시 `Rc<Node>`에 대한 유일한 참조이기 때문입니다.

카운트와 값 드롭을 관리하는 모든 로직은 `Rc<T>`와 `Weak<T>` 그리고 그들의
`Drop` 트레이트 구현에 내장되어 있습니다. `Node` 정의에서 자식에서 부모로의
관계가 `Weak<T>` 참조여야 한다고 명시함으로써, 부모 노드가 자식 노드를 가리
키고 그 반대도 가능한 참조 순환 및 메모리 누수 없이 구현할 수 있습니다.

## 요약

이 장은 러스트가 일반 참조로 기본 제공하는 것과는 다른 보장과 트레이드오프를
만들기 위해 스마트 포인터를 사용하는 방법을 다뤘습니다. `Box<T>` 타입은 알려진
크기를 가지며 힙에 할당된 데이터를 가리킵니다. `Rc<T>` 타입은 힙의 데이터에
대한 참조 수를 추적하여 데이터가 여러 소유자를 가질 수 있게 해 줍니다.
`RefCell<T>` 타입은 그 내부 가변성으로, 불변 타입이 필요하지만 그 타입 내부
값을 바꿔야 할 때 사용할 수 있는 타입을 제공합니다. 또 컴파일 타임이 아닌
런타임에 빌림 규칙을 강제합니다.

스마트 포인터 기능의 대부분을 가능하게 하는 `Deref`와 `Drop` 트레이트도
논의했습니다. 메모리 누수를 일으킬 수 있는 참조 순환과 `Weak<T>`로 이를
방지하는 방법을 탐구했습니다.

이 장에 관심이 갔고 여러분만의 스마트 포인터를 구현하고 싶다면, 더 유용한
정보를 얻으려면 [“러스토노미콘(The Rustonomicon)”][nomicon]을 확인하세요.

다음으로는 러스트의 동시성을 이야기하겠습니다. 몇 가지 새로운 스마트 포인터도
배우게 됩니다.

[nomicon]: ../nomicon/index.html
