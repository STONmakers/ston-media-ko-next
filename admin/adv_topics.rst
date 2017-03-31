.. _adv_topics:

15장. 최적화와 그 밖의 것들
****************************

이 장에서는 최적화와 그 밖의 잡다하지만 깊이 있는 주제에 대해 다룬다.
최적화는 고성능(High Performance)을 위한 방법이며 이는 우리가 추구하는 가장 큰 가치다.
엔터프라이즈 환경에서의 고성능은 주어진 하드웨어 자원을 최대한 활용하는 것을 의미하기도 한다.

그 중 메모리는 모든 설계 및 정책을 결정하는 가장 중요한 자원이다.
특히 인덱싱(요청된 URL을 빠르게 찾는 것)에 대해서는 반드시 이해해야 한다.
왜냐하면 서비스 품질을 결정짓는 것은 인덱싱이기 때문이다.
앞으로 설명할 모든 내용은 다음 표 "물리 메모리 크기에 따른 기본설정"와 관련이 있다.

============= ============== =============== ============= ========
Physical RAM  System Free    Contents        Caching Count Sockets
============= ============== =============== ============= ========
1GB           409.60MB       188.37MB        219,469       5,000
2GB           819.20MB       446.74MB        520,494       10,000
4GB           1.60GB         963.49MB        1,122,544     20,000
8GB           3.20GB         2.05GB          2,440,422     20,000
16GB          6.40GB         4.45GB          5,303,733     20,000
32GB          12.80GB        9.25GB          11,030,356    20,000
64GB          25.60GB        18.85GB         22,483,603    20,000
128GB         51.20GB        38.05GB         45,390,095    20,000
============= ============== =============== ============= ========



.. toctree::
   :maxdepth: 2


.. _adv_topics_mem_control:

메모리 조절
====================================

STON 미디어 서버는 구동될 때 물리 메모리 크기에 기반하여 캐싱 메모리 사용량을 결정한다. ::

   # server.xml - <Server><Cache>

   <SystemMemoryRatio>100</SystemMemoryRatio>

-  ``<SystemMemoryRatio> (기본: 100)`` 물리메모리를 기준으로 사용할 메모리 비율을 설정한다.

예를 들어 8GB장비에서 ``<SystemMemoryRatio>`` 를 50으로 설정하면 물리 메모리가 4GB인 것처럼 동작한다.
이는 메모리를 점유하는 다른 프로세스와 같이 구동될 때 유용하게 사용될 수 있다.

좀 더 구체적으로 서비스 형식에 따라 메모리에 적재되는 데이터 비율을 조절하면 효과적이다. ::

   # server.xml - <Server><Cache>

   <ContentMemoryRatio>50</ContentMemoryRatio>

-  ``<ContentMemoryRatio> (기본: 50)`` STON이 사용하는 전체 메모리 중 서비스 데이터 메모리 적재비율을 설정한다.

예를 들어 4K UHD 영상처럼 파일개수는 적지만 컨텐츠 크기가 클 경우엔 이 수치를 늘리면 파일 I/O가 감소된다.
반대로 아주 작은 샘플 영상만 많은 경우는 반대로 줄이는 설정이 유용할 수 있다.


.. _adv_topics_sys_free_mem:

시스템 Free 메모리
====================================

OS(Operating System)가 느리면 어떠한 프로그램도 제 성능을 내지 못한다.
STON 미디어 서버는 OS를 위해 일부 메모리를 사용하지 않는다.
OS의 성능을 극대화하기 위해서이며 이를 시스템 Free메모리라 부른다.

.. note::

   이에 대해 권위있는 설명을 제시하고 싶으나 아쉽게도 찾지 못하였다.
   구글링을 통해 가장 많이 `인용된 글 <http://www.sysxperts.com/home/announce/vmdirtyratioandvmdirtybackgroundratio>`_ 을 제시한다.

============== ===============
Physical RAM   System Free
============== ===============
1GB	           409.6MB
2GB	           819.2MB
4GB            1.6GB
8GB	           3.2GB
16GB	         6.4GB
32GB	         12.8GB
64GB	         25.6GB
128GB	         51.2GB
============== ===============

고급 사용자의 경우 서비스 형태에 맞추어 Free메모리 비율을 줄일 수 있다. Free메모리가 줄어들면 더 많은 Contents를 메모리에 적재할 수 있다. ::

   # server.xml - <Server><Cache>

   <SystemFreeMemoryRatio>40</SystemFreeMemoryRatio>

-  ``<SystemFreeMemoryRatio> (기본: 40, 최대: 40)`` 물리 메모리를 기준으로 설정된 비율만큼을 Free메모리로 남겨둔다.



.. _adv_topics_tso:

TCP Segmentation Offload
====================================

.. important::

   10G NIC를 사용한다면 TSO(TCP Segmentation Offload)를 OFF로 설정하길 권장한다.

TCP는 전송시 패킷을 분할(Segmentation)하는데, 이 작업을 CPU가 아닌 NIC가 수행하도록 설정하는 것이 TSO이다.
(기본 값은 ON이다.)
하지만 10G NIC 서비스 환경에서 우리는 이와 관련된 많은 장애를 겪었다.

-  TCP 패킷 유실 및 지연
-  TCP 연결 종료
-  Load Average의 비정상적인 증가

결론적으로 TSO는 모두의 기대만큼 높은 성능을 내지 못하는 것으로 추정된다.
(NIC만 1G로 바꿔도 이런 문제는 발생하지 않았다.)
결론적으로 TSO를 OFF로 설정함으로써 서비스는 정상화되었다.
이에 따른 CPU 사용량은 우려할 수준이 아니며 서비스 규모와 비례하는 정직한 지표를 보여 준다.

TSO 설정은 다음과 같이 설정/확인할 수 있다. (K의 대/소문자에 유의한다.) ::

   # ethtool -K ethX tso off        // TSO OFF 설정
   # ethtool -k ethX                // 설정 열람
   ...
   tcp segmentation offload: on
   ...

.. tip::

   더 자세한 정보는 다음 링크를 참조한다.

   -  `http://sandilands.info/sgordon/segmentation-offloading-with-wireshark-and-ethtool <http://sandilands.info/sgordon/segmentation-offloading-with-wireshark-and-ethtool>`_
   -  `http://www.linuxfoundation.org/collaborate/workgroups/networking/tso <http://www.linuxfoundation.org/collaborate/workgroups/networking/tso>`_
   -  `http://www.packetinside.com/2013/02/mtu-1500.html <http://www.packetinside.com/2013/02/mtu-1500.html>`_




클라이언트 접속 제한
====================================

제한없이 클라이언트 요청을 모두 허용하면 시스템에 지나친 부하가 발생할 수 있다.
시스템 과부하는 사실상 장애이다.
적절한 수치에서 클라이언트 요청을 거부하여 시스템을 보호한다. ::

   # server.xml - <Server><Cache>

   <MaxSockets Reopen="75">80000</MaxSockets>

-  ``<MaxSockets> (기본: 80000, 최대: 100000)`` 연결을 허용할 최대 클라이언트 소켓 수.
   이 수치를 넘으면 신규 클라이언트 접속을 즉시 닫는다.
   ``<MaxSockets>`` 의 ``Reopen (기본: 75%)`` 비율만큼 소켓 수가 감소하면 다시 접속을 허용한다.

.. figure:: img/maxsockets.png
   :align: center

(기본 설정에서) 전체 클라이언트 소켓 수가 8만을 넘으면 신규 클라이언트 접속은 즉시 종료된다.
전체 클라이언트 소켓 수가 6만(8만의 75%)이 되면 다시 접근을 허용한다.

예를 들어 3만개의 클라이언트 세션을 처리할 때 원본 서버들이 모두 한계에 도달하면
이 수치를 3~4만 정도로 설정하는 것이 좋다.
이로 인해 얻을 수 있는 효과는 다음과 같다.

-  별다른 Network 구성(e.g. L4 세션조절 등)이 필요 없다.
-  불필요한(원본 부하로 처리될 수 없는) 클라이언트 요청을 방지한다.
-  서비스의 신뢰성을 높인다. 서비스 Burst 이후 재시작 등 점검 작업이 필요 없다.



클라이언트 세션풀
====================================

클라이언트 연결을 처리하기 위한 초기/증설 세션 수를 설정한다. 
HLS/MPEG-DASH등 HTTP기반 프로토콜은 HTTP 클라이언트 세션 설정에 포함된다. ::

    # server.xml - <Server><Cache>

    <HttpClientSession>
       <Init>10000</Init>
       <TopUp>3000</TopUp>
    </HttpClientSession>
    
    <RtmpClientSession>
       <Init>10000</Init>
       <TopUp>3000</TopUp>
    </RtmpClientSession>

-  ``<Init>`` STON 미디어 서버 시작 시 미리 생성해놓는 소켓 수

-  ``<TopUp>`` 생성해놓은 소켓 수를 초과했을 때 추가로 생성할 소켓 수

별도로 설정하지 않을 경우 물리 메모리 크기에 따라 자동으로 설정된다.

=============== =========================
물리메모리	      <Init>, <TopUp>
=============== =========================
1GB             5천, 1천
2GB             1만, 2천
4GB             2만, 4천
8GB 이상         2만, 6천
=============== =========================
제한적인 환경에서 적은 수의 소켓만으로도 서비스가 가능할 때 소켓 수를 줄이면 메모리를 절약할 수 있다.



.. _adv_topics_req_hit_ratio:

Request hit ratio
====================================

먼저 클라이언트의 HTTP요청이 어떻게 처리되는지 이해해야 한다.
캐시처리 결과는 Squid와 동일하게 TCP_*로 명명되며 각 표현마다 캐시서버가 처리한 방식을 의미한다.

-  ``TCP_HIT`` 요청된 리소스(만료되지 않음)가 캐싱되어 있어 즉시 응답함.
-  ``TCP_IMS_HIT`` IMS(If-Modified-Since)헤더와 함께 요청된 리소스가 만료되지 않은 상태로 캐싱되어 있어 304 NOT MODIFIED로 응답함. TTLExtensionBy4xx, TTLExtensionBy5xx설정에 해당하는 경우에도 이에 해당함.
-  ``TCP_REFRESH_HIT`` 요청된 리소스가 만료되어 원본서버 확인(원본 미변경, 304 NOT MODIFIED) 후 응답함. 리소스 만료시간 연장됨.
-  ``TCP_REF_FAIL_HIT`` TCP_REFRESH_HIT과정 중 원본서버에서 확인이 실패(접속실패, 전송지연)한 경우 만료된 컨텐츠로 응답함.
-  ``TCP_NEGATIVE_HIT`` 요청된 리소스가 비정상적인 상태(원본서버 접속/전송 실패, 4xx응답, 5xx응답)로 캐싱되어 있고 해당상태를 응답함.
-  ``TCP_REDIRECT_HIT`` 서비스 허용/거부/Redirect 조건에 의해 Redirect를 응답함.
-  ``TCP_MISS`` 요청된 리소스가 캐싱되어 있지 않음(=최초 요청). 원본서버에서 가져온 결과를 응답함.
-  ``TCP_REF_MISS`` 요청된 리소스가 만료되어 원본서버 확인(원본 변경, 200 OK) 후 응답함. 새로운 리소스가 캐싱됨.
-  ``TCP_CLIENT_REFRESH_MISS`` 요청을 원본서버로 바이패스.
-  ``TCP_ERROR`` 요청된 리소스가 캐싱되어 있지 않음(=최초 요청). 원본서버 장애(접속실패, 전송지연, 원본배제)로 인해 리소스를 캐싱하지 못함. 클라이언트에게 500 Internal Error로 응답함.
-  ``TCP_DENIED`` 요청이 거부되었음.

이상을 종합하여 Request hit ratio계산 공식은 다음과 같다. ::

   TCP_HIT + TCP_IMS_HIT + TCP_REFRESH_HIT + TCP_REF_FAIL_HIT + TCP_NEGATIVE_HIT + TCP_REDIRECT_HIT
   ------------------------------------------------------------------------------------------------
                                            SUM(TCP_*)


Byte hit ratio
====================================

클라이언트에게 전송한 트래픽(Client Outbound)대비 원본서버로부터 전송받은 트래픽(Origin Inbound)의 비율을 나타낸다.
원본서버 트래픽이 클라이언트 트래픽보다 높은 경우 음수가 나올 수 있다. ::

   Client Outbound - Origin Inbound
   --------------------------------
           Client Outbound


원본서버 장애상황 정책
====================================

고객이 언제든지 원본서버를 점검 할 수 있도록 하는 것이 개발팀의 목표다.
원본서버의 장애가 감지되면 해당 서버는 자동으로 배제되어 복구모드로 전환된다.
장애서버가 재가동되었더라도 정상 서비스 상태를 확인해야만 다시 투입한다.

만약 모든 원본서버의 장애를 감지한 경우 현재 캐싱된 컨텐츠로 서비스를 진행한다.
TTL이 만료된 컨텐츠는 원본서버가 복구될 때까지 자동으로 연장된다.
심지어 Purge된 컨텐츠의 경우에도 원본서버에서 캐싱할 수 없다면 복구시켜 서비스에 문제가 없도록 동작한다.
최대한 클라이언트에게 장애상황을 노출해선 안된다는 정책이다.
완전 장애상황에서 신규 컨텐츠 요청이 들어오면 다음과 같은 에러 페이지와 이유가 명시된다.

.. figure:: img/faq_stonerror.jpg
   :align: center

   왠만하면 이런 화면은 보여주기 싫다.


시간단위 표현과 범위
====================================

기준 시간이 "초"인 항목에 대하여 문자열로 시간표현이 가능하다.
다음은 지원되는 시간표현 목록과 환산된 초(sec) 다.

=========================== =========================
표현	                      환산
=========================== =========================
year(s)                     31536000 초 (=365 days)
month(s)                    2592000 초 (=30 days)
week(s)                     604800 초 (=7 days)
day(s)                      86400 초 (=24 hours)
hour(s)	                    3600 초 (=60 mins)
minute(s), min(s)	          60 초
second(s), sec(s), (생략)	  1 초
=========================== =========================

다음과 같이 조합된 시간표현이 가능하다. ::

    1year 3months 2weeks 4days 7hours 10mins 36secs

현재 지원대상은 다음과 같다.

- Custom TTL의 시간표현
- TTL의 Ratio를 제외한 모두
- ClientKeepAliveSec
- ConnectTimeout
- ReceiveTimeout
- BypassConnectTimeout
- BypassReceiveTimeout
- ReuseTimeout
- Recovery의 Cycle속성
- Bandwidth Throttling



디스크 Hot-Swap
====================================

서비스 중단없이 디스크를 교체한다.
파라미터는 반드시 ``<Disk>`` 설정과 같아야 한다. ::

   http://127.0.0.1:20040/command/unmount?disk=...
   http://127.0.0.1:20040/command/umount?disk=...

배제된 디스크는 즉시 사용되지 않으며 해당 디스크에 저장되었던 모든 컨텐츠는 무효화된다.
관리자에 의해 배제된 디스크의 상태는 "Unmounted"로 설정된다.

디스크를 서비스에 재투입하려면 다음과 같이 호출한다. ::

   http://127.0.0.1:20040/command/mount?disk=...

재투입된 디스크의 모든 콘텐츠는 무효화된다.


.. _adv_topics_syncstale:

SyncStale
====================================

(인덱싱시점과 성능상의 이유로) 비정상 서비스 종료시 관리자가 :ref:`api-cmd-purge` , :ref:`api-cmd-expire` , :ref:`api-cmd-hardpurge` 한 컨텐츠가 인덱싱에서 누락될 수 있다.
이를 보완하기 위해 API호출을 로그로 기록하여 서비스 재가동시 반영한다. ::

    # server.xml - <Server><Cache>

    <SyncStale>ON</SyncStale>

-  ``<SyncStale>``

   - ``ON  (기본)`` 구동될 때 동기화한다.

   - ``OFF`` 무시한다.

로그는 ./stale.log에 기록되며 정상종료 또는 정기 인덱싱 시점에 초기화된다.
