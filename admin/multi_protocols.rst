.. _multi_protocols:

3장. 멀티 프로토콜
******************

이 장에서는 예제 가상호스트를 이용하여 멀티 프로토콜(HTTP/HLS/RTMP)로 서비스하는 방법에 대해 설명한다.
STON 미디어서버의 멀티 프로토콜 표현은 Adobe 미디어 서버(= Flash 미디어 서버)와 호환된다.

현재 지원되는 프로토콜과 규격은 아래와 같다.
======================== =============== =============== ===============
프로토콜                   확장자            Video Codec     Audio Codec
======================== =============== =============== ===============
RTMP                     .mp4            H.264           AAC
Apple HLS                .mp4            H.264           AAC
HTTP Pseudo-Streaming    .mp4            H.264           AAC
======================== =============== =============== ===============


.. toctree::
   :maxdepth: 2



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

- 도메인 (media.example.com) + 디렉토리(/vod)
- 도메인 (media.example.com)
- 1 depth 디렉토리 (/vod). Adobe 미디어서버의 어플리케이션 개념으로 이해할 수 있다.

열거한 순서대로 선택 우선순위를 가지며 가장 명시적인 표현을 우선적으로 선택한다.
다음은 예제 URL에 따른 가상호스트 선택 결과이다.

============================================== ========================
URL                                            가상호스트
============================================== ========================
http://media.example.com/vod/video.mp4         media.example.com/vod
http://media.example.com/sports/highlight.mp4  media.example.com
http://www.foobar.com/vod/video.mp4            /vod
http://www.foobar.com/sports/highlight.mp4     (찾을 수 없음)
============================================== ========================

.. note::

   1 depth 디렉토리만 사용할 경우 Adobe 미디어서버의 Application과 같은 개념이다.

가상호스트 ``<Alias>`` 를 이용하여 패턴표현과 디렉토리 표현이 가능하다. ::

  # vhosts.xml - <Vhosts>

  <Vhost Name="example.com">
     <Alias>another.com</Alias>
     <Alias>*.sub.example.com</Alias>
     <Alias>sports.com/highlight</Alias>
     <Alias>/video2</Alias>
  </Vhost>

패턴표현(*)은 도메인에만 사용할 수 있으며 가상호스트 ``Name`` 과 ``<Alias>`` 는 중복될 수 없다.



멀티 프로토콜 URL
====================================

URL 표현은 Adobe 미디어서버와 호환성을 가진다.
웹서버의 서비스파일 URL이 /subdir/iu.mp4 라면 서비스 주소는 아래와 같다. ::

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
    // <Vhost Name="/hello">
    //////////////////////////////////////////////////////

    // Adobe Flash Player (RTMP)
    Server: rtmp://1.1.1.1/hello
    Stream: mp4:subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://1.1.1.1/hello/mp4:subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming
    http://1.1.1.1/hello/mp4:subdir/iu.mp4


    //////////////////////////////////////////////////////
    // <Vhost Name="media.example.com">
    //////////////////////////////////////////////////////

    // Adobe Flash Player (RTMP)
    Server: rtmp://media.example.com/
    Stream: mp4:subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://media.example.com/mp4:subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming (+ Bandwidth-Throttling)
    http://media.example.com/mp4:subdir/iu.mp4


이미 배포된 URL과의 호환성을 위해 가상호스트 Prefix 속성을 제공한다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost Name="media.example.com/hello"
             Prefix="http/"> ... </Vhost>
   </Vhosts>

Prefix는 URL에만 추가될 뿐 아무런 역할을 수행하지 않는다.
Prefix가 추가된 주소는 아래와 같다. ::

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
   다음 주소에서 대해서도 정상적인 서비스가 가능하다. ::

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

원하는 임의의 포트를 열 수는 있지만, 가상호스트끼리 이미 특정 프로토콜에 바인딩된 포트를 다른 프로토콜에서 열 수 없다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="foo.com">
        <Listen>
            <Http>*:80</Http>
            <Rtmp>*:1935</Rtmp>
        </Listen>
    </Vhost>

    <Vhost Name="bar.com">
        <Listen>
            <Http>*:8080</Http>   // 가능
            <Rtmp>*:80</Rtmp>     // 불가능 - 이미 HTTP에서 사용
        </Listen>
    </Vhost>




