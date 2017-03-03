.. _media_draft_01:

Media. v0.1 Draft
******************

.. warning::

   개발용 임시 문서입니다. 설정 중 기본 값에 대해서는 정식버전 릴리스 전까지 변경될 수 있습니다.


약어
====================================

- SES - STON Edge Server
- SMS - STON Media Server


호환성
====================================

- Adobe Media Server (구 Flash Media Server), WOWZA 등과 URL호환성을 가진다.
- SES 경험에 익숙한 고객들에게 되도록 친숙한 표현을 제공한다.

서비스 범위
====================================

프로토콜에 따라 지원되는 파일규격은 아래와 같다.

======================== =============== =============== ===============
프로토콜                   확장자            Video Codec     Audio Codec
======================== =============== =============== ===============
RTMP                     .mp4            H.264           AAC
Apple HLS                .mp4            H.264           AAC
HTTP Pseudo-Streaming    .mp4            H.264           AAC
======================== =============== =============== ===============

.. note::

   [연구소 요청사항] 지원되는 항목이 더 있으면 내용 알려주세요.

지원되지 않거나 분석할 수 없는 파일에 대한 요청은 다음과 같이 예외처리된다.

====================== ===============================
프로토콜                 예외처리 응답
====================== ===============================
RTMP                   NetStream.Play.StreamNotFound
Apple HLS              406 Not Acceptable
HTTP Pseudo-Streaming  406 Not Acceptable
====================== ===============================


기본설정
====================================

- Bandwidth-Throttling (Active)
- MP4HLS (Active)

가상호스트
====================================

가상호스트는 서비스의 기본단위이다.
가상호스트가 생성되면 HTTP (80)포트와 RTMP (1935)포트가 오픈된다.
클라이언트가 접속하는 URL을 가상호스트 ``Name`` 과 매칭하여 서비스할 가상호스트를 결정한다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost Name="www.example.com"> ... </Vhost>
      <Vhost Name="/vod"> ... </Vhost>
      <Vhost Name="www.example.com/vod"> ... </Vhost>
   </Vhosts>

가상호스트 ``Name`` 은 3가지 형태로 구성할 수 있으며, 아래 열거한 순서대로 선택 우선순위를 가진다.

- 도메인 (www.example.com) + 디렉토리(/vod)
- 도메인 (www.example.com)
- 디렉토리 (/vod)

디렉토리는 1-depth만 가능하다.
URL에 따라 가상호스트는 다음과 같이 선택된다.

============================================== ====================
URL                                            가상호스트
============================================== ====================
http://www.example.com/vod/video.mp4           www.example.com/vod
http://www.example.com/sports/highlight.mp4    www.example.com
http://www.foobar.com/vod/video.mp4            /vod
http://www.foobar.com/sports/highlight.mp4     (찾을 수 없음)
============================================== ====================

가상호스트 ``<Alias>`` 를 통해 패턴표현과 디렉토리 표현이 가능하다. ::

   # vhosts.xml - <Vhosts>

   <Vhost Name="example.com">
      <Alias>another.com</Alias>
      <Alias>*.sub.example.com</Alias>
      <Alias>sports.com/highlight</Alias>
      <Alias>/video2</Alias>
   </Vhost>

패턴표현(*)은 도메인에만 사용할 수 있으며 가상호스트 ``Name`` 과 ``<Alias>`` 는 중복될 수 없다.


원본요청 Host헤더
====================================

가상호스트 ``Name`` 은 원본요청의 Host헤더의 기본 값이 된다.
``Name`` 에 따른 Host헤더는 아래와 같다.

====================== ===============================
Vhost ``Name``         HTTP Host Header
====================== ===============================
www.example.com/vod    www.example.com
www.example.com        www.example.com
/vod                   (보내지 않음)
====================== ===============================

HTTP Host Header는 다음 설정을 통해 설정이 가능하다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <Host />


URL 표현
====================================

URL 표현은 Adobe Media Server(구 FMS)를 따르며
파생된 미디어 서버(i.e. WOWZA)들과 호환성을 가진다.
원본서버 경로가 /subdir/iu.mp4 라면 서비스 주소는 아래와 같다. ::

    //////////////////////////////////////////////////////
    // <Vhost Name="www.example.com/exam_vod">
    //////////////////////////////////////////////////////

    // Adobe Flash Player (RTMP)
    Server: rtmp://www.example.com/exam_vod
    Stream: mp4:subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://www.example.com/exam_vod/mp4:subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming (+ Bandwidth-Throttling)
    http://www.example.com/exam_vod/mp4:subdir/iu.mp4

    //////////////////////////////////////////////////////
    // <Vhost Name="www.example.com">
    //////////////////////////////////////////////////////

    // Adobe Flash Player (RTMP)
    Server: rtmp://www.example.com/
    Stream: mp4:subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://www.example.com/mp4:subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming (+ Bandwidth-Throttling)
    http://www.example.com/mp4:subdir/iu.mp4


가상호스트의 Prefix 속성을 설정하면 URL 호환성을 더 강화할 수 있다. ::

   # vhosts.xml

   <Vhosts>
      <Vhost Name="www.example.com/exam_vod"
             Prefix="http/"> ... </Vhost>
   </Vhosts>

Prefix는 URL에만 추가될 뿐 아무런 역할을 수행하지 않는다.
Prefix가 추가된 주소는 아래와 같다. ::

    // Adobe Flash Player (RTMP)
    Server: rtmp://www.example.com/exam_vod
    Stream: mp4:http/subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://www.example.com/exam_vod/mp4:http/subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming (+ Bandwidth-Throttling)
    http://www.example.com/exam_vod/mp4:http/subdir/iu.mp4

WOWZA의 경우 Application이름 뒤에 application-instance명을 함께 명시하고 있다.
(이 값은 대부분 _definst_ 이다.)
다음 주소에서 대해서도 정상적인 서비스가 가능하다. ::

    // Adobe Flash Player (RTMP) - 동일
    Server: rtmp://www.example.com/exam_vod
    Stream: mp4:http/subdir/iu.mp4

    // Apple iOS device (Cupertino/Apple HTTP Live Streaming)
    http://www.example.com/exam_vod/_definst_/mp4:http/subdir/iu.mp4/playlist.m3u8

    // HTTP Pseudo-Streaming (+ Bandwidth-Throttling)
    http://www.example.com/exam_vod/_definst_/mp4:http/subdir/iu.mp4



서비스 포트
====================================

프로토콜별로 서비스 포트를 설정한다. 기본 포트로 HTTP는 80, RTMP는 1935를 사용한다. ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com">
        <Listen>
          <Http>*:80</Http>
          <Rtmp>*:1935</Rtmp>
        </Listen>
    </Vhost>

사용자가 원하는 임의의 포트를 열 수는 있지만, 가상호스트끼리 서로 다른 프로토콜에 대해 포트를 공유할 수 없다. ::

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
