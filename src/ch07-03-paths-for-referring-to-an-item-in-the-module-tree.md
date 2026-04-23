## 모듈 트리에서 항목을 참조하는 경로

러스트에게 모듈 트리에서 아이템을 어디서 찾아야 하는지 알려 주기 위해, 파일 시스템
을 탐색할 때 경로를 쓰는 것과 같은 방식으로 경로를 사용합니다. 함수를 호출하려면
그 함수의 경로를 알아야 합니다.

경로는 두 가지 형태를 취할 수 있습니다.

- *절대 경로(absolute path)*는 크레이트 루트에서 시작하는 전체 경로입니다. 외부
  크레이트의 코드라면 절대 경로는 크레이트 이름으로 시작하고, 현재 크레이트의
  코드라면 리터럴 `crate`로 시작합니다.
- *상대 경로(relative path)*는 현재 모듈에서 시작하며, `self`, `super`, 또는
  현재 모듈의 식별자를 사용합니다.

절대 경로든 상대 경로든 뒤에 더블 콜론(`::`)으로 구분된 하나 이상의 식별자가
이어집니다.

Listing 7-1로 돌아가, `add_to_waitlist` 함수를 호출하고 싶다고 해 봅시다. 이는
`add_to_waitlist` 함수의 경로가 무엇인지 묻는 것과 같습니다. Listing 7-3은
Listing 7-1에서 일부 모듈과 함수를 뺀 코드입니다.

크레이트 루트에 정의된 새 함수 `eat_at_restaurant`에서 `add_to_waitlist` 함수를
호출하는 두 가지 방법을 보여 드리겠습니다. 이 경로들은 올바르지만, 이 예제가 그대로
컴파일되지 않도록 막는 또 다른 문제가 남아 있습니다. 그 이유는 곧 설명합니다.

`eat_at_restaurant` 함수는 우리 라이브러리 크레이트의 공개 API의 일부이므로,
`pub` 키워드로 표시합니다. `pub`에 대해서는 [“`pub` 키워드로 경로
노출하기”][pub]<!-- ignore --> 절에서 더 자세히 다룹니다.

<Listing number="7-3" file-name="src/lib.rs" caption="절대 경로와 상대 경로를 사용해 `add_to_waitlist` 함수 호출하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

`eat_at_restaurant`에서 `add_to_waitlist` 함수를 처음 호출할 때에는 절대 경로를
사용합니다. `add_to_waitlist` 함수는 `eat_at_restaurant`과 같은 크레이트에 정의
되어 있으므로, 절대 경로를 `crate` 키워드로 시작할 수 있습니다. 그런 다음
`add_to_waitlist`에 도달할 때까지 연속된 모듈들을 하나씩 포함시킵니다. 같은 구조의
파일 시스템을 상상해 볼 수 있습니다. `add_to_waitlist` 프로그램을 실행하기 위해
경로 `/front_of_house/hosting/add_to_waitlist`를 지정할 것입니다. 크레이트 루트
에서 시작하기 위해 `crate` 이름을 사용하는 것은, 셸에서 파일 시스템 루트에서 시작
하기 위해 `/`를 사용하는 것과 같습니다.

`eat_at_restaurant`에서 `add_to_waitlist`를 두 번째로 호출할 때에는 상대 경로를
사용합니다. 경로는 `eat_at_restaurant`과 같은 모듈 트리 수준에 정의된 모듈 이름인
`front_of_house`로 시작합니다. 여기서 파일 시스템에 해당하는 것은
`front_of_house/hosting/add_to_waitlist` 경로를 사용하는 것이 될 것입니다. 모듈
이름으로 시작한다는 것은 경로가 상대적이라는 뜻입니다.

상대 경로를 쓸지 절대 경로를 쓸지는 프로젝트에 따라 여러분이 내릴 결정이며,
아이템 정의 코드를 아이템을 사용하는 코드와 별개로 옮길 가능성이 더 높은지, 함께
옮길 가능성이 더 높은지에 따라 달라집니다. 예를 들어, `front_of_house` 모듈과
`eat_at_restaurant` 함수를 `customer_experience`라는 이름의 모듈 안으로 옮긴다면,
`add_to_waitlist`로의 절대 경로는 갱신해야 하지만 상대 경로는 여전히 유효합니다.
그러나 `eat_at_restaurant` 함수를 별도로 `dining`이라는 모듈로 옮긴다면,
`add_to_waitlist` 호출의 절대 경로는 그대로이지만 상대 경로는 갱신해야 합니다.
우리는 일반적으로 절대 경로를 지정하는 것을 선호합니다. 코드 정의와 아이템 호출을
서로 독립적으로 옮기고 싶은 경우가 더 많기 때문입니다.

Listing 7-3을 컴파일해 보고, 왜 아직 컴파일되지 않는지 알아봅시다! 얻게 되는
에러는 Listing 7-4에 나와 있습니다.

<Listing number="7-4" caption="Listing 7-3의 코드를 빌드할 때의 컴파일러 에러">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

에러 메시지는 `hosting` 모듈이 비공개라고 말합니다. 다시 말해, `hosting` 모듈과
`add_to_waitlist` 함수에 대한 경로는 맞지만, 러스트가 비공개 섹션에 접근할 수 없기
때문에 그것을 사용하지 못하게 막는 것입니다. 러스트에서는 모든 아이템(함수, 메서드,
구조체, 열거형, 모듈, 상수)이 기본적으로 부모 모듈에 대해 비공개입니다. 함수나
구조체 같은 아이템을 비공개로 만들고 싶다면, 그것을 모듈 안에 둡니다.

부모 모듈의 아이템은 자식 모듈 안의 비공개 아이템을 사용할 수 없지만, 자식 모듈의
아이템은 조상 모듈의 아이템을 사용할 수 있습니다. 이는 자식 모듈이 구현 세부 사항을
감싸고 숨기지만, 자식 모듈은 자신이 정의된 맥락을 볼 수 있기 때문입니다. 우리의
비유를 이어 가자면, 비공개성 규칙을 식당의 백오피스처럼 생각해 보세요. 그 안에서
일어나는 일은 식당 고객에게 비공개이지만, 오피스 매니저는 자신이 운영하는 식당에서
일어나는 모든 일을 볼 수 있고 할 수 있습니다.

러스트는 모듈 시스템이 이런 방식으로 동작하도록 선택해, 내부 구현 세부 사항을
기본적으로 숨겼습니다. 그 덕분에 바깥 코드를 깨뜨리지 않고 내부 코드의 어느 부분을
바꿀 수 있는지 알 수 있습니다. 그렇지만 러스트는 `pub` 키워드로 아이템을 공개로
만들어, 자식 모듈 코드의 내부 부분을 외부 조상 모듈에 노출하는 선택지도 제공합니다.

### `pub` 키워드로 경로 노출하기

Listing 7-4의 `hosting` 모듈이 비공개라고 알려 준 에러로 돌아가 봅시다. 부모 모듈
의 `eat_at_restaurant` 함수가 자식 모듈의 `add_to_waitlist` 함수에 접근할 수 있기를
원하므로, Listing 7-5에 보이듯 `hosting` 모듈에 `pub` 키워드를 표시합니다.

<Listing number="7-5" file-name="src/lib.rs" caption="`eat_at_restaurant`에서 사용하기 위해 `hosting` 모듈을 `pub`으로 선언하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

안타깝게도 Listing 7-5의 코드는 Listing 7-6에 보이듯 여전히 컴파일러 에러를
냅니다.

<Listing number="7-6" caption="Listing 7-5의 코드를 빌드할 때의 컴파일러 에러">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

무슨 일이 벌어졌을까요? `mod hosting` 앞에 `pub` 키워드를 추가하면 모듈이 공개가
됩니다. 이 변경으로, `front_of_house`에 접근할 수 있다면 `hosting`에도 접근할
수 있습니다. 하지만 `hosting`의 *내용*은 여전히 비공개입니다. 모듈을 공개로 만들어도
그 내용이 공개가 되지는 않습니다. 모듈에 붙은 `pub` 키워드는 조상 모듈의 코드가
그것을 참조하도록만 허용할 뿐, 내부 코드에 접근하도록 허용하지는 않습니다. 모듈은
컨테이너이기 때문에, 모듈만 공개로 만들어서는 할 수 있는 일이 많지 않습니다.
한 걸음 더 나아가 모듈 안의 아이템 중 하나 이상을 공개로 만드는 선택도 해야 합니다.

Listing 7-6의 에러는 `add_to_waitlist` 함수가 비공개라고 말합니다. 비공개성 규칙은
모듈뿐 아니라 구조체, 열거형, 함수, 메서드에도 적용됩니다.

Listing 7-7처럼 `add_to_waitlist` 함수의 정의 앞에도 `pub` 키워드를 추가해 공개로
만듭시다.

<Listing number="7-7" file-name="src/lib.rs" caption="`mod hosting`과 `fn add_to_waitlist`에 `pub` 키워드를 추가해 `eat_at_restaurant`에서 함수를 호출할 수 있게 하기.">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

이제 코드가 컴파일됩니다! 비공개성 규칙을 기준으로 `pub` 키워드를 추가하는 것이
왜 `eat_at_restaurant`에서 이 경로들을 사용하게 해 주는지 이해하기 위해, 절대
경로와 상대 경로를 살펴봅시다.

절대 경로에서는 `crate`, 즉 크레이트 모듈 트리의 루트에서 시작합니다. `front_of_house`
모듈은 크레이트 루트에 정의되어 있습니다. `front_of_house`가 공개는 아니지만,
`eat_at_restaurant` 함수가 `front_of_house`와 같은 모듈에 정의되어 있기 때문에
(즉, `eat_at_restaurant`와 `front_of_house`는 형제이므로), `eat_at_restaurant`
에서 `front_of_house`를 참조할 수 있습니다. 다음은 `pub`으로 표시된 `hosting`
모듈입니다. `hosting`의 부모 모듈에 접근할 수 있으므로 `hosting`에 접근할 수
있습니다. 마지막으로 `add_to_waitlist` 함수는 `pub`으로 표시되어 있고, 그 부모
모듈에 접근할 수 있으므로 이 함수 호출이 동작합니다!

상대 경로에서 로직은 첫 단계만 빼고 절대 경로와 같습니다. 크레이트 루트에서
시작하는 대신, 경로가 `front_of_house`에서 시작합니다. `front_of_house` 모듈은
`eat_at_restaurant`과 같은 모듈 안에 정의되어 있으므로, `eat_at_restaurant`이
정의된 모듈에서 시작하는 상대 경로가 동작합니다. 그런 다음 `hosting`과
`add_to_waitlist`가 `pub`으로 표시되어 있으므로 나머지 경로도 동작하며, 이 함수
호출은 유효합니다!

여러분의 라이브러리 크레이트를 다른 프로젝트에서 사용할 수 있도록 공유할 계획
이라면, 여러분의 공개 API는 사용자가 여러분의 코드와 어떻게 상호작용할 수 있는
지를 결정하는, 크레이트 사용자와의 계약입니다. 사람들이 여러분의 크레이트에 더
쉽게 의존할 수 있도록, 공개 API에 대한 변경을 관리할 때 고려해야 할 점들이 많습
니다. 이 주제는 이 책의 범위를 벗어납니다. 관심이 있다면 [러스트 API
지침(API Guidelines)][api-guidelines]을 참고하세요.

> #### 바이너리와 라이브러리가 있는 패키지의 모범 사례
>
> 패키지는 *src/main.rs* 바이너리 크레이트 루트와 *src/lib.rs* 라이브러리 크레
> 이트 루트를 모두 포함할 수 있고, 두 크레이트 모두 기본적으로 패키지 이름을
> 가진다고 이야기했습니다. 일반적으로 라이브러리와 바이너리 크레이트를 모두 가진
> 이 패턴의 패키지는, 바이너리 크레이트에 라이브러리 크레이트에 정의된 코드를
> 호출하는 실행 파일을 시작할 만큼의 코드만 둡니다. 이렇게 하면 라이브러리 크레
> 이트의 코드를 공유할 수 있으므로 다른 프로젝트들이 이 패키지가 제공하는 기능
> 으로부터 최대한 이득을 얻을 수 있습니다.
>
> 모듈 트리는 *src/lib.rs*에 정의되어야 합니다. 그러면 어떤 공개 아이템이든 패키지
> 이름으로 시작하는 경로로 바이너리 크레이트에서 사용할 수 있습니다. 바이너리
> 크레이트는 완전히 외부의 크레이트가 라이브러리 크레이트를 사용하는 것과 똑같이
> 라이브러리 크레이트의 사용자가 됩니다. 공개 API만 사용할 수 있습니다. 이는 좋은
> API를 설계하는 데 도움이 됩니다. 여러분이 저자이면서 동시에 고객이 되니까요!
>
> [12장][ch12]<!-- ignore -->에서는 이 조직화 관례를, 바이너리 크레이트와 라이브
> 러리 크레이트를 모두 포함하는 명령줄 프로그램으로 시연합니다.

### `super`로 상대 경로 시작하기

경로의 시작에 `super`를 사용해, 현재 모듈이나 크레이트 루트가 아닌 부모 모듈에서
시작하는 상대 경로를 만들 수 있습니다. 이는 파일 시스템 경로를 `..`로 시작해서
부모 디렉터리로 가는 것과 비슷합니다. `super`를 사용하면 부모 모듈에 있다는 것을
우리가 아는 아이템을 참조할 수 있습니다. 이는 모듈이 부모와 밀접하게 관련되어
있지만 부모가 언젠가 모듈 트리의 다른 곳으로 옮겨질 수도 있을 때 모듈 트리를
재배치하기 쉽게 만들어 줍니다.

셰프가 잘못된 주문을 수정해 직접 고객에게 전달하는 상황을 모델링한 Listing 7-8의
코드를 봅시다. `back_of_house` 모듈에 정의된 `fix_incorrect_order` 함수는 부모
모듈에 정의된 `deliver_order` 함수를 `super`로 시작하는 경로로 지정해 호출합니다.

<Listing number="7-8" file-name="src/lib.rs" caption="`super`로 시작하는 상대 경로로 함수 호출하기">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

`fix_incorrect_order` 함수는 `back_of_house` 모듈 안에 있으므로, `super`를 사용해
`back_of_house`의 부모 모듈로 갈 수 있습니다. 이 경우는 루트인 `crate`입니다.
그 자리에서 `deliver_order`를 찾아 발견합니다. 성공! 우리는 `back_of_house` 모듈
과 `deliver_order` 함수가 서로 같은 관계를 유지할 가능성이 높고, 크레이트의 모듈
트리를 재조직하기로 결정한다면 함께 옮겨질 가능성이 높다고 생각합니다. 따라서 이
코드가 다른 모듈로 옮겨진다면 미래에 코드를 갱신할 곳이 더 적도록 `super`를
사용했습니다.

### 구조체와 열거형을 공개로 만들기

`pub`을 사용해 구조체와 열거형도 공개로 지정할 수 있지만, 구조체와 열거형에서
`pub`을 사용할 때에는 몇 가지 추가 세부 사항이 있습니다. 구조체 정의 앞에 `pub`
을 쓰면 구조체는 공개가 되지만, 구조체의 필드는 여전히 비공개입니다. 각 필드를
경우에 따라 공개로 만들거나 비공개로 둘 수 있습니다. Listing 7-9에서는 공개 `toast`
필드와 비공개 `seasonal_fruit` 필드를 가진 공개 `back_of_house::Breakfast` 구조체
를 정의했습니다. 이는 식당에서 고객이 식사와 함께 나오는 빵의 종류는 고를 수
있지만, 제철 과일과 재고에 따라 어떤 과일이 식사에 곁들여지는지는 셰프가 결정하는
경우를 모델링한 것입니다. 사용 가능한 과일이 빠르게 바뀌므로 고객은 과일을 고를
수도, 어떤 과일이 나올지 볼 수도 없습니다.

<Listing number="7-9" file-name="src/lib.rs" caption="일부 공개 필드와 일부 비공개 필드를 가진 구조체">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

`back_of_house::Breakfast` 구조체의 `toast` 필드가 공개이므로, `eat_at_restaurant`
에서 점 표기법으로 `toast` 필드에 쓰고 읽을 수 있습니다. `seasonal_fruit`은
비공개이기 때문에, `eat_at_restaurant`에서는 `seasonal_fruit` 필드를 사용할 수
없다는 점에 주목하세요. `seasonal_fruit` 필드 값을 수정하는 줄의 주석을 풀어 보고,
어떤 에러가 나는지 확인해 보세요!

또한, `back_of_house::Breakfast`에는 비공개 필드가 있기 때문에, 구조체는 `Breakfast`의
인스턴스를 생성하는 공개 연관 함수를 제공해야 한다는 점에 유의하세요(여기서는
`summer`라고 이름 붙였습니다). `Breakfast`에 그런 함수가 없다면, `eat_at_restaurant`
에서 비공개 `seasonal_fruit` 필드의 값을 설정할 수 없으므로 `Breakfast`의 인스턴
스를 만들 수 없습니다.

반면에 열거형을 공개로 만들면, 그 배리언트는 모두 공개가 됩니다. Listing 7-10에
보이듯 `enum` 키워드 앞에 `pub`만 있으면 됩니다.

<Listing number="7-10" file-name="src/lib.rs" caption="열거형을 공개로 지정하면 모든 배리언트도 공개가 됩니다.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

`Appetizer` 열거형을 공개로 만들었기 때문에, `eat_at_restaurant`에서 `Soup`와
`Salad` 배리언트를 사용할 수 있습니다.

배리언트가 공개가 아니면 열거형은 그리 유용하지 않습니다. 매번 모든 열거형 배리
언트에 `pub`을 다는 것은 성가실 것이므로, 열거형 배리언트의 기본값은 공개입니다.
구조체는 필드가 공개가 아닌 상태에서도 유용할 때가 많으므로, 구조체 필드는 `pub`
으로 명시되지 않는 한 기본적으로 모든 것이 비공개라는 일반 규칙을 따릅니다.

`pub`과 관련해 아직 다루지 않은 상황이 하나 더 있고, 그것이 모듈 시스템의 마지막
기능인 `use` 키워드입니다. 먼저 `use`를 그 자체로 다룬 뒤, `pub`과 `use`를 어떻게
결합하는지 보여 드리겠습니다.

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
