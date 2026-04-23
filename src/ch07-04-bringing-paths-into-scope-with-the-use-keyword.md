## `use` 키워드로 경로를 스코프로 가져오기

함수를 호출하기 위해 경로를 매번 써내야 하는 것은 불편하고 반복적으로 느껴질 수
있습니다. Listing 7-7에서는 `add_to_waitlist` 함수로 가는 절대 경로를 썼든 상대
경로를 썼든, `add_to_waitlist`를 호출하고 싶을 때마다 `front_of_house`와 `hosting`
도 함께 지정해야 했습니다. 다행히 이 과정을 단순화할 방법이 있습니다. `use`
키워드로 한 번 경로에 대한 바로가기를 만든 뒤, 그 스코프의 다른 모든 곳에서는
짧은 이름을 쓸 수 있습니다.

Listing 7-11에서는 `crate::front_of_house::hosting` 모듈을 `eat_at_restaurant`
함수의 스코프로 가져와, `eat_at_restaurant`에서 `add_to_waitlist` 함수를 호출하기
위해 `hosting::add_to_waitlist`만 지정하면 되도록 합니다.

<Listing number="7-11" file-name="src/lib.rs" caption="`use`로 모듈을 스코프로 가져오기">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

스코프에 `use`와 경로를 추가하는 것은 파일 시스템에서 심볼릭 링크를 만드는 것과
비슷합니다. 크레이트 루트에 `use crate::front_of_house::hosting`을 추가함으로써,
이제 `hosting`은 그 스코프에서 유효한 이름이 되며, 마치 `hosting` 모듈이 크레이트
루트에 정의되어 있었던 것처럼 동작합니다. `use`로 스코프에 가져온 경로 또한 다른
경로들처럼 비공개성을 검사합니다.

`use`는 오직 그 `use`가 등장한 특정 스코프에만 바로가기를 만든다는 점에 유의하세요.
Listing 7-12는 `eat_at_restaurant` 함수를 `customer`라는 새 자식 모듈로 옮깁니다.
그러면 이 모듈은 `use` 문과는 다른 스코프가 되므로, 함수 본문이 컴파일되지 않습
니다.

<Listing number="7-12" file-name="src/lib.rs" caption="`use` 문은 자신이 속한 스코프에만 적용됩니다.">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

컴파일러 에러는 바로가기가 `customer` 모듈 안에서는 더 이상 적용되지 않는다는
것을 보여 줍니다.

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

`use`가 이제 자신의 스코프에서 사용되지 않는다는 경고도 있다는 점에 주목하세요!
이 문제를 해결하려면 `use`도 `customer` 모듈 안으로 옮기거나, 자식 `customer`
모듈 안에서 `super::hosting`으로 부모 모듈의 바로가기를 참조하세요.

### 관용적인 `use` 경로 만들기

Listing 7-11에서 `use crate::front_of_house::hosting`을 지정한 뒤 `eat_at_restaurant`
에서 `hosting::add_to_waitlist`를 호출한 것을 보고, 왜 같은 결과를 얻기 위해
Listing 7-13처럼 `use` 경로를 `add_to_waitlist` 함수까지 지정하지 않았는지 궁금
했을 수도 있습니다.

<Listing number="7-13" file-name="src/lib.rs" caption="`use`로 `add_to_waitlist` 함수를 스코프로 가져오기. 이는 관용적이지 않습니다.">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

Listing 7-11과 Listing 7-13이 같은 일을 해내기는 하지만, `use`로 함수를 스코프로
가져오는 관용적인 방법은 Listing 7-11입니다. `use`로 함수의 부모 모듈을 스코프로
가져온다는 것은, 함수를 호출할 때 부모 모듈을 지정해야 한다는 뜻입니다. 함수를
호출할 때 부모 모듈을 지정하면 함수가 로컬에 정의된 것이 아님이 분명해지고, 전체
경로의 반복은 최소화됩니다. Listing 7-13의 코드는 `add_to_waitlist`가 어디에 정의
되어 있는지 명확하지 않습니다.

반면에 `use`로 구조체, 열거형, 그 외 아이템을 가져올 때에는 전체 경로를 지정하는
것이 관용적입니다. Listing 7-14는 표준 라이브러리의 `HashMap` 구조체를 바이너리
크레이트의 스코프로 가져오는 관용적인 방법을 보여 줍니다.

<Listing number="7-14" file-name="src/main.rs" caption="관용적인 방법으로 `HashMap`을 스코프로 가져오기">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

이 관용 뒤에 강한 이유가 있는 것은 아닙니다. 그냥 자리잡은 관례일 뿐이며, 사람들은
이 방식으로 러스트 코드를 읽고 쓰는 데 익숙해져 있습니다.

이 관용에 예외가 있다면, 같은 이름의 두 아이템을 `use` 문으로 스코프에 가져오는
경우입니다. 러스트는 그것을 허용하지 않기 때문이죠. Listing 7-15는 같은 이름이지만
부모 모듈이 다른 두 `Result` 타입을 스코프로 가져와 어떻게 참조하는지 보여 줍니다.

<Listing number="7-15" file-name="src/lib.rs" caption="같은 이름의 두 타입을 같은 스코프로 가져오려면 부모 모듈을 사용해야 합니다.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

보시다시피, 부모 모듈을 사용해 두 `Result` 타입을 구분합니다. 만약 대신
`use std::fmt::Result`와 `use std::io::Result`를 지정했다면, 같은 스코프에 두
`Result` 타입을 가지게 되었을 것이고, `Result`를 사용했을 때 러스트가 어느 쪽을
의미하는지 알 수 없었을 것입니다.

### `as` 키워드로 새 이름 제공하기

같은 이름의 두 타입을 `use`로 같은 스코프로 가져오는 문제에 대한 또 다른 해결책이
있습니다. 경로 뒤에 `as`와, 그 타입에 대한 새 로컬 이름 또는 *별칭(alias)*을
지정하는 것입니다. Listing 7-16은 `as`를 사용해 두 `Result` 타입 중 하나의 이름을
바꿔서 Listing 7-15의 코드를 다른 방식으로 작성한 것을 보여 줍니다.

<Listing number="7-16" file-name="src/lib.rs" caption="`as` 키워드로 스코프로 가져올 때 타입의 이름 바꾸기">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

두 번째 `use` 문에서 `std::io::Result` 타입에 새 이름 `IoResult`를 선택했는데,
이는 스코프로 함께 가져온 `std::fmt`의 `Result`와 충돌하지 않습니다. Listing 7-15
와 Listing 7-16 모두 관용적으로 여겨지므로, 선택은 여러분에게 달려 있습니다!

### `pub use`로 이름 재내보내기

`use` 키워드로 이름을 스코프로 가져올 때, 그 이름은 우리가 가져온 스코프에 대해
비공개입니다. 그 스코프 밖의 코드도 그 이름이 그 스코프에 정의된 것처럼 참조할
수 있게 하려면, `pub`과 `use`를 결합할 수 있습니다. 이 기법을 *재내보내기
(re-exporting)*라고 부릅니다. 아이템을 스코프로 가져오는 동시에, 다른 이들이
자신의 스코프로 가져올 수 있도록 그 아이템을 사용 가능하게 만들기 때문입니다.

Listing 7-17은 Listing 7-11의 루트 모듈에 있던 `use`를 `pub use`로 바꾼 코드를
보여 줍니다.

<Listing number="7-17" file-name="src/lib.rs" caption="`pub use`로 새 스코프에서 어떤 코드든 이름을 사용할 수 있게 만들기">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

이 변경 전에는, 외부 코드가 `add_to_waitlist` 함수를 호출하려면
`restaurant::front_of_house::hosting::add_to_waitlist()` 경로를 써야 했을 것이고,
그러려면 `front_of_house` 모듈을 `pub`으로 표시해야 했을 것입니다. 이제 이 `pub
use`가 루트 모듈에서 `hosting` 모듈을 재내보냈으므로, 외부 코드는
`restaurant::hosting::add_to_waitlist()` 경로를 대신 사용할 수 있습니다.

재내보내기는 코드의 내부 구조가 여러분의 코드를 호출하는 프로그래머가 도메인에
대해 생각하는 방식과 다를 때 유용합니다. 예를 들어, 이 식당 비유에서 식당을
운영하는 사람들은 “프론트 오브 하우스”와 “백 오브 하우스”를 떠올립니다. 하지만
식당을 찾는 고객은 아마 그런 용어로 식당의 부분들을 생각하지 않을 것입니다. `pub
use`를 사용하면 한 구조로 코드를 작성하면서도 다른 구조를 노출할 수 있습니다.
그렇게 하면 라이브러리 작업을 하는 프로그래머와 라이브러리를 호출하는 프로그래머
모두에게 잘 조직화된 라이브러리가 됩니다. `pub use`의 또 다른 예와 그것이 크레
이트 문서에 어떤 영향을 주는지는 14장의 [“편리한 공개 API
내보내기”][ch14-pub-use]<!-- ignore -->에서 살펴보겠습니다.

### 외부 패키지 사용하기

2장에서 우리는 난수를 얻기 위해 `rand`라는 외부 패키지를 사용하는 숫자 맞히기 게임
프로젝트를 작성했습니다. 프로젝트에서 `rand`를 사용하기 위해 *Cargo.toml*에 다음
줄을 추가했습니다.

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

*Cargo.toml*에 의존성으로 `rand`를 추가하면, Cargo가 `rand` 패키지와 그 의존성을
[crates.io](https://crates.io/)에서 내려받아 우리 프로젝트에서 `rand`를 사용할
수 있게 해 줍니다.

그런 다음 `rand` 정의를 우리 패키지의 스코프로 가져오기 위해, 크레이트 이름인
`rand`로 시작하는 `use` 줄을 추가하고 스코프로 가져오고 싶은 아이템을 나열했습
니다. 2장의 [“난수 생성하기”][rand]<!-- ignore -->에서 `Rng` 트레이트를 스코프로
가져와 `rand::thread_rng` 함수를 호출했던 것을 떠올려 보세요.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

러스트 커뮤니티 구성원들은 많은 패키지를 [crates.io](https://crates.io/)에 공개해
두었습니다. 그것들을 여러분의 패키지로 끌어오는 일은 같은 과정을 따릅니다.
패키지의 *Cargo.toml* 파일에 나열하고, `use`로 그 크레이트들의 아이템을 스코프로
가져오는 것입니다.

표준 `std` 라이브러리 또한 우리 패키지 외부의 크레이트라는 점에 유의하세요. 표준
라이브러리는 러스트 언어와 함께 배포되므로, `std`를 포함시키기 위해 *Cargo.toml*
을 수정할 필요는 없습니다. 하지만 거기서 아이템을 패키지의 스코프로 가져오려면
`use`로 참조해야 합니다. 예를 들어 `HashMap`에는 다음 줄을 쓸 것입니다.

```rust
use std::collections::HashMap;
```

이것은 표준 라이브러리 크레이트의 이름인 `std`로 시작하는 절대 경로입니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-nested-paths-to-clean-up-large-use-lists"></a>

### 중첩 경로로 긴 `use` 목록 정리하기

같은 크레이트 또는 같은 모듈에 정의된 여러 아이템을 사용하고 있다면, 각 아이템을
자기 줄에 나열하는 것은 파일에서 많은 세로 공간을 차지할 수 있습니다. 예를 들어,
Listing 2-4의 숫자 맞히기 게임에 있었던 이 두 `use` 문은 `std`에서 아이템을 스코프로
가져옵니다.

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

대신 중첩 경로를 사용해 같은 아이템을 한 줄로 스코프에 가져올 수 있습니다. 경로의
공통 부분을 지정하고 더블 콜론을 쓴 다음, 서로 다른 경로 부분의 목록을 중괄호로
감싸서 이를 수행할 수 있습니다. Listing 7-18을 봅시다.

<Listing number="7-18" file-name="src/main.rs" caption="같은 접두사를 가진 여러 아이템을 스코프로 가져오기 위해 중첩 경로 지정하기">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

더 큰 프로그램에서는 같은 크레이트나 모듈에서 많은 아이템을 중첩 경로로 스코프에
가져오면 필요한 별도의 `use` 문 수를 크게 줄일 수 있습니다!

경로의 어떤 수준에서든 중첩 경로를 사용할 수 있으며, 이는 하위 경로를 공유하는 두
`use` 문을 결합할 때 유용합니다. 예를 들어, Listing 7-19는 두 `use` 문을 보여 줍
니다. 하나는 `std::io`를 스코프로 가져오고, 다른 하나는 `std::io::Write`를 스코프
로 가져옵니다.

<Listing number="7-19" file-name="src/lib.rs" caption="둘 중 하나가 다른 하나의 하위 경로인 두 `use` 문">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

이 두 경로의 공통 부분은 `std::io`이며, 그것이 첫 번째 경로의 전부입니다. 이 두
경로를 하나의 `use` 문으로 합치려면, Listing 7-20에 보이듯 중첩 경로에서 `self`를
사용할 수 있습니다.

<Listing number="7-20" file-name="src/lib.rs" caption="Listing 7-19의 경로들을 하나의 `use` 문으로 결합하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

이 한 줄이 `std::io`와 `std::io::Write`를 모두 스코프로 가져옵니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="the-glob-operator"></a>

### 글롭 연산자로 아이템 가져오기

어떤 경로에 정의된 *모든* 공개 아이템을 스코프로 가져오고 싶다면, 그 경로 뒤에
`*` 글롭(glob) 연산자를 지정할 수 있습니다.

```rust
use std::collections::*;
```

이 `use` 문은 `std::collections`에 정의된 모든 공개 아이템을 현재 스코프로 가져
옵니다. 글롭 연산자를 사용할 때에는 주의하세요! 글롭은 어떤 이름이 스코프 안에
있는지, 그리고 프로그램에서 사용된 이름이 어디에 정의되어 있는지 알아내기 어렵게
만들 수 있습니다. 또한 의존성이 자기 정의를 바꾸면 가져온 것도 바뀌므로, 예를
들어 의존성이 같은 스코프에 있는 여러분의 정의와 같은 이름의 정의를 추가하면
의존성을 업그레이드할 때 컴파일러 에러로 이어질 수 있습니다.

글롭 연산자는 테스트할 때 테스트 대상 아래의 모든 것을 `tests` 모듈로 가져오기 위해
자주 쓰입니다. 이에 대해서는 11장의 [“테스트 작성 방법”][writing-tests]<!-- ignore
-->에서 이야기합니다. 글롭 연산자는 때로 프렐류드 패턴의 일부로도 사용됩니다.
이 패턴에 대한 자세한 정보는 [표준 라이브러리
문서](../std/prelude/index.html#other-preludes)<!-- ignore -->를 참고하세요.

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
