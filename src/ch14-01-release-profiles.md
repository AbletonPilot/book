## 릴리스 프로필로 빌드 커스터마이즈하기

러스트에서 _릴리스 프로필(release profiles)_ 은 미리 정의된, 커스터마이즈
가능한 프로필로, 프로그래머가 코드 컴파일에 대한 다양한 옵션을 더 잘 제어할
수 있게 해 주는 서로 다른 구성을 갖습니다. 각 프로필은 다른 프로필과 독립적
으로 구성됩니다.

카고에는 두 개의 주요 프로필이 있습니다. `cargo build`를 실행할 때 카고가
사용하는 `dev` 프로필과, `cargo build --release`를 실행할 때 카고가 사용하는
`release` 프로필입니다. `dev` 프로필은 개발에 좋은 기본값으로 정의되어 있고,
`release` 프로필은 릴리스 빌드에 좋은 기본값을 가지고 있습니다.

이 프로필 이름들은 빌드 출력에서 이미 익숙할 수도 있습니다.

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev`와 `release`는 컴파일러가 사용하는 이 서로 다른 프로필들입니다.

프로젝트의 _Cargo.toml_ 파일에 명시적으로 `[profile.*]` 섹션을 추가하지 않았을
때, 카고는 각 프로필에 대한 기본 설정을 적용합니다. 커스터마이즈하고 싶은
프로필에 대해 `[profile.*]` 섹션을 추가하면, 기본 설정의 어떤 부분이든
재정의할 수 있습니다. 예를 들어 다음은 `dev`와 `release` 프로필의 `opt-level`
설정의 기본값입니다.

<span class="filename">파일명: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` 설정은 러스트가 코드에 적용할 최적화의 수를 제어하며, 0에서 3
사이의 값을 가집니다. 더 많은 최적화를 적용하면 컴파일 시간이 늘어나므로,
개발 중이어서 자주 컴파일하는 상황이라면 결과 코드가 더 느리게 실행되더라도
더 빠르게 컴파일하기 위해 최적화를 덜 하는 것이 좋습니다. 그래서 `dev`의
기본 `opt-level`은 `0`입니다. 코드를 릴리스할 준비가 되면, 컴파일에 더 많은
시간을 쓰는 것이 가장 좋습니다. 릴리스 모드로는 한 번만 컴파일하지만, 컴파일된
프로그램은 여러 번 실행할 테니, 릴리스 모드는 긴 컴파일 시간을 코드가 더 빠르게
실행되는 것과 맞바꿉니다. 그래서 `release` 프로필의 기본 `opt-level`이 `3`인
것입니다.

_Cargo.toml_ 에 다른 값을 추가하면 기본 설정을 재정의할 수 있습니다. 예를 들어
개발 프로필에서 최적화 레벨 1을 사용하고 싶다면, 프로젝트의 _Cargo.toml_
파일에 다음 두 줄을 추가하면 됩니다.

<span class="filename">파일명: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

이 코드는 기본값 `0`을 재정의합니다. 이제 `cargo build`를 실행하면 카고는
`dev` 프로필의 기본값과 우리의 `opt-level` 커스터마이제이션을 사용합니다.
`opt-level`을 `1`로 설정했으므로 카고는 기본값보다 더 많은 최적화를 적용
하지만, 릴리스 빌드만큼은 아닙니다.

각 프로필의 구성 옵션 전체 목록과 기본값에 대해서는 [카고 문서][Cargo’s documentation]
를 참고하세요.

[Cargo’s documentation]: https://doc.rust-lang.org/cargo/reference/profiles.html
