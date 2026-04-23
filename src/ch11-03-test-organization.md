## 테스트 구성

장 시작에서 언급했듯이, 테스트는 복잡한 분야이고 사람마다 쓰는 용어와 구성
방식이 다릅니다. 러스트 커뮤니티는 테스트를 크게 두 범주로 생각합니다. 단위
테스트와 통합 테스트입니다. _단위 테스트(unit tests)_ 는 작고 더 집중되어 있어,
한 번에 한 모듈을 격리된 상태로 테스트하며 비공개 인터페이스도 테스트할 수
있습니다. _통합 테스트(integration tests)_ 는 라이브러리 외부에 완전히 있으며,
다른 외부 코드가 사용하는 것과 동일한 방식으로 공개 인터페이스만 사용하고
테스트 하나에서 여러 모듈을 실행할 수 있습니다.

라이브러리의 구성 요소들이 각각 또는 함께 여러분이 기대하는 대로 동작하는지
확인하려면 두 종류의 테스트를 모두 작성하는 것이 중요합니다.

### 단위 테스트

단위 테스트의 목적은 코드의 각 단위를 코드의 나머지 부분과 격리해 테스트하여,
코드가 기대대로 동작하는 곳과 그렇지 않은 곳을 빠르게 짚어 내는 것입니다. 단위
테스트는 테스트 대상 코드와 같은 파일의 _src_ 디렉터리에 둡니다. 관례적으로는
각 파일에 `tests`라는 모듈을 만들어 테스트 함수들을 담고, 그 모듈에 `cfg(test)`
애너테이션을 붙입니다.

#### `tests` 모듈과 `#[cfg(test)]`

`tests` 모듈의 `#[cfg(test)]` 애너테이션은 러스트에게 `cargo test`를 실행할
때만 테스트 코드를 컴파일하고 실행하도록, `cargo build`를 실행할 때는 그러지
말도록 지시합니다. 라이브러리만 빌드하고 싶을 때 컴파일 시간을 절약해 주며,
테스트가 포함되지 않으므로 결과 컴파일 산출물의 공간도 절약됩니다. 통합 테스트는
다른 디렉터리에 두기 때문에 `#[cfg(test)]` 애너테이션이 필요 없음을 알게 될
것입니다. 그러나 단위 테스트는 코드와 같은 파일에 두기 때문에, 컴파일 결과에
포함되지 말아야 함을 명시하기 위해 `#[cfg(test)]`를 사용합니다.

이 장의 첫 절에서 새 `adder` 프로젝트를 생성했을 때 카고가 우리에게 다음 코드를
생성해 줬음을 떠올려 보세요.

<span class="filename">파일명: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

자동 생성된 `tests` 모듈에서 `cfg` 속성은 _configuration(구성)_ 을 의미하며,
러스트에게 특정 구성 옵션이 주어졌을 때만 그다음 항목이 포함되어야 함을 알려
줍니다. 이 경우 구성 옵션은 `test`로, 러스트가 테스트를 컴파일하고 실행하기
위해 제공합니다. `cfg` 속성을 사용하면, 우리가 `cargo test`로 적극적으로 테스트를
실행할 때만 카고가 우리 테스트 코드를 컴파일합니다. 이는 `#[test]`가 붙은 함수
뿐 아니라 이 모듈 안에 있을 수 있는 헬퍼 함수도 포함합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-private-functions"></a>

#### 비공개 함수 테스트하기

비공개 함수를 직접 테스트해야 하는지 여부에 대해 테스트 커뮤니티 내에서 논쟁이
있으며, 다른 언어들은 비공개 함수를 테스트하는 것을 어렵거나 불가능하게 만듭니다.
어떤 테스트 이념을 따르든, 러스트의 비공개 규칙은 비공개 함수 테스트를 허용
합니다. Listing 11-12에서 비공개 함수 `internal_adder`가 있는 코드를 살펴봅시다.

<Listing number="11-12" file-name="src/lib.rs" caption="비공개 함수 테스트하기">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

`internal_adder` 함수가 `pub`으로 표시되지 않았다는 점에 유의하세요. 테스트도
그냥 러스트 코드이고, `tests` 모듈도 또 다른 모듈일 뿐입니다. [“모듈 트리에서
항목을 참조하는 경로”][paths]<!-- ignore --> 절에서 다뤘듯이, 자식 모듈의 항목은
조상 모듈의 항목을 사용할 수 있습니다. 이 테스트에서는 `use super::*`로 `tests`
모듈 부모의 모든 항목을 스코프로 가져오고, 그러면 테스트는 `internal_adder`를
호출할 수 있습니다. 비공개 함수를 테스트해서는 안 된다고 생각한다면, 러스트에는
그것을 강제하는 무언가가 없습니다.

### 통합 테스트

러스트에서 통합 테스트는 라이브러리 외부에 완전히 있습니다. 다른 코드가
라이브러리를 사용하는 것과 동일한 방식으로 라이브러리를 사용하므로, 라이브러리의
공개 API에 속한 함수만 호출할 수 있습니다. 통합 테스트의 목적은 라이브러리의
여러 부분이 함께 올바르게 동작하는지 테스트하는 것입니다. 각자 개별로는 올바르게
동작하는 코드 단위들이 통합 시 문제를 가질 수 있으므로, 통합된 코드의 테스트
커버리지도 중요합니다. 통합 테스트를 만들려면 먼저 _tests_ 디렉터리가 필요합니다.

#### _tests_ 디렉터리

프로젝트 디렉터리의 최상위 레벨, _src_ 옆에 _tests_ 디렉터리를 만듭니다. 카고는
이 디렉터리에서 통합 테스트 파일을 찾아야 함을 알고 있습니다. 그 다음 원하는
만큼 많은 테스트 파일을 만들 수 있고, 카고는 각 파일을 개별 크레이트로 컴파일
합니다.

통합 테스트를 하나 만들어 봅시다. _src/lib.rs_ 파일에 Listing 11-12의 코드가
그대로 있는 상태에서, _tests_ 디렉터리를 만들고 _tests/integration_test.rs_
라는 새 파일을 만드세요. 디렉터리 구조는 다음과 같을 것입니다.

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

_tests/integration_test.rs_ 파일에 Listing 11-13의 코드를 입력하세요.

<Listing number="11-13" file-name="tests/integration_test.rs" caption="`adder` 크레이트의 함수에 대한 통합 테스트">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

_tests_ 디렉터리의 각 파일은 개별 크레이트이므로, 각 테스트 크레이트의 스코프에
우리 라이브러리를 가져와야 합니다. 그래서 코드 맨 위에 `use adder::add_two;`를
추가했는데, 이는 단위 테스트에서는 필요하지 않았습니다.

_tests/integration_test.rs_ 의 어떤 코드에도 `#[cfg(test)]`를 붙일 필요는
없습니다. 카고는 _tests_ 디렉터리를 특별하게 취급하여 `cargo test`를 실행할
때만 이 디렉터리의 파일을 컴파일합니다. 이제 `cargo test`를 실행해 보세요.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

출력의 세 섹션은 단위 테스트, 통합 테스트, 문서 테스트를 포함합니다. 어느
섹션에서든 테스트가 실패하면 이후 섹션은 실행되지 않음에 유의하세요. 예를
들어 단위 테스트가 실패하면 통합 테스트와 문서 테스트에 대한 출력은 없게
되는데, 모든 단위 테스트가 통과할 때만 그 테스트들이 실행되기 때문입니다.

단위 테스트의 첫 번째 섹션은 지금까지 본 것과 같습니다. 각 단위 테스트(여기서는
Listing 11-12에서 추가한 `internal`이라는 테스트)마다 한 줄이 있고, 단위 테스트
요약 줄이 이어집니다.

통합 테스트 섹션은 `Running tests/integration_test.rs` 줄로 시작합니다. 그
다음, 해당 통합 테스트 안의 각 테스트 함수마다 줄이 있고, `Doc-tests adder`
섹션이 시작되기 직전에 통합 테스트 결과의 요약 줄이 있습니다.

각 통합 테스트 파일에는 자체 섹션이 있으므로, _tests_ 디렉터리에 파일을 더
추가하면 통합 테스트 섹션이 더 많아집니다.

여전히 `cargo test`의 인수로 테스트 함수 이름을 지정해 특정 통합 테스트 함수를
실행할 수 있습니다. 특정 통합 테스트 파일의 모든 테스트를 실행하려면,
`cargo test`에 `--test` 인수를 파일 이름과 함께 사용하세요.

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

이 명령은 _tests/integration_test.rs_ 파일의 테스트만 실행합니다.

#### 통합 테스트의 하위 모듈

통합 테스트를 더 추가하면서, 정리를 돕기 위해 _tests_ 디렉터리에 파일을 더
만들고 싶을 수 있습니다. 예를 들어 테스트 함수를 테스트하는 기능별로 묶을 수
있습니다. 앞서 언급했듯이 _tests_ 디렉터리의 각 파일은 개별 크레이트로 컴파일
되므로, 최종 사용자가 크레이트를 사용하는 방식을 더 잘 흉내 내기 위해 분리된
스코프를 만드는 데 유용합니다. 그러나 이는 _tests_ 디렉터리의 파일이 _src_의
파일과 같은 동작을 공유하지 않는다는 의미입니다. 코드를 모듈과 파일로 분리하는
방법은 7장에서 배운 대로입니다.

_tests_ 디렉터리 파일의 이러한 다른 동작은, 여러 통합 테스트 파일에서 사용할
헬퍼 함수 집합이 있어서 7장의 [“모듈을 여러 파일로 분리하기”][separating-modules-into-files]<!-- ignore -->
절의 단계를 따라 공통 모듈로 추출하려 할 때 가장 두드러집니다. 예를 들어
_tests/common.rs_를 만들고 `setup`이라는 함수를 두면, 여러 테스트 파일의
여러 테스트 함수에서 호출하려는 어떤 코드를 `setup`에 추가할 수 있습니다.

<span class="filename">파일명: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

다시 테스트를 실행하면, 이 파일에 테스트 함수가 없고 `setup` 함수를 어디에서도
호출하지 않았음에도 테스트 출력에 _common.rs_ 파일에 대한 새 섹션이 보입니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

`common`이 테스트 결과에 나타나고 `running 0 tests`가 표시되는 것은 우리가
원한 바가 아닙니다. 우리는 단지 다른 통합 테스트 파일과 코드 일부를 공유하고
싶었을 뿐입니다. `common`이 테스트 출력에 나타나지 않도록 하려면,
_tests/common.rs_를 만드는 대신 _tests/common/mod.rs_를 만듭니다. 프로젝트
디렉터리는 이제 다음과 같이 보입니다.

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

이것은 7장의 [“대체 파일 경로”][alt-paths]<!-- ignore -->에서 언급한, 러스트가
이해하는 옛날 명명 관례입니다. 파일의 이름을 이렇게 지으면 러스트는 `common`
모듈을 통합 테스트 파일로 취급하지 않게 됩니다. `setup` 함수 코드를
_tests/common/mod.rs_로 옮기고 _tests/common.rs_ 파일을 삭제하면, 그 섹션이
테스트 출력에 더 이상 나타나지 않게 됩니다. _tests_ 디렉터리의 하위 디렉터리에
있는 파일은 개별 크레이트로 컴파일되지 않으며 테스트 출력에 섹션을 갖지
않습니다.

_tests/common/mod.rs_를 만든 뒤에는, 이를 모듈로 삼아 어느 통합 테스트
파일에서든 사용할 수 있습니다. 다음은 _tests/integration_test.rs_ 의
`it_adds_two` 테스트에서 `setup` 함수를 호출하는 예입니다.

<span class="filename">파일명: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

`mod common;` 선언이 Listing 7-21에서 시연한 모듈 선언과 같음에 유의하세요.
그런 다음 테스트 함수 안에서 `common::setup()` 함수를 호출할 수 있습니다.

#### 바이너리 크레이트의 통합 테스트

프로젝트가 _src/main.rs_ 파일만 있고 _src/lib.rs_ 파일은 없는 바이너리 크레이트
라면, _tests_ 디렉터리에 통합 테스트를 만들어 _src/main.rs_ 파일에 정의된 함수를
`use` 구문으로 스코프에 가져올 수 없습니다. 라이브러리 크레이트만이 다른
크레이트가 사용할 수 있는 함수를 노출하며, 바이너리 크레이트는 단독으로 실행
되도록 만들어졌기 때문입니다.

바이너리를 제공하는 러스트 프로젝트가 _src/lib.rs_ 파일에 사는 로직을 호출하는
단순한 _src/main.rs_ 파일을 갖는 이유 중 하나가 바로 이것입니다. 그런 구조를
사용하면, 통합 테스트가 `use`로 중요한 기능을 사용해 라이브러리 크레이트를
테스트할 _수_ 있습니다. 중요한 기능이 동작한다면, _src/main.rs_ 파일에 있는
소량의 코드도 동작할 것이며, 그 소량의 코드는 테스트할 필요가 없습니다.

## 요약

러스트의 테스트 기능은 코드가 어떻게 동작해야 하는지 명시하는 방법을 제공해,
여러분이 변경을 가하더라도 코드가 계속 기대한 대로 동작하도록 보장해 줍니다.
단위 테스트는 라이브러리의 서로 다른 부분을 따로따로 연습하며, 비공개 구현
세부 사항도 테스트할 수 있습니다. 통합 테스트는 라이브러리의 여러 부분이 함께
올바르게 동작하는지 검사하며, 외부 코드가 코드를 사용하는 방식과 동일하게
라이브러리의 공개 API를 사용합니다. 러스트의 타입 시스템과 소유권 규칙이 일부
버그를 예방해 주지만, 여러분의 코드가 어떻게 동작해야 하는지와 관련된 논리
버그를 줄이려면 여전히 테스트가 중요합니다.

이 장과 이전 장에서 배운 지식을 결합해 프로젝트를 진행해 봅시다!

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
