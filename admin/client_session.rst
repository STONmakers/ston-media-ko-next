.. _client_session:

7장. 클라이언트 세션
******************

이 장에서는 프로토콜에 따른 클라이언트 세션과 세부적인 응답을 처리하는 방식에 대해 설명한다.
너무 많은 클라이언트 세션이 유지되도록 설정하면 자칫 시스템 부하가 높아질 수 있다.
따라서 프로토콜 특성에 맞는 설정을 정확히 이해할 수 있도록 하자.


.. toctree::
   :maxdepth: 2


.. _client_session_rtmp_session:

RTMP 세션
====================================

RTMP 클라이언트와 STON 미디어서버에 사이에 생성되는 RTMP 세션에 대해 설정한다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Rtmp>
       <BufferSize>3</BufferSize>
       <ClientKeepAliveSec>10</ClientKeepAliveSec>
   </Rtmp>

-  ``<BufferSize> (기본: 3초)``
   PLAY가 시작되면 설정된 시간(초)만큼을 대역폭 제한없이 클라이언트에게 전송한다.

-  ``<ClientKeepAliveSec> (기본: 10초)``
   아무런 통신이 없는 상태로 설정된 시간이 경과하면 RTMP 클라이언트 세션에게 Ping Request을 보낸다.
   RTMP 클라이언트가 Ping Response를 보내면 세션은 유지되지만 응답하지 않을 경우 연결을 종료한다.



.. _client_session_http_session:

HTTP 세션
====================================

HTTP 클라이언트가 STON 미디어서버에 접속하면 HTTP세션이 생성된다.
HTTP 클라이언트는 HTTP 세션을 통해 서버에 저장된 여러 콘텐츠를 서비스 받는다.
요청부터 응답까지를 하나의 **HTTP 트랜잭션** 이라고 부른다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Http>
       <ClientKeepAliveSec>10</ClientKeepAliveSec>
       <ConnectionHeader>keep-alive</ConnectionHeader>
       <KeepAliveHeader Max="0">ON</KeepAliveHeader>
   </Http>

-  ``<ClientKeepAliveSec> (기본: 10초)``
   아무런 통신이 없는 상태로 설정된 시간이 경과하면 연결을 종료한다.

-  ``<ConnectionHeader> (기본: keep-alive)``
   HTTP 클라이언트에게 보내는 응답의 Connection헤더( ``keep-alive`` 또는 ``close`` )를 설정한다.

-  ``<KeepAliveHeader>``

   - ``ON (기본)`` HTTP응답에 Keep-Alive헤더를 명시한다.
     ``Max (기본: 0)`` 를 0보다 크게 설정하면 Keep-Alive헤더의 값으로 ``Max`` 값이 명시된다.
    이후 HTTP 트랜잭션이 발생할때마다 1씩 차감된다.

   - ``OFF`` HTTP응답에 Keep-Alive헤더를 생략한다.


.. _client_session_http_session_lifecyle:

세션 유지정책
---------------------

STON 미디어서버의 HTTP 세션 유지정책은 Apache의 정책을 따른다.
HTTP 헤더 값에 따른 변수가 많아 다소 복잡하다.
HTTP 세션 유지정책에 영향을 주는 요소는 다음과 같다.

- HTTP 클라이언트 요청에 명시된 Connection헤더 ("Keep-Alive" 또는 "Close")
- 가상호스트 ``<Connection>`` 설정
- 가상호스트 세션 Keep-Alive시간 설정
- 가상호스트 ``<Keep-Alive>`` 설정


1. HTTP 클라이언트 요청에 "Connection: Close"로 명시되어 있는 경우 ::

      GET / HTTP/1.1
      ...(생략)...
      Connection: Close

   이같은 요청에 대해서는 가상호스트 설정여부와 상관없이
   "Connection: Close"로 응답한다. Keep-Alive헤더는 명시되지 않는다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Close

   이 HTTP 트랜잭션이 완료되면 HTTP 연결을 종료한다.


2. ``<ConnectionHeader>`` 가 ``Close`` 로 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>

      <ConnectionHeader>Close</ConnectionHeader>

   HTTP 클라이언트 요청과 상관없이 "Connection: Close"로 응답한다.
   Keep-Alive헤더는 명시되지 않는다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Close


3. ``<KeepAliveHeader>`` 가 ``OFF`` 로 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>

      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <KeepAliveHeader>OFF</KeepAliveHeader>

   Keep-Alive헤더가 명시되지 않는다. HTTP 세션은 지속적으로 재사용가능하다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Keep-Alive


4. ``<KeepAliveHeader>`` 가 ``ON`` 으로 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>

      <HttpClientKeepAliveSec>10</HttpClientKeepAliveSec>
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <KeepAliveHeader>ON</KeepAliveHeader>

   Keep-Alive헤더가 명시된다.
   timeout값은 세션 Keep-Alive시간 설정을 사용한다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10

   .. note::

      < ``<Keep-Alive>`` 와 ``<HttpClientKeepAliveSec>`` 의 관계 >

      ``<Keep-Alive>`` 설정시 ``<HttpClientKeepAliveSec>`` 를 참고하지만 ``<HttpClientKeepAliveSec>`` 는 보다 근본적인 문제와 관련이 있다.
      성능이나 자원적으로 가장 중요한 이슈는 Idle세션(=HTTP 트랜잭션이 발생되지 않는 세션)의 정리시점을 잡는 것이다.
      HTTP 헤더 설정은 동적으로 변경되거나 때로 생략될 수 있지만 Idle세션 정리는 훨씬 민감한 문제이다.
      이런 이유 때문에 ``<HttpClientKeepAliveSec>`` 는 ``<KeepAliveHeader>`` 에 통합되지 않고 별도로 존재한다.


5. ``<KeepAliveHeader>`` 의 ``Max`` 속성이 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options>
      # vhosts.xml - <Vhosts><Vhost><Options>

      <HttpClientKeepAliveSec>10</HttpClientKeepAliveSec>
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <KeepAliveHeader Max="50">ON</KeepAliveHeader>

   Keep-Alive헤더에 max값을 명시한다.
   이 세션은 max회만큼 사용이 가능하며 HTTP 트랜잭션이 진행될때마다 1씩 감소된다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10, max=50


6. Keep-Alive의 max가 만료된 경우 ::

   위의 설정대로 max가 설정되었다면 max는 점차 줄어 다음처럼 1까지 도달하게 된다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10, max=1

   이 응답은 현재 세션으로 앞으로 1번 HTTP 트랜잭션 진행이 가능하다는 의미이다.
   이 세션으로 HTTP 요청이 한번 더 진행될 경우 다음과 같이 "Connection: Close"로 응답한다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Close



.. _client_session_http_session_upfrontheader:

MP4 헤더위치 변경
---------------------

MP4(M4A 포함)의 경우 인코딩 과정 중에는 헤더를 완성할 수 없기 때문에 완료 후 파일의 맨 뒤에 붙인다.
헤더를 앞으로 옮기려면 별도의 작업이 필요하다.
헤더가 뒤에 있다면 이를 지원하지 않는 플레이어에서 Pseudo-Streaming이 불가능하다.
헤더위치 변경을 통해 HTTP Pseudo-Streaming을 간편하게 지원할 수 있다.

헤더위치 변경은 전송단계에서만 발생할 뿐 원본의 형태를 변경하지 않는다.
별도의 저장공간을 사용하지도 않는다. ::

   # server.xml - <Server><VHostDefault><Options><Http>
   # vhosts.xml - <Vhosts><Vhost><Options><Http>

   <UpfrontMP4Header>ON</UpfrontMP4Header>

-  ``<UpfrontMP4Header>``

   - ``ON (기본)`` 확장자가 .mp4 또는 .m4a이고 헤더가 뒤에 있다면 헤더를 앞으로 옮겨서 전송한다.

   - ``OFF`` 아무 것도 하지 않는다.

처음 요청되는 콘텐츠의 헤더를 앞으로 옮겨야 한다면 헤더를 옮기기위해 필요한 부분을 우선적으로 다운로드 받는다.
STON 미디어서버는 아주 영리할뿐만 아니라 빠르게 동작한다.
커튼 뒤의 복잡한 과정과는 상관없이, HTTP Pseudo-Streaming은 자연스럽게 이루어진다.

.. note::

   분석할 수 없거나 깨진 파일이라면 원본형태 그대로 서비스된다.



.. _client_session_http_session_headermodify:

요청/응답 헤더변경
---------------------

HTTP 클라이언트 요청과 응답을 특정 조건에 따라 변경한다. ::

   # server.xml - <Server><VHostDefault><Options><Http>
   # vhosts.xml - <Vhosts><Vhost><Options><Http>

   <ModifyHeader FirstOnly="OFF">OFF</ModifyHeader>

-  ``<ModifyHeader>``

   -  ``OFF (기본)`` 변경하지 않는다.

   -  ``ON`` 헤더 변경조건에 따라 헤더를 변경한다.

헤더 변경시점을 정확히 이해하자.

-  **HTTP 요청헤더 변경시점**

   HTTP 요청을 최초로 인식하는 시점에 헤더를 변경한다.
   헤더가 변경되었다면 변경된 상태로 Cache 모듈에서 처리된다.
   단, Host헤더와 URI는 변경할 수 없다.

-  **HTTP 응답헤더 변경시점**

   HTTP 응답 직전에 헤더를 변경한다.
   단, Content-Length는 변경할 수 없다.


헤더 변경조건은 /svc/{가상호스트 이름}/http_headers.txt에 설정한다.
헤더는 멀티로 설정이 가능하므로 조건과 일치한다면 모든 변경설정이 순차적으로 모두 적용된다.

최초 조건에만 변경을 원할 경우 ``FirstOnly`` 속성을 ``ON`` 으로 설정한다.
서로 다른 조건이 같은 헤더를 변경하는 경우 ``set`` 에 의해 Last-Win이 되거나 명시적으로 ``put`` ``append`` 할 수 있다. ::

   # /svc/www.example.com/http_headers.txt
   # 구분자는 콤마(,)이다.

   # 요청변경
   # {Match}, {$REQ}, {Action(set|put|append|unset)} 순서로 표기한다.
   $IP[192.168.1.1], $REQ[SOAPAction], unset
   $IP[192.168.2.1-255], $REQ[accept-encoding: gzip], set
   $IP[192.168.3.0/24], $REQ[cache-control: no-cache], append
   $IP[192.168.4.0/255.255.255.0], $REQ[x-custom-header], unset
   $IP[AP], $REQ[X-Forwarded-For], unset
   $HEADER[user-agent: *IE6*], $REQ[accept-encoding], unset
   $HEADER[via], $REQ[via], unset
   $URL[/source/*.zip], $REQ[accept-encoding: deflate], set

   # 응답변경
   # {Match}, {$RES}, {Action(set|put|append|unset)}, {condition} 순서로 표기한다.
   # {condition}은 특정 응답코드에 한하여 헤더를 변경할 수 있지만 필수는 아니다.
   $IP[192.168.1.1], $RES[via: STON for CDN], set
   $IP[192.168.2.1-255], $RES[X-Cache], unset, 200
   $IP[192.168.3.0/24], $RES[cache-control: no-cache, private], append, 3xx
   $IP[192.168.4.0/255.255.255.0], $RES[x-custom-header], unset
   $HEADER[user-agent: *IE6*], $RES[vary], unset
   $HEADER[x-custom-header], $RES[cache-control: no-cache, private], append, 5xx
   $URL[/source/*], $RES[cache-control: no-cache], set, 404
   /secure/*.dat, $RES[x-custom], unset, 200
   /*.mp4, $RES[Access-Control-Allow-Origin: example1.com], set
   /*.mp4, $RES[Access-Control-Allow-Origin: example2.com], put

{Match}는 IP, GeoIP, Header, URL 4가지로 설정이 가능하다.

-  **IP**
   $IP[...]로 표기하며 IP, IP Range, Bitmask, Subnet 네 가지 형식을 지원한다.

-  **GeoIP**
   $IP[...]로 표기하며 반드시 :ref:`access-control-geoip` 가 설정되어 있어야 한다.
   국가코드는 `ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ 와 `ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ 를 지원한다.

-  **Header**
   $HEADER[Key : Value]로 표기한다.
   Value는 명확한 표현과 패턴을 지원한다.
   Value가 생략된 경우에는 Key에 해당하는 헤더의 존재유무를 조건으로 판단한다.

-  **URL**
   $URL[...]로 표기하며 생략이 가능하다. 명확한 표현과 패턴을 인식한다.

{$REQ}와 {$RES}는 헤더변경 방법을 설정한다.
``set`` ``put`` ``append`` 의 경우 {Key: Value}로 설정하며,
Value가 입력되지 않은 경우 빈 값("")이 입력된다.
``unset`` 의 경우 {Key}만 입력한다.

{Action}은 ``set`` , ``put`` , ``append`` , ``unset``  4가지로 설정이 가능하다.

-  ``set``  요청/응답 헤더에 설정되어 있는 Key와 Value를 헤더에 추가한다.
   이미 같은 Key가 존재한다면 이전 값을 덮어쓴다.

-  ``put``  ( ``set`` 과 유사하나) 같은 Key가 존재하면, 덮어쓰지 않고 새로운 라인으로 붙여 넣는다.

-  ``append`` ( ``set`` 과 유사하나) 같은 Key가 존재하면, 기존의 Value와 설정된 Value사이에 Comma(,)로 구분하여 값을 결합한다.

-  ``unset`` 요청/응답 헤더에 설정되어 있는 Key에 해당하는 헤더를 삭제한다.

{Condition}은 200이나 304같은 구체적인 응답 코드외에 2xx, 3xx, 4xx, 5xx처럼 응답코드 계열조건으로 설정한다.
{Match}와 일치하더라도 {Condition}과 일치하지 않는다면 변경이 반영되지 않는다.
{Condition}이 생략된 경우 응답코드를 검사하지 않는다.




.. _client_session_hls_session:

HLS 세션
====================================

HLS 클라이언트가 STON 미디어서버에 접속하면 HLS세션이 생성된다.
HTTP 클라이언트는 HTTP 세션을 통해 서버에 저장된 여러 콘텐츠를 서비스 받는다.
요청부터 응답까지를 하나의 **HTTP 트랜잭션** 이라고 부른다. ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Http>
       <ClientKeepAliveSec>10</ClientKeepAliveSec>
       <ConnectionHeader>keep-alive</ConnectionHeader>
       <KeepAliveHeader Max="0">ON</KeepAliveHeader>
   </Http>

-  ``<ClientKeepAliveSec> (기본: 10초)``
   아무런 통신이 없는 상태로 설정된 시간이 경과하면 연결을 종료한다.

-  ``<ConnectionHeader> (기본: keep-alive)``
   HTTP 클라이언트에게 보내는 응답의 Connection헤더( ``keep-alive`` 또는 ``close`` )를 설정한다.

-  ``<KeepAliveHeader>``

   - ``ON (기본)`` HTTP응답에 Keep-Alive헤더를 명시한다.
     ``Max (기본: 0)`` 를 0보다 크게 설정하면 Keep-Alive헤더의 값으로 ``Max`` 값이 명시된다.
    이후 HTTP 트랜잭션이 발생할때마다 1씩 차감된다.

   - ``OFF`` HTTP응답에 Keep-Alive헤더를 생략한다.

