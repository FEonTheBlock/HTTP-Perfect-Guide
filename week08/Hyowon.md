웹 캐시는 자주 쓰이는 문서의 사본을 자동으로 보관하는 HTTP 장치다.

웹 요청이 캐시에 도착했을 때, 캐시된 로컬 사본이 존재한다면,

그 문서는 원 서버가 아니라 그 캐시로부터 제공된다.

캐시는 다음과 같은 혜택을 준다

- 불필요한 테이터 전송을 줄인다
    
    → 네트워크 요금으로 인한 비용을 줄인다
    
- 네트워크 병목을 줄인다
    
    → 대역폭을 늘리지 않고도 페이지를 빨리 불러올 수 있다
    
- 원 서버에 대한 요청을 줄인다
    
    → 서버는 부하를 줄일 수 있고 더 빨리 응답할 수 있게된다.
    
- 거리로 인한 지연을 줄인다

## 7.1 불필요한 데이터 전송

복수의 클라이언트가 자주 쓰이는 원 서버 페이지에 접근 할때,

서버는 같은 문서를 클라이언트들에게 각각 한 번씩 전송하게 된다.

똑같은 바이트들이 네트워크를 통해 계속 반복해서 이동한다

불필요한 데이터 전송은 값비싼 네트워크 대역폭을 잡아먹고,

전송을 느리게 만들며, 웹서버에 부하를 준다

캐시를 이용하면, 첫 번째 서버 응답은 캐시에 보관된다.

캐시된 사본이 뒤이은 요청들에 대한 응답으로 사용될 수 있기 때문에

원 서버가 중복해서 트래픽을 주고 받는 낭비가 줄어들게 된다.

## 7.2 대역폭 병목

캐시는 네트워크 병목을 줄인다

많은 네트워크가 원격 서버보다 로컬 네트워크 클라이언트에 더 넓은 대역폭을 제공한다

클라이언트들이 서버에 접근할 때의 속도는, 그 경로에 있는 가장 느린 네트워크의 속도와 같다.

만약 클라이언트가 빠른 LAN에 있는 캐시로부터 사본을 가져온다면, 캐싱은 성능을 대폭 개선할 수 있을 것이다 (특히 큰 문서들에 대해서)

대역폭이 네트워크 속도와 문서 크기에 따라 전송 시간에 얼마나 영향을 주는지 보여준다

대역폭은 큰 문서에 대해 현저한 지연을 일으키며, 속도는 네트워크 종류에 따라 극적으로 달라진다

## 7.3 갑작스런 요청 쇄도(Flash Crowds)

캐싱은 갑작스런 요청 쇄도에 대처하기 위해 특히 중요하다.

## 7.4 거리로인한지연

대역폭이 문제가 되지 않더라도, 거리가 문제가 될 수 있다.

캐시를 사용하면 문서가 전송되는 물리적인 거리를 수천 킬로미터에서 수십 미터로 줄일 수 있다

## 7.5 적중과 부적중

캐시가 모든 문서의 사본을 저장하지 않는다

캐시에 요청이 도착했을 때 대응하는 사본이 있음 → 캐시 적중 (cache hit)라고 부른다

대응하는 사본이 없다면 그냥 원서버로 전달되기만 한다. → 캐시 부적중(cache miss)

### 7.5.1 재검사(Revalidation)

원 서버 콘텐츠는 변경될 수 있기 때문에, 캐시는 사본이 최신인지 때때로 점검해야한다

효과적인 재검사를 위해 전체를 가져오지 않고도 여전이 신선한지 빠르게 검사할 수 있는 요청을 정의했다.

캐시는 스스로 원한다면 언제든지 사본을 재검사할 수 있지만 문서를 수백만개 가지고 있기 때문에

대부분의 캐시는 클라이언트가 사본을 요청하였으며 사본이 검사가 필요할 정도로 오래된 경우에만 재검사한다.

- 캐시는 재검사가 필요할때 서버에 재검사 요청을 보낸다
- 변경되지 않았다면 304 Not Modified 응답을 보낸다.
- 사본이 유효함을 알게 된 캐시는 사본이 신선하다고 임시로 표시하고 클라이언트에게 제공한다
    - 재검사 적중 혹은 느린 적중이라고 부른다
    - 순수 캐시 적중보다는 느리다
    - 캐시 부적중 보다는 빠르다
    

**If-Modified-Since 헤더**

캐시된 객체를 재확인하기 위한 몇가지 도구가 있는데

그 중에서 가장 많이 쓰이는 것은 If-Modified-Since 헤더다

서버에 보내는 get 요청에 이 헤더를 추가하면 캐시된 시간 이후에 변경된 경우에만 사본을 보내달라는 의미

GET If-Modified-Since 요청이 서버에 도착했을 때 일어 날 수 있는 세가지 상황

- 서버 콘텐츠가 변경되지 않은 경우 (재검사 적중)
    - 304 Not Modified 응답을 보낸다.
- 서버 콘텐츠가 변경된 경우 (재검사 부적중)
    - 콘텐츠 전체와 함께 200 응답
- 객체가 삭제된 경우
    - 404 Not Found 응답, 캐시 사본 삭제

### 7.5.2 적중률

캐시가 요청을 처리하는 비율

0% 는 모든 요청이 캐시 부적중

100% 는 모든 요청이 캐시 적중

### 7.5.3 바이트 적중률

문서 적중률이 모든 것을 말해주지 않음

몇몇 큰 객체는 덜 접근되지만 그 크기 때문에 전체 트래픽에는 더 크게 기여한다

캐시를 통해 제공된 모든 바이트의 비율을 표현

### 7.5.4 적중과 부적중의 구별

- http는 캐시 적중이었는지 서버 접근인지 구분하는 방법을 제공하지 않음
- 어떤 상용 프락시 캐시는 Via헤더에 추가정보를 붙이기도한다
- 클라이언트가 응답이 캐시에서 왔는지 알아내는 방법은 Date 헤더를 이용하는 것
    - 응답의 Date 헤더 값을 현재 시각과 비교하여, 응답의 생성일이 더 오래되었다면 캐시된 응답인 것을 알 수 있다
    - 다른 방법으로는 응답이 얼마나 오래되었는지 말해주는 Age 헤더 이용하는 것

## 7.6 캐시토폴로지

캐시는 한 명의 사용자에게만 할당될 수 도 있고

수 천 명의 사용자들 간에 공유될 수도 있다

### 7.6.1 개인 전용 캐시

웹브라우저는 개인 전용 캐시를 내장하고 있다

### 7.6.2 공용 프락시 캐시

공용 캐시는 캐시 프락시 서버 혹은 프락시 캐시라고 불리는 특별한 종류의 공유된 프락시 서버다

개인 전용 캐시는 같은 문서를 네트워크를 거쳐 여러 번 가져온다

공용 캐시는 자주 찾는 객체를 한번만 가져와 모든 요청에 대해 공유된 사본을 제공함으로써 네트워크 트래픽을 줄인다

### 7.6.3 프락시 캐시 계층들

작은 캐시에서 캐시 부적중이 발생했을 때 더 큰 부모가 그 ‘걸러 남겨진’ 트래픽을 처리하도록 하는 계층을 만드는 방식이 합리적인 경우가 많다.

### 7.6.4 캐시망, 콘텐츠 라우팅, 피어링

몇몇 네트워크 아키텍처는 단순한 캐시 계층 대신 복잡한 캐시망을 만든다.

캐시망외 프락시 캐시는 복잡한 방법으로 서로 대화하여, 어떤 부모 캐시와 대화할 것인지, 아니면 요청이 캐시를 완전히 우회해서 원 서버로 바로 가도록 할 것인지에 대한 캐시 커뮤니케이션 결정을 동적으로 내린다.

캐시망 안에서의 콘텐츠 라우팅을 위해 설계된 캐시들은 다음에 나열된 일들을 모두 할 수 있다

- URL에 근거하여, 부모 캐시와 원 서버 중 하나를 동적으로 선택
- URL에 근거하여 특정 부모 캐시를 동적으로 선택
- 부모 캐시에게 가기 전에 캐시된 사본을 로컬에서 찾아본다
- 다른 캐시들이 그들의 캐시된 콘텐츠에 부분적으로 접근할 수 있도록 허용하되, 그들의 캐시를 통한 인터넷 트랜짓은 허용하지 않는다.

## 7.7 캐시처리 단계

요청 받기 → 파싱 → 검색 → 신선도 검사 → 응답 생성 → 발송 → 로깅

## 7.8 사본을 신선하게 유지하기

### 7.8.1 문서 만료

HTTP는 Cache-Control과 Expires라는 특별한 헤더들을 이용해서 원 서버가 각 문서에 유효기간을 붙일 수 있게 해준다. (우유 유통기간와 마찬가지)

### 7.8.2 유효기간과 나이

서버는 응답 본문과 함께 Expires 나 Cache-Control: max-age 응답 헤더를 이용해서 유효기간을 명시한다

**Cache-Control: max-age**
max-age 값은 문서의 최대 나이를 정의한다. 최대 나이는 문서가 처음 생성된 이후부터, 제공하기엔 더 이상 신선하지 않다고 간주될 때까지 경과한 시간의 합벅적인 최댓값이다.

`Cache-Control: max-age=484200`

**Expires**

절대 유효기간을 명시한다. 만약 유효기간이 경과했다면, 그 문서는 더이상 신선하지 않다

`Expires: Fri, 0S Jul 2002, 0S:00:00 GMT`

### 7.8.4 조건부 메서드와의 재검사

http의 조건부 메서드는 재검사를 효율적으로 만들어준다

조건부 get은 get 요청 메시지에 특별한 조건부 헤더를 추가하면 된다.

**If-Modified-Since: <date>**

만약 문서가 주어진 날짜 이후로 수정되었다면 요청 메서드를 처리

이것은 캐시된 버전으로부터 콘텐츠가 변경된 경우에만 콘텐츠를 가져오기 위해

Last-Modified 서버 응답 헤더와 함께 사용된다.

**If-None-Match: <tags>**

마지막 변경된 날짜를 맞춰보는 대신, 서버는 문서에 대한 일련번호와 같이 동작하는 특별한 태그(”ETag”)를 제공할 수 있다. If-None-Match 헤더는 캐시된 태그가 서버에 있는 문서의 태그와 다를 때만 요청을 처리한다.

### 7.8.5 If-Modified-Since: 날짜 재검사

가장 흔히 쓰이는 캐시 재검사 헤더

IMS 요청으로 불린다

### 7.8.6 If-None-Match: 엔터티 태그 재검사

다음과 같이 최근 변경 일시 재검사가 적절히 행해지기 어려운 상황이 몇가지 있다

- 일정 시간 간격으로 다시 쓰여지지만 실제로는 같은 데이터를 포함하고있다.
- 어떤 문서들의 변경은 캐시가 다시 읽어들이기엔 사소한 것일 수도 있다
- 어떤 서버들은 그들이 갖고 있는 페이지에 대한 최근 변경 일시를 판별할 수 없다.
- 1초보다 작은 간격으로 생신되는 문서를 제공하는 서버들에게는 변경일에 대한 1초의 정밀도는 충분하지 않을 수 있다.

엔터티 태그가 변경되었다면, 캐시는 새 문서의 사본을 얻기위해서  If-None-Match 조건부 헤더를 사용할 수 있다.

캐시는 엔터티 태그 “v2.6”인 문서를 갖고 있다.

캐시는 원서버에게 태그가 더이상 “v2.6”이 아닌 경우에만

새객체를 달라고 요청하는 방법으로 유효한지 여부를 검사한다

변경 되지 않은 경우 304 Not Modified 응답이 반환된다.

만약 서버의 엔터티 태그가 변경되었다면 서버는 200 OK 응답으로 새 콘텐츠를 새 ETag와 함께 반환한다.

캐시가 객체에 대한 여러 개의 사본을 갖고 있는 경우，그사실을 서버에게 알리기 위해 하나의 If-None-Match 헤더에 여러 개의 엔터티 태그를 포함시킬 수 있다.

### 7.8.7 약한 검사기와 강한 검사기

서버는 때때로 모든 캐시된 사본을 무효화시키지 않고 문서를 살짝 고칠 수 있도록 허용하고 싶은 경우가 있다. 조금 변경되었더라도 `그 정도면 같은 것` 이라고 주장할 수 있도록 해주는 `약한 검사기` 를 지원한다

`강한 검사기`는 컨텐츠가 바뀔 때마다 바뀐다.

서버는 `W/` 접두사로 약한 검사기를 구분한다.

강한 엔터티 태그는 대응하는 엔터티 값이 어떻게 바뀌든 매번 반드시 같이 바뀌어야한다.

약한 엔터티 태그는 엔터티에 유의미한 변경이 있을 때마다 같이 변경되어야 한다.

원 서버는 서로 다른 두 엔터티에 대해 강한 엔터티 태그 값을 재활용해서는 안되고 약한 엔터티 태그 값이라고 할지라도 서로 의미가 다른 두 엔터티에 대해서 재활용해서는 안된다.

### 7.8.8 언제 엔터티 태그를 사용하고 언제 Last-Modified 일시를 사용하는가

- 서버가 엔터티 태그를 반환했다면, 반드시 엔터티 태그 검사기를 사용해야함
- 서버가 Last-Modified 값만을 반환했다면 클라이언트는 If-Modified-Since 검사를 사용할 수 있다.
- 모두 사용 가능하다면 적절히 응답할 수 있도록 두 가지의 재검사 정책을 모두 사용해야한다.

## 7.9 캐시 제어

문서가 만료되기 전까지 얼마나 오랫동안 캐시될 수 있게 할 것인지 서버가 설정할수 있는 여러 가지 방법을 정의한다.

- Cache-ControI: no-store 헤더를 응답에 첨부할 수 있다.
- Cache-ControI: no-cache 헤더를 응답에 첨부할 수 있다.
- Cache-ControI: must-revalidate 헤더를 응답에 첨부할 수 있다.
- Cache-ControI: max-age 헤더를 응답에 첨부할 수 있다.
- Expires 날짜 헤더를 응답에 첨부할 수 있다.
- 아무 만료 정보도 주지 않고，캐시가 스스로 체험적인(휴리스틱) 방법으로 결정하게할수있다.

### 7.9.1 no-cache와 no-store 응답 헤더

no-store와 no-cache 헤더는 캐시가 검증되지 않은 캐시된 객체로 응답하는 것을 막는다.

no-store가 표시된 응답은 캐시가 그 응답의 사본을 만드는 것을 금지한다

no-cache는 서버와 재검사를 하지 않고서는 클라이언트에 제공할 수 없다

### 7.9.4 Must-Revalidate 응답 헤더

캐시가 만료 정보를 엄격하게 따르기 원한다면 

응답 헤더에 Cache-Control: must-revalidate 을 붙일 수 있다.

이 헤더는 최초의 재검사 없이는 제공해서는 안됨을 의미

### 7.9.5 휴리스틱만료

응답이 Cache-Control: max-age 헤더나 Expires 헤더 중 어느 것도 포함하지 않고 있다면, 캐시는 경험적인 방법으로 최대 나이를 계산한다.

### 7.9.6 클라이언트 신선도 제약

클라이언트는 Cache-Control 요청 헤더를 사용하여 만료 제약을 엄격하게 하거나 느슨하게 할 수 있다.
