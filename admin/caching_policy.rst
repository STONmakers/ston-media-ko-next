.. _caching_policy:

7장. Caching 정책
******************

이 장에서는 서비스의 핵심이 되는 TTL(Time To Live)과 Caching-Key 그리고 만료정책에 대해 설명한다.
저장된 콘텐츠는 TTL동안 유효하다.
HTTP 규격은 TTL을 설정할 수 있도록 Cache-Control을 명시하고 있다.
하지만 이는 절대적인 것은 아니다.
다양한 방식의 TTL 정책과 :ref:`caching-purge` 를 통해 서비스 품질을 높일 수 있다.

HTTP에는 콘텐츠를 구분하는 다양한 규격이 존재한다.
그만큼 Caching-Key도 다양하게 존재할 수 있다.
콘텐츠 변경이 없을수록 원본부하를 줄일 수 있을뿐만 아니라 쉽게 확장할 수 있다.
서비스에 최적화된 만료정책을 수립하는 다양한 방식에 대해 설명한다.

**Caching-Key**란 콘텐츠를 구분하는 고유 값이다.
파일시스템에서 파일들과 구분되는 고유경로(예. /usr/conf.txt)를 가지는 것과 같은 개념이다.
흔히 Caching-Key는 URL과 혼동되기 쉽다.
HTTP의 여러 기능에 따라 같은 URL이라고 하더라도 콘텐츠가 달라질 수 있다.


.. toctree::
   :maxdepth: 2



.. _caching_policy-ttl:

TTL (Time To Live)
====================================

TTL이란 저장된 콘텐츠의 유효시간이다.
TTL을 길게 설정하면 원본서버의 부하는 줄어들지만 변경사항이 늦게 반영된다.
반대로 짧게 설정하면 너무 빈번한 변경확인 요청으로 원본서버 부하가 높아진다.
운영의 묘미는 TTL을 적절히 설정하여 원본부하를 줄이는 것에 있다.
TTL은 한번 설정되면 만료되기 전까지 바뀌지 않는다.
새로운 TTL은 파일이 만료되었을 때 적용된다.
관리자는 :ref:`api-cmd-purge` , :ref:`api-cmd-expire` , :ref:`api-cmd-expireafter` , :ref:`api-cmd-hardpurge` 등의 API를 사용해 TTL을 변경할 수 있다.


기본 TTL
---------------------

TTL은 원본서버의 응답에 따라 결정된다.
TTL이 만료되기 전까지 저장된 콘텐츠로 서비스 된다.
TTL이 만료되면 원본서버로 콘텐츠 변경여부( **If-Modified-Since** 또는 **If-None-Match** )를 확인한다.
원본서버가 **304 Not Modified** 응답을 준다면 TTL은 연장된다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <TTL>
        <Res2xx Ratio="20" Max="86400">1800</Res2xx>
        <NoCache Ratio="0" Max="5" MaxAge="0">5</NoCache>
        <Res3xx>300</Res3xx>
        <Res4xx>30</Res4xx>
        <Res5xx>30</Res5xx>
        <ConnectTimeout>3</ConnectTimeout>
        <ReceiveTimeout>3</ReceiveTimeout>
        <OriginBusy>3</OriginBusy>
    </TTL>

``Ratio`` (0~100)를 제외한 모든 설정 단위는 초(sec) 다.

-  ``<Res2xx> (기본: 1800초, Ratio: 20, Max=86400)``
   원본서버가 200 OK로 응답했을 때 TTL을 설정한다.
   콘텐츠를 처음 저장할 때 ``<Res2xx>`` 초 뒤에 콘텐츠가 만료(TTL)되도록 설정한다.
   (TTL만료 후) 원본서버에서 변경되지 않았다면(304 Not Modified) ``Ratio`` 비율(0~100)만큼 TTL을 연장한다.
   TTL은 최대 ``Max`` 까지 증가한다.

-  ``<NoCache> (기본: 5초, Ratio: 0, Max=5, MaxAge=0)``
   ``<Res2xx>`` 와 동일하나 원본서버가 no-cache로 응답하는 경우에만 적용된다. ::

      cache-control: no-cache 또는 private 또는 must-revalidate

   ``MaxAge`` 가 0보다 크다면 max-age를 줄 수 있다.

   .. figure:: img/sms_nocache_maxage1.png
      :align: center

      Max-Age만큼 클라이언트에 Caching된다.

-  ``<Res3xx> (기본: 300초)``
   원본서버가 3xx로 응답했을 때 TTL을 설정한다.
   Redirect용도로 사용되는 경우가 많다.

-  ``<Res4xx> (기본: 30초)``
   원본서버가 4xx로 응답했을 때 TTL을 설정한다.
   **404 Not Found** 인 경우가 많다.

-  ``<Res5xx> (기본: 30초)``
   원본서버가 5xx로 응답했을 때 TTL을 설정한다.
   원본서버 내부 장애상황인 경우가 많다.

-  ``<ConnectTimeout> (기본: 3초)``
   원본서버로 접속하지 못하는 경우 TTL을 설정한다.
   콘텐츠가 이미 저장되어 있다면 ``<ConnectTimeout>`` 초 만큼 TTL을 연장한다.
   콘텐츠가 저장되어 있지 않다면 ``<ConnectTimeout>`` 초 만큼 장애상황으로 응답한다.
   이는 장애상황을 서비스한다는 의미보다는 TTL시간동안 (아마도 장애상황일) 원본서버에 부담을 주지 않기 위함이다.

-  ``<ReceiveTimeout> (기본: 3초)``
   접속은 됐으나 데이터를 수신하지 못하는 경우 TTL을 설정한다.
   ``<ConnectTimeout>`` 과 의미적으로 동일하다.

-  ``<OriginBusy> (기본: 3초)``
   :ref:`http-origin-busysessioncount` 조건을 만족하면 원본서버 요청없이 만료된 콘텐츠의 TTL을 설정된 시간만큼 연장한다.
   이는 원본서버의 부하를 가중시키지 않기 위함이다.

.. note::

   TTL 값을 0으로 설정하면 서비스 직후 곧바로 만료된다.
   만약 모든 요청에 대해 원본서버의 응답을 주고 싶다면 바이패스할 것을 권장한다.


.. _caching_policy-customttl:

Custom TTL
---------------------

URL마다 별도로 TTL을 설정한다.
명확한 URL 또는 패턴 URL에 매칭되는 콘텐츠마다 고정된 TTL을 설정할 수 있다.
/svc/{가상호스트 이름}/ttl.txt 에 설정한다. ::

    # /svc/www.example.com/ttl.txt
    # 구분자는 콤마(,)이며 시간 단위는 초이다.

    /hot/*.mp4, 300
    /trip.mp4, 3600

프로토콜에 따라 클라이언트가 접속하는 URL이 다르기 때문에 원본서버에서 제공하는 URL을 설정해야 한다. 
예를 들어 원본서버 URL이 /hot/sample.mp4라면 위 설정(/hot/*.mp4, 300)에 의해 300초 TTL이 설정된다. 
이때 클라이언트 URL은 아래와 같다. ::

   //////////////////////////////////////////////////////
   // <Vhost Name="www.example.com/bar">
   //////////////////////////////////////////////////////

   // Adobe Flash Player (RTMP)
   Server: rtmp://www.example.com/bar
   Stream: mp4:hot/sample.mp4

   // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
   http://www.example.com/bar/mp4:hot/sample.mp4/playlist.m3u8

   // HTTP Pseudo-Streaming
   http://www.example.com/bar/mp4:hot/sample.mp4


TTL 우선순위
---------------------

적용할 TTL설정의 우선순위를 설정한다. ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <TTL Priority="cc_nocache, custom, cc_maxage, rescode">
        ... (생략) ...
    </TTL>

``<TTL>`` 의 ``Priority (기본: cc_nocache, custom, cc_maxage, rescode)`` 속성으로 설정한다.

- ``cc_nocache`` 원본이 Cache-Control: no-cache로 응답한 경우
- ``custom`` `caching-policy-customttl`
- ``cc_maxage`` 원본이 Cache-Control에 maxage를 명시한 경우
- ``rescode`` 원본 응답코드별 기본 TTL



.. _caching_policy-casesensitive:

대소문자 구분
====================================

원본서버의 대소문자 구분여부를 능동적으로 알 수 없다.

   .. figure:: img/sms_casesensitive1.png
      :align: center

      아마도 같은 콘텐츠이거나 404가 발생한다.

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <CaseSensitive>ON</CaseSensitive>

-  ``<CaseSensitive>``

   -  ``ON (기본)`` URL 대소문자를 구문한다.

   -  ``OFF`` URL 대소문자를 구분하지 않는다. 모두 소문자로 처리된다.


.. _caching_policy-applyquerystring:

QueryString 구분
====================================

QueryString에 의하여 동적으로 생성되는 콘텐츠가 아니라면 QueryString을 인식하는 것은 불필요하다.
아무 의미없는 Random값이나 항상 변하는 시간 값이 매번 붙는다면 원본에 엄청난 부하가 발생할 수 있다.

   .. figure:: img/sms_querystring1.png
      :align: center

      동적 콘텐츠가 아니라면 같은 콘텐츠일 가능성이 높다.

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <ApplyQueryString Collective="OFF">ON</ApplyQueryString>

-  ``<ApplyQueryString>``

   -  ``ON (기본)`` QueryString을 인식한다. 예외조건에 만족하면 QueryString이 무시된다.

   -  ``OFF`` QueryString을 무시한다. 예외조건에 만족하면 QueryString을 인식한다.

QueryString-예외조건은 /svc/{가상호스트 이름}/querystring.txt에 설정한다. ::

    # ./svc/www.example.com/querystring.txt

    /private/personal.mp4?login=ok*
    /ad/spot.mp4

예외조건이 ``<ApplyQueryString>`` 설정에 따라 의미가 달라짐에 주의한다.
명확한 URL또는 패턴(*만 허용한다)으로 설정이 가능하다.

``Collective`` 속성은 Purge API가 호출되었을 때 대상을 지정한다.

-  ``Collective``

   -  ``OFF (기본)`` 파라미터 URL만을 대상으로 지정한다.

   -  ``ON`` 파라미터 URL뿐만 아니라 URL에 QueryString이 존재하는 모든 컨텐츠를 대상으로 지정한다.

``Collective`` 속성이 ON이고 파일이 많을수록 Purge API를 수행할 때 CPU부하가 높아진다.
관련 파일을 검색하는 소요시간이 길어질 수 있어 예기치 않은 문제를 일으킬 수 있다.
가급적 QueryString까지 붙은 명확한 URL로 Purge API를 호출할 것을 권장한다.
