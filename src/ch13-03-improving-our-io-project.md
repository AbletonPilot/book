## I/O 프로젝트 개선하기

이터레이터에 대한 새 지식으로, 12장의 I/O 프로젝트의 여러 부분을 더 명확하고
간결하게 만들기 위해 이터레이터를 사용할 수 있습니다. 이터레이터가 `Config::build`
함수와 `search` 함수의 구현을 어떻게 개선할 수 있는지 살펴봅시다.

### 이터레이터로 `clone` 제거하기

Listing 12-6에서는 `String` 값의 슬라이스를 받아 슬라이스를 인덱싱해 값을
복제하여 `Config` 구조체 인스턴스를 만들게 해서, `Config` 구조체가 그 값을
소유할 수 있게 하는 코드를 추가했습니다. Listing 13-17에는 Listing 12-23 당시
의 `Config::build` 함수 구현을 재현했습니다.

<Listing number="13-17" file-name="src/main.rs" caption="Listing 12-23의 `Config::build` 함수 재현">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

당시에 비효율적인 `clone` 호출은 나중에 제거할 것이니 걱정하지 말라고 했죠.
이제 그때가 되었습니다!

여기서 `clone`이 필요했던 이유는 매개변수 `args`가 `String` 요소의 슬라이스
이지만 `build` 함수가 `args`를 소유하지 않기 때문이었습니다. `Config` 인스턴스
의 소유권을 반환하려면, `Config`가 자신의 값을 소유할 수 있도록 `Config`의
`query`와 `file_path` 필드에 해당하는 값을 복제해야 했습니다.

이터레이터에 대한 새 지식으로, `build` 함수가 슬라이스를 빌리는 대신 이터레이터
의 소유권을 인수로 가져가도록 바꿀 수 있습니다. 슬라이스의 길이를 확인하고
특정 위치에 인덱싱하는 코드 대신 이터레이터 기능을 사용할 것입니다. 그러면
이터레이터가 값들에 접근하므로 `Config::build` 함수가 무엇을 하는지 명확해집니다.

`Config::build`가 이터레이터의 소유권을 가져가 빌리는 인덱싱 연산을 중단하면,
`clone`을 호출해 새로 할당하는 대신 이터레이터에서 `String` 값을 `Config`로
이동할 수 있습니다.

#### 반환된 이터레이터 직접 사용하기

I/O 프로젝트의 _src/main.rs_ 파일을 여세요. 다음과 같이 보일 것입니다.

<span class="filename">파일명: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

먼저 Listing 12-24의 `main` 함수 시작 부분을 Listing 13-18의 코드로 바꾸겠습니다.
이번에는 이터레이터를 사용합니다. `Config::build`도 함께 업데이트하기 전까지는
컴파일되지 않을 것입니다.

<Listing number="13-18" file-name="src/main.rs" caption="`env::args`의 반환 값을 `Config::build`에 전달하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

`env::args` 함수는 이터레이터를 반환합니다! 이터레이터 값을 벡터로 모은 뒤
`Config::build`에 슬라이스를 전달하는 대신, 이제는 `env::args`가 반환한
이터레이터의 소유권을 `Config::build`에 직접 전달하고 있습니다.

다음으로 `Config::build`의 정의를 업데이트해야 합니다. `Config::build`의
시그니처를 Listing 13-19처럼 바꿔 봅시다. 함수 본문을 업데이트해야 하므로
아직 컴파일되지 않습니다.

<Listing number="13-19" file-name="src/main.rs" caption="이터레이터를 기대하도록 `Config::build`의 시그니처 업데이트하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

`env::args` 함수의 표준 라이브러리 문서는 이 함수가 반환하는 이터레이터의
타입이 `std::env::Args`이며, 이 타입이 `Iterator` 트레이트를 구현하고 `String`
값을 반환함을 보여 줍니다.

`Config::build` 함수의 시그니처를 업데이트해, 매개변수 `args`가 `&[String]`
대신 `impl Iterator<Item = String>` 트레이트 경계를 가진 제네릭 타입을 가지도록
했습니다. 10장의 [“매개변수로서의 트레이트”][impl-trait]<!-- ignore --> 절에서
다룬 `impl Trait` 문법의 이러한 사용은, `args`가 `Iterator` 트레이트를
구현하며 `String` 항목을 반환하는 어떤 타입이든 될 수 있음을 의미합니다.

`args`의 소유권을 가져가고 순회하며 `args`를 변경할 것이므로, 가변으로 만들기
위해 `args` 매개변수 명세에 `mut` 키워드를 추가할 수 있습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-iterator-trait-methods-instead-of-indexing"></a>

#### `Iterator` 트레이트 메서드 사용하기

다음으로 `Config::build`의 본문을 고치겠습니다. `args`가 `Iterator` 트레이트를
구현하므로, 거기에 `next` 메서드를 호출할 수 있음을 압니다! Listing 13-20은
Listing 12-23의 코드를 `next` 메서드를 사용하도록 업데이트합니다.

<Listing number="13-20" file-name="src/main.rs" caption="이터레이터 메서드를 사용하도록 `Config::build`의 본문 바꾸기">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

`env::args`의 반환 값에서 첫 번째 값은 프로그램 이름임을 기억하세요. 그것은
무시하고 다음 값으로 넘어가고 싶으므로, 먼저 `next`를 호출하되 반환 값은
아무것도 하지 않습니다. 그런 다음 `next`를 호출해 `Config`의 `query` 필드에
넣고 싶은 값을 얻습니다. `next`가 `Some`을 반환하면 `match`로 값을 추출합니다.
`None`을 반환하면 충분한 인수가 주어지지 않았다는 뜻이므로 `Err` 값으로 조기
반환합니다. `file_path` 값에 대해서도 같은 일을 합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="making-code-clearer-with-iterator-adapters"></a>

### 이터레이터 어댑터로 코드 명확하게 하기

I/O 프로젝트의 `search` 함수에서도 이터레이터를 활용할 수 있습니다. 여기
Listing 13-21에 Listing 12-19 당시의 모습을 재현했습니다.

<Listing number="13-21" file-name="src/lib.rs" caption="Listing 12-19의 `search` 함수 구현">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

이터레이터 어댑터 메서드를 사용해 이 코드를 더 간결하게 작성할 수 있습니다.
그렇게 하면 가변 중간 `results` 벡터를 갖지 않아도 됩니다. 함수형 프로그래밍
스타일은 코드를 명확하게 만들기 위해 가변 상태의 양을 최소화하는 것을
선호합니다. 가변 상태를 제거하면 향후에 검색을 병렬로 수행하는 개선을 가능
하게 할 수 있습니다. `results` 벡터에 대한 동시 접근을 관리하지 않아도 되기
때문입니다. Listing 13-22는 이 변경을 보여 줍니다.

<Listing number="13-22" file-name="src/lib.rs" caption="`search` 함수 구현에서 이터레이터 어댑터 메서드 사용하기">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

`search` 함수의 목적은 `query`를 포함하는 `contents`의 모든 줄을 반환하는
것임을 기억하세요. Listing 13-16의 `filter` 예제와 비슷하게, 이 코드는
`line.contains(query)`가 `true`를 반환하는 줄만 유지하기 위해 `filter` 어댑터
를 사용합니다. 그런 다음 일치하는 줄을 `collect`로 다른 벡터에 모읍니다.
훨씬 단순합니다! `search_case_insensitive` 함수도 이터레이터 메서드를 사용
하도록 같은 변경을 자유롭게 해 보세요.

더 나아간 개선으로, `search` 함수에서 `collect` 호출을 제거하고 반환 타입을
`impl Iterator<Item = &'a str>`로 바꿔 함수가 이터레이터 어댑터가 되도록
이터레이터를 반환해 보세요. 테스트도 업데이트해야 한다는 점에 유의하세요!
이 변경 전후로 `minigrep` 도구로 큰 파일을 검색해 동작 차이를 관찰해 보세요.
이 변경 전에는 프로그램이 모든 결과를 모은 뒤에야 아무것도 출력하지 않지만,
변경 후에는 `run` 함수의 `for` 루프가 이터레이터의 게으름을 활용할 수 있기
때문에 일치하는 각 줄이 발견될 때마다 출력됩니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="choosing-between-loops-or-iterators"></a>

### 루프와 이터레이터 사이에서 선택하기

다음으로 생길 자연스러운 질문은 여러분의 코드에서 어느 스타일을 선택해야 할지와
그 이유입니다. Listing 13-21의 원래 구현과 Listing 13-22의 이터레이터를 사용한
버전(이터레이터를 반환하는 대신 반환 전에 모든 결과를 모은다는 가정 하에)
중 말입니다. 대부분의 러스트 프로그래머는 이터레이터 스타일을 선호합니다.
처음에는 감을 잡기가 조금 더 어렵지만, 다양한 이터레이터 어댑터와 그 동작을
익히고 나면 이터레이터가 이해하기 더 쉬워질 수 있습니다. 루프를 이리저리
만지고 새 벡터를 만드는 부분들에 매달리는 대신, 코드는 루프의 상위 수준
목적에 집중합니다. 이는 흔한 코드 일부를 추상화해, 이터레이터의 각 요소가
통과해야 하는 필터링 조건처럼 이 코드에 고유한 개념을 보기 더 쉽게 해 줍니다.

하지만 두 구현이 정말로 같을까요? 직관적인 가정은 더 낮은 수준의 루프가 더
빠를 것이라는 점일 것입니다. 성능에 대해 이야기해 봅시다.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
