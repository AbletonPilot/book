<!-- Old headings. Do not remove or links may break. -->

<a id="defining-modules-to-control-scope-and-privacy"></a>

## 모듈로 스코프와 비공개성 제어하기

이 절에서는 모듈과, 모듈 시스템의 다른 부분들을 이야기합니다. 아이템에 이름을 붙일
수 있게 해 주는 *경로(path)*, 경로를 스코프로 가져오는 `use` 키워드, 아이템을
공개(public)로 만드는 `pub` 키워드가 그것들입니다. `as` 키워드, 외부 패키지, 그리고
글롭(glob) 연산자에 대해서도 이야기합니다.

### 모듈 치트 시트

모듈과 경로의 세부 사항으로 들어가기 전에, 컴파일러에서 모듈, 경로, `use` 키워드,
`pub` 키워드가 어떻게 동작하는지, 그리고 대부분의 개발자가 코드를 어떻게 조직화
하는지에 대한 빠른 참고를 제공합니다. 이 장 전체에 걸쳐 이 규칙들의 예시를 하나하나
살펴볼 테지만, 모듈이 어떻게 동작하는지 다시 떠올리기 위해 참고하기에 좋은
자리입니다.

- **크레이트 루트에서 시작**: 크레이트를 컴파일할 때, 컴파일러는 먼저 크레이트
  루트 파일(보통 라이브러리 크레이트는 *src/lib.rs*, 바이너리 크레이트는
  *src/main.rs*)에서 컴파일할 코드를 찾습니다.
- **모듈 선언하기**: 크레이트 루트 파일에서는 새 모듈을 선언할 수 있습니다. 예를
  들어 `mod garden;`으로 “garden” 모듈을 선언한다고 해 봅시다. 컴파일러는 다음
  위치들에서 모듈의 코드를 찾습니다.
  - 인라인으로, `mod garden` 뒤의 세미콜론을 대체하는 중괄호 안
  - *src/garden.rs* 파일
  - *src/garden/mod.rs* 파일
- **서브모듈 선언하기**: 크레이트 루트가 아닌 어떤 파일에서든 서브모듈을 선언할
  수 있습니다. 예를 들어 *src/garden.rs*에 `mod vegetables;`라고 선언할 수 있습
  니다. 컴파일러는 부모 모듈의 이름을 가진 디렉터리 안의 다음 위치들에서 서브모듈
  의 코드를 찾습니다.
  - 인라인으로, `mod vegetables` 바로 뒤에 세미콜론 대신 중괄호 안
  - *src/garden/vegetables.rs* 파일
  - *src/garden/vegetables/mod.rs* 파일
- **모듈 안 코드로의 경로**: 모듈이 크레이트의 일부가 되면, 비공개성 규칙이 허용
  하는 한, 그 크레이트의 다른 어디에서든 코드로의 경로를 사용해 해당 모듈의 코드를
  참조할 수 있습니다. 예를 들어, garden vegetables 모듈의 `Asparagus` 타입은
  `crate::garden::vegetables::Asparagus`에서 찾을 수 있습니다.
- **비공개 vs. 공개**: 모듈 안의 코드는 기본적으로 부모 모듈에 대해 비공개입니다.
  모듈을 공개로 만들려면 `mod` 대신 `pub mod`로 선언합니다. 공개 모듈 안의 아이템도
  공개로 만들려면 그 선언 앞에 `pub`을 사용합니다.
- **`use` 키워드**: 스코프 안에서 `use` 키워드는 긴 경로의 반복을 줄이기 위해
  아이템에 대한 바로가기를 만듭니다. `crate::garden::vegetables::Asparagus`를
  참조할 수 있는 어떤 스코프에서든, `use crate::garden::vegetables::Asparagus;`
  로 바로가기를 만들 수 있고, 그 스코프에서는 이제 `Asparagus`만 써도 그 타입을
  사용할 수 있습니다.

여기서는 이 규칙들을 보여 주는 `backyard`라는 이름의 바이너리 크레이트를 만듭니다.
크레이트의 디렉터리는 *backyard*라는 같은 이름으로, 다음 파일과 디렉터리를
포함합니다.

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

이 경우 크레이트 루트 파일은 *src/main.rs*이고, 그 내용은 다음과 같습니다.

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

`pub mod garden;` 줄은 *src/garden.rs*에서 찾은 코드를 포함시키라고 컴파일러에게
알립니다. 그 내용은 다음과 같습니다.

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

여기서 `pub mod vegetables;`는 *src/garden/vegetables.rs*의 코드도 포함시킨다는
뜻입니다. 그 코드는 다음과 같습니다.

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

이제 이 규칙들의 세부 사항으로 들어가 실제로 시연해 봅시다!

### 관련 코드를 모듈로 묶기

*모듈(modules)*은 크레이트 안에서 읽기 쉽고 재사용하기 좋게 코드를 조직화할 수
있게 해 줍니다. 모듈은 또한 아이템의 *비공개성(privacy)*을 제어할 수 있게 해
줍니다. 모듈 안의 코드는 기본적으로 비공개이기 때문입니다. 비공개 아이템은 외부에서
사용할 수 없는 내부 구현 세부 사항입니다. 모듈과 그 안의 아이템을 공개로 만드는
선택도 할 수 있으며, 이는 외부 코드가 사용하고 의존할 수 있도록 노출합니다.

예시로, 식당의 기능을 제공하는 라이브러리 크레이트를 작성해 봅시다. 함수의 시그
니처는 정의하되, 본문은 비워 두어 식당의 구현이 아니라 코드의 조직화에 집중하도록
하겠습니다.

식당 업계에서는 식당의 어떤 부분을 프론트 오브 하우스(front of house), 다른 부분을
백 오브 하우스(back of house)라고 부릅니다. *프론트 오브 하우스*는 고객이 있는
공간입니다. 호스트가 고객을 좌석으로 안내하고, 서버가 주문과 결제를 받고, 바텐더가
음료를 만드는 공간을 포함합니다. *백 오브 하우스*는 셰프와 요리사가 주방에서 일하고,
설거지 담당자가 정리하고, 매니저가 행정 업무를 하는 공간입니다.

이렇게 크레이트를 구조화하기 위해, 함수를 중첩된 모듈로 조직화할 수 있습니다.
`cargo new restaurant --lib`로 `restaurant`이라는 이름의 새 라이브러리를 만드
세요. 그런 다음 Listing 7-1의 코드를 *src/lib.rs*에 입력해 몇몇 모듈과 함수
시그니처를 정의합니다. 이 코드는 프론트 오브 하우스 부분입니다.

<Listing number="7-1" file-name="src/lib.rs" caption="다른 모듈들을 담고 그 모듈들이 함수를 담는 `front_of_house` 모듈">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

모듈은 `mod` 키워드와 모듈 이름(이 경우 `front_of_house`)으로 정의합니다. 그런
다음 모듈 본문이 중괄호 안에 들어갑니다. 모듈 안에는 다른 모듈을 둘 수 있는데,
여기서는 `hosting`과 `serving` 모듈이 그 예입니다. 모듈은 구조체, 열거형, 상수,
트레이트, 그리고 Listing 7-1처럼 함수와 같은 다른 아이템의 정의도 담을 수 있습
니다.

모듈을 사용하면 관련 정의들을 함께 묶고 왜 연관되어 있는지 이름을 붙일 수 있습
니다. 이 코드를 사용하는 프로그래머는 모든 정의를 훑지 않고도 그룹을 기준으로
코드를 탐색할 수 있으므로, 자신과 관련된 정의를 더 쉽게 찾을 수 있습니다. 이
코드에 새 기능을 추가하는 프로그래머는 프로그램이 조직화된 상태를 유지하도록
코드를 어디에 둘지 알게 됩니다.

앞서 *src/main.rs*와 *src/lib.rs*를 *크레이트 루트*라고 부른다고 언급했습니다.
이 이름의 이유는, 두 파일 중 어느 쪽의 내용이든 크레이트의 모듈 구조의 루트에서
`crate`라는 이름의 모듈을 형성하기 때문입니다. 이 모듈 구조를 *모듈 트리(module
tree)*라고 부릅니다.

Listing 7-2는 Listing 7-1 구조의 모듈 트리를 보여 줍니다.

<Listing number="7-2" caption="Listing 7-1의 코드에 대한 모듈 트리">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

이 트리는 어떤 모듈이 다른 모듈 안에 어떻게 중첩되는지 보여 줍니다. 예를 들어,
`hosting`은 `front_of_house` 안에 중첩되어 있습니다. 또한 이 트리는 어떤 모듈은
*형제(sibling)*, 즉 같은 모듈에 정의되어 있음을 보여 줍니다. `hosting`과 `serving`
은 `front_of_house` 안에 정의된 형제입니다. 모듈 A가 모듈 B 안에 포함되어 있다면,
모듈 A를 모듈 B의 *자식(child)*이라고 하고 모듈 B를 모듈 A의 *부모(parent)*라고
부릅니다. 모듈 트리 전체가 `crate`라는 이름의 암묵적 모듈 아래에 뿌리내리고 있다는
점에 주목하세요.

모듈 트리는 여러분 컴퓨터의 파일 시스템의 디렉터리 트리를 떠올리게 할지도 모릅
니다. 매우 적절한 비유입니다! 파일 시스템의 디렉터리처럼 모듈을 사용해 코드를
조직화합니다. 그리고 디렉터리 안의 파일처럼, 우리 모듈을 찾을 방법이 필요합니다.
