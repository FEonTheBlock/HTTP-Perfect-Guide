# 20220326 6장 프락시

- 웹 프락시 서버: 클라이언트와 서버사이에 위치하여 메시지를 정리, 가공, 전달하는 중개 서버

## 6.1. 웹 중개자

- 클라이언트에게는 웹 서버, 서버에게는 웹 클라이언트와 같이 동작
- 개인 프락시 vs. 공유 프락시
    - 대부분의 프락시는 공용으로, 중앙 집중형으로 관리되며 여러 사용자들의 요청을 처리
    - 흔하지는 않지만 브라우저의 기능 확장, 성능 개선 등을 위해 클라이언트의 컴퓨터에서 직접 실행되는 형태로 개인 프락시가 존재한다.
- 프락시 vs. 게이트웨이
    - 프락시: 같은 프로토콜을 사용하는 둘 이상의 애플리케이션을 연결하지만 약간의 프로토콜 변환을 하기도 하므로 게이트웨이와의 차이점은 모호하다.
    - 게이트웨이: 서로다른 프로토콜을 사용하는 둘 이상을 연결(HTTP 프론트엔드에 이메일 백엔드를 연결하는 웹 기반 이메일 프로그램 등)

## 6.2. 왜 프락시를 사용하는가?

- 실용적이고 유용한 기능을 제공: 보안 개선, 성능 향상, 비용 절약 등
- 부가적인 유용한 웹 서비스를 구현하기 위해 HTTP 트래픽을 감시하고 수정
- 프락시 애플리케이션 사례
    - 어린이 필터: 학교에서 성인 콘텐츠 차단을 위한 필터링 프락시
    - 문서 접근 제어: 회사에서 리소스에 대한 단일한 접근 제어 전략을 구현, 감사 추적하는 중앙 프락시
    - 보안 방화벽: 조직을 출입하는 프로토콜 흐름을 네트워크의 한 지점에서 통제하여 바이러스 제거
    - 웹 캐시: 자주 사용되는 문서의 로컬 사본을 관리하여 빠르게 제공
    - 대리(리버스) 프락시: 웹 서버처럼 위장하여 공용 콘텐츠에 대한 서버 성능 개선
    - 콘텐츠 라우터: 인터넷 트래픽 조건과 콘텐츠의 종류에 따라 요청을 특정 웹 서버로 유도
    - 트랜스코더: 본문 포맷을 수정하여 데이터 표현방식 변환 (e.g. gif ⇒ jpg, 외국어 문서로 변환 등)
    - 익명화 프락시: 신원을 식별할 수 있는 HTTP 메시지의 속성(User-Agent, From, Referer, Cookie 헤더 등) 제거

## 6.3. 프락시는 어디에 있는가?

- 프락시 서버는 어떻게 사용할지에 따라 어디에든 배치가 가능
    - 출구 프락시: 로컬 네트워크 출구에 배치하여 로컬 네트워크와 더 큰 인터넷 사이 트래픽을 제어. 회사의 방화벽이나 학교의 필터링 프락시 등
    - 접근(입구) 프락시: 사요앚들의 모든 요청을 종합적으로 처리하기 위해 ISP 접근지점에 배치. 다운로드 속도 개선과 인터넷 대역폭 비용 줄이기 위한 캐시 프락시 등
    - 대리(리버스) 프락시: 서버의 바로 앞에 위치하여 서버 보안과 성능 개선. 웹서버의 이름과 IP로 가장
    - 네트워크 교환 프락시: 네트워크 사이 인터넷 피어링 교환 지점에 배치.
- 프락시 계층
    - 메시지는 여러 프락시를 거쳐 최종적으로 웹 서버에 도착하고 반대로도 여러 프락시를 거쳐 클라이언트로 돌아온다.
    - 서버에 가까운 쪽이 인바운드 프락시, 부모 프락시 (반대로 클라이언트에 가까운 쪽이 아웃바운드이자 자식 프락시)
    - 계층은 반드시 정해지지 않고 프락시는 부하를 분산하거나 지리적 인접성에 근거한 라우팅, URI에 따라 메시지를 다양하고 유동적이게 보낼 수 있는데 이를 동적 부모 선택이라고 한다.
- 클라이언트 트래픽이 프락시로 가도록 처리하는 방법
    - 클라이언트 수정: 웹 클라이언트에서 프락시를 사용하도록 설정
    - 네트워크 수정: 네트워크 인프라를 가로채 프락시로 가도록 조정, 스위칭 장치와 라우팅 장치를 필요로하며 이를 인터셉트 프락시라고 한다.
    - DNS namespace 수정: 실제 서버의 IP 주소와 이름 변경하고 프락시에 이전의 주소와 이름을 주는 등
- 웹 서버 수정: 리다이렉션 응답(305)을 통해 클라이언트의 요청을 프락시로 보낸다.

## 6.4 클라이언트 프락시 설정

## 6.5 프락시 요청의 미묘한 특징들

## 6.6 메시지 추적

## 6.7 프락시 인증

## 6.8 프락시 상호운용성

- 지원하지 않는 헤더와 메서드 다루기
- OPTIONS: 어떤 기능을 지원하는지 알아보기
- Allow 헤더