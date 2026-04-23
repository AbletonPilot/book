<!-- Old headings. Do not remove or links may break. -->
<a id="developing-the-librarys-functionality-with-test-driven-development"></a>

## 테스트 주도 개발로 기능 추가하기

이제 검색 로직이 `main` 함수와 분리되어 _src/lib.rs_ 에 있으므로, 코드의 핵심
기능에 대한 테스트를 작성하기가 훨씬 쉬워졌습니다. 명령행에서 바이너리를
호출할 필요 없이, 다양한 인수로 함수를 직접 호출하고 반환 값을 확인할 수
있습니다.

이 절에서는 테스트 주도 개발(TDD) 프로세스를 사용해 `minigrep` 프로그램에
검색 로직을 추가하겠습니다. 다음 단계를 따릅니다.

1. 실패하는 테스트를 작성하고, 기대한 이유로 실패함을 확인하기 위해 실행한다.
2. 새 테스트를 통과시키기에 딱 충분한 만큼의 코드를 작성하거나 수정한다.
3. 방금 추가하거나 변경한 코드를 리팩터링하고, 테스트가 계속 통과하는지
   확인한다.
4. 1단계부터 반복한다!

TDD는 소프트웨어를 작성하는 여러 방법 중 하나일 뿐이지만, 코드 설계를 이끄는
데 도움이 됩니다. 테스트를 통과시키는 코드를 작성하기 전에 테스트를 먼저
작성하면, 과정 전반에 걸쳐 높은 테스트 커버리지를 유지하는 데 도움이 됩니다.

파일 내용에서 쿼리 문자열을 실제로 검색하고 쿼리와 일치하는 줄 목록을 만들
기능의 구현을 테스트로 이끌어 나가겠습니다. 이 기능은 `search`라는 함수에
추가할 것입니다.

### 실패하는 테스트 작성하기

_src/lib.rs_ 에 [11장][ch11-anatomy]<!-- ignore -->에서 한 것처럼 테스트 함수를
가진 `tests` 모듈을 추가하겠습니다. 테스트 함수는 `search` 함수가 가져야 할
동작을 명시합니다. 쿼리와 검색할 텍스트를 받아, 쿼리를 포함한 텍스트의 줄만
반환해야 합니다. Listing 12-15가 이 테스트를 보여 줍니다.

<Listing number="12-15" file-name="src/lib.rs" caption="원하는 기능에 대해 `search` 함수의 실패하는 테스트 만들기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

이 테스트는 문자열 `"duct"`를 검색합니다. 검색 대상 텍스트는 세 줄이며, 그중
`"duct"`를 포함하는 것은 하나뿐입니다(여는 큰따옴표 뒤의 백슬래시는 러스트에게
이 문자열 리터럴 내용의 시작에 개행 문자를 넣지 말라고 알려 준다는 점에
유의하세요). 우리는 `search` 함수가 반환한 값이 우리가 기대한 줄만 포함함을
단언합니다.

지금 이 테스트를 실행하면 `unimplemented!` 매크로가 “not implemented”라는
메시지와 함께 패닉하기 때문에 실패할 것입니다. TDD 원칙에 따라, 함수를 호출
해도 패닉하지 않을 만큼만 코드를 추가하는 작은 단계를 밟겠습니다. Listing
12-16처럼 `search` 함수가 항상 빈 벡터를 반환하도록 정의합니다. 그러면 테스트는
컴파일되고, 빈 벡터가 `"safe, fast, productive."` 줄을 담은 벡터와 일치하지
않기 때문에 실패해야 합니다.

<Listing number="12-16" file-name="src/lib.rs" caption="호출해도 패닉하지 않을 만큼만 `search` 함수 정의하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

이제 `search`의 시그니처에 명시적 라이프타임 `'a`를 정의하고 그 라이프타임을
`contents` 인수와 반환 값에 사용해야 하는 이유를 이야기해 봅시다.
[10장][ch10-lifetimes]<!-- ignore -->에서 라이프타임 매개변수는 어떤 인수의
라이프타임이 반환 값의 라이프타임과 연결되는지를 명시함을 떠올려 보세요. 이
경우 반환된 벡터가 (인수 `query`가 아니라) 인수 `contents` 슬라이스를 참조하는
문자열 슬라이스를 담아야 함을 나타냅니다.

다시 말해, 우리는 러스트에게 `search` 함수가 반환한 데이터가 `contents` 인수로
`search` 함수에 전달된 데이터가 살아 있는 동안 살아 있을 것이라고 말합니다.
이는 중요합니다! 참조가 유효하려면 슬라이스가 _참조하는_ 데이터가 유효해야
합니다. 컴파일러가 우리가 `contents`가 아니라 `query`의 문자열 슬라이스를 만든
다고 가정하면, 안전성 검사를 잘못 수행할 것입니다.

라이프타임 애너테이션을 잊고 이 함수를 컴파일하려 하면 다음 오류가 납니다.

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

두 매개변수 중 어느 것이 출력에 필요한지 러스트는 알 수 없으므로, 우리가
명시적으로 알려 줘야 합니다. 도움말 텍스트가 모든 매개변수와 출력 타입에 같은
라이프타임 매개변수를 지정하라고 제안하지만, 이는 올바르지 않음에 유의하세요!
`contents`는 우리의 모든 텍스트를 담은 매개변수이며, 우리는 그 텍스트 중
일치하는 부분을 반환하려 하므로, 라이프타임 문법으로 반환 값과 연결되어야 할
유일한 매개변수는 `contents`임을 압니다.

다른 프로그래밍 언어들은 시그니처에서 인수와 반환 값을 연결하라고 요구하지
않지만, 이 관행은 시간이 지남에 따라 더 쉬워질 것입니다. 이 예제를 10장의
[“라이프타임으로 참조 검증하기”][validating-references-with-lifetimes]<!-- ignore -->
절의 예제들과 비교해 보는 것도 좋습니다.

### 테스트를 통과시키는 코드 작성하기

현재 우리 테스트는 항상 빈 벡터를 반환하기 때문에 실패합니다. 이를 고치고
`search`를 구현하려면, 프로그램은 다음 단계를 따라야 합니다.

1. 내용의 각 줄을 순회한다.
2. 그 줄이 우리 쿼리 문자열을 포함하는지 확인한다.
3. 포함한다면, 반환할 값 목록에 추가한다.
4. 포함하지 않는다면 아무것도 하지 않는다.
5. 일치하는 결과 목록을 반환한다.

줄을 순회하는 것부터 각 단계를 차례로 살펴봅시다.

#### `lines` 메서드로 줄 단위 순회하기

러스트에는 편리하게도 `lines`라는 이름의, 문자열의 줄 단위 순회를 다루는
유용한 메서드가 있으며 Listing 12-17처럼 동작합니다. 이 코드는 아직 컴파일
되지 않는다는 점에 유의하세요.

<Listing number="12-17" file-name="src/lib.rs" caption="`contents`의 각 줄 순회하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

`lines` 메서드는 이터레이터를 반환합니다. 이터레이터는 [13장][ch13-iterators]<!-- ignore -->
에서 깊이 다룹니다. 하지만 [Listing 3-5][ch3-iter]<!-- ignore -->에서 컬렉션의
각 항목에 대해 어떤 코드를 실행하기 위해 `for` 루프를 이터레이터와 함께 사용
하는 방식을 이미 본 적이 있음을 떠올려 보세요.

#### 각 줄에서 쿼리 검색하기

다음으로, 현재 줄이 우리 쿼리 문자열을 포함하는지 확인합니다. 다행히 문자열에는
이를 대신 해 주는 `contains`라는 유용한 메서드가 있습니다! Listing 12-18처럼
`search` 함수에 `contains` 메서드 호출을 추가하세요. 이 코드도 아직 컴파일되지
않는다는 점에 유의하세요.

<Listing number="12-18" file-name="src/lib.rs" caption="줄이 `query`의 문자열을 포함하는지 확인하는 기능 추가하기">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

지금은 기능을 쌓아 가는 중입니다. 코드를 컴파일하려면, 함수 시그니처에서
나타낸 것처럼 본문에서 값을 반환해야 합니다.

#### 일치하는 줄 저장하기

이 함수를 완성하려면, 반환하고 싶은 일치하는 줄을 저장할 방법이 필요합니다.
이를 위해 `for` 루프 전에 가변 벡터를 만들고 `push` 메서드를 호출해 벡터에
`line`을 저장할 수 있습니다. `for` 루프 이후에는 Listing 12-19처럼 벡터를
반환합니다.

<Listing number="12-19" file-name="src/lib.rs" caption="반환하기 위해 일치하는 줄 저장하기">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

이제 `search` 함수는 `query`를 포함한 줄만 반환해야 하고, 테스트는 통과해야
합니다. 테스트를 실행해 봅시다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

테스트가 통과했으니 동작함을 알 수 있습니다!

이 시점에 같은 기능을 유지하면서 테스트가 계속 통과하도록 하면서 검색 함수의
구현을 리팩터링할 기회를 고려해 볼 수 있습니다. 검색 함수의 코드가 나쁘지는
않지만, 이터레이터의 유용한 기능을 활용하지 않고 있습니다. 이 예제는
[13장][ch13-iterators]<!-- ignore -->에서 이터레이터를 자세히 다룰 때 다시
살펴보며 어떻게 개선할지 살펴보겠습니다.

이제 전체 프로그램이 동작해야 합니다! 먼저 에밀리 디킨슨 시에서 정확히 한 줄을
반환해야 하는 단어인 _frog_ 로 실행해 봅시다.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

좋습니다! 이제 여러 줄에 일치하는 단어, 예를 들어 _body_ 로 실행해 봅시다.

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

마지막으로, 시 어디에도 없는 단어, 예를 들어 _monomorphization_ 을 검색할 때
아무 줄도 얻지 않도록 확인합시다.

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

훌륭합니다! 고전적인 도구의 우리 미니 버전을 만들었고, 애플리케이션을 구성
하는 방법에 관해 많이 배웠습니다. 파일 입출력, 라이프타임, 테스트, 명령행
파싱에 관해서도 조금 배웠습니다.

이 프로젝트를 마무리하기 위해, 환경 변수를 다루는 방법과 표준 오류에 출력하는
방법을 간략히 시연하겠습니다. 둘 다 명령행 프로그램을 작성할 때 유용합니다.

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
