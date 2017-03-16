.. _multi_protocol:

5장. 클라이언트 요청/응답
******************

이 장에서는 클라이언트 연결과 요청을 세밀하게 처리하는 방법에 대해 설명한다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Name="www.example.com/bar">
            <Options>
                <Rtmp>...</Rtmp>
                <Http>...</Http>
                <Hls>...</Hls>
            </Options>
        </Vhost>
    </Vhosts>

이 장의 모든 설정은 ``<Options>`` 하위 프로토콜에 따라 분류된다.
프로토콜마다 설정할 수 있는 범위가 다르지만 HTTP기반의 프로토콜(HLS, MPEG-DASH)은 많은 공통점을 가진다.


.. toctree::
   :maxdepth: 2


.. _multi_protocol_rtmp_session:

RTMP 클라이언트
====================================

RTMP 클라이언트에 대해 설정한다. ::

   # server.xml - <Server><VHostDefault><Options><Rtmp>
   # vhosts.xml - <Vhosts><Vhost><Options><Rtmp>
   
   
   <BufferSize>3</BufferSize>
   <ClientKeepAliveSec>10</ClientKeepAliveSec>

-  ``<BufferSize> (기본: 3초)``
   PLAY가 시작되면 설정된 시간(초)만큼을 대역폭 제한없이 클라이언트에게 전송한다.

-  ``<ClientKeepAliveSec> (기본: 10초)``
   아무런 통신이 없는 상태로 설정된 시간(초)이 경과하면 RTMP 클라이언트에게 Ping Request을 보낸다.
   RTMP 클라이언트가 Ping Response를 보내지 않으면 연결을 종료한다.



.. _multi_protocol_http_session:

HTTP 클라이언트
====================================

HTTP 클라이언트가 요청을 보내고 응답이 완료되기 까지를 HTTP 트랜잭션이라고 부른다.
HTTP 클라이언트는 하나의 연결을 통해 여러 번의 HTTP 트랜잭션을 진행한다. ::

   # server.xml - <Server><VHostDefault><Options><Http>
   # vhosts.xml - <Vhosts><Vhost><Options><Http>
   
   <ClientKeepAliveSec>10</ClientKeepAliveSec>
   <ConnectionHeader>keep-alive</ConnectionHeader>
   <KeepAliveHeader Max="0">ON</KeepAliveHeader>

-  ``<ClientKeepAliveSec> (기본: 10초)``
   아무런 통신이 없는 상태로 설정된 시간이 경과하면 연결을 종료한다.

-  ``<ConnectionHeader> (기본: keep-alive)``
   HTTP 클라이언트에게 보내는 응답의 Connection헤더( ``keep-alive`` 또는 ``close`` )를 설정한다.

-  ``<KeepAliveHeader>``

   - ``ON (기본)`` HTTP응답에 Keep-Alive헤더를 명시한다.
     ``Max (기본: 0)`` 를 0보다 크게 설정하면 Keep-Alive헤더의 값으로 ``Max`` 값이 명시된다. 이후 HTTP 트랜잭션이 진행될때마다 1씩 차감된다.

   - ``OFF`` HTTP응답에 Keep-Alive헤더를 생략한다.


.. _multi_protocol_http_session_lifecyle:

연결 유지정책
---------------------

HTTP 연결 유지정책은 Apache의 정책을 따른다.
HTTP 헤더 값에 따른 변수가 많아 다소 복잡하다.

- HTTP 클라이언트 요청에 명시된 Connection헤더 ("Keep-Alive" 또는 "Close")
- 가상호스트 ``<ConnectionHeader>`` 설정
- 가상호스트 연결 Keep-Alive시간 설정
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

      # server.xml - <Server><VHostDefault><Options><Http>
      # vhosts.xml - <Vhosts><Vhost><Options><Http>

      <ConnectionHeader>Close</ConnectionHeader>

   HTTP 클라이언트 요청과 상관없이 "Connection: Close"로 응답한다.
   Keep-Alive헤더는 명시되지 않는다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Close


3. ``<KeepAliveHeader>`` 가 ``OFF`` 로 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options><Http>
      # vhosts.xml - <Vhosts><Vhost><Options><Http>

      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <KeepAliveHeader>OFF</KeepAliveHeader>

   Keep-Alive헤더가 명시되지 않는다. HTTP 연결은 지속적으로 재사용가능하다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Keep-Alive


4. ``<KeepAliveHeader>`` 가 ``ON`` 으로 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options><Http>
      # vhosts.xml - <Vhosts><Vhost><Options><Http>

      <HttpClientKeepAliveSec>10</HttpClientKeepAliveSec>
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <KeepAliveHeader>ON</KeepAliveHeader>

   Keep-Alive헤더가 명시된다.
   timeout값은 연결 Keep-Alive시간 설정을 사용한다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Keep-Alive
      Keep-Alive: timeout=10


5. ``<KeepAliveHeader>`` 의 ``Max`` 속성이 설정된 경우 ::

      # server.xml - <Server><VHostDefault><Options><Http>
      # vhosts.xml - <Vhosts><Vhost><Options><Http>

      <HttpClientKeepAliveSec>10</HttpClientKeepAliveSec>
      <ConnectionHeader>Keep-Alive</ConnectionHeader>
      <KeepAliveHeader Max="50">ON</KeepAliveHeader>

   Keep-Alive헤더에 max값을 명시한다.
   이 연결은 max회만큼 사용이 가능하며 HTTP 트랜잭션이 진행될때마다 1씩 감소된다. ::

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

   이 응답은 현재 연결으로 앞으로 1번 HTTP 트랜잭션 진행이 가능하다는 의미이다.
   이 연결으로 HTTP 요청이 한번 더 진행될 경우 다음과 같이 "Connection: Close"로 응답한다. ::

      HTTP/1.1 200 OK
      ...(생략)...
      Connection: Close



.. _multi_protocol_http_session_upfrontheader:

MP4 헤더위치 변경
---------------------

MP4파일의 헤더가 뒤에 있다면 플레이어에 따라 HTTP Pseudo-Streaming이 원활하지 않을 수 있다.
전송 단계에서 헤더 위치를 앞으로 배치하면 이런 문제를 해결할 수 있다. ::

   # server.xml - <Server><VHostDefault><Options><Http>
   # vhosts.xml - <Vhosts><Vhost><Options><Http>

   <UpfrontMP4Header>ON</UpfrontMP4Header>

-  ``<UpfrontMP4Header>``

   - ``ON (기본)`` 확장자가 .mp4, .m4a인 파일의 헤더가 뒤에 있다면 앞으로 옮겨서 전송한다.

   - ``OFF`` 아무 것도 하지 않는다.

처음 요청되는 콘텐츠의 헤더를 앞으로 옮겨야 한다면 필요한 부분을 우선적으로 다운로드 받는다.
헤더위치 변경은 전송단계에서만 발생할 뿐 원본의 형태를 변경하거나 별도의 저장공간을 사용하지 않는다.


.. note::

   분석할 수 없거나 깨진 파일이라면 원본형태 그대로 서비스된다.



.. _multi_protocol_http_session_bt:

Bandwidth Throttling
--------------------------

Bandwidth Throttling(이하 쓰로틀링)이란 (각 연결마다) 대역폭을 최적화하여 전송하는 기능이다.
일반적인 미디어 파일의 내부는 다음과 같이 헤더, V(Video), A(Audio)로 구성되어 있다.

.. figure:: img/conf_media_av.png
   :align: center

   헤더는 쓰로틀링의 대상이 아니다.

헤더는 재생시간이 길거나 키 프레임(Key Frame)주기가 짧을수록 커진다.
그러므로 인식할 수 있는 미디어 파일이라면 원활한 재생을 위해 헤더는 대역폭 제한없이 전송한다.
다음 그림처럼 헤더가 완전히 전송된 뒤 쓰로틀링이 시작된다.

.. figure:: img/conf_bandwidththrottling2.png
   :align: center

   최적화된 대역폭 활용

::

   # server.xml - <Server><VHostDefault><Options><Http>
   # vhosts.xml - <Vhosts><Vhost><Options><Http>

   <BandwidthThrottling>
      <Settings>
         <Bandwidth Unit="kbps">2000</Bandwidth>
         <Ratio>150</Ratio>
         <Boost>5</Boost>
      </Settings>
      <Throttling>OFF</Throttling>
   </BandwidthThrottling>

``<BandwidthThrottling>`` 태그 하위에 기본동작을 설정한다.

-  ``<Settings>``

   기본 동작을 설정한다.

   -  ``<Bandwidth> (기본: 2000 Kbps)``
      클라이언트 전송 대역폭을 설정한다.
      ``Unit`` 속성을 통해 기본 단위( ``kbps`` , ``mbps`` , ``bytes`` , ``kb`` , ``mb`` )를 설정한다.

   -  ``<Ratio> (기본: 150 %)``
      ``<Bandwidth>`` 설정에 비율을 반영하여 대역폭을 설정한다.

   -  ``<Boost> (기본: 5 초)``
      일정 시간만큼의 데이터를 속도제한 없이 클라이언트에게 전송한다.
      데이터의 양은 ``<Boost>`` X ``<Bandwidth>`` X ``<Ratio>`` 공식으로 계산한다.

-  ``<Throttling>``

   -  ``OFF (기본)`` BT를 적용하지 않는다.
   -  ``ON`` 조건목록과 일치하면 BT를 적용한다.


쓰로틀링은 조건목록을 설정해야 동작한다.
설정된 순서대로 우선순위를 가진다.
전송 정책은 /svc/{가상호스트 이름}/http_throttling.txt 에 설정한다. ::

   # /svc/www.example.com/http_throttling.txt
   # 구분자는 콤마(,)이며 {조건},{Bandwidth},{Ratio},{Boost} 순서로 표기한다.
   # {조건}을 제외한 모든 필드는  생략가능하다.
   # 생략된 필드는 ``<Settings>`` 에 설정된 기본 값을 사용한다.
   # 모든 조건표현은 acl.txt설정과 동일하다.
   # {Bandwidth} 단위는 ``<Settings>`` ``<Bandwidth>`` 의 ``Unit`` 속성을 사용한다.

   # 3초의 데이터를 속도 제한없이 전송한 후 3Mbps(3000Kbps = 2000Kbps X 150%)로 클라이언트에게 전송한다.
   $IP[192.168.1.1], 2000, 150, 3

   # bandwidth만 정의. 5(기본)초의 데이터를 속도 제한없이 전송한 후 800 Kbps로 클라이언트에게 전송한다.
   !HEADER[referer], 800

   # boost만 정의. 10초의 데이터를 속도 제한없이 전송한 후 1000 Kbps로 클라이언트에게 전송한다.
   HEADER[cookie], , , 10

   # 확장자가 m4a인 경우 BT를 적용하지 않는다.
   $URL[*.m4a], no

미디어 파일(MP4, M4A, MP3)을 분석하면 Encoding Rate로부터 Bandwidth를 얻을 수 있다.
접근되는 콘텐츠의 확장자는 반드시 .mp4, .m4a, .mp3 중 하나여야 한다.
동적으로 Bandwidth를 추출하려면 다음과 같이 Bandwidth뒤에 **x** 를 붙인다. ::

   # /vod/*.mp4 파일에 대한 접근이라면 bandwidth를 구한다. 구할 수 없다면 1000을 bandwidth로 사용한다.
   $URL[/vod/*.mp4], 1000x, 120, 5

   # user-agent헤더가 없다면 bandwidth를 구한다. 구할 수 없다면 500을 bandwidth로 사용한다.
   !HEADER[user-agent], 500x

   # /low_quality/* 파일에 대한 접근이라면 bandwidth를 구한다. 구할 수 없다면 기본 값을 bandwidth로 사용한다.
   $URL[/low_quality/*], x, 200

HTTP QueryString을 사용하여 ``<Bandwidth>`` , ``<Ratio>`` , ``<Boost>`` 를 URL로 지정할 수 있다.
QueryString 조건은 http_throttling.txt보다 우선한다. ::

   # server.xml - <Server><VHostDefault><Options><Http>
   # vhosts.xml - <Vhosts><Vhost><Options><Http>

   <BandwidthThrottling>
      <Settings>
         <Bandwidth Param="mybandwidth" Unit="mbps">2</Bandwidth>
         <Ratio Param="myratio">100</Ratio>
         <Boost Param="myboost">3</Boost>
      </Settings>
      <Throttling QueryString="ON">ON</Throttling>
   </BandwidthThrottling>

-  ``<Bandwidth>`` , ``<Ratio>`` , ``<Boost>`` 의 ``Param``

    용도별 HTTP QueryString 키를 설정한다.

-  ``<Throttling>`` 의 ``QueryString``

   - ``ON (기본)`` QueryString으로 조건을 재정의한다.

   - ``OFF`` QueryString으로 조건을 재정의하지 않는다.

위와 같이 설정되어 있다면 다음과 같이 클라이언트가 요청한 URL에 따라 쓰로틀링이 동적으로 설정된다. ::

    # 10초의 데이터를 속도 제한없이 전송한 후 1.3Mbps(1mbps X 130%)로 클라이언트에게 전송한다.
    http://www.example.com/bar/mp4:trip.mp4?myboost=10&mybandwidth=1&myratio=130

반드시 모든 파라미터를 명시할 필요는 없다. ::

    http://www.example.com/bar/mp4:trip.mp4?myratio=150

위와 같이 일부 조건이 생략된 경우 나머지 조건(여기서는 bandwidth, boost)을 결정하기 위해 조건목록을 검색한다.
여기서도 적합한 조건을 찾지 못하는 경우 ``<Settings>`` 에 설정된 기본 값을 사용한다.
QueryString이 일부 존재하더라도 조건목록에서 미적용 옵션(no)이 설정되어 있다면 쓰로틀링은 적용되지 않는다.

QueryString을 사용하므로 자칫 :ref:`caching-policy-applyquerystring` 과 혼동을 일으킬 소지가 있다.
:ref:`caching-policy-applyquerystring` 이 ``ON`` 인 경우 클라이언트가 요청한 URL의 QueryString이
모두 인식되지만 ``BoostParam`` , ``BandwidthParam`` , ``RatioParam`` 은 제외된다. ::

   GET /bar/mp4:trip.mp4?mybandwidth=2000&myratio=130&myboost=10
   GET /bar/mp4:trip.mp4?tag=3277&myboost=10&date=20130726

예를 들어 위와 같은 입력은 쓰로틀링 정책을 결정하는데 쓰일 뿐 Caching-Key를 생성하거나 HTTP 원본서버로 요청을 보낼 때는 제거된다.
즉 각각 다음과 같이 인식된다. ::

    GET /trip.mp4
    GET /trip.mp4?tag=3277&date=20130726



.. _multi_protocol_http_session_headermodify:

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



.. _multi_protocol_http_session_acceptencoding:

Accept-Encoding 헤더
---------------------

같은 URL에 대한 HTTP요청이라도 Accept-Encoding헤더의 존재 유무에 따라 다른 콘텐츠가 캐싱될 수 있다.
원본서버에 요청을 보내는 시점에 압축여부를 알 수 없다.
응답을 받았다고해도 압축여부를 매번 비교할 수도 없다.

   .. figure:: img/acceptencoding.png
      :align: center

      원본서버가 어떤 응답을 줄지 알 수 없다.

::

    # server.xml - <Server><VHostDefault><Options><Http>
    # vhosts.xml - <Vhosts><Vhost><Options><Http>

    <AcceptEncoding>ON</AcceptEncoding>

-  ``<AcceptEncoding>``

   -  ``OFF (기본)`` HTTP 클라이언트가 보내는 Accept-Encoding 헤더를 무시한다.

   -  ``ON`` HTTP 클라이언트가 보내는 Accept-Encoding 헤더를 인식한다.

원본서버에서 압축을 지원하지 않거나, 압축이 필요없는 대용량 파일의 경우 ``OFF`` 로 설정하는 것이 바람직하다.


.. _multi_protocol_http_session_server:

Server 헤더
---------------------

HTTP 클라이언트에게 보내는 HTTP 응답에 Server 헤더 명시여부를 설정한다. ::

   # server.xml - <Server><VHostDefault><Options><Http>
   # vhosts.xml - <Vhosts><Vhost><Options><Http>

   <ServerHeader>ON</ServerHeader>

-  ``<ServerHeader>``

   -  ``ON (기본)`` 원본서버의 Server헤더를 명시한다. ::

   -  ``OFF``  Server헤더를 생략한다.



.. _multi_protocol_http_session_originalheader:

원본 비표준 헤더
---------------------

성능과 보안상의 이유로 원본서버가 보내는 헤더 중 표준헤더만을 선택적으로 인식한다. ::

   # server.xml - <Server><VHostDefault><Options><Http>
   # vhosts.xml - <Vhosts><Vhost><Options><Http>

   <OriginalHeader>OFF</OriginalHeader>

-  ``<OriginalHeader>``

   -  ``OFF (기본)`` 표준헤더가 아니라면 무시한다.

   -  ``ON`` cookie, set-cookie, set-cookie2를 제외한 모든 헤더를 저장하여 클라이언트에게 전달한다.
      단, 메모리와 저장비용을 좀 더 소비한다.




.. _multi_protocol_hls_session:

HLS 클라이언트
====================================

미디어 파일을 HLS(HTTP Live Streaming)로 서비스한다.
원본서버는 더 이상 HLS 서비스를 위해 파일을 분할/저장할 필요가 없다.

..  note::

    STON 미디어 서버가 지원하는 HLS는 Elementary Stream(Video 또는 Audio)을 변환하는 트랜스코딩(Transcoding)이 아니다.
    그러므로 HLS에 적합한 형식으로 인코딩된 파일에 한해서 원활한 단말 재생이 가능하다.
    인코딩이 적합하지 않을 경우 화면이나 깨지거나 소리가 재생되지 않을 수 있다.
    Apple에서 밝히고 있는 Video/Audio 인코딩 규격은 다음과 같다.

    What are the specifics of the video and audio formats supported?
    Although the protocol specification does not limit the video and audio formats, the current Apple implementation supports the following formats:

    [Video]
    H.264 Baseline Level 3.0, Baseline Level 3.1, Main Level 3.1, and High Profile Level 4.1.

    [Audio]
    HE-AAC or AAC-LC up to 48 kHz, stereo audio
    MP3 (MPEG-1 Audio Layer 3) 8 kHz to 48 kHz, stereo audio
    AC-3 (for Apple TV, in pass-through mode only)

    Note: iPad, iPhone 3G, and iPod touch (2nd generation and later) support H.264 Baseline 3.1. If your app runs on older versions of iPhone or iPod touch, however, you should use H.264 Baseline 3.0 for compatibility. If your content is intended solely for iPad, Apple TV, iPhone 4 and later, and Mac OS X computers, you should use Main Level 3.1.


기존 방식의 HTTP Pseudo-Streaming과 HLS를 위해 다음과 같이 원본파일과 분할된 파일이 각각 존재해야 한다.
이 방식의 단점은 분할 파일로 인한 저장공간 점유와 관리의 어려움이다.

.. figure:: img/conf_media_mp4hls1.png
   :align: center

   수고가 많은 HLS

STON 미디어 서버는 원본파일로부터 HLS서비스에 필요한 파일을 동적으로 생성한다.

.. figure:: img/conf_media_mp4hls2.png
   :align: center

   똑똑한 HLS

모든 .m3u8/.ts파일은 원본파일에서 파생되며 별도의 저장공간을 소비하지 않는다.
서비스 즉시 임시적으로 생성되며 서비스되지 않을 때 자동으로 없어진다.


.. _multi_protocol_hls_session_clientkeepalivesec:

연결 유지시간
---------------------
::

   # server.xml - <Server><VHostDefault><Options><Hls>
   # vhosts.xml - <Vhosts><Vhost><Options><Hls>
   
   <ClientKeepAliveSec>30</ClientKeepAliveSec>

-  ``<ClientKeepAliveSec> (기본: 30초)``
   아무런 통신이 없는 상태로 설정된 시간이 경과하면 연결을 종료한다.



.. _multi_protocol_hls_session_mp4:

MP4 분할
---------------------
MP4 파일을 MPEG2-TS(Transport Stream)로 변환하고 인덱스 파일을 구성하는 정책을 설정한다.  ::

   # server.xml - <Server><VHostDefault><Options><Hls>
   # vhosts.xml - <Vhosts><Vhost><Options><Hls>

   <MP4 Status="Active">
      <Index Ver="3" Alternates="ON">index.m3u8</Index>
      <Sequence>0</Sequence>
      <Duration>10</Duration>
      <AlternatesName>playlist.m3u8</AlternatesName>
   </MP4>

-  ``<MP4>``

   - ``Status (기본: Active)`` 값이 ``Inactive`` 라면 비활성화된다.

-  ``<Index> (기본: index.m3u8)`` HLS 인덱스(.m3u8) 파일명

   - ``Ver (기본 3)`` 인덱스 파일 버전.
     3인 경우 ``#EXT-X-VERSION:3`` 헤더가 명시되며 ``#EXTINF`` 의 시간 값이 소수점 3째 자리까지 표시된다.
     1인 경우 ``#EXT-X-VERSION`` 헤더가 없으며, ``#EXTINF`` 의 시간 값이 정수(반올림)로 표시된다.

   - ``Alternates (기본: ON)`` Stream Alternates 사용여부.

     .. figure:: img/hls_alternates_on.png
        :align: center

        ON. ``<AlternatesName>`` 에서 TS목록을 서비스한다.

     .. figure:: img/hls_alternates_off.png
        :align: center

        OFF. ``<Index>`` 에서 TS목록을 서비스한다.

-  ``<Sequence> (기본: 0)`` .ts 파일의 시작 번호. 이 수를 기준으로 순차적으로 증가한다.

-  ``<Duration> (기본: 10초)`` MP4를 분할(Segmentation)하는 기준 시간(초).
   분할의 기준은 Video/Audio의 KeyFrame이다.
   KeyFrame은 들쭉날쭉할 수 있으므로 정확히 분할되지 않을 수 있다.
   만약 10초로 분할하려는데 KeyFrame이 9초와 12초에 있다면 가까운 값(9초)을 선택한다.

-  ``<AlternatesName> (기본: playlist.m3u8)`` Stream Alternates 파일명. ::

      http://www.example.com/bar/mp4:trip.mp4/playlist.m3u8


다음 URL이 호출되면 HTTP 원본서버의 /trip.mp4로부터 인덱스 파일을 생성한다. ::

   http://www.example.com/bar/mp4:trip.mp4/index.m3u8

``Alternates`` 속성이 ON이라면 ``<Index>`` 파일은 ``<AlternatesName>`` 파일을 서비스한다. ::

   #EXTM3U
   #EXT-X-VERSION:3
   #EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=200000,RESOLUTION=720x480
   /bar/mp4:trip.mp4/playlist.m3u8

``#EXT-X-STREAM-INF`` 의 Bandwidth와 Resolution은 영상을 분석하여 동적으로 제공한다.


최종적으로 생성된 .ts 목록(버전 3)은 다음과 같다. ::

   #EXTM3U
   #EXT-X-TARGETDURATION:10
   #EXT-X-VERSION:3
   #EXT-X-MEDIA-SEQUENCE:0
   #EXTINF:11.637,
   /bar/mp4:trip.mp4/0.ts
   #EXTINF:10.092,
   /bar/mp4:trip.mp4/1.ts
   #EXTINF:10.112,
   /bar/mp4:trip.mp4/2.ts

   ... (중략)...

   #EXTINF:10.847,
   /bar/mp4:trip.mp4/161.ts
   #EXTINF:9.078,
   /bar/mp4:trip.mp4/162.ts
   #EXT-X-ENDLIST


.. _multi_protocol_hls_session_mp3:

MP3 분할
---------------------

MP3 파일을 분할하고 인덱스 파일을 구성하는 정책을 설정한다.  ::

   # server.xml - <Server><VHostDefault><Options><Hls>
   # vhosts.xml - <Vhosts><Vhost><Options><Hls>

   <MP3 Status="Active" SegmentType="TS">
       <Index Ver="3" Alternates="ON">index.m3u8</Index>
       <Sequence>0</Sequence>
       <Duration>10</Duration>
       <AlternatesName>playlist.m3u8</AlternatesName>
   </MP3>

-  ``<MP3>``

   - ``SegmentType (기본: TS)`` 분할 포맷을 설정한다. (TS 또는 MP3)

그외 모든 설정과 동작방식은 ``<MP4>`` 와 동일하다.




.. _multi_protocol_hls_session_segmentation:

키 프레임과 분할
---------------------

키 프레임에 따라 분할(Segmentation)에는 3가지 경우가 있다.

-  **KeyFrame 간격보다** ``<Duration>`` **설정이 큰 경우**
   KeyFrame이 3초, ``<Duration>`` 이 20초라면 20초를 넘지 않는 KeyFrame의 배수인 18초로 분할된다.

-  **KeyFrame 간격과** ``<Duration>`` **이 비슷한 경우**
   KeyFrame이 9초, ``<Duration>`` 이 10초라면 10초를 넘지 않는 KeyFrame의 배수인 9초로 분할된다.

-  **KeyFrame 간격이** ``<Duration>`` **설정보다 큰 경우**
   KeyFrame단위로 분할된다.

다음 클라이언트 요청에 대해 STON이 어떻게 동작하는지 이해해보자. ::

   GET /bar/mp4:trip.mp4/99.ts HTTP/1.1
   Range: bytes=0-512000
   Host: www.example.com

1.	``STON`` 최초 로딩 (아무 것도 캐싱되어 있지 않음.)
#.	``Client`` HTTP Range 요청 (100번째 파일의 최초 500KB 요청)
#.	``STON`` /trip.mp4 파일 캐싱객체 생성
#.	``STON`` /trip.mp4 파일 분석을 위해 필요한 부분만을 원본서버에서 다운로드
#.	``STON`` 100번째(99.ts)파일 서비스를 위해 필요한 부분만을 원본서버에서 다운로드
#.	``STON`` 100번째(99.ts)파일 생성 후 Range 서비스
#.	``STON`` 서비스가 완료되면 99.ts파일 파괴

.. note::

   ``MP4Trimming`` 기능이 ``ON`` 이라면 Trimming된 MP4를 HLS로 변환할 수 있다. (HLS영상을 Trimming할 수 없다. HLS는 MP4가 아니라 MPEG2TS 임에 주의하자.)
   영상을 Trimming한 뒤, HLS로 변환하기 때문에 다음과 같이 표현하는 것이 자연스럽다. ::

      /bar/mp4:trip.mp4?start=0&end=60/index.m3u8

   동작에는 문제가 없지만 QueryString을 맨 뒤에 붙이는 HTTP 규격에 어긋난다.
   이를 보완하기 위해 다음과 같은 표현해도 동작은 동일하다. ::

      /bar/mp4:trip.mp4/playlist.m3u8?start=0&end=60
      /bar/mp4:trip.mp4?start=0/playlist.m3u8?end=60