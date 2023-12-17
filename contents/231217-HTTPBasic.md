# HTTP Basic 🤿

Vapor를 시작하기 전에, 먼저 웹과 HTTP가 어떻게 작동하는지에 대한 기본 사항을 검토할 것입니다.<br>

이 장에서는 HTTP, 그 방법등, 그리고 가장 흔한 응답 코드들에 대해 알아야 할 것들을 설명합니다.<br>
또한, Vapor가 웹 개발 경험을 어떻게 향상시킬 수 있는지, 그 이점들, 그리고 다른 스위프트 프레임워크들과 구별되는 점들에 대해서도 배울 것입니다.

# Powering the web 🤿

Hyper Text Transfer Protocol, 또는 HTTP는 웹의 기반이 됩니다.<br>
웹사이트를 방문할 때마다 브라우저는 서버에 HTTP 요청을 보내고 서버로부터 응답을 받습니다.<br>
스마트폰으로 커피를 주문하거나, TV로 비디오를 스트리밍하거나, 온라인 게임을 하는 등 많은 전용 앱들이 뒤에서 HTTP를 사용합니다.<br>

기본적으로 HTTP는 단순합니다.<br>
클라이언트가 있습니다.<br>
iOS 애플리케이션, 웹 브라우저 또는 단순한 cURL 세션일 수 있습니다.<br>
그리고 서버가 있습니다.<br>
클라이언트는 서버에 HTTP 요청을 보내고, 서버는 HTTP 응답을 반환합니다.<br>

<img src = "https://github.com/devKobe24/images/blob/main/VDD.jpeg?raw=true">

## 🤔 HTTP requests

HTTP 요청은 여러 부분으로 구성됩니다.<br>

1️⃣ **`The request line(요청 라인)`**<br>
사용할 HTTP 메소드, 요청된 자원, 그리고 HTTP 버전을 지정합니다.<br>
예를 들어 `GET/about.html HTTP/1.1`이 있습니다.<br>

2️⃣ **`The host(호스트)`**<br>
요청을 처리할 서버의 이름입니다.<br>
이는 같은 주소에서 여러 서버가 호스팅될 때 필요합니다.<br>

3️⃣ **`Other request headers`**<br>
Authorization, Accept, Cache-Control, Content-Length, Content-Type 등과 같은 기타 요청 헤더들.<br>

4️⃣ **`Optional request data`**<br>
HTTP 메소드에 필요한 경우 선택적 요청 데이터.<br>

HTTP 메소드는 클라이언트가 요청하는 작업의 유형을 지정합니다.<br>
HTTP 사양은 다음과 같은 메소드를 정의합니다.<br>

- **GET**
- **HEAD**
- **POST**
- **PUT**
- **DELETE**
- **CONNECT**
- **OPTIONS**
- **TRACE**
- **PATCH**

가장 일반적인 HTTP 메소드는 `GET`입니다.<br>
이는 클라이언트가 서버에서 자원을 검색할 수 있게 합니다.<br>
브라우저에서 링크를 클릭하거나 뉴스 앱에서 스토리를 탭하는 것은 모두 서버에 대한 `GET` 요청을 트리거합니다.<br>

또 다른 일반적인 HTTP 메소드는 `POST`입니다.<br>
이는 클라이언트가 서버에 데이터를 보낼 수 있게 합니다.<br>
사용자 이름과 비밀 번호를 입력한 후 로그인 버튼을 클릭하면 서버에 대한 `POST` 요청을 트리거할 수 있습니다.<br>

자주, 서버는 요청을 제대로 처리하기 위해 자원의 이름 이상의 정보가 필요합니다.<br>
이 추가 정보는 요청 헤더에서 전송됩니다.<br>
요청 헤더는 단순히 키-값 쌍일 뿐입니다.<br>

일반적인 요청 헤더에는 `Authorization`, `Cookie`, `Content-Type`, `Accept` 등이 있습니다.<br>

## 🤔 HTTP responses

서버 요청을 처리한 후 HTTP 응답을 반환합니다.<br>
HTTP 응답은 다음으로 구성됩니다.<br>

1️⃣ **`The status line(상태 라인)`**<br>
버전, 상태 코드 및 메시지를 포함합니다.<br>

2️⃣ **`Response headers(응답 헤더들)`**<br>

3️⃣ **`An optional response body(선택적 응답 본문)`**<br>

상태 코드와 관련 메시지는 요청의 결과를 나타냅니다.<br>
많은 산태 코드들이 있지만 대부분을 사용하거나 마주치지 않을 것입니다.<br>
이들은 첫 번째 숫자에 따라 5개의 그룹으로 나뉩니다.<br>

1️⃣ **`informational response(정보 응답)`**<br>
이들은 자주 발생하지 않습니다.<br>

2️⃣ **`success response(성공 응답)`**<br>
가장 흔한 것은 `200 OK`로, 요청이 성공적으로 완료되었음을 의미합니다.<br>

3️⃣ **`redirection response(리디렉션 응답)`**<br>
이들은 자주 사용됩니다.<br>

4️⃣ **`client error(클라이언트 오류)`**<br>
가장 흔한 것 중 하나는 `404 Not Found`입니다.<br>
아마도 다양하고 재미있는 404 페이지를 몇 개 보셨을 것입니다!<br>

5️⃣ **`server error(서버 오류)`**<br>
이는 종종 잘못 구성된 서버, 자원 고갈 또는 서버 측 앱의 버그를 나타냅니다.<br>

심지어 만우절 농담 상태 코드도 있습니다: `418 I'm a teapot!`<br>

응답에는 페이지의 HTML 내용, 이미지 파일, 또는 자원에 대한 JSON 설명과 같은 응답 본문이 포함 될 수 있습니다.<br>
그러나 응답 본문은 선택적이며, 예를 들어 `204 Not Content`와 같은 일부 응답 코드는 본문이 없을 수 있습니다.<br>

마지막으로, 응답에는 일부 응답 헤더들이 포함될 수 있습니다.<br>
이들은 앞서 설명된 요청 헤더들과 유사합니다.<br>
일반적인 헤더에는 `Set-Cookie`, `WWW-Authenticate`, `Cache-Control` 및 `Content-Length` 가 있습니다.
