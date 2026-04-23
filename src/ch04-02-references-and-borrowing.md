## 참조와 빌림

Listing 4-5의 튜플 코드의 문제는, `calculate_length` 호출 뒤에도 `String`을 계속
사용하려면 `String`이 `calculate_length`로 이동된 이후에도 호출한 함수로 `String`을
돌려받아야 한다는 것입니다. 그 `String`이 `calculate_length`로 이동되었기 때문
이죠. 대신 `String` 값에 대한 참조(reference)를 제공할 수 있습니다. 참조는
포인터와 비슷합니다. 주소를 따라가면 그 위치에 저장된 데이터에 접근할 수 있고,
그 데이터는 다른 어떤 변수에 의해 소유되어 있다는 점에서 그렇습니다. 포인터와
달리, 참조는 그 참조의 수명 동안 특정 타입의 유효한 값을 가리키도록 보장됩니다.

값의 소유권을 가져가는 대신 객체에 대한 참조를 매개변수로 받는 `calculate_length`
함수를 정의하고 사용하는 방법은 다음과 같습니다.

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

먼저, 변수 선언과 함수 반환값에 있던 튜플 코드가 모두 사라졌다는 점에 유의하세요.
둘째, `calculate_length`에 `&s1`을 전달하고, 그 정의에서는 `String` 대신
`&String`을 받는다는 점에 주목하세요. 이 앰퍼샌드(ampersand, `&`)는 참조를
나타내며, 값을 소유하지 않고도 그 값을 가리킬 수 있게 해 줍니다. Figure 4-6이
이 개념을 보여 줍니다.

<img alt="세 개의 표: s 표에는 s1 표를 가리키는 포인터만 들어 있습니다. s1
표에는 s1의 스택 데이터가 있고, 힙에 있는 문자열 데이터를 가리킵니다." src="img/trpl04-06.svg" class="center" />

<span class="caption">Figure 4-6: `&String` `s`가 `String` `s1`을 가리키는
그림</span>

> 참고: `&`를 사용하는 참조의 반대는 *역참조(dereferencing)*이고, 이는 역참조
> 연산자 `*`로 수행합니다. 역참조 연산자의 사용은 8장에서 몇 가지 볼 것이고,
> 역참조에 대한 세부 내용은 15장에서 다룹니다.

여기서 함수 호출 부분을 좀 더 자세히 살펴봅시다.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

`&s1` 문법은 `s1`의 값을 *가리키는*, 그러나 소유하지는 않는 참조를 만들어 줍니다.
참조가 값을 소유하지 않기 때문에, 그 참조가 더 이상 사용되지 않아도 그 참조가
가리키는 값은 버려지지(drop) 않습니다.

마찬가지로, 함수 시그니처는 매개변수 `s`의 타입이 참조임을 나타내기 위해 `&`를
사용합니다. 설명하는 주석을 덧붙여 봅시다.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

변수 `s`가 유효한 스코프는 여느 함수 매개변수의 스코프와 같지만, `s`의 사용이
끝났다고 해서 참조가 가리키는 값이 버려지지는 않습니다. `s`가 소유권을 가지고
있지 않기 때문입니다. 함수가 실제 값 대신 참조를 매개변수로 받으면, 소유권을
돌려주기 위해 값을 반환할 필요가 없습니다. 애초에 소유권을 가진 적이 없으니까요.

참조를 만드는 행위를 우리는 *빌림(borrowing)*이라고 부릅니다. 실생활에서도 누군가가
어떤 것을 소유하고 있다면 그 사람에게서 빌릴 수 있고, 다 사용한 뒤에는 돌려주어야
합니다. 소유하는 것은 아니니까요.

그렇다면 빌린 것을 수정하려고 하면 어떻게 될까요? Listing 4-6의 코드를 시도해
봅시다. 스포일러 경고: 동작하지 않습니다!

<Listing number="4-6" file-name="src/main.rs" caption="빌린 값을 수정하려는 시도">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

다음은 에러입니다.

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

변수가 기본적으로 불변인 것과 마찬가지로, 참조도 그렇습니다. 참조하고 있는 것을
수정할 수는 없습니다.

### 가변 참조

Listing 4-6의 코드를 고쳐 빌린 값을 수정할 수 있게 하려면, 몇 가지 작은 수정만으로
*가변 참조(mutable reference)*를 사용하면 됩니다.

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

먼저, `s`를 `mut`로 바꿉니다. 그런 다음 `change` 함수를 호출하는 곳에서 `&mut s`로
가변 참조를 만들고, 함수 시그니처를 `some_string: &mut String`으로 바꿔 가변
참조를 받도록 합니다. 이렇게 하면 `change` 함수가 빌린 값을 변경할 것임을 매우
명확하게 드러냅니다.

가변 참조에는 큰 제약이 하나 있습니다. 어떤 값에 가변 참조를 가지고 있다면, 그
값에 대한 다른 참조를 가질 수 없습니다. `s`에 두 개의 가변 참조를 만들려고 하는
다음 코드는 실패합니다.

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

다음은 에러입니다.

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

에러는 같은 시점에 `s`를 두 번 이상 가변으로 빌릴 수 없기 때문에 이 코드가 유효
하지 않다고 말합니다. 첫 가변 빌림은 `r1`에 있고, 이 빌림은 `println!`에서 사용
될 때까지 지속되어야 하는데, 그 가변 참조의 생성과 사용 사이에 같은 데이터를
빌리는 또 다른 가변 참조를 `r2`에 만들려고 한 것입니다.

같은 시점에 같은 데이터에 여러 가변 참조를 두지 못하도록 하는 이 제약은 변경을
허용하되, 아주 통제된 방식으로만 허용합니다. 대부분의 언어는 원할 때마다 변경을
허용하기 때문에, 새로운 러스타시안이 씨름하게 되는 부분이기도 합니다. 이 제약의
이점은 러스트가 컴파일 타임에 데이터 레이스(data race)를 막을 수 있다는 것입니다.
*데이터 레이스*는 경쟁 상태(race condition)와 비슷하며, 다음 세 가지 동작이 일어
날 때 발생합니다.

- 둘 이상의 포인터가 같은 데이터에 동시에 접근합니다.
- 그 포인터 중 적어도 하나가 데이터에 쓰기 위해 사용되고 있습니다.
- 데이터 접근을 동기화(synchronize)하는 메커니즘이 사용되고 있지 않습니다.

데이터 레이스는 정의되지 않은 동작(undefined behavior)을 유발하며, 런타임에 추적
하려고 하면 진단과 수정이 어렵습니다. 러스트는 데이터 레이스가 있는 코드의 컴파일
자체를 거부함으로써 이 문제를 예방합니다!

늘 그렇듯, 중괄호를 사용해 새 스코프를 만들어 여러 가변 참조를 허용할 수는
있습니다. *동시에* 존재하지 않기만 하면 됩니다.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

러스트는 가변 참조와 불변 참조의 조합에도 비슷한 규칙을 강제합니다. 다음 코드는
에러를 발생시킵니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

다음은 에러입니다.

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

휴! 같은 값에 대한 불변 참조가 있는 동안에는 가변 참조도 가질 수 *없습니다*.

불변 참조의 사용자는 값이 자기도 모르게 갑자기 바뀌기를 기대하지 않습니다! 그러나
여러 불변 참조는 허용됩니다. 데이터를 그저 읽기만 하는 누구도 다른 이의 읽기에
영향을 줄 능력이 없기 때문입니다.

참조의 스코프는 도입된 지점에서 시작해서, 그 참조가 마지막으로 사용된 시점까지
이어진다는 점에 유의하세요. 예를 들어, 다음 코드는 가변 참조가 도입되기 전에 불변
참조의 마지막 사용이 `println!`에서 이뤄지기 때문에 컴파일됩니다.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

불변 참조 `r1`과 `r2`의 스코프는 이들이 마지막으로 사용되는 `println!` 이후에
끝나며, 이는 가변 참조 `r3`가 생성되기 전입니다. 이 스코프들은 겹치지 않으므로
이 코드는 허용됩니다. 컴파일러는 스코프가 끝나기 전에 참조가 더 이상 사용되지
않는 지점을 알아낼 수 있습니다.

빌림 에러가 때로 답답하더라도, 이는 러스트 컴파일러가 (런타임이 아닌 컴파일
타임에) 잠재적 버그를 일찍 짚어 주고 문제의 정확한 위치를 보여 주고 있는 것임을
기억하세요. 그러면 데이터가 왜 기대와 다른지 추적할 필요가 없습니다.

### 댕글링 참조

포인터가 있는 언어에서는 어떤 메모리를 해제하면서 그 메모리를 가리키는 포인터를
유지함으로써, 다른 누군가에게 넘어갔을 수도 있는 메모리 위치를 참조하는 포인터,
즉 *댕글링 포인터(dangling pointer)*를 잘못 만들기 쉽습니다. 이에 비해 러스트에서는
컴파일러가 참조가 절대 댕글링 참조가 되지 않도록 보장합니다. 어떤 데이터에 대한
참조가 있다면, 컴파일러는 그 데이터에 대한 참조가 스코프를 벗어나기 전에 데이터가
스코프를 벗어나지 않도록 보장합니다.

댕글링 참조를 만들어 보면서 러스트가 컴파일 타임 에러로 어떻게 이를 막는지 살펴
봅시다.

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

다음은 에러입니다.

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

이 에러 메시지는 아직 다루지 않은 기능인 라이프타임(lifetimes)을 가리킵니다. 라이프
타임에 대해서는 10장에서 자세히 다루겠습니다. 다만 라이프타임에 관한 부분을
제쳐 두고 보면, 메시지에는 이 코드가 문제인 핵심 이유가 담겨 있습니다.

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

`dangle` 코드의 각 단계에서 정확히 어떤 일이 벌어지는지 좀 더 자세히 살펴봅시다.

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

`s`는 `dangle` 안에서 생성되었기 때문에, `dangle`의 코드가 끝나면 `s`가 할당 해제
됩니다. 그런데 우리는 그에 대한 참조를 반환하려고 했습니다. 즉, 이 참조는 유효
하지 않은 `String`을 가리키게 될 것입니다. 이건 안 되죠! 러스트는 이를 허용하지
않습니다.

여기서의 해결책은 `String`을 직접 반환하는 것입니다.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

이 코드는 아무 문제 없이 동작합니다. 소유권이 바깥으로 이동되고, 아무것도 할당
해제되지 않습니다.

### 참조의 규칙

참조에 대해 지금까지 논의한 내용을 정리해 봅시다.

- 어느 시점에든, *하나의 가변 참조* 또는 *임의 개수의 불변 참조* 중 하나만 가질
  수 있습니다.
- 참조는 항상 유효해야 합니다.

다음으로, 또 다른 종류의 참조인 슬라이스(slice)를 살펴보겠습니다.
