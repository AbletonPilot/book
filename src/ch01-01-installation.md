## 설치

첫 번째 단계는 러스트를 설치하는 것입니다. 러스트 버전과 관련 도구를 관리하기 위한
명령줄 도구 `rustup`을 통해 러스트를 내려받겠습니다. 다운로드에는 인터넷 연결이
필요합니다.

> 참고: 어떤 이유로든 `rustup`을 사용하고 싶지 않다면, 다른 설치 방법에 대한 [Other
> Rust Installation Methods 페이지][otherinstall]를 참고하세요.

아래 단계는 러스트 컴파일러의 최신 안정(stable) 버전을 설치합니다. 러스트의 안정성
보증 덕분에, 이 책에서 컴파일되는 모든 예제는 더 새로운 러스트 버전에서도 계속
컴파일됩니다. 러스트가 에러 메시지나 경고를 지속적으로 개선하기 때문에, 버전에 따라
출력이 약간 다를 수는 있습니다. 다시 말해, 이 단계로 설치하는 러스트의 새로운 안정
버전은 이 책의 내용과 문제없이 잘 동작할 것입니다.

> ### 명령줄 표기법
>
> 이 장을 포함해 책 전반에 걸쳐, 터미널에서 사용하는 명령을 몇 가지 보여 드릴
> 것입니다. 터미널에 입력해야 하는 줄은 모두 `$`로 시작합니다. `$` 문자는 직접
> 입력할 필요가 없습니다. 각 명령의 시작을 표시하기 위한 명령줄 프롬프트일 뿐입니다.
> `$`로 시작하지 않는 줄은 보통 이전 명령의 출력 결과를 나타냅니다. 또한 파워셸
> (PowerShell) 전용 예제에서는 `$` 대신 `>`를 사용합니다.

### 리눅스 또는 맥OS에 `rustup` 설치하기

리눅스나 맥OS를 사용하신다면 터미널을 열고 다음 명령을 입력하세요.

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

이 명령은 스크립트를 내려받아 `rustup` 도구를 설치하기 시작합니다. `rustup`은 러스트의
최신 안정 버전을 함께 설치합니다. 중간에 비밀번호를 묻는 프롬프트가 뜰 수도 있습니다.
설치가 성공하면 다음과 같은 줄이 나타납니다.

```text
Rust is installed now. Great!
```

또한 *링커(linker)*도 필요합니다. 링커는 러스트가 컴파일된 결과물을 하나의 파일로
합치는 데 사용하는 프로그램입니다. 대부분 이미 링커가 설치되어 있을 것입니다. 링커
에러가 발생한다면, 링커가 함께 포함되는 경우가 많은 C 컴파일러를 설치해야 합니다.
C 컴파일러는 또 다른 이유로도 유용한데, 자주 쓰이는 러스트 패키지 중 일부는 C
코드에 의존하고 있어서 C 컴파일러가 필요하기 때문입니다.

맥OS에서는 다음 명령을 실행해서 C 컴파일러를 얻을 수 있습니다.

```console
$ xcode-select --install
```

리눅스 사용자라면 일반적으로 배포판 문서를 참고해 GCC 또는 Clang을 설치하면 됩니다.
예를 들어 우분투를 사용한다면 `build-essential` 패키지를 설치할 수 있습니다.

### 윈도우에 `rustup` 설치하기

윈도우에서는 [https://www.rust-lang.org/tools/install][install]<!-- ignore -->에
접속해 러스트 설치 안내를 따르면 됩니다. 설치 중 어느 시점에 Visual Studio를
설치하라는 프롬프트가 표시됩니다. 이는 프로그램을 컴파일하는 데 필요한 링커와
네이티브 라이브러리를 제공합니다. 이 단계에 대한 추가 도움이 필요하다면
[https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]<!--
ignore -->를 참고하세요.

이 책의 나머지 부분에서는 *cmd.exe*와 파워셸 모두에서 동작하는 명령을 사용합니다.
구체적으로 다른 점이 있다면, 어느 것을 사용해야 하는지 설명하겠습니다.

### 문제 해결

러스트가 올바르게 설치되었는지 확인하려면 셸을 열고 다음 줄을 입력하세요.

```console
$ rustc --version
```

다음과 같은 형식으로 가장 최근에 출시된 안정 버전의 버전 번호, 커밋 해시, 커밋
날짜가 보여야 합니다.

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

이 정보가 보인다면 러스트가 성공적으로 설치된 것입니다! 만약 이 정보가 보이지
않는다면, 러스트가 여러분의 `%PATH%` 시스템 변수에 등록되어 있는지 다음과 같이
확인하세요.

윈도우 CMD에서는 다음을 사용하세요.

```console
> echo %PATH%
```

파워셸에서는 다음을 사용하세요.

```powershell
> echo $env:Path
```

리눅스와 맥OS에서는 다음을 사용하세요.

```console
$ echo $PATH
```

이 모든 것이 올바른데도 러스트가 여전히 동작하지 않는다면, 도움을 받을 수 있는 곳이
여러 군데 있습니다. [커뮤니티 페이지][community]에서 다른 러스타시안(Rustacean,
러스트 사용자들이 스스로를 장난스럽게 부르는 별명)들과 연락할 방법을 찾을 수
있습니다.

### 업데이트와 제거

`rustup`을 통해 러스트를 설치했다면, 새로 출시된 버전으로의 업데이트가 쉽습니다.
셸에서 다음 업데이트 스크립트를 실행하세요.

```console
$ rustup update
```

러스트와 `rustup`을 제거하려면, 셸에서 다음 제거 스크립트를 실행하세요.

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->
<a id="local-documentation"></a>

### 로컬 문서 읽기

러스트를 설치하면 오프라인에서도 읽을 수 있도록 문서의 로컬 사본도 함께 설치됩니다.
`rustup doc`을 실행하면 브라우저에서 로컬 문서가 열립니다.

표준 라이브러리가 어떤 타입이나 함수를 제공하는지 알고 있지만 그 기능이 무엇이고
어떻게 써야 할지 확실하지 않을 때마다, API(application programming interface)
문서에서 확인해 보세요!

<!-- Old headings. Do not remove or links may break. -->
<a id="text-editors-and-integrated-development-environments"></a>

### 텍스트 에디터와 IDE 사용하기

이 책은 여러분이 러스트 코드를 작성할 때 어떤 도구를 사용하는지에 대해 별다른 가정을
하지 않습니다. 거의 어떤 텍스트 에디터든 일을 해낼 수 있습니다! 다만 많은 텍스트
에디터와 통합 개발 환경(IDE)은 러스트에 대한 기본 지원을 제공합니다. 러스트 웹사이트의
[도구 페이지][tools]에서 비교적 최신의 여러 에디터와 IDE 목록을 항상 확인할 수
있습니다.

### 이 책을 오프라인에서 다루기

몇몇 예제에서는 표준 라이브러리 외의 러스트 패키지도 사용합니다. 해당 예제를
진행하려면 인터넷에 연결되어 있거나, 미리 해당 의존성을 내려받아 두어야 합니다.
의존성을 미리 내려받으려면 다음 명령을 실행할 수 있습니다. (`cargo`가 무엇이며 이
명령들이 각각 어떤 일을 하는지는 나중에 자세히 설명하겠습니다.)

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

이렇게 하면 이 패키지들의 다운로드가 캐시되므로, 나중에 다시 내려받을 필요가
없습니다. 이 명령을 실행한 뒤에는 `get-dependencies` 폴더를 남겨 둘 필요는 없습니다.
이 명령을 실행해 두었다면, 이 책의 나머지 부분에서 모든 `cargo` 명령에 `--offline`
플래그를 사용해 네트워크를 쓰지 않고 캐시된 버전을 이용할 수 있습니다.

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
