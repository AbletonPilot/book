## 주석

모든 프로그래머는 코드를 이해하기 쉽게 만들기 위해 애쓰지만, 때로는 추가 설명이
필요한 경우가 있습니다. 그런 경우에 프로그래머는 소스 코드에 *주석(comment)*을
남깁니다. 주석은 컴파일러는 무시하지만, 소스 코드를 읽는 사람에게는 유용할 수
있습니다.

다음은 간단한 주석입니다.

```rust
// hello, world
```

러스트에서 관용적인 주석 스타일은 슬래시 두 개로 주석을 시작해서, 줄의 끝까지
주석이 이어지도록 하는 것입니다. 한 줄을 넘어가는 주석의 경우, 다음과 같이 각
줄에 `//`을 넣어 주어야 합니다.

```rust
// So we're doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what's going on.
```

주석은 코드를 담은 줄의 끝에도 둘 수 있습니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

그러나 더 자주 보게 되는 형식은 다음처럼, 주석이 설명하려는 코드 위에 별도의 줄로
쓰이는 형식입니다.

<span class="filename">파일명: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

러스트에는 또 다른 종류의 주석인 문서 주석(documentation comment)도 있습니다.
이는 14장의 [“Crates.io에 크레이트
게시하기”][publishing]<!-- ignore --> 절에서 다룹니다.

[publishing]: ch14-02-publishing-to-crates-io.html
