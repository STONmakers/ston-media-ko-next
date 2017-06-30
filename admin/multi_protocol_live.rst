.. _multi-protocol-live:

[v1.x] 5장. LIVE
******************

이 장에서는 STON 미디어 서버의 LIVE 서비스 구성에 대해 설명한다.
STON 미디어 서버는 원본서버로부터 RTMP/HLS 라이브를 송신받아 멀티 프로토콜로 전송한다.
프로토콜별 URL 표현은 :ref:`multi-protocol-url` 을 참고한다.

.. figure:: img/sms_live_workflow.png
   :align: center

가상호스트의 ``Type`` 속성이 반드시 Live로 설정되어 있어야 한다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Name="www.example.com/bar" Type="LIVE">
            ...
        </Vhost>
    </Vhosts>

서로 다른 프로토콜 변환이 발생할 때(RTMP to HLS/HLS to RTMP) 기술적인 제약사항이 있을 수 있다.

.. warning::

   VOD와 LIVE는 동적으로 변경할 수 없다. 
   같은 이름의 가상호스트를 사용하려면 삭제 후 다시 추가해주어야 한다.


.. toctree::
   :maxdepth: 2



.. _multi-protocol-live-channel:

채널(Channel)
====================================

채널(Channel)은 1개의 LIVE 서비스를 의미한다.
채널은 클라이언트의 첫번째 요청에 의해 생성되고, 마지막 클라이언트가 종료되면 자동으로 파괴된다.

.. figure:: img/sms_live_channel_lifycycle.png
   :align: center

   채널의 Life Cycle

가상호스트는 여러 개의 채널을 동시에 서비스할 수 있다.

.. figure:: img/sms_live_channel_multi.png
   :align: center

   멀티 채널

단, 가상호스트에 속한 모든 채널의 원본 프로토콜(RTMP 또는 HLS)은 동일해야 한다.

.. note::

   채널은 LIVE의 특성상 메모리만을 이용해 동작하기 때문에 메모리 크기따라 생성할 수 있는 수가 제한된다. 
   예를 들어 사용가능한 메모리 크기가 10GB이고 채널 하나당 10MB를 소비한다면 약 1000개의 채널이 서비스 가능하다.
   메모리 한계를 초과할 경우 채널이 생성되지 않는다.



.. _multi-protocol-live-channel-create:

생성
------------------------------------

채널은 명확하게 원본서버와 어떤 프로토콜을 이용해 통신할 것인지 알고 있어야 한다. ::

    # vhosts.xml

    <Vhosts>
        <Vhost Name="www.example.com/bar" Type="LIVE">
            <Origin Protocol="RTMP">
               ...
            </Origin>
        </Vhost>
    </Vhosts>

-  ``<Origin>``

   - ``Protocol (기본: RTMP)`` LIVE를 위해 원본서버와 통신할 프로토콜을 설정한다. (RTMP 또는 HLS)

클라이언트 요청 프로토콜과 상관없이 ``<Origin Protocol="...">`` 설정으로 원본서버와 통신한다.
예를 들어 원본서버 주소(mov/trip.mp4)가 RTMP로 게시되어 있다면 다음 주소 중 먼저 오는 요청에 의해 채널은 생성된다.

- rtmp://www.example.com/bar/mp4:mov/trip.mp4
- http://www.example.com/bar/mp4:mov/trip.mp4/playlist.m3u8


.. _multi-protocol-live-channel-destroy:

파괴
------------------------------------

채널은 마지막 클라이언트와 연결이 종료된 후(LIVE가 종료될 때가 아님) ``<ClientKeepAliveSec>`` 시간(초)만큼 채널을 유지한 뒤 파괴된다. ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>
   
   <Rtmp>
      <ClientKeepAliveSec>60</ClientKeepAliveSec>
   </Rtmp>

   <Hls>
      <ClientKeepAliveSec>60</ClientKeepAliveSec>
   </Hls>

-  ``<ClientKeepAliveSec> (기본: 60초)``
   마지막 클라이언트의 연결이 종료된 후 설정된 시간(초)만큼 경과 후 채널을 파괴한다.

(HLS처럼) 클라이언트가 항상 채널에 연결되어 있는 것은 아니다.
채널을 즉시 파괴하면 자칫 너무 많은 생성/파괴가 발생할 수 있으며 이는 서비스 품질에 영향을 준다.
따라서 서비스 특성에 맞추어 일정시간 채널을 유지하도록하여 서비스 품질을 보장한다.



.. _multi-protocol-live-adobe-rtmp:

Adobe RTMP
====================================

원본서버에서 RTMP로 스트리밍(Streaming)되는 영상을 RTMP/HLS로 전송한다.

.. figure:: img/sms_live_workflow_rtmp.png
   :align: center


.. _multi-protocol-live-adobe-rtmp-client:

RTMP 클라이언트
------------------------------------

:ref:`multi-protocol-vod-adobe-rtmp-session` 설정을 그대로 사용하지만, ``<BufferSize>`` 의 의미가 다르다. ::

   # server.xml - <Server><VHostDefault><Options><Rtmp>
   # vhosts.xml - <Vhosts><Vhost><Options><Rtmp>
   
   <BufferSize>3</BufferSize>

-  ``<BufferSize> (기본: 3초)``
   클라이언트가 PLAY를 요청했을 때 "현재시점"에서 설정된 시간(초) 이전부터 전송한다.

      .. figure:: img/sms_live_channel_multi.png
      :align: center
      
   값이 0이라면 PLAY 요청 시 채널의 "현재시점"을 전송한다. 

LIVE 서비스의 특성상 방송 시점과 클라이언트 시청 시점의 차이가 짧을수록 좋다.

.. figure:: img/sms_live_channel_multi.png
   :align: center

   BufferSize , 시점, 네트워크 안정성, 원활한 재생의 관계

하지만 3G/공용Wi-Fi 등 불안정한 네트워크 환경이라면 영상이 자주 끊기는 등 재생이 원활하지 않을 가능성이 높다.
클라이언트가 일정시간을 버퍼링한다면 순간적인 네트워크 지연에도 끊김없는 재생이 가능하다.



.. _multi-protocol-live-apple-hls-client:

HLS 클라이언트
------------------------------------

HLS 전송을 위해서는 RTMP 스트림을 Packetizing해야 한다.
:ref:`multi-protocol-vod-apple-hls-session` 설정은 동일하며 :ref:`multi-protocol-vod-apple-hls-packetizing` 설정의 다른 점에 대해서만 설명한다. ::

   # server.xml - <Server><VHostDefault><Options><Hls>
   # vhosts.xml - <Vhosts><Vhost><Options><Hls>

   <Packetizing Status="Active">
      <Index Ver="3" Alternates="ON">index.m3u8</Index>
      <Sequence>0</Sequence>
      <Duration ChunkCount="3">10</Duration>
      <AlternatesName>playlist.m3u8</AlternatesName>
      <MP3SegmentType>TS</MP3SegmentType>
   </Packetizing>

-  ``<Packetizing>`` LIVE가 이미 진행 중이라면 ``<Packetizing>`` 및 하위 설정의 값을 바꾸어도 적용되지 않는다.

-  ``<Duration> (기본: 10초)`` Streaming된 데이터가 Duration동안 저장되면 Chunk가 생성되고 인덱스파일(m3u8)이 갱신된다.

   - ``ChunkCount (기본 3)`` 인덱스파일(m3u8)에서 제공할 Chunk개수를 지정한다.

RTMP를 HLS로 변환할 때는 Streaming되는 Audio/Video를 Chunk(MPEG2-TS)파일로 만들어야 한다. 
LIVE가 진행되면서 (기본 ``<Duration>`` 설정에서) 인덱스파일은 아래와 같이 변한다.

.. figure:: img/sms_live_workflow_rtmp_hls_duration10.png
   :align: center
   
   RTMP시점보다 30초 전 시점부터 시청한다.

``<Duration>`` 을 아래와 같이 줄이면 시청 시점을 RTMP와 최대한 맞출 수 있다. ::

   # server.xml - <Server><VHostDefault><Options><Hls>
   # vhosts.xml - <Vhosts><Vhost><Options><Hls>

   <Packetizing>
      <Duration ChunkCount="3">2</Duration>
   </Packetizing>

.. figure:: img/sms_live_workflow_rtmp_hls_duration2.png
   :align: center

   RTMP시점보다 6초 전 시점부터 시청한다.




.. _multi-protocol-live-apple-hls:

Apple HLS
====================================

원본서버에서 HTTP로 다운로드한 영상을 HLS(HTTP Live Streaming)으로 전송한다.

.. figure:: img/vod_workflow_hls.png
   :align: center

모든 인덱스/Chunk 파일은 동적으로 생성되며 별도의 저장공간을 소비하지 않는다.
서비스 시점에 임시적으로 생성되며 서비스가 끝나면 사라진다.


.. _multi-protocol-live-apple-hls-session:

세션
------------------------------------
::

   # server.xml - <Server><VHostDefault><Options><Hls>
   # vhosts.xml - <Vhosts><Vhost><Options><Hls>
   
   <ClientKeepAliveSec>30</ClientKeepAliveSec>

-  ``<ClientKeepAliveSec> (기본: 30초)``
   아무런 통신이 없는 상태로 설정된 시간이 경과하면 연결을 종료한다.



.. _multi-protocol-live-apple-hls-mp4segmentation:

Packetizing
------------------------------------
MPEG2-TS(Transport Stream)로 Packetizing하고 인덱스 파일을 구성하는 정책을 설정한다.  ::

   # server.xml - <Server><VHostDefault><Options><Hls>
   # vhosts.xml - <Vhosts><Vhost><Options><Hls>

   <Packetizing Status="Active">
      <Index Ver="3" Alternates="ON">index.m3u8</Index>
      <Sequence>0</Sequence>
      <Duration>10</Duration>
      <AlternatesName>playlist.m3u8</AlternatesName>
      <MP3SegmentType>TS</MP3SegmentType>
   </Packetizing>


-  ``<Packetizing>``

   - ``Status (기본: Active)`` 값이 ``Inactive`` 라면 Packetizing하지 않고 원본서버의 HLS 파일들을 릴레이한다.

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

-  ``<Duration> (기본: 10초)`` 콘텐츠를 분할(Segmentation)하는 기준 시간(초).
   분할의 기준은 Video/Audio의 KeyFrame이다.
   KeyFrame은 들쭉날쭉할 수 있으므로 정확히 분할되지 않을 수 있다.
   만약 10초로 분할하려는데 KeyFrame이 9초와 12초에 있다면 가까운 값(9초)을 선택한다.

-  ``<AlternatesName> (기본: playlist.m3u8)`` Stream Alternates 파일명. ::

      http://www.example.com/bar/mp4:trip.mp4/playlist.m3u8

-  ``<MP3SegmentType> (기본: TS)`` MP3라면 Chunk포맷을 설정한다. (TS 또는 MP3)


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



.. _multi-protocol-live-apple-hls-keyframe-duration:

키 프레임과 <Duration>
------------------------------------

분할(Segmentation)의 경우 ``<Duration>`` 보다 Key Frame 간격이 우선한다. 아래 3가지 경우에서 분할이 어떻게 되는지 설명한다.

-  **KeyFrame 간격보다** ``<Duration>`` **설정이 큰 경우**
   KeyFrame이 3초, ``<Duration>`` 이 20초라면 20초를 넘지 않는 KeyFrame의 배수인 18초로 분할된다.

-  **KeyFrame 간격과** ``<Duration>`` **이 비슷한 경우**
   KeyFrame이 9초, ``<Duration>`` 이 10초라면 10초를 넘지 않는 KeyFrame의 배수인 9초로 분할된다.

-  **KeyFrame 간격이** ``<Duration>`` **설정보다 큰 경우**
   KeyFrame단위로 분할된다.

다음 클라이언트 요청에 대해 STON 미디어 서버가 어떻게 동작하는지 이해해보자. ::

   GET /bar/mp4:trip.mp4/99.ts HTTP/1.1
   Range: bytes=0-512000
   Host: www.example.com

1.	**STON Media Server** : 최초 로딩 (아무 것도 캐싱되어 있지 않음.)
#.	**HTTP/HLS Client** : HTTP Range 요청 (100번째 파일의 최초 500KB 요청)
#.	**STON Media Server** : /trip.mp4 파일 캐싱객체 생성
#.	**STON Media Server** : /trip.mp4 파일 분석을 위해 필요한 부분만을 원본서버에서 다운로드
#.	**STON Media Server** : 100번째(99.ts)파일 서비스를 위해 필요한 부분만을 원본서버에서 다운로드
#.	**STON Media Server** : 100번째(99.ts)파일 생성 후 Range 서비스
#.	**STON Media Server** : 서비스가 완료되면 99.ts파일 파괴

.. note::

   ``MP4Trimming`` 기능이 ``ON`` 이라면 Trimming된 MP4를 HLS로 변환할 수 있다. (HLS영상을 Trimming할 수 없다. HLS는 MP4가 아니라 MPEG2-TS 임에 주의하자.)
   영상을 Trimming한 뒤, HLS로 변환하기 때문에 다음과 같이 표현하는 것이 자연스럽다. ::

      /bar/mp4:trip.mp4?start=0&end=60/playlist.m3u8

   동작에는 문제가 없지만 QueryString을 맨 뒤에 붙이는 HTTP 규격에 어긋난다.
   이를 보완하기 위해 다음과 같은 표현해도 동작은 동일하다. ::

      /bar/mp4:trip.mp4/playlist.m3u8?start=0&end=60
      /bar/mp4:trip.mp4?start=0/playlist.m3u8?end=60

