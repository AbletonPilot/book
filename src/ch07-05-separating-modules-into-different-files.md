## 모듈을 여러 파일로 분리하기

지금까지 이 장의 모든 예제는 하나의 파일에 여러 모듈을 정의했습니다. 모듈이 커지면
코드를 더 쉽게 탐색하기 위해 정의를 별도의 파일로 옮기고 싶어질 수 있습니다.

예를 들어, 여러 식당 모듈을 담고 있던 Listing 7-17의 코드에서 시작해 봅시다. 모든
모듈을 크레이트 루트 파일에 정의하는 대신, 모듈들을 파일로 추출하겠습니다. 이 경우
크레이트 루트 파일은 *src/lib.rs*이지만, 이 절차는 크레이트 루트 파일이 *src/main.rs*
인 바이너리 크레이트에서도 동작합니다.

먼저, `front_of_house` 모듈을 자기만의 파일로 추출하겠습니다. `front_of_house`
모듈의 중괄호 안의 코드를 제거하고 `mod front_of_house;` 선언만 남겨, *src/lib.rs*
가 Listing 7-21의 코드를 담게 합니다. 이 코드는 Listing 7-22의 *src/front_of_house.rs*
파일을 만들기 전까지는 컴파일되지 않는다는 점에 유의하세요.

<Listing number="7-21" file-name="src/lib.rs" caption="본문이 *src/front_of_house.rs*에 있게 될 `front_of_house` 모듈 선언">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

다음으로, 중괄호 안에 있던 코드를 Listing 7-22처럼 *src/front_of_house.rs*라는
이름의 새 파일에 넣습니다. 컴파일러는 크레이트 루트에서 `front_of_house`라는
이름의 모듈 선언을 만났기 때문에, 이 파일에서 찾아야 한다는 것을 압니다.

<Listing number="7-22" file-name="src/front_of_house.rs" caption="*src/front_of_house.rs* 안에 있는 `front_of_house` 모듈의 정의">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

`mod` 선언으로 파일을 로드하는 것은 모듈 트리에서 *한 번만* 하면 된다는 점에 유의
하세요. 컴파일러가 파일이 프로젝트의 일부임을 알게 되고(그리고 `mod` 문을 둔 위치
덕분에 코드가 모듈 트리의 어디에 위치하는지도 알게 되고) 나면, 프로젝트의 다른
파일들은 [“모듈 트리에서 항목을 참조하는 경로”][paths]<!-- ignore --> 절에서 다룬
것처럼, 로드된 파일의 코드를 그것이 선언된 곳으로의 경로로 참조해야 합니다. 다시
말해, `mod`는 다른 프로그래밍 언어에서 봤을 “include” 동작이 *아닙니다*.

다음으로, `hosting` 모듈을 자기만의 파일로 추출하겠습니다. `hosting`은 루트 모듈의
자식이 아니라 `front_of_house`의 자식 모듈이기 때문에 과정이 조금 다릅니다.
`hosting`을 위한 파일은 모듈 트리에서 그 조상들의 이름을 따서 붙인 새 디렉터리에
둘 것입니다. 이 경우에는 *src/front_of_house*입니다.

`hosting`을 옮기기 시작하기 위해, *src/front_of_house.rs*를 다음처럼 `hosting`
모듈의 선언만 담도록 바꿉니다.

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

그런 다음, `hosting` 모듈에서 만든 정의를 담을 *src/front_of_house* 디렉터리와
*hosting.rs* 파일을 만듭니다.

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

만약 대신 *hosting.rs*를 *src* 디렉터리에 두면, 컴파일러는 *hosting.rs*의 코드가
크레이트 루트에 선언된 `hosting` 모듈에 있다고 기대하고, `front_of_house` 모듈의
자식으로 선언되었다고는 기대하지 않을 것입니다. 어떤 모듈의 코드를 어떤 파일에서
찾아야 하는지에 대한 컴파일러의 규칙은, 디렉터리와 파일이 모듈 트리에 더 근접하게
대응되도록 만듭니다.

> ### 대안 파일 경로
>
> 지금까지 러스트 컴파일러가 사용하는 가장 관용적인 파일 경로를 다뤘지만, 러스트
> 는 더 오래된 스타일의 파일 경로도 지원합니다. 크레이트 루트에 선언된
> `front_of_house`라는 이름의 모듈의 경우, 컴파일러는 다음 위치에서 모듈의 코드를
> 찾습니다.
>
> - *src/front_of_house.rs* (앞서 다룬 방식)
> - *src/front_of_house/mod.rs* (오래된 방식, 여전히 지원됨)
>
> `front_of_house`의 서브모듈인 `hosting`이라는 이름의 모듈의 경우, 컴파일러는
> 다음 위치에서 모듈의 코드를 찾습니다.
>
> - *src/front_of_house/hosting.rs* (앞서 다룬 방식)
> - *src/front_of_house/hosting/mod.rs* (오래된 방식, 여전히 지원됨)
>
> 같은 모듈에 두 스타일을 함께 쓰면 컴파일러 에러가 납니다. 같은 프로젝트에서
> 서로 다른 모듈에 대해 두 스타일을 섞어 쓰는 것은 허용되지만, 프로젝트를 탐색
> 하는 사람들에게는 혼란스러울 수 있습니다.
>
> *mod.rs*라는 이름의 파일을 사용하는 스타일의 주된 단점은, 프로젝트에 *mod.rs*
> 라는 이름의 파일이 여러 개 생길 수 있다는 것입니다. 에디터에서 그것들을 동시에
> 열어 두면 헷갈릴 수 있습니다.

각 모듈의 코드를 별도의 파일로 옮겼지만, 모듈 트리는 그대로입니다. 정의가 서로
다른 파일에 있더라도 `eat_at_restaurant`의 함수 호출은 수정 없이 동작합니다. 이
기법을 사용하면 모듈의 크기가 커짐에 따라 모듈을 새 파일로 옮길 수 있습니다.

*src/lib.rs*의 `pub use crate::front_of_house::hosting` 문도 바뀌지 않았음에 유의
하세요. `use`는 어떤 파일이 크레이트의 일부로 컴파일되는지에 영향을 주지 않습
니다. `mod` 키워드가 모듈을 선언하고, 러스트는 모듈에 들어가는 코드를 모듈과 같은
이름의 파일에서 찾습니다.

## 요약

러스트는 한 모듈에 정의된 아이템을 다른 모듈에서 참조할 수 있도록, 패키지를 여러
크레이트로, 크레이트를 여러 모듈로 쪼갤 수 있게 해 줍니다. 절대 경로나 상대 경로를
지정해 이를 할 수 있습니다. 이 경로들은 `use` 문으로 스코프에 가져와, 해당 스코프
에서 아이템을 여러 번 사용할 때 더 짧은 경로를 쓸 수 있게 해 줍니다. 모듈 코드는
기본적으로 비공개이지만, `pub` 키워드를 추가해 정의를 공개로 만들 수 있습니다.

다음 장에서는 잘 조직화된 코드 안에서 쓸 수 있는 표준 라이브러리의 몇 가지 컬렉션
자료 구조를 살펴봅니다.

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
