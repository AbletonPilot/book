## `Drop` 트레이트로 정리 시 코드 실행하기

스마트 포인터 패턴에 중요한 두 번째 트레이트는 `Drop`으로, 값이 스코프를
벗어나려 할 때 어떤 일이 일어날지 커스터마이즈할 수 있게 해 줍니다. 어떤
타입에든 `Drop` 트레이트 구현을 제공할 수 있고, 그 코드로 파일이나 네트워크
연결 같은 자원을 해제할 수 있습니다.

`Drop` 트레이트의 기능이 스마트 포인터를 구현할 때 거의 항상 사용되기 때문에
스마트 포인터의 맥락에서 `Drop`을 소개합니다. 예를 들어 `Box<T>`가 드롭될 때,
그 박스가 가리키는 힙의 공간을 할당 해제합니다.

일부 언어에서는, 어떤 타입의 인스턴스를 다 쓸 때마다 프로그래머가 메모리나
자원을 해제하는 코드를 호출해야 하는 타입이 있습니다. 파일 핸들, 소켓, 잠금
등이 그 예입니다. 프로그래머가 잊으면 시스템이 과부하되어 충돌할 수 있습니다.
러스트에서는 값이 스코프를 벗어날 때마다 실행되는 특정 코드를 지정할 수 있고,
컴파일러가 이 코드를 자동으로 삽입합니다. 그 결과 어떤 타입의 인스턴스를 다
쓴 프로그램의 모든 곳에 정리 코드를 두는 데 신경 쓰지 않아도 자원이 새지
않습니다!

값이 스코프를 벗어날 때 실행할 코드는 `Drop` 트레이트를 구현해 지정합니다.
`Drop` 트레이트는 `self`에 대한 가변 참조를 받는 `drop`이라는 메서드 하나를
구현하도록 요구합니다. 러스트가 언제 `drop`을 호출하는지 보기 위해, 지금은
`println!` 구문이 있는 `drop`을 구현해 봅시다.

Listing 15-14는 그 유일한 커스텀 기능이 인스턴스가 스코프를 벗어날 때
`Dropping CustomSmartPointer!`를 출력하는, 러스트가 `drop` 메서드를 언제
실행하는지 보여 주는 `CustomSmartPointer` 구조체를 보여 줍니다.

<Listing number="15-14" file-name="src/main.rs" caption="정리 코드를 둘 곳에 `Drop` 트레이트를 구현한 `CustomSmartPointer` 구조체">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

`Drop` 트레이트는 프렐루드에 포함되어 있으므로 스코프로 가져올 필요가 없습니다.
`CustomSmartPointer`에 `Drop` 트레이트를 구현하고, `println!`을 호출하는 `drop`
메서드 구현을 제공합니다. `drop` 메서드의 본문은 여러분의 타입 인스턴스가
스코프를 벗어날 때 실행하고 싶은 어떤 로직이든 놓을 곳입니다. 러스트가 `drop`을
언제 호출하는지 시각적으로 보여 주기 위해 여기서 텍스트를 출력합니다.

`main`에서는 두 개의 `CustomSmartPointer` 인스턴스를 만들고 `CustomSmartPointers
created`를 출력합니다. `main`의 끝에서 `CustomSmartPointer` 인스턴스들은 스코프
를 벗어나고, 러스트는 우리가 `drop` 메서드에 넣은 코드를 호출해 최종 메시지를
출력합니다. `drop` 메서드를 명시적으로 호출할 필요는 없었음에 유의하세요.

이 프로그램을 실행하면 다음 출력이 보입니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

러스트가 인스턴스들이 스코프를 벗어날 때 우리를 위해 자동으로 `drop`을 호출해,
우리가 지정한 코드를 호출했습니다. 변수는 생성의 역순으로 드롭되므로, `d`가
`c`보다 먼저 드롭되었습니다. 이 예제의 목적은 `drop` 메서드가 어떻게 동작하는지
시각적인 가이드를 제공하는 것입니다. 보통은 출력 메시지 대신 여러분 타입이
실행해야 하는 정리 코드를 지정할 것입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="dropping-a-value-early-with-std-mem-drop"></a>

안타깝게도 자동 `drop` 기능을 비활성화하는 것은 간단하지 않습니다. `drop`
비활성화는 보통 필요하지 않습니다. `Drop` 트레이트의 요점 자체가 자동으로
처리된다는 것이니까요. 그러나 가끔 값을 일찍 정리하고 싶을 수 있습니다. 한
예로 잠금을 관리하는 스마트 포인터를 사용할 때입니다. 같은 스코프의 다른
코드가 잠금을 획득할 수 있도록 잠금을 해제하는 `drop` 메서드를 강제로 실행
하고 싶을 수 있습니다. 러스트는 `Drop` 트레이트의 `drop` 메서드를 수동으로
호출하도록 허용하지 않습니다. 대신 스코프 끝 전에 강제로 값을 드롭하고
싶다면 표준 라이브러리가 제공하는 `std::mem::drop` 함수를 호출해야 합니다.

Listing 15-14의 `main` 함수를 수정해 `Drop` 트레이트의 `drop` 메서드를 수동
으로 호출하려 하면 Listing 15-15처럼 동작하지 않습니다.

<Listing number="15-15" file-name="src/main.rs" caption="일찍 정리하기 위해 `Drop` 트레이트의 `drop` 메서드를 수동으로 호출하려는 시도">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

이 코드를 컴파일하려 하면 다음 오류가 납니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

이 오류 메시지는 `drop`을 명시적으로 호출하는 것이 허용되지 않음을 말합니다.
이 오류 메시지는 인스턴스를 정리하는 함수를 가리키는 일반 프로그래밍 용어인
_destructor(소멸자)_ 라는 용어를 사용합니다. _소멸자_ 는 인스턴스를 만드는
_생성자(constructor)_ 에 대응됩니다. 러스트의 `drop` 함수는 특정한 하나의
소멸자입니다.

러스트가 `drop`을 명시적으로 호출하지 못하게 하는 이유는, 러스트가 `main`의
끝에서 값에 대해 자동으로 `drop`을 호출하기 때문입니다. 이는 같은 값을 두 번
정리하려 하게 되어 이중 해제(double free) 오류를 일으킵니다.

값이 스코프를 벗어날 때 `drop`이 자동 삽입되는 것을 비활성화할 수 없고,
`drop` 메서드를 명시적으로 호출할 수도 없습니다. 따라서 값을 일찍 정리하도록
강제해야 한다면 `std::mem::drop` 함수를 사용합니다.

`std::mem::drop` 함수는 `Drop` 트레이트의 `drop` 메서드와 다릅니다. 강제로
드롭하고 싶은 값을 인수로 전달해 호출합니다. 이 함수는 프렐루드에 있으므로,
Listing 15-16처럼 Listing 15-15의 `main`을 수정해 `drop` 함수를 호출할 수
있습니다.

<Listing number="15-16" file-name="src/main.rs" caption="값이 스코프를 벗어나기 전에 명시적으로 드롭하기 위해 `std::mem::drop` 호출하기">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

이 코드를 실행하면 다음이 출력됩니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

``Dropping CustomSmartPointer with data `some data`!`` 텍스트가
`CustomSmartPointer created`와 `CustomSmartPointer dropped before the end of
main` 텍스트 사이에 출력되어, 그 시점에 `c`를 드롭하기 위해 `drop` 메서드
코드가 호출됨을 보여 줍니다.

편리하고 안전하게 정리하기 위해 `Drop` 트레이트 구현에 지정된 코드를 여러
방법으로 사용할 수 있습니다. 예를 들어 직접 메모리 할당자를 만드는 데 사용할
수도 있습니다! `Drop` 트레이트와 러스트의 소유권 시스템 덕분에 정리를 기억할
필요가 없습니다. 러스트가 자동으로 해 주기 때문입니다.

참조가 항상 유효하도록 보장하는 소유권 시스템은 값이 더 이상 사용되지 않을
때 `drop`이 단 한 번만 호출되도록 보장하기도 하므로, 여전히 사용 중인 값을
실수로 정리하는 데서 비롯되는 문제를 걱정하지 않아도 됩니다.

이제 `Box<T>`와 스마트 포인터의 몇 가지 특징을 살펴보았으므로, 표준 라이브러리
에 정의된 다른 스마트 포인터 몇 가지를 살펴봅시다.
