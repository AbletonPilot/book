<!-- Old headings. Do not remove or links may break. -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## `cargo install`로 바이너리 설치하기

`cargo install` 명령은 바이너리 크레이트를 로컬에 설치해 사용할 수 있게 해
줍니다. 이는 시스템 패키지를 대체하려는 것이 아니라, 러스트 개발자가 다른
사람들이 [crates.io](https://crates.io/)<!-- ignore -->에 공유한 도구를
설치하는 편리한 방법으로 의도된 것입니다. 바이너리 타겟이 있는 패키지만 설치
할 수 있음에 유의하세요. _바이너리 타겟(binary target)_ 은 크레이트에
_src/main.rs_ 파일이 있거나 바이너리로 지정된 다른 파일이 있으면 생성되는,
실행 가능한 프로그램입니다. 이는 단독으로 실행 가능하지 않지만 다른 프로그램
에 포함되기에 적합한 라이브러리 타겟과는 반대입니다. 보통 크레이트가 라이
브러리인지, 바이너리 타겟이 있는지, 아니면 둘 다인지에 대한 정보는 README
파일에 있습니다.

`cargo install`로 설치되는 모든 바이너리는 설치 루트의 _bin_ 폴더에 저장됩니다.
러스트를 _rustup.rs_ 로 설치했고 커스텀 구성이 없다면 이 디렉터리는
*$HOME/.cargo/bin*이 됩니다. 이 디렉터리가 `$PATH`에 있어야 `cargo install`로
설치한 프로그램을 실행할 수 있습니다.

예를 들어 12장에서 파일 검색용 `grep` 도구의 러스트 구현체인 `ripgrep`에 대해
언급했습니다. `ripgrep`을 설치하려면 다음을 실행할 수 있습니다.

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

출력의 끝에서 두 번째 줄은 설치된 바이너리의 위치와 이름을 보여 주는데,
`ripgrep`의 경우 `rg`입니다. 앞서 언급한 것처럼 설치 디렉터리가 `$PATH`에
있기만 하면, `rg --help`를 실행해 더 빠르고 더 러스트다운 파일 검색 도구를
사용하기 시작할 수 있습니다!
