## 해시 맵으로 키와 연관된 값 저장하기

공통 컬렉션 중 마지막은 해시 맵입니다. `HashMap<K, V>` 타입은 *해싱 함수(hashing
function)*를 사용해 `K` 타입의 키와 `V` 타입의 값의 매핑을 저장합니다. 해싱 함수는
이 키와 값을 메모리에 어떻게 배치할지 결정합니다. 많은 프로그래밍 언어가 이런
종류의 자료 구조를 지원하지만, 이름은 *해시*, *맵*, *객체(object)*, *해시
테이블(hash table)*, *딕셔너리(dictionary)*, *연관 배열(associative array)* 등
여러 가지를 사용하는 경우가 많습니다.

해시 맵은 벡터처럼 인덱스가 아니라 어떤 타입이든 될 수 있는 키로 데이터를 조회하고
싶을 때 유용합니다. 예를 들어, 게임에서 각 키가 팀 이름이고 값이 각 팀의 점수인
해시 맵으로 각 팀의 점수를 추적할 수 있습니다. 팀 이름이 주어지면 그 팀의 점수를
조회할 수 있죠.

이 절에서는 해시 맵의 기본 API를 살펴보지만, 표준 라이브러리가 `HashMap<K, V>`에
정의한 함수 안에 훨씬 더 많은 좋은 것들이 숨겨져 있습니다. 언제나처럼, 더 많은
정보는 표준 라이브러리 문서를 확인하세요.

### 새 해시 맵 만들기

비어 있는 해시 맵을 만드는 한 가지 방법은 `new`를 사용하고 `insert`로 요소를
추가하는 것입니다. Listing 8-20에서는 이름이 각각 *Blue*와 *Yellow*인 두 팀의
점수를 추적합니다. Blue 팀은 10점으로 시작하고, Yellow 팀은 50점으로 시작합니다.

<Listing number="8-20" caption="새 해시 맵을 만들고 일부 키와 값을 삽입하기">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

먼저 표준 라이브러리의 컬렉션 부분에서 `HashMap`을 `use`해야 한다는 점에 유의
하세요. 세 가지 공통 컬렉션 중 이 타입이 가장 적게 쓰이므로, 프렐류드에서 자동으로
스코프로 가져오는 기능에는 포함되어 있지 않습니다. 또한 해시 맵은 표준 라이브러리
의 지원도 덜 받습니다. 예를 들어, 이를 만드는 내장 매크로는 없습니다.

벡터와 마찬가지로 해시 맵은 데이터를 힙에 저장합니다. 이 `HashMap`은 `String` 타입의
키와 `i32` 타입의 값을 가집니다. 벡터처럼 해시 맵도 동종(homogeneous)입니다.
모든 키는 같은 타입이어야 하고, 모든 값도 같은 타입이어야 합니다.

### 해시 맵의 값에 접근하기

Listing 8-21에 보이듯, 해시 맵에 키를 전달해 `get` 메서드로 값을 가져올 수
있습니다.

<Listing number="8-21" caption="해시 맵에 저장된 Blue 팀의 점수에 접근하기">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

여기서 `score`는 Blue 팀과 연관된 값을 가지고, 결과는 `10`이 됩니다. `get` 메서드는
`Option<&V>`를 반환합니다. 해시 맵에 그 키에 대한 값이 없다면 `get`은 `None`을
반환합니다. 이 프로그램은 `Option<&i32>` 대신 `Option<i32>`를 얻기 위해 `copied`
를 호출하고, 그 뒤 `scores`에 그 키에 대한 엔트리가 없으면 `score`를 0으로 설정
하기 위해 `unwrap_or`를 호출함으로써 이 `Option`을 처리합니다.

해시 맵의 각 키-값 쌍은, `for` 루프를 사용해 벡터와 비슷한 방식으로 순회할 수
있습니다.

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

이 코드는 임의의 순서로 각 쌍을 출력합니다.

```text
Yellow: 50
Blue: 10
```

<!-- Old headings. Do not remove or links may break. -->

<a id="hash-maps-and-ownership"></a>

### 해시 맵의 소유권 관리하기

`i32`처럼 `Copy` 트레이트를 구현하는 타입에서는 값이 해시 맵 안으로 복사됩니다.
`String`처럼 소유된 값은 값이 이동하고 해시 맵이 그 값들의 소유자가 됩니다.
Listing 8-22에서 이를 시연했습니다.

<Listing number="8-22" caption="삽입 이후 키와 값이 해시 맵에 의해 소유됨을 보여 주기">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

`insert` 호출로 해시 맵에 이동된 이후에는, `field_name`과 `field_value` 변수를
사용할 수 없습니다.

값에 대한 참조를 해시 맵에 삽입하면, 값은 해시 맵으로 이동되지 않습니다. 참조가
가리키는 값은 적어도 해시 맵이 유효한 동안은 유효해야 합니다. 이 문제들에 대해서는
10장의 [“라이프타임으로 참조 유효성
검증하기”][validating-references-with-lifetimes]<!-- ignore -->에서 더 이야기
하겠습니다.

### 해시 맵 갱신하기

키와 값 쌍의 개수는 자라날 수 있지만, 각 고유한 키는 한 번에 하나의 값만 가질 수
있습니다(그 반대는 아닙니다. 예를 들어 Blue 팀과 Yellow 팀 모두 `scores` 해시 맵에
`10` 값을 저장할 수 있습니다).

해시 맵의 데이터를 바꾸고 싶을 때, 이미 키에 값이 할당되어 있는 경우를 어떻게
처리할지 결정해야 합니다. 기존 값을 새 값으로 대체할 수도 있고, 기존 값을 완전히
무시할 수도 있습니다. 기존 값을 유지하고 새 값을 무시할 수도 있고, 키에 값이 아직
*없는* 경우에만 새 값을 추가할 수도 있습니다. 또는 기존 값과 새 값을 결합할 수도
있습니다. 이들 각각을 어떻게 하는지 살펴봅시다!

#### 값 덮어쓰기

키와 값을 해시 맵에 삽입한 뒤 같은 키를 다른 값으로 삽입하면, 그 키에 연관된 값은
교체됩니다. Listing 8-23의 코드가 `insert`를 두 번 호출하더라도, Blue 팀의 키에
대해 두 번 모두 값을 삽입하기 때문에, 해시 맵은 하나의 키-값 쌍만 담게 됩니다.

<Listing number="8-23" caption="특정 키에 저장된 값 교체하기">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

이 코드는 `{"Blue": 25}`를 출력합니다. 원래 값 `10`이 덮어써졌습니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### 키가 없을 때에만 키와 값 추가하기

해시 맵에 특정 키가 값과 함께 이미 존재하는지 확인한 뒤 다음과 같은 동작을 취하는
일은 흔합니다. 키가 해시 맵에 존재한다면 기존 값을 그대로 유지하고, 키가 없다면
그 키와 그에 대한 값을 삽입하는 것입니다.

해시 맵에는 이를 위한 특별한 API인 `entry`가 있으며, 확인하려는 키를 매개변수로
받습니다. `entry` 메서드의 반환값은 존재할 수도 있고 존재하지 않을 수도 있는
값을 나타내는 `Entry`라는 열거형입니다. Yellow 팀 키에 값이 연관되어 있는지
확인하고, 없다면 값 `50`을 삽입하고, Blue 팀에 대해서도 똑같이 하고 싶다고 해
봅시다. `entry` API를 사용하면 코드는 Listing 8-24처럼 보입니다.

<Listing number="8-24" caption="키에 아직 값이 없는 경우에만 삽입하기 위해 `entry` 메서드 사용하기">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

`Entry`의 `or_insert` 메서드는, 대응하는 `Entry` 키가 존재한다면 그 키에 해당하는
값에 대한 가변 참조를 반환하고, 그렇지 않다면 매개변수를 이 키의 새 값으로 삽입
하고 그 새 값에 대한 가변 참조를 반환하도록 정의되어 있습니다. 이 기법은 우리가
직접 로직을 작성하는 것보다 훨씬 깔끔하고, 덤으로 빌림 검사기와도 더 잘 어울립
니다.

Listing 8-24의 코드를 실행하면 `{"Yellow": 50, "Blue": 10}`을 출력합니다. `entry`
의 첫 호출은 Yellow 팀에 아직 값이 없으므로 Yellow 팀의 키를 값 `50`과 함께
삽입합니다. `entry`의 두 번째 호출은 Blue 팀이 이미 값 `10`을 가지고 있으므로 해시
맵을 바꾸지 않습니다.

#### 기존 값을 기반으로 값 갱신하기

해시 맵의 또 다른 흔한 사용 사례는, 키의 값을 조회한 뒤 기존 값을 기반으로 값을
갱신하는 것입니다. 예를 들어 Listing 8-25는 어떤 텍스트에서 각 단어가 몇 번 등장
하는지 세는 코드를 보여 줍니다. 단어를 키로 하는 해시 맵을 사용하고, 그 단어를
본 횟수를 추적하기 위해 값을 증가시킵니다. 어떤 단어를 처음 본 것이라면, 먼저 값
`0`을 삽입합니다.

<Listing number="8-25" caption="단어와 횟수를 저장하는 해시 맵을 사용해 단어의 등장 횟수 세기">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

이 코드는 `{"world": 2, "hello": 1, "wonderful": 1}`를 출력합니다. 같은 키-값 쌍이
다른 순서로 출력되는 것을 볼 수도 있습니다. [“해시 맵의 값에 접근하기”][access]
<!-- ignore -->에서 해시 맵을 순회하는 것은 임의의 순서로 일어난다는 점을
떠올리세요.

`split_whitespace` 메서드는 `text`의 값을 공백으로 구분한 서브슬라이스에 대한
이터레이터를 반환합니다. `or_insert` 메서드는 지정한 키에 대한 값에 대한 가변
참조(`&mut V`)를 반환합니다. 여기서는 그 가변 참조를 `count` 변수에 저장하므로,
그 값을 할당하려면 먼저 `count`를 별표(`*`)를 사용해 역참조해야 합니다. 가변
참조는 `for` 루프의 끝에서 스코프를 벗어나므로, 이 모든 변경은 안전하고 빌림
규칙에 의해 허용됩니다.

### 해싱 함수

기본적으로 `HashMap`은 해시 테이블을 관련된 서비스 거부(DoS) 공격에 대한 내성을
제공할 수 있는 *SipHash*라는 해싱 함수를 사용합니다[^siphash]<!-- ignore -->.
이는 사용 가능한 가장 빠른 해싱 알고리즘은 아니지만, 성능 하락과 함께 오는 더 나은
보안의 절충은 그럴 만한 가치가 있습니다. 여러분의 코드를 프로파일링해서 기본 해시
함수가 여러분의 용도에 너무 느리다면, 다른 해셔(hasher)를 지정해 다른 함수로 전환
할 수 있습니다. *해셔*는 `BuildHasher` 트레이트를 구현하는 타입입니다. 트레이트
와 구현 방법은 [10장][traits]<!-- ignore -->에서 이야기하겠습니다. 여러분이 직접
해셔를 바닥부터 구현해야 하는 것은 아닙니다. [crates.io](https://crates.io/)
<!-- ignore -->에 다른 러스트 사용자들이 공유한, 많은 흔한 해싱 알고리즘을 구현
하는 해셔를 제공하는 라이브러리들이 있습니다.

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## 요약

벡터, 문자열, 해시 맵은 데이터를 저장하고 접근하고 수정해야 할 때 프로그램에서
필요한 많은 기능을 제공합니다. 다음은 이제 여러분이 풀 수 있어야 하는 연습 문제
입니다.

1. 정수 목록이 주어졌을 때, 벡터를 사용해 그 목록의 중앙값(정렬했을 때 가운데
   위치의 값)과 최빈값(가장 자주 등장하는 값. 여기서는 해시 맵이 도움이 될
   것입니다)을 반환하세요.
1. 문자열을 Pig Latin으로 변환하세요. 각 단어의 첫 자음을 단어의 끝으로 옮기고
   *ay*를 추가하므로, *first*는 *irst-fay*가 됩니다. 모음으로 시작하는 단어에는
   대신 *hay*를 끝에 추가합니다(*apple*은 *apple-hay*가 됩니다). UTF-8 인코딩에
   대한 세부 사항을 염두에 두세요!
1. 해시 맵과 벡터를 사용해, 사용자가 회사의 부서에 직원 이름을 추가할 수 있도록
   하는 텍스트 인터페이스를 만드세요. 예를 들어, “Add Sally to Engineering” 또는
   “Add Amir to Sales” 같은 방식입니다. 그런 다음 사용자가 부서의 모든 사람 목록
   또는 회사의 모든 사람 목록을 부서별로, 알파벳순으로 정렬해 조회할 수 있게
   하세요.

표준 라이브러리 API 문서는 이 연습 문제에 도움이 될 벡터, 문자열, 해시 맵의
메서드들을 설명합니다!

연산이 실패할 수 있는 더 복잡한 프로그램으로 들어가고 있으니, 에러 처리에 대해
이야기하기에 완벽한 시점입니다. 다음으로 그것을 하겠습니다!

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
