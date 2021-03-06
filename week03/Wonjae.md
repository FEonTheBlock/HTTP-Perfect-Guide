# 4장 커넥션 관리

## TCP 커넥션

> 전 세계 모든 HTTP 통신은 TCP/IP를 통해 이루어진다.

- 한번 연결이 맺어지면 클라이언트와 서버 컴퓨터 간에 주고받는 메세지들은 손실 혹은 손상되거나 순서가 바뀌지 않고 안전하게 전달된다.

### 브라우저가 TCP 커넥션을 통해 웹 서버에 요청을 보내는 과정

1. 호스트명 추출
2. DNS를 통해 IP주소를 얻음
3. 포트번호를 얻음
4. 브라우저가 TCP 커넥션 생성
5. 브라우저가 서버로 HTTP GET 요청
6. 서버로부터 응답메세지를 읽음
7. 커넥션 종료

### 신뢰할 수 있는 데이터 전송 통로인 TCP

> 인터넷을 안정적으로 연결해주고 HTTP에게 신뢰할만한 통신 방식을 제공한다.

### TCP 스트림은 세그먼트로 나뉘어 IP패킷을 통해 전송된다.

> HTTP에서 메세지를 전송하는 경우 연결되어 있는 TCP 커넥션을 통해 `TCP 세그먼트`라는 단위로 데이터 스트림을 잘게 나누고, `세그먼트 IP 패킷`이라는 봉투에 담아서 인터넷을 통해 데이터를 전달한다.

**세그먼트 IP 패킷이 포함하는 데이터**

- **IP 패킷 헤더**: 발신지와 목적지의 IP주소, 크기, 기타 플래그...
- **TCP 세그먼트 헤더**: TCP 포트번호, TCP 제어 플래그, 데이터 순서와 무결성 검사
- **TCP 데이터 조각**

### TCP 커넥션 유지하기

> 특정 커넥션들은 같은 목적지 포트 번호를 가리킬 수 있다.

TCP 커넥션은 아래와 같이 4가지 값으로 식별한다.(이 4가지 값이 모두 같은 경우는 없으므로 유일하다.)

```shell
〈발신지 IP 주소， 발신지 포트， 수신지 IP 주소， 수신지 포트〉
```

<br>

## TCP 성능에 대한 고려

> HTTP는 TCP 바로 위에 있는 계층이므로 HTTP 트랜잭션의 성능은 TCP 성능에 영향을 받는다.

### HTTP 트랜잭션 지연

> 하드웨어 성능, 네트워크와 서버의 전송 속도, 요청과 응답 메세지의 크기, 클라이언트와 서버 간의 거리에 따라 크게 달라지며, TCP 프로토콜의 기술적인 복잡성도 지연에 영향을 미친다.

**트랜잭션 지연의 원인**

- 방문한 적 없는 호스트 접근시 DNS 이름 분석 인프라를 사용하여 URI에 있는 호스트명을 IP 주소로 변환하는데 수십초의 시간
- 커넥션 설정시간(클라이언트 -> 서버로 TCP 커넥션 요청)
  - HTTP 트랙잭션이 여러개일수록 시간 증가
- 웹 서버가 데이터 요청을 받고 이를 처리하는 시간
- 웹 서버가 HTTP 응답을 보내는 시간

**성능 관련 중요 요소**

- TCP 커넥션의 핸드 셰이크 설정
  - 크기가 작은 HTTP 트랜잭션은 TCP 커넥션을 구성하는데에만 50%이상 시간을 소비
  ```text
  클라이언트 -> 서버: SYN 플래그를 보내 커넥션 생성 요청
  서버 -> 클라이언트: SYN + ACK 플래그를 보내 커넥션 요청을 받아들였음을 응답
  클라이언트 -> 서버: ACK + get 요청 등 확인 응답 신호
  ```
- TCP의 편승 확인 응답을 위한 확인응답 지연 알고리즘
  - 각 TCP 세그먼트는 순번과 데이터 무결성 체크섬을 갖고, 수신자가 이를 온전히 받으면 확인 응답 메세지를 다시 보낸다.
  - 그러나 크기가 작아 같은 TCP 방향으로 송충되는 데이터 패킷에 편승하려고 한다.
  - 이때 확인 응답 지연 알고리즘을 구현하는데 이로 인한 지연이 자주 발생하기도 한다.
- 인터넷의 혼잡 제어를 위한 TCP의 느린시작
  - TCP 커넥션은 지속되고 있는 커넥션에서는 자체적으로 `튜닝`되어 처음에는 보내는 패킷 수를 제한하다가, 전송이 성공한 뒤 점점 패킷 수를 늘려간다.
  - 혼잡 윈도우를 연다 === 보낼 수 있는 패킷 수가 늘어난다
  - 이러한 혼잡 제어 기능으로 인해 새로운 커넥션은 느리다.
- 데이터를 한데 모아 전송하기 위한 네이글 알고리즘
  - 작은 크기의 데이터를 TCP로 보내면 배보다 배꼽이 더 큰 상황이 생긴다.
  - `40바이트 상당의 플래그, 헤더`를 포함하여 전송하기 때문
  - 이 때문에 네트워크 성능이 떨어지므로 이를 예방하기 위해 네이글 알고리즘을 사용한다.
  - 세그먼트가 최대 크기가 되지 않으면 전송을 하지 않고 확인 응답을 받았을 때 작은 패킷 전송을 허락하며 아직 패킷이 전송중이면 버퍼에 저장하고 확인응답 혹은 충분한 패킷이 쌓였을때 버퍼에서 다시 전송한다.
  - 하지만 이 또한 다른 문제를 발생시킨다.
  - 크기가 작은 HTTP 메세지는 패킷을 채우지 못해서 추가적인 데이터를 기다리며 지연
  - 확인 응답 지연과 함께 쓰일 경우 그만큼 확인응답이 늦어지므로 지연
- `TIME_WAIT` 지연과 포트 고갈
  - TCP 커넥션이 종료되면 패킷의 중복과 TCP 데이터 충돌을 막기 위해 약 2분 이내에 같은 IP 주소와 포트 번호를 가지는 커넥션이 생성되는 것을 막아준다.
  - 하지만 현대에는 높은 성능의 서버 덕분에 이런 일이 거의 없다.
  - 일반적으로 커넥션 종료 지연이 문제되지는 않지만 성능 측정 시 문제가 될 수 있다.

## HTTP 커넥션 관리

### Connection 헤더

- HTTP 메세지는 `클라이언트 -> 서버` 사이 중개 서버들을 거치며 전달되는데 현재 맺고 있는 커넥션에서만 적용될 옵션을 지정해야하는 경우가 있다.
- 커넥션 토큰들: HTTP 헤더 필드명, 임시 토큰 값, close 값
- HTTP 헤더 필드명은 이 커넥션에만 해당하는 헤더들을 나열한다.
- 임시 토큰 값은 커넥션에 대한 비표준 옵션을 의미한다.
- close 값은 커넥션 작업이 완료되면 종료되어야함을 의미한다.

### 순차적인 트랜잭션 처리에 의한 지연

> 한 페이지를 위한 트랜잭션이 많을 경우, 각 트랜잭션이 새로운 커넥션을 필요로 할때 지연 발생
> <br>이를 해결하기 위해 아래와 같은 최신 기술들이 있음

- **병렬 커넥션**: 여러개의 TCP 커넥션을 통한 동시 HTTP 요청
- **지속 커넥션**: 커넥션을 맺고 끊는 데서 발생하는 지연을 제거하기 위한 TCP 커넥션의 재활용
- **파이프라인 커넥션**: 공유 TCP 커넥션을 통한 병렬 HTTP 요청
- **다중 커넥션**: 요청과 응답들에 대한 중재이며 실험적인 기술

### 병렬 커넥션

> 클라이언트가 여러 개의 커넥션을 맺어 HTTP 트랜잭션을 병렬로 처리한다.

- 하지만 병렬 커넥션이 항상 빠르다는 것을 보장하지는 않는다.
- 네트워크 대역폭이 좁은 경우 대부분의 시간을 데이터 전송하는 데만 사용하게 된다.
- 다수의 커넥션은 메모리를 많이 소모하고 자체적인 성능 문제를 발생시킨다.
- 병렬 커넥션이 페이지를 항상 더 빠르게 로드하는 것은 아니지만 화면에 여러 개의 객체가 동시에 보이면서 내려받으면 사용자는 더 빠르게 내려받고 있는 것처럼 체감할 수 있다.

### 지속 커넥션

> 처리가 완료된 후에도 계쏙 연결된 상태로 있는 TCP 커넥션을 지속 커넥션이라고 한다.

- 지속 커넥션을 재사용함으로서 커넥션을 맺는 준비작업에 따르는 시간을 절약할 수 있다.

**병렬 커넥션의 단점**

- 각 트랜잭션 마다 새로운 커넥션을 맺고 끊어 시간과 대역폭 소요
- 각각의 새로운 커넥션은 TCP의 느린시작 때문에 성능이 떨어짐
- 실제로 연결할 수 있는 병렬 커넥션의 수에는 제한이 없음

지속 커넥션은 이런 병렬 커넥션은 단점을 해결하지만 잘못 관리하는 경우 계속 연결된 상태의 커넥션이 쌓여 클라이언트와 서버의 리소스에 불필요한 소모를 발생시킨다.

따라서 지속 + 병렬 조합이 가장 효과적이며 지속 커넥션 타입으로 `HTTP/1.0+`에서는 `keep-alive`커넥션을, `HTTP/1.1`에는 `지속`커넥션이 있다.

**`keep-alive` 커넥션**

> `keep-alive`커넥션을 통해 커넥션을 맺고 끊는데 필요한 작업을 없애 시간을 단축시킬 수있다.

`keep-alive`는 헤더에 `Connection: Keep-Alive`를 포함시켜 보내고 서버는 다음 요청도 이 커넥션을 통해 받고자하면 응답에 이를 포함시켜 보낸다.

만약 헤더가 없으면 서버 커넥션을 끊을 것이라고 클라이언트가 추정한다.

**`keep-alive` 커넥션 제한과 규칙**

- `HTTP/1.0`에서 기본으로 사용되지 않아 요청 헤더를 따로 보내야함
- 이를 계속 유지하려면 모든 메세지에 `Connection: Keep-Alive` 헤더가 포함되어 있어야 함
- 끊어지기 전, 엔터티 본문의 길이를 알 수 있어야 커넥션 유지 가능
- 프락시와 게이트웨이는 Connection 헤더의 규칙을 철저히 지켜야함
- 이 헤더를 인식 못하는 프락시 서버와는 맺어지면 안됨
- `HTTP/1.0`을 따르는 기기로부터 받는 모든 Connection 헤더 필드는 무시해야함
- 클라이언트는 응답 전체를 모두 받기 전에 커넥션이 끊어졌을 경우, 별다른 문제가 없으면 요청을 다시 보낼 수 있게 준비되어야 함

**`HTTP/1.1`의 지속 커넥션**

> 해당 버전에서는 지속 커넥션이 기본으로 활성화되어 있다.

트랜잭션이 끝난 후 커넥션을 종료하려면 `Connection: close`헤더를 명시해야한다.
<br>`HTTP/1.1` 클라이언트는 응답에 이 헤더가 없으면 응답 후에도 커넥션을 계속 유지하는 것으로 추정한다.
<br>그러나 클라이언트와 서버는 언제든지 커넥션을 끊을 수 있다.

**지속 커넥션 제한과 규칙**

- 클라이언트가 해당 커넥션으로 추가적인 요청을 보내지 않을 것이라면 마지막 요청에 `Connection: close` 헤더를 보내고, 이를 보내면 그 커넥션으로 추가적인 요청을 보낼 수 없다.
- 커넥션에 있는 모든 메세지가 자신의 길이 정보를 정확히 가지고 있을 때에만 커넥션을 지속시킬 수 있다.
- `HTTP/1.1`의 프락시는 클라이언트와 서버 각각에 대한 별도의 지속 커넥션을 맺고 관리해야 한다.
- 커넥션 관련 기능에 대한 클라이언트의 지원 범위를 알고 있지 않은 한 지속 커넥션을 맺으면 안된다.
- `HTTP/1.1` 기기는 언제든 커넥션을 끊을 수 있다.
- `HTTP/1.1` 애플리케이션은 중간에 끊어지는 커넥션을 복구할 수 있어야만 한다.
- 클라이언트는 전체 응답을 받기전 커넥션이 끊어지면 요청을 반복해서 보내도 문제가 없는 경우 요청을 다시 보낼 준비가 되어있어야한다.
- 하나의 사용자 클라이언트는 서버의 과부화를 방지하기 위해 넉넉히 2개의 지속 커넥션만 유지해야한다.
