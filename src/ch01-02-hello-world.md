## Hello, World!

러스트를 설치했으니 이제 첫 러스트 프로그램을 작성할 차례입니다. 새 언어를 배울 때는
화면에 `Hello, world!`라는 글자를 출력하는 작은 프로그램을 만드는 것이 전통이므로,
우리도 똑같이 해 보겠습니다!

> 참고: 이 책은 명령줄 사용에 대한 기본적인 친숙함을 전제로 합니다. 러스트는
> 에디터나 도구, 코드의 위치에 대해 어떤 특별한 요구 사항도 두지 않기 때문에,
> 명령줄보다 IDE를 선호한다면 좋아하는 IDE를 편하게 사용해도 됩니다. 현재 많은
> IDE가 일정 수준의 러스트 지원을 제공하니 자세한 내용은 IDE 문서를 확인해 보세요.
> 러스트 팀은 `rust-analyzer`를 통해 뛰어난 IDE 지원을 제공하는 데 집중해
> 왔습니다. 자세한 내용은 [부록 D][devtools]<!-- ignore -->를 참고하세요.

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-a-project-directory"></a>

### 프로젝트 디렉터리 설정

먼저 러스트 코드를 보관할 디렉터리를 만드는 것부터 시작하겠습니다. 러스트 입장에서
코드가 어디에 있든 상관없지만, 이 책의 연습 문제와 프로젝트를 위해서는 홈 디렉터리에
*projects* 디렉터리를 만들어 두고 모든 프로젝트를 그 안에 보관하는 것을 권장합니다.

터미널을 열고 다음 명령을 입력해 *projects* 디렉터리와 그 안에 “Hello, world!”
프로젝트용 디렉터리를 만드세요.

리눅스, 맥OS, 그리고 윈도우의 파워셸에서는 다음과 같이 입력합니다.

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

윈도우 CMD에서는 다음과 같이 입력합니다.

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-and-running-a-rust-program"></a>

### 러스트 프로그램 기본

다음으로 새 소스 파일을 만들고 이름을 *main.rs*로 지정합니다. 러스트 파일은 항상
*.rs* 확장자로 끝납니다. 파일 이름에 여러 단어를 사용할 경우, 관례적으로 밑줄(`_`)로
단어를 구분합니다. 예를 들어 *helloworld.rs*보다 *hello_world.rs*를 사용하세요.

방금 만든 *main.rs* 파일을 열고 Listing 1-1의 코드를 입력하세요.

<Listing number="1-1" file-name="main.rs" caption="`Hello, world!`를 출력하는 프로그램">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

파일을 저장한 뒤, *~/projects/hello_world* 디렉터리에 있는 터미널 창으로 돌아오세요.
리눅스나 맥OS에서는 다음 명령을 입력해 파일을 컴파일하고 실행합니다.

```console
$ rustc main.rs
$ ./main
Hello, world!
```

윈도우에서는 `./main` 대신 `.\main` 명령을 입력합니다.

```powershell
> rustc main.rs
> .\main
Hello, world!
```

운영체제가 무엇이든 `Hello, world!`라는 문자열이 터미널에 출력되어야 합니다. 이 출력이
보이지 않는다면, 도움을 받을 수 있는 방법에 대해 설치 절의 [“문제
해결”][troubleshooting]<!-- ignore --> 부분을 다시 확인해 보세요.

`Hello, world!`가 출력됐다면 축하드립니다! 여러분은 공식적으로 러스트 프로그램을
작성한 것입니다. 이제 여러분은 러스트 프로그래머입니다. 환영합니다!

<!-- Old headings. Do not remove or links may break. -->

<a id="anatomy-of-a-rust-program"></a>

### 러스트 프로그램의 해부학

방금 만든 “Hello, world!” 프로그램을 자세히 살펴봅시다. 퍼즐의 첫 번째 조각은
다음과 같습니다.

```rust
fn main() {

}
```

이 몇 줄은 `main`이라는 이름의 함수를 정의합니다. `main` 함수는 특별합니다. 모든
실행 가능한 러스트 프로그램에서 항상 가장 먼저 실행되는 코드가 바로 `main`
함수입니다. 여기서 첫 번째 줄은 매개변수가 없고 아무것도 반환하지 않는 `main`이라는
이름의 함수를 선언합니다. 만약 매개변수가 있었다면, 괄호(`()`) 안에 들어갔을
것입니다.

함수 본문은 `{}`로 감쌉니다. 러스트는 모든 함수 본문에 중괄호를 요구합니다. 여는
중괄호를 함수 선언과 같은 줄에 두고 그 사이에 한 칸의 공백을 넣는 것이 좋은
스타일입니다.

> 참고: 러스트 프로젝트 전반에 걸쳐 표준 스타일을 유지하고 싶다면, `rustfmt`라는
> 자동 포매터 도구로 코드를 특정 스타일에 맞게 포맷할 수 있습니다(`rustfmt`에 대한
> 자세한 내용은 [부록 D][devtools]<!-- ignore --> 참고). 러스트 팀은 `rustc`처럼
> 이 도구도 표준 러스트 배포에 포함시켜 놓았으므로, 여러분의 컴퓨터에 이미 설치되어
> 있을 것입니다!

`main` 함수의 본문에는 다음 코드가 들어 있습니다.

```rust
println!("Hello, world!");
```

이 한 줄이 이 작은 프로그램에서 모든 일을 합니다. 화면에 텍스트를 출력하는 것이죠.
여기서 주목해야 할 중요한 세부 사항이 세 가지 있습니다.

첫 번째로, `println!`은 러스트의 매크로를 호출합니다. 만약 함수를 호출하는 것이었다면
`println`(`!` 없이)으로 적었을 것입니다. 러스트 매크로는 러스트 문법을 확장하기 위해
코드를 생성하는 코드를 작성하는 방법이며, [20장][ch20-macros]<!-- ignore -->에서 좀
더 자세히 다룹니다. 지금은 `!`를 사용하면 일반 함수가 아니라 매크로를 호출하는
것이며, 매크로는 항상 함수와 동일한 규칙을 따르는 것은 아니라는 점만 알아 두면
됩니다.

두 번째로, `"Hello, world!"` 문자열이 보입니다. 우리는 이 문자열을 `println!`에
인자로 전달하고, 이 문자열이 화면에 출력됩니다.

세 번째로, 줄 끝은 세미콜론(`;`)으로 마칩니다. 세미콜론은 이 표현식이 끝났고 다음
표현식을 시작할 준비가 되었음을 나타냅니다. 대부분의 러스트 코드 줄은 세미콜론으로
끝납니다.

<!-- Old headings. Do not remove or links may break. -->
<a id="compiling-and-running-are-separate-steps"></a>

### 컴파일과 실행

방금 새로 만든 프로그램을 실행해 봤으니, 그 과정에서 각 단계가 어떻게 일어나는지
살펴봅시다.

러스트 프로그램을 실행하기 전에, 우선 러스트 컴파일러를 이용해 프로그램을 컴파일해야
합니다. 다음과 같이 `rustc` 명령을 입력하고 소스 파일 이름을 전달하세요.

```console
$ rustc main.rs
```

C나 C++ 배경이 있다면, 이것이 `gcc`나 `clang`과 비슷하다는 것을 알아챌 것입니다.
성공적으로 컴파일한 후, 러스트는 실행 가능한 바이너리를 출력합니다.

리눅스, 맥OS, 그리고 윈도우의 파워셸에서는 셸에 `ls` 명령을 입력해 실행 파일을
확인할 수 있습니다.

```console
$ ls
main  main.rs
```

리눅스와 맥OS에서는 두 개의 파일이 보입니다. 윈도우의 파워셸에서는 CMD를 사용할
때와 동일한 세 개의 파일이 보입니다. 윈도우의 CMD에서는 다음을 입력합니다.

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

여기에는 *.rs* 확장자의 소스 코드 파일, 실행 파일(윈도우에서는 *main.exe*, 그 외
모든 플랫폼에서는 *main*), 그리고 윈도우를 사용할 경우 *.pdb* 확장자의 디버깅
정보를 담은 파일이 보입니다. 이제 다음과 같이 *main* 또는 *main.exe* 파일을
실행합니다.

```console
$ ./main # or .\main on Windows
```

*main.rs*가 여러분의 “Hello, world!” 프로그램이었다면, 이 명령은 터미널에 `Hello,
world!`를 출력합니다.

루비, 파이썬, 자바스크립트와 같은 동적 언어에 더 익숙하다면, 컴파일과 실행을 별도의
단계로 나누는 것이 익숙하지 않을 수 있습니다. 러스트는 *사전 컴파일(ahead-of-time
compiled)* 언어로, 여러분이 프로그램을 컴파일해 실행 파일을 다른 사람에게 건네면
그 사람은 러스트가 설치되어 있지 않아도 실행할 수 있음을 의미합니다. 누군가에게
*.rb*, *.py*, *.js* 파일을 전달한다면, 각각 루비, 파이썬, 자바스크립트 구현체가
설치되어 있어야 실행할 수 있습니다. 다만 그런 언어에서는 프로그램을 컴파일하고
실행하는 데 명령 하나면 됩니다. 언어 설계에서는 모든 것이 절충(trade-off)입니다.

단순한 프로그램이라면 `rustc`만으로 컴파일해도 괜찮지만, 프로젝트가 커지면 온갖
옵션을 관리하고 코드를 쉽게 공유할 수 있게 만들고 싶어질 것입니다. 이어서 실제
러스트 프로그램을 작성하는 데 도움이 되는 Cargo 도구를 소개하겠습니다.

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html
