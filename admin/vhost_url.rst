.. _vhost_url:

4장. 가상호스트와 URL
******************

이 장에서는 가상호스트를 통해 제공되는 멀티 프로토콜 서비스 URL(Uniform Resource Locator)표현에 대해 설명한다.
STON 미디어서버의 URL 표현은 Adobe 미디어 서버(= Flash 미디어 서버)와 완벽히 호환된다.
가상호스트는 구분되는 URL표현을 통해 어떤 프로토콜을 사용해서 전송해야 하는지 알 수 있다.

지원되는 프로토콜과 규격은 아래와 같다.

========================== =============== =============== ===============
프로토콜                     확장자            Video Codec     Audio Codec
========================== =============== =============== ===============
RTMP                       .mp4            H.264           AAC
Apple HLS                  .mp4            H.264           AAC
HTTP Pseudo-Streaming      .mp4            H.264           AAC
========================== =============== =============== ===============


.. toctree::
   :maxdepth: 2



.. _env-vhost:

vhosts.xml 가상호스트 설정
====================================

실행파일과 같은 경로에 존재하는 vhosts.xml파일을 가상호스트 설정파일로 인식한다.
가상호스트 개수에 제한은 없다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="foo.com"> ... </Vhost>
        <Vhost Status="Active" Name="bar.com"> ... </Vhost>
    </Vhosts>


.. _env-vhost-create-destroy:

생성/파괴
------------------------------------

``<Vhosts>`` 하위에 ``<Vhost>`` 로 가상호스트를 설정한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Status="Active" Name="www.example.com">
        <Origin>
            <Address>10.10.10.10</Address>
        </Origin>
    </Vhost>

-  ``<Vhost>`` 가상호스트를 설정한다.

   - ``Status (기본: Active)`` Inactive인 경우 해당 가상호스트를 서비스하지 않는다. 캐싱된 콘텐츠는 유지된다.
   - ``Name`` 가상호스트 이름. 중복될 수 없다.

``<Vhost>`` 를 삭제하면 해당 가상호스트가 삭제된다.
삭제된 가상호스트의 모든 콘텐츠는 삭제대상이 된다.
다시 추가해도 콘텐츠는 되살아나지 않는다.


.. _env-vhost-find:

찾기
------------------------------------

다음은 가장 간단한 형태의 HTTP요청이다. ::

    GET / HTTP/1.1
    Host: www.example.com

일반적인 Web서버는 Host헤더로 가상호스트를 찾는다.
하나의 가상호스트를 여러 이름으로 서비스하고 싶다면 ``<Alias>`` 를 사용한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="example.com">
        <Alias>another.com</Alias>
        <Alias>*.sub.example.com</Alias>
    </Vhost>

-  ``<Alias>``

   가상호스트의 별명을 설정한다.
   개수는 제한이 없다.
   명확한 표현(another.com)과 패턴표현(*.sub.example.com)을 지원한다.
   패턴은 복잡한 정규표현식이 아닌 prefix에 * 표현을 하나만 붙일 수 있는 간단한 형식만을 지원한다.


가상호스트 검색 순서는 다음과 같다.

1. ``<Vhost>`` 의 ``Name`` 과 일치하는가?
2. 명시적인 ``<Alias>`` 와 일치하는가?
3. 패턴 ``<Alias>`` 를 만족하는가?


.. _env-vhost-defaultvhost:

Default 가상호스트
------------------------------------

요청을 처리할 가상호스트를 찾지못한 경우 선택될 가상호스트를 지정할 수 있다.
요청을 처리하고 싶지 않다면 설정하지 않아도 된다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Default>www.example.com</Default>
    </Vhosts>

-  ``<Default>``

   기본 가상호스트 이름을 설정한다.
   반드시 ``<Vhost>`` 의 ``Name`` 속성과 똑같은 문자열로 설정해야 한다.


.. _env-vhost-listen:

서비스주소
------------------------------------
서비스 주소를 설정한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com">
        <Listen>*:80</Listen>
    </Vhost>

-  ``<Listen> (기본: *:80)``

   {IP}:{Port} 형식으로 서비스 주소를 설정한다.
   *:80 표현은 모든 NIC로부터의 80포트로 오는 요청을 처리한다는 의미다.
   예를 들어 특정 IP(1.1.1.1)의 90포트로 서비스하고 싶다면 다음과 같이 설정한다. ::

       # vhosts.xml - <Vhosts>

       <Vhost Name="www.example.com">
           <Listen>1.1.1.1:90</Listen>
       </Vhost>

.. note:

   서비스 포트를 열지 않으려면 ``OFF`` 로 설정한다. ::

      # vhosts.xml - <Vhosts>

      <Vhost Name="www.example.com">
         <Listen>OFF</Listen>
      </Vhost>


.. _env-vhost-txt:

가상호스트-예외조건 (.txt)
---------------------------------------

서비스 중 다음과 같이 예외적인 상황이 필요할 때가 있다.

- 모든 POST요청은 허용하지 않지만, 특정 URL에 대한 POST요청은 허가한다.
- 모든 GET요청은 STON이 응답하지만, 특정 IP대역에 대해서는 원본서버로 바이패스한다.
- 특정 국가에 대해서는 전송속도를 제한한다.

이와같은 예외조건은 XML에 설정하지 않는다.
모든 가상호스트는 독립적인 예외조건을 가진다.
예외조건은 ./svc/가상호스트/ 디렉토리 하위에 TXT로 존재한다.
관련 기능에 대해 설명할 때 예외조건도 함께 다룬다.


가상호스트 목록확인
====================================

가상호스트 목록을 조회한다. ::

   http://127.0.0.1:20040/monitoring/vhostslist

결과는 JSON형식으로 제공된다. ::

   {
      "version": "1.1.9",
      "method": "vhostslist",
      "status": "OK",
      "result": [ "www.example.com","www.foobar.com", "site1.com" ]
   }




가상호스트
====================================

가상호스트는 서비스의 기본단위이다.
가상호스트가 생성되면 HTTP (80)포트와 RTMP (1935)포트가 오픈된다.
클라이언트가 접속하는 URL을 가상호스트 ``Name`` 과 매칭하여 서비스할 가상호스트를 결정한다. ::

  # vhosts.xml

  <Vhosts>
     <Vhost Name="media.example.com"> ... </Vhost>
     <Vhost Name="/vod"> ... </Vhost>
     <Vhost Name="media.example.com/vod"> ... </Vhost>
  </Vhosts>

가상호스트 ``Name`` 은 3가지 형태로 구성할 수 있다.

- 도메인 (media.example.com) + 1depth 디렉토리(/vod)
- 도메인 (media.example.com)
- 1 depth 디렉토리 (/vod)

열거한 순서대로 명시적인 표현을 우선으로 선택한다.
예제 URL에 따른 가상호스트 선택 결과는 아래와 같다.

============================================== ========================
URL                                            가상호스트
============================================== ========================
http://media.example.com/vod/video.mp4         media.example.com/vod
http://media.example.com/sports/highlight.mp4  media.example.com
http://www.foobar.com/vod/video.mp4            /vod
http://www.foobar.com/sports/highlight.mp4     (찾을 수 없음)
============================================== ========================

.. note::

   1 depth 디렉토리는 Adobe 미디어서버의 Application과 같은 개념이다.

가상호스트 ``<Alias>`` 를 이용하여 패턴표현과 디렉토리 표현이 가능하다. ::

  # vhosts.xml - <Vhosts>

  <Vhost Name="media.example.com">
     <Alias>myvideo.com</Alias>
     <Alias>*.submedia.example.com</Alias>
     <Alias>sports.com/highlight</Alias>
     <Alias>/video2</Alias>
  </Vhost>

패턴표현(*)은 도메인에만 사용할 수 있으며 가상호스트 ``Name`` 과 ``<Alias>`` 는 중복될 수 없다.



멀티 프로토콜 URL
====================================

URL 표현은 Adobe 미디어서버와 호환성을 가진다.
웹서버 URL이 /subdir/iu.mp4 라면 서비스 주소는 아래와 같다. ::

    //////////////////////////////////////////////////////
    // <Vhost Name="media.example.com/hello">
    //////////////////////////////////////////////////////

    // Adobe Flash Player (RTMP)
    Server: rtmp://media.example.com/hello
    Stream: mp4:subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://media.example.com/hello/mp4:subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming
    http://media.example.com/hello/mp4:subdir/iu.mp4


    //////////////////////////////////////////////////////
    // <Vhost Name="media.example.com">
    //////////////////////////////////////////////////////

    // Adobe Flash Player (RTMP)
    Server: rtmp://media.example.com/
    Stream: mp4:subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://media.example.com/mp4:subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming
    http://media.example.com/mp4:subdir/iu.mp4


    //////////////////////////////////////////////////////
    // <Vhost Name="/hello">
    //////////////////////////////////////////////////////

    // Adobe Flash Player (RTMP)
    Server: rtmp://1.1.1.1/hello
    Stream: mp4:subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://1.1.1.1/hello/mp4:subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming
    http://1.1.1.1/hello/mp4:subdir/iu.mp4


이미 배포된 URL과의 호환성을 위해 가상호스트 Prefix 속성을 제공한다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost Name="media.example.com/hello" Prefix="http/"> ... </Vhost>
   </Vhosts>

Prefix가 추가된 주소는 다음과 같다. ::

    // Adobe Flash Player (RTMP)
    Server: rtmp://media.example.com/hello
    Stream: mp4:http/subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://media.example.com/hello/mp4:http/subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming
    http://media.example.com/hello/mp4:http/subdir/iu.mp4

.. note::

   WOWZA의 경우 Application이름 뒤에 application-instance명을 함께 명시하고 있다.
   (이 값은 대부분 _definst_ 이다.)
   STON은 별도의 설정없이 다음 주소를 인식한다. ::

      // Adobe Flash Player (RTMP) - 동일
      Server: rtmp://media.example.com/hello
      Stream: mp4:http/subdir/iu.mp4

      // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
      http://media.example.com/hello/_definst_/mp4:http/subdir/iu.mp4/playlist.m3u8

      // HTTP Pseudo-Streaming (+ Bandwidth-Throttling)
      http://media.example.com/hello/_definst_/mp4:http/subdir/iu.mp4



서비스 포트
====================================

프로토콜별로 서비스 포트를 설정한다. 기본 포트로 HTTP는 80, RTMP는 1935를 사용한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="media.example.com">
        <Listen>
          <Http>*:80</Http>
          <Rtmp>*:1935</Rtmp>
        </Listen>
    </Vhost>

포트설정에 제한은 없지만, 이미 특정 프로토콜에 바인딩된 포트를 다른 프로토콜에서 열 수 없다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="/foo">
        <Listen>
            <Http>*:80</Http>
            <Rtmp>*:1935</Rtmp>
        </Listen>
    </Vhost>

    <Vhost Name="/bar">
        <Listen>
            <Http>*:8080</Http>   // 가능
            <Rtmp>*:80</Rtmp>     // 불가능 - 이미 HTTP에서 사용
        </Listen>
    </Vhost>
