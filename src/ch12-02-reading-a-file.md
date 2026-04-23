## 파일 읽기

이제 `file_path` 인수로 지정된 파일을 읽는 기능을 추가하겠습니다. 먼저 테스트에
사용할 샘플 파일이 필요합니다. 여러 줄에 걸쳐 적은 양의 텍스트가 있고 반복되는
단어가 몇 개 있는 파일을 사용하겠습니다. Listing 12-3에 잘 어울리는 에밀리
디킨슨(Emily Dickinson)의 시가 있습니다! 프로젝트 루트 레벨에 _poem.txt_ 라는
파일을 만들고, “I’m Nobody! Who are you?” 시를 입력하세요.

<Listing number="12-3" file-name="poem.txt" caption="에밀리 디킨슨의 시는 좋은 테스트 사례가 됩니다.">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

텍스트가 준비되었으면 _src/main.rs_ 를 편집해서, Listing 12-4처럼 파일을 읽는
코드를 추가하세요.

<Listing number="12-4" file-name="src/main.rs" caption="두 번째 인수로 지정된 파일의 내용 읽기">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

먼저 `use` 구문으로 표준 라이브러리의 관련 부분을 가져옵니다. 파일을 다루려면
`std::fs`가 필요합니다.

`main`에서 새 구문 `fs::read_to_string`은 `file_path`를 받아 그 파일을 열고,
파일의 내용을 담은 `std::io::Result<String>` 타입의 값을 반환합니다.

그 후, 파일을 읽은 뒤 `contents`의 값을 출력하는 임시 `println!` 구문을 다시
추가해, 지금까지 프로그램이 동작하는지 확인할 수 있게 합니다.

이 코드를 첫 번째 명령행 인수로는 임의의 문자열을(검색 부분은 아직 구현하지
않았으므로) 주고, 두 번째 인수로는 _poem.txt_ 파일을 줘서 실행해 봅시다.

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

좋습니다! 코드가 파일을 읽고 그 내용을 출력했습니다. 하지만 이 코드에는 몇
가지 결함이 있습니다. 지금 `main` 함수는 여러 책임을 지고 있습니다. 일반적으로
함수는 각 함수가 하나의 아이디어에만 책임을 질 때 더 명확하고 유지보수하기
쉽습니다. 또 다른 문제는 오류 처리를 충분히 잘하지 못하고 있다는 점입니다.
프로그램이 아직 작으므로 이 결함들이 큰 문제는 아니지만, 프로그램이 커질수록
깔끔하게 고치기가 더 어려워질 것입니다. 프로그램을 개발할 때 일찍부터 리팩터링을
시작하는 것은 좋은 습관인데, 코드의 양이 적을 때 리팩터링하는 편이 훨씬 쉽기
때문입니다. 다음으로 그것을 해 보겠습니다.
