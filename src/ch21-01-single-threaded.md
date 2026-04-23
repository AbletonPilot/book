## 단일 스레드 웹 서버 만들기

먼저 단일 스레드 웹 서버를 동작시키는 것부터 시작하겠습니다. 시작하기 전에 웹 서버를 만드는 데 관여하는 프로토콜의 간단한 개요를 살펴봅시다. 이 프로토콜들의 세부 사항은 이 책의 범위를 넘어서지만, 짧은 개요로 필요한 정보를 얻을 수 있습니다.

웹 서버에 관여하는 두 가지 주요 프로토콜은 _Hypertext Transfer Protocol(HTTP)_ 과 _Transmission Control Protocol(TCP)_ 입니다. 두 프로토콜 모두 _요청-응답(request-response)_ 프로토콜이며, _클라이언트(client)_ 가 요청을 시작하고 _서버(server)_ 가 요청을 수신 대기하면서 클라이언트에 응답을 제공한다는 뜻입니다. 그러한 요청과 응답의 내용은 프로토콜이 정의합니다.

TCP는 한 서버에서 다른 서버로 정보가 어떻게 전달되는지에 대한 세부 사항을 기술하는 더 낮은 수준의 프로토콜이지만, 그 정보가 무엇인지는 명시하지 않습니다. HTTP는 요청과 응답의 내용을 정의함으로써 TCP 위에 만들어집니다. 기술적으로 HTTP를 다른 프로토콜과 함께 사용하는 것이 가능하지만, 대부분의 경우 HTTP는 TCP를 통해 데이터를 보냅니다. 우리는 TCP와 HTTP 요청 및 응답의 원시 바이트를 다루겠습니다.

### TCP 연결 수신 대기

우리 웹 서버는 TCP 연결을 수신 대기해야 하므로 그것이 우리가 작업할 첫 부분입니다. 표준 라이브러리는 이를 할 수 있게 해 주는 `std::net` 모듈을 제공합니다. 평소처럼 새 프로젝트를 만듭시다.

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

이제 시작을 위해 _src/main.rs_ 에 Listing 21-1의 코드를 입력하세요. 이 코드는 로컬 주소 `127.0.0.1:7878`에서 들어오는 TCP 스트림을 수신 대기합니다. 들어오는 스트림을 받으면 `Connection established!`를 출력합니다.

<Listing number="21-1" file-name="src/main.rs" caption="들어오는 스트림을 수신 대기하고 스트림을 받으면 메시지 출력하기">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

`TcpListener`를 사용해, 주소 `127.0.0.1:7878`에서 TCP 연결을 수신 대기할 수 있습니다. 주소에서 콜론 앞부분은 자신의 컴퓨터를 나타내는 IP 주소이고(이것은 모든 컴퓨터에서 동일하며 저자의 컴퓨터를 특정해 가리키는 것이 아닙니다), `7878`은 포트입니다. 이 포트를 선택한 데는 두 가지 이유가 있습니다. HTTP는 보통 이 포트로 받지 않으므로, 우리 서버는 머신에서 실행 중일 수 있는 다른 웹 서버와 충돌할 가능성이 낮습니다. 그리고 7878은 전화기에서 _rust_ 라고 입력한 숫자입니다.

이 시나리오에서 `bind` 함수는 새로운 `TcpListener` 인스턴스를 반환한다는 점에서 `new` 함수와 비슷하게 동작합니다. 이 함수가 `bind`라고 불리는 이유는, 네트워킹에서 어떤 포트에 연결해 그것을 수신 대기하는 것을 “포트에 바인드한다”라고 부르기 때문입니다.

`bind` 함수는 `Result<T, E>`를 반환하는데, 이는 바인드가 실패할 가능성이 있음을 나타냅니다. 예를 들어 우리 프로그램의 두 인스턴스를 실행해 두 프로그램이 같은 포트를 수신 대기하는 경우입니다. 우리는 학습 목적으로만 기본 서버를 작성하고 있으므로 이런 종류의 오류를 처리하는 것에 신경 쓰지 않습니다. 대신 오류가 발생하면 프로그램을 멈추기 위해 `unwrap`을 사용합니다.

`TcpListener`의 `incoming` 메서드는 우리에게 일련의 스트림(더 구체적으로는 `TcpStream` 타입의 스트림)을 주는 이터레이터를 반환합니다. 단일 _스트림(stream)_ 은 클라이언트와 서버 사이의 열린 연결 하나를 나타냅니다. _연결(connection)_ 은 클라이언트가 서버에 연결하고 서버가 응답을 생성하고 서버가 연결을 닫는 전체 요청-응답 과정의 이름입니다. 따라서 `TcpStream`에서 읽어 클라이언트가 보낸 것이 무엇인지 확인하고, 데이터를 클라이언트로 돌려 보내기 위해 우리 응답을 스트림에 씁니다. 전체적으로 이 `for` 반복은 각 연결을 차례로 처리하고, 우리가 다룰 일련의 스트림을 만들어 냅니다.

지금은 스트림 처리는 스트림에 어떤 오류가 있으면 프로그램을 종료하기 위해 `unwrap`을 호출하는 것으로 구성되어 있습니다. 오류가 없으면 프로그램이 메시지를 출력합니다. 다음 리스팅에서 성공 케이스에 대한 더 많은 기능을 추가합니다. 클라이언트가 서버에 연결할 때 `incoming` 메서드에서 오류가 발생할 수 있는 이유는, 우리가 사실 연결들을 순회하는 것이 아니기 때문입니다. 대신 _연결 시도(connection attempts)_ 를 순회하고 있습니다. 연결은 여러 이유로 성공하지 못할 수 있으며, 그 중 많은 부분은 운영체제에 따라 다릅니다. 예를 들어 많은 운영체제는 동시에 열 수 있는 연결의 수에 제한을 둡니다. 그 수를 넘는 새 연결 시도는 일부 열린 연결이 닫힐 때까지 오류를 만들어 냅니다.

이 코드를 실행해 봅시다! 터미널에서 `cargo run`을 호출한 다음 웹 브라우저에서 _127.0.0.1:7878_ 을 열어 보세요. 서버가 현재 어떤 데이터도 돌려보내지 않으므로 브라우저는 “Connection reset”과 같은 오류 메시지를 표시할 것입니다. 그러나 터미널을 보면 브라우저가 서버에 연결될 때 출력된 여러 메시지를 볼 수 있을 것입니다!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

가끔은 한 번의 브라우저 요청에 대해 여러 메시지가 출력되는 것을 보게 됩니다. 그 이유는 브라우저가 페이지에 대한 요청뿐 아니라, 브라우저 탭에 표시되는 _favicon.ico_ 아이콘 같은 다른 리소스에 대한 요청도 만들고 있기 때문일 수 있습니다.

서버가 어떤 데이터도 응답하지 않으므로 브라우저가 여러 번 서버에 연결을 시도할 수도 있습니다. `stream`이 스코프를 벗어나 반복의 끝에서 드롭되면, `drop` 구현의 일부로 연결이 닫힙니다. 브라우저는 때때로 닫힌 연결을 재시도로 처리합니다. 문제가 일시적일 수 있기 때문입니다.

브라우저는 또한 때때로 어떤 요청도 보내지 않은 채 서버에 여러 연결을 열어 두기도 합니다. 그래야 나중에 *실제로* 요청을 보낼 때 그 요청이 더 빨리 일어날 수 있기 때문입니다. 이런 경우 우리 서버는 그 연결에 어떤 요청이 있든 없든 각 연결을 보게 됩니다. 예를 들어 많은 Chrome 기반 브라우저들이 그렇게 합니다. 비공개 브라우징 모드를 사용하거나 다른 브라우저를 사용해 그 최적화를 비활성화할 수 있습니다.

중요한 점은 우리가 TCP 연결의 핸들을 성공적으로 얻었다는 것입니다!

특정 버전의 코드를 실행하는 것을 마치면 <kbd>ctrl</kbd>-<kbd>C</kbd>를 눌러 프로그램을 멈추는 것을 잊지 마세요. 그런 다음, 가장 새로운 코드를 실행하고 있는지 확인하기 위해 코드 변경 사항 한 묶음을 만들 때마다 `cargo run` 명령으로 프로그램을 다시 시작하세요.

### 요청 읽기

이제 브라우저로부터 요청을 읽는 기능을 구현해 봅시다! 먼저 연결을 얻는 일과 그 다음 그 연결로 어떤 동작을 취하는 일을 분리하기 위해, 연결을 처리할 새 함수를 시작하겠습니다. 이 새 `handle_connection` 함수에서는 TCP 스트림으로부터 데이터를 읽고, 브라우저에서 보내는 데이터를 볼 수 있도록 출력합니다. 코드를 Listing 21-2와 같이 변경하세요.

<Listing number="21-2" file-name="src/main.rs" caption="`TcpStream`에서 읽어 데이터 출력하기">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

`std::io::BufReader`와 `std::io::prelude`를 스코프로 가져와 스트림으로부터 읽고 쓸 수 있게 해 주는 트레이트와 타입에 접근합니다. `main` 함수의 `for` 반복 안에서, 연결이 만들어졌다는 메시지를 출력하는 대신, 이제 새 `handle_connection` 함수를 호출하고 `stream`을 전달합니다.

`handle_connection` 함수에서는 `stream`에 대한 참조를 감싸는 새 `BufReader` 인스턴스를 만듭니다. `BufReader`는 `std::io::Read` 트레이트 메서드 호출을 우리 대신 관리해 버퍼링을 추가합니다.

브라우저가 우리 서버로 보내는 요청의 줄들을 모으기 위해 `http_request`라는 변수를 만듭니다. `Vec<_>` 타입 표기를 추가해 이 줄들을 벡터에 모으겠다는 의도를 나타냅니다.

`BufReader`는 `lines` 메서드를 제공하는 `std::io::BufRead` 트레이트를 구현합니다. `lines` 메서드는 데이터 스트림을 개행 바이트가 보일 때마다 분할하여 `Result<String, std::io::Error>`의 이터레이터를 반환합니다. 각 `String`을 얻기 위해 각 `Result`를 `map`하고 `unwrap`합니다. `Result`는 데이터가 유효한 UTF-8이 아니거나 스트림에서 읽는 중에 문제가 있었다면 오류일 수 있습니다. 다시 한번, 운영 프로그램에서는 이런 오류를 더 우아하게 처리해야 하지만, 단순함을 위해 오류 케이스에서는 프로그램을 멈추는 것으로 선택합니다.

브라우저는 두 개의 개행 문자를 연속으로 보냄으로써 HTTP 요청의 끝을 신호합니다. 그래서 스트림에서 한 요청을 얻기 위해 빈 문자열인 줄에 도달할 때까지 줄들을 가져옵니다. 줄들을 벡터에 모으고 나면, 웹 브라우저가 우리 서버로 보내는 지시를 살펴볼 수 있도록 보기 좋게 디버그 포맷으로 그것들을 출력합니다.

이 코드를 시도해 봅시다! 프로그램을 시작하고 웹 브라우저에서 다시 요청을 만드세요. 브라우저에서는 여전히 오류 페이지를 보겠지만, 이제 터미널에 있는 프로그램의 출력은 다음과 비슷할 것입니다.

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-02
cargo run
make a request to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

브라우저에 따라 약간 다른 출력을 볼 수 있습니다. 이제 요청 데이터를 출력하므로, 요청의 첫 줄에서 `GET` 다음의 경로를 보면 한 번의 브라우저 요청에서 여러 연결을 얻는 이유를 알 수 있습니다. 반복되는 연결들이 모두 _/_ 를 요청하고 있다면, 브라우저가 우리 프로그램으로부터 응답을 받지 못해 _/_ 를 반복적으로 가져오려 하고 있다는 것을 알 수 있습니다.

이 요청 데이터를 분해해서 브라우저가 우리 프로그램에 무엇을 요구하는지 이해해 봅시다.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-closer-look-at-an-http-request"></a>
<a id="looking-closer-at-an-http-request"></a>

### HTTP 요청을 더 자세히 살펴보기

HTTP는 텍스트 기반 프로토콜이며, 요청은 다음 형식을 가집니다.

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

첫 줄은 클라이언트가 무엇을 요청하는지에 대한 정보를 담고 있는 _요청 줄(request line)_ 입니다. 요청 줄의 첫 부분은 사용되는 메서드를 나타냅니다. `GET`이나 `POST` 같은 것이며, 클라이언트가 이 요청을 어떻게 만들고 있는지를 설명합니다. 우리 클라이언트는 `GET` 요청을 사용했으며, 이는 정보를 요청한다는 뜻입니다.

요청 줄의 다음 부분은 _/_ 인데, 이는 클라이언트가 요청하는 _uniform resource identifier(URI)_ 를 나타냅니다. URI는 _uniform resource locator(URL)_ 와 거의 같지만 정확히 같지는 않습니다. URI와 URL의 차이는 이 장의 목적상 중요하지 않지만, HTTP 명세는 _URI_ 라는 용어를 사용하므로 여기에서는 그저 머릿속으로 _URI_ 를 _URL_ 로 바꿔 생각해도 됩니다.

마지막 부분은 클라이언트가 사용하는 HTTP 버전이며, 그 다음 요청 줄은 CRLF 시퀀스로 끝납니다. (_CRLF_ 는 _carriage return_ 과 _line feed_ 의 약자로, 타자기 시절의 용어입니다!) CRLF 시퀀스는 `\r\n`으로도 쓸 수 있는데, `\r`은 carriage return이고 `\n`은 line feed입니다. _CRLF 시퀀스_ 는 요청 줄을 나머지 요청 데이터로부터 분리합니다. CRLF가 출력되면 `\r\n`이 아니라 새 줄이 시작되는 것을 보게 된다는 점에 주의하세요.

지금까지 우리 프로그램을 실행해서 받은 요청 줄 데이터를 보면, `GET`이 메서드, _/_ 가 요청 URI, `HTTP/1.1`이 버전임을 볼 수 있습니다.

요청 줄 다음으로, `Host:`부터 시작하는 나머지 줄들은 헤더입니다. `GET` 요청은 본문이 없습니다.

다른 브라우저에서 요청을 만들거나 _127.0.0.1:7878/test_ 같은 다른 주소를 요청해서 요청 데이터가 어떻게 변하는지 보세요.

이제 브라우저가 무엇을 요청하는지 알았으니, 어떤 데이터를 돌려보냅시다!

### 응답 작성하기

클라이언트 요청에 대한 응답으로 데이터를 보내는 것을 구현하겠습니다. 응답은 다음 형식을 가집니다.

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

첫 줄은 응답에서 사용된 HTTP 버전, 요청의 결과를 요약하는 숫자 상태 코드, 그리고 상태 코드의 텍스트 설명을 제공하는 사유 문구(reason phrase)를 담은 _상태 줄(status line)_ 입니다. CRLF 시퀀스 다음에는 헤더가 있고, 그 뒤 또 다른 CRLF 시퀀스, 응답 본문이 옵니다.

다음은 HTTP 버전 1.1을 사용하고 상태 코드 200, OK 사유 문구, 헤더 없음, 본문 없음인 예시 응답입니다.

```text
HTTP/1.1 200 OK\r\n\r\n
```

상태 코드 200은 표준 성공 응답입니다. 이 텍스트는 작은 성공적인 HTTP 응답입니다. 성공한 요청에 대한 응답으로 이를 스트림에 작성합시다! `handle_connection` 함수에서 요청 데이터를 출력하던 `println!`을 제거하고 Listing 21-3의 코드로 교체하세요.

<Listing number="21-3" file-name="src/main.rs" caption="작은 성공 HTTP 응답을 스트림에 작성하기">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

첫 새 줄은 성공 메시지의 데이터를 담는 `response` 변수를 정의합니다. 그런 다음 문자열 데이터를 바이트로 변환하기 위해 `response`에 `as_bytes`를 호출합니다. `stream`의 `write_all` 메서드는 `&[u8]`을 받아 그 바이트를 연결을 통해 직접 보냅니다. `write_all` 연산이 실패할 수 있으므로, 이전처럼 어떤 오류 결과에든 `unwrap`을 사용합니다. 다시, 실제 애플리케이션에서는 여기에 오류 처리를 추가할 것입니다.

이러한 변경 사항을 적용한 채 코드를 실행하고 요청을 만들어 봅시다. 더 이상 어떤 데이터도 터미널에 출력하지 않으므로, Cargo의 출력 외에는 어떤 출력도 보이지 않을 것입니다. 웹 브라우저에서 _127.0.0.1:7878_ 을 열면 오류 대신 빈 페이지를 받을 것입니다. 방금 HTTP 요청을 받고 응답을 보내는 것을 직접 작성한 것입니다!

### 실제 HTML 반환하기

빈 페이지 이상을 반환하는 기능을 구현해 봅시다. 프로젝트 디렉터리의 루트(즉 _src_ 디렉터리가 아니라)에 새 파일 _hello.html_ 을 만드세요. 원하는 어떤 HTML이라도 입력할 수 있습니다. Listing 21-4는 한 가능한 예시를 보여 줍니다.

<Listing number="21-4" file-name="hello.html" caption="응답으로 반환할 샘플 HTML 파일">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

이는 제목과 약간의 텍스트가 있는 최소한의 HTML5 문서입니다. 요청을 받았을 때 서버에서 이를 반환하기 위해, Listing 21-5에서 보이듯 HTML 파일을 읽고 응답에 본문으로 추가해 보내도록 `handle_connection`을 수정합니다.

<Listing number="21-5" file-name="src/main.rs" caption="응답 본문으로 *hello.html* 의 내용 보내기">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

표준 라이브러리의 파일시스템 모듈을 스코프로 가져오기 위해 `use` 문에 `fs`를 추가했습니다. 파일의 내용을 문자열로 읽는 코드는 친숙해 보일 것입니다. 12장 Listing 12-4의 I/O 프로젝트에서 파일 내용을 읽을 때 사용했습니다.

다음으로 `format!`을 사용해 파일의 내용을 성공 응답의 본문으로 추가합니다. 유효한 HTTP 응답을 보장하기 위해, 응답 본문의 크기로 설정된 `Content-Length` 헤더를 추가합니다. 이 경우에는 `hello.html`의 크기입니다.

이 코드를 `cargo run`으로 실행하고 브라우저에서 _127.0.0.1:7878_ 을 열어 보세요. 여러분의 HTML이 렌더링되는 것을 볼 수 있을 것입니다!

현재는 `http_request`의 요청 데이터를 무시하고 무조건 HTML 파일의 내용을 돌려 보내고 있습니다. 즉, 브라우저에서 _127.0.0.1:7878/something-else_ 를 요청해도 여전히 같은 HTML 응답을 받습니다. 지금 우리 서버는 매우 제한적이고 대부분의 웹 서버가 하는 일을 하지 않습니다. 우리는 요청에 따라 응답을 사용자 정의하고, _/_ 에 대한 잘 형성된 요청에 대해서만 HTML 파일을 돌려보내고 싶습니다.

### 요청 검증과 선택적 응답

지금 우리 웹 서버는 클라이언트가 무엇을 요청하든 파일 안의 HTML을 반환합니다. HTML 파일을 반환하기 전에 브라우저가 _/_ 를 요청하고 있는지 확인하고, 브라우저가 다른 무엇을 요청하면 오류를 반환하는 기능을 추가합시다. 이를 위해 Listing 21-6에서 보이듯 `handle_connection`을 수정해야 합니다. 이 새 코드는 받은 요청의 내용을 _/_ 에 대한 요청이 어떻게 생겼는지에 대한 우리 지식과 비교 검사하고, 요청을 다르게 다루기 위해 `if`와 `else` 블록을 추가합니다.

<Listing number="21-6" file-name="src/main.rs" caption="*/*에 대한 요청을 다른 요청과 다르게 처리하기">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

HTTP 요청의 첫 줄만 볼 것이므로, 전체 요청을 벡터에 읽어 들이는 대신 이터레이터에서 첫 항목을 얻기 위해 `next`를 호출합니다. 첫 `unwrap`은 `Option`을 처리하고 이터레이터에 어떤 항목도 없으면 프로그램을 멈춥니다. 두 번째 `unwrap`은 `Result`를 처리하며, Listing 21-2에서 추가한 `map` 안의 `unwrap`과 같은 효과를 가집니다.

다음으로, `request_line`이 _/_ 경로에 대한 GET 요청의 요청 줄과 같은지 확인합니다. 같다면 `if` 블록은 우리 HTML 파일의 내용을 반환합니다.

`request_line`이 _/_ 경로에 대한 GET 요청과 같지 _않다면_, 다른 요청을 받았다는 뜻입니다. 잠시 후 다른 모든 요청에 응답하기 위한 코드를 `else` 블록에 추가하겠습니다.

이 코드를 지금 실행하고 _127.0.0.1:7878_ 을 요청하세요. _hello.html_ 의 HTML을 받을 것입니다. _127.0.0.1:7878/something-else_ 같은 다른 요청을 만들면, Listing 21-1과 21-2의 코드를 실행할 때 보았던 것 같은 연결 오류를 받게 됩니다.

이제 요청한 콘텐츠를 찾을 수 없음을 신호하는 상태 코드 404로 응답을 반환하기 위해 Listing 21-7의 코드를 `else` 블록에 추가합시다. 또한 최종 사용자에게 응답을 보여 주는 페이지에 대한 어떤 HTML도 반환할 것입니다.

<Listing number="21-7" file-name="src/main.rs" caption="*/* 외의 다른 것이 요청되었을 때 상태 코드 404와 오류 페이지로 응답하기">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

여기서 우리 응답은 상태 코드 404와 사유 문구 `NOT FOUND`를 가진 상태 줄을 가집니다. 응답 본문은 _404.html_ 파일의 HTML이 됩니다. 오류 페이지를 위해 _hello.html_ 옆에 _404.html_ 파일을 만들어야 합니다. 다시 한번, 원하는 어떤 HTML을 사용해도 좋고, Listing 21-8의 예시 HTML을 사용해도 됩니다.

<Listing number="21-8" file-name="404.html" caption="404 응답과 함께 보낼 페이지의 샘플 콘텐츠">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

이러한 변경 사항을 적용해 서버를 다시 실행하세요. _127.0.0.1:7878_ 을 요청하면 _hello.html_ 의 내용을 반환해야 하고, _127.0.0.1:7878/foo_ 같은 다른 어떤 요청도 _404.html_ 의 오류 HTML을 반환해야 합니다.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-touch-of-refactoring"></a>

### 리팩터링

지금 `if`와 `else` 블록은 반복이 많습니다. 둘 다 파일을 읽고 파일의 내용을 스트림에 씁니다. 유일한 차이는 상태 줄과 파일명입니다. 이러한 차이점들을 별도의 `if`와 `else` 줄로 빼내어 상태 줄과 파일명의 값들을 변수에 할당함으로써 코드를 더 간결하게 만들어 봅시다. 그러면 파일을 읽고 응답을 쓰는 코드에서 그 변수들을 무조건적으로 사용할 수 있습니다. Listing 21-9는 큰 `if`와 `else` 블록을 교체한 결과 코드를 보여 줍니다.

<Listing number="21-9" file-name="src/main.rs" caption="두 케이스 사이에 다른 코드만 담도록 `if`와 `else` 블록 리팩터링">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

이제 `if`와 `else` 블록은 상태 줄과 파일명에 적절한 값들을 튜플로 반환하기만 합니다. 그런 다음 19장에서 논의했듯이 `let` 문에서 패턴을 사용해 이 두 값을 `status_line`과 `filename`에 할당하기 위해 분해를 사용합니다.

이전에 중복되었던 코드는 이제 `if`와 `else` 블록 바깥에 있고 `status_line`과 `filename` 변수를 사용합니다. 이는 두 케이스 사이의 차이를 보기 더 쉽게 해 주며, 파일 읽기와 응답 쓰기 동작을 변경하고 싶을 때 코드를 갱신할 곳이 단 한 곳뿐임을 의미합니다. Listing 21-9의 코드 동작은 Listing 21-7의 동작과 같을 것입니다.

훌륭합니다! 이제 한 요청에는 콘텐츠가 있는 페이지로 응답하고 다른 모든 요청에는 404 응답으로 응답하는, 약 40줄의 러스트 코드로 된 단순한 웹 서버를 갖게 되었습니다.

지금 우리 서버는 단일 스레드에서 실행되므로 한 번에 한 요청만 서비스할 수 있습니다. 느린 요청 몇 개를 시뮬레이션해 그것이 어떻게 문제가 될 수 있는지 살펴봅시다. 그런 다음 서버가 한 번에 여러 요청을 처리할 수 있도록 고치겠습니다.
